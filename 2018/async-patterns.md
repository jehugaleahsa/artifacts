# Async Patterns - Consolidation, Parallelization and Caching
As the author of several open source projects, I ran into an interesting, recurring challenge when providing an async implementation for my libraries. Basically, I ended up rewriting the same logic for my synchronous code and my asynchronous code, except returning `Task`s and using `async`/`await`. Obviously, duplicate code is undesirable, so I started researching how I could consolidate as much code as possible. I found this [StackOverflow post](https://stackoverflow.com/questions/23932885/avoid-duplicate-code-with-async), that led me to believe I was not the only person experiencing this phenomenon. Most of the time, you really can't do anything about duplicate code when supporting both synchronous and asynchronous code. This mostly impacts library writers since we have to support both use-cases, where in a specific application's codebase you really only need to worry about one or the other.

A common mistake is to wrap an async implementation with `Task.Run` and expect calls to `Result` or `Wait` to work. In several scenarios, this will lead a to a dealock. I would suggest limiting your use of this technique to console apps, Windows services and certain desktop apps. For ASP.NET, while it might look like it's working in development, it could cause [deadlocks under heavy load](http://blog.stephencleary.com/2012/07/dont-block-on-async-code.html). Personally, I am often left asking how you interact with async code at all when working with older Windows/.NET technology stacks. Most of the time, this wouldn't be an issue if classes like `HttpClient` provided a [synchronous implementation](https://stackoverflow.com/questions/48489762/using-httpclient-in-the-synchronous-class-library).

I've yet to hear a satisfying approach to safely use `HttpClient` in a synchronous context - I want a synchronous implementation, Microsoft! As the author of several libraries, I perfectly understand the .NET development team not wanting to provide both sync/async support, as it will undoubtably mean writing all the same code twice. For that reason, I am not getting my hopes up.

Personally, I found adding support for async in my OSS projects came with a plethora of other annoying issues. I was greatly relieve when Microsoft released [ValueTask](http://blog.i3arnon.com/2015/11/30/valuetask/), which reduces the overhead of async calls when ,most of the time, an async function will return immediately. Of course, this optimization is only relevant when simple value types are being returned.

## The obvious solution
A great deal of code has this pattern:

```
Grab data from some source
Filter/map/reduce and store data in one or more objects
Return objects
```

Whenever you have this pattern, you can easily consolidate your logic across sync/async implementations. Everything after "grab data" can be executed synchronously (assuming it isn't computationally expensive). This is by far the most common situation I've encountered at work. This is great because you are not constantly mixing calls to sync/async code. It also means you are not making potentially long-running, expensive calls inside of a loop.

There's no reason "grab data" can't involve several, sequential calls to the database, network or file system... the point is they are all done up-front. In fact, this is one of the greatest benefits to async code... you can parallelize several I/O calls (*so long as they aren't dependent on one another*) using `Task.WhenAll`. Taken to the extreme, you can define a topological sort of parallel and sequential calls, like the `auto` operation in [caolan async](https://caolan.github.io/async/docs.html#auto).

In real code bases, sync/async code isn't usually this cleanly separated out, though. It's pretty common to grab some data, process it synchronously, grab some more data, process it, repeat... repeat... repeat. In many cases, it'd be possible to execute the asynchronous operations in parallel but we execute them sequentially anyway. I mean, it's usually a lot easier to write `await await await` than it is to write `await Task.WhenAll(task1, task2, task3)`. Most of the time, the difference in efficiency is irrelevant, so obscuring the code with `Task.WhenAll` doesn't justify the performance gains. Remember, the goal of asynchronous code isn't to boost performance - it's to boost *throughput under load*.

Really, the only time you'll likely see a performance difference is if you can avoid waiting on asynchronous operations inside of a loop. If you are `await`-ing these operations inside of a loop, it has the same performance implications as synchronous operations inside of a loop (just without hogging up system resources). The one nice thing about `async`/`await` is it's harder to ignore these sorts of performance killers - there's a tendency for `async`/`await` to trickle through your entire codebase unless you take specific steps to avoid it.

### A real-world example
Here's a representation of some code I wrote in a recent project. I wanted to show how it could be refactored from sequential operations to parallel operations.

Here's the original code:
```csharp
var account = await accountRepository.GetAccountAsync(accountNumber, token);
var offers = await offerRepository.GetOffersAsync(accountNumber, token);
var offerDetails = offerManager.GetOfferDetails(account.Balance, offers);
var paymentOptions = await paymentOptionsFactory.GetPaymentOptionsAsync(token);
var model = offerMapper.GetModel(offerDetails, paymentOptions);
return model;
```

If you look at what values are being passed as parameters, you'll see there's no dependencies between the different `await` calls. They all just accept `accountNumber` (or nothing). Only at the very bottom of the method does everything come together. Here's a dependency graph of the calls (async in red/sync in green):

![async dependency graph](https://raw.githubusercontent.com/jehugaleahsa/artifacts/master/2018/async%20dependency%20graph.png).

The async leaf nodes of the dependency graph can all be executed in parallel. That means I can rewrite my code like this:

```csharp
var accountTask = accountRepository.GetAccountAsync(accountNumber, token);
var offersTask = offersRepository.GetOffersAsync(accountNumber, token);
var paymentOptionsTask = paymentOptionsFactory.GetPaymentOptionsAsync(token);
await Task.WhenAll(accountTask, offersTask, paymentOptionsTask);
var account = accountTask.Result;
var offers = offersTask.Result;
var paymentOptions = paymentOptionsTask.Result;
var offerDetails = offerManager.GetOfferDetails(account.Balance, offers);
var model = offerMapper.GetModel(offerDetails, paymentOptions);
return model;
```

As you can clearly see, while this *might* improve performance, it is certainly harder to read. Part of what obscures this code is that the return type of each operation is different. When the return type is the same, `Task.WhenAll` will simply return an array of values. Easy! However, since the return types are different, each `Result` must be inspected separately. Hopefully, .NET will eventually support a version of `Task.WhenAll` that returns a [ValueTuple](https://blogs.msdn.microsoft.com/mazhou/2017/05/26/c-7-series-part-1-value-tuples/): https://github.com/dotnet/corefx/issues/25756. That way, when destructuring becomes widely available in C#, it will free us of this syntactic nastiness. We may be able to simply write this in the not-too-distant future:

```csharp
var accountTask = accountRepository.GetAccountAsync(accountNumber, token);
var offersTask = offersRepository.GetOffersAsync(accountNumber, token);
var paymentOptionsTask = paymentOptionsFactory.GetPaymentOptionsAsync(token);
var (account, offers, paymentOptions) = await Task.WhenAll(accountTask, offersTask, paymentOptionsTask);
// ... 
```

## Model building and partial initialization
We spend *waaaay* too much time building models for our UIs and APIs these days. I guess that's a good thing. A challenge that comes up with building models is it can take several phases. You're either grabbing data from several sources (the previous section) or you are grabbing additional data as you go. In the latter case, you don't know what you need until after the first set of results comes back. It's pretty easy to find yourself making calls out to the database/network/file system inside of a loop, which is usually bad for performance.

For example, on my recent project, after grabbing a record from the database, I had to inspect its values and grab the corresponding legal text to send along to the front-end. Since the legal text was in a file, I had to grab it asynchronously. This posed a challenge because the rest of the model-building exercise was synchronous. Even though only the one property was populated asynchronously, I had to make all of the model building methods `async` down the stack.

```csharp
public async Task<AccountSummaryModel> GetModelAsync(IEnumerable<Account> accounts, CancellationToken token)
{
    var model = new AccountSummaryModel()
    {
        TotalBalance = accounts.Sum(a => a.Balance),
        Accounts = await GetAccountModelsAsync(accounts, token),
    };
    return model;
}

private Task<AccountModel[]> GetAccountModelsAsync(IEnumerable<Account> accounts, CancellationToken token)
{
    var tasks = accounts.Select(a => GetAccountModelAsync(a, token)).ToArray();
    return Task.WhenAll(tasks);
}

private async Task<AccountModel> GetAccountModelAsync(Account account, CancellationToken token)
{
    var model = new AccountModel()
    {
        AccountId = account.Id,
        Balance = account.Balance,
        // ...
        LegalText = await GetLegalTextAsync(account, token)
    };
    return model;
}
```

As a side note, I wanted to draw attention to the fact that `GetAccountModelsAsync` is not marked `async`. Since `Task.WhenAll` returns a `Task<TResult>[]`, an `await` isn't necessary. I frequently use `await` unnecessarily and only clean it up later after realizing it, so it's worth mentioning. Adding an `await` here could result in a performance penalty if not optimized away by the compiler.

More importantly, I am calling `GetLegalTextAsync` inside of a loop (LINQ's `Select`). As I explained, this is hitting the file system. One redeeming quality is I am using `Task.WhenAll`, so I may be grabbing all the files in parallel. This is a little scary, though. What if there are 200 accounts? Would I be trying to open 200 files in parallel? Furthermore, there's a good chance the same legal text is being used by multiple accounts; am I hitting the file system for the same file over and over? More on this later...

Generally, I like to remove any and all data access and business logic from my model building code. Model building should be, for the most part, just copying values from one object to another or doing some basic calculations. I think of grabbing text from a file as a form of data access, so it shouldn't be a part of the model building code. It turns out this philosophy of keeping things separate can help avoid async code from polluting your code. It can also help to replace multiple calls in a loop with a single up-front call. The caveat is you have to make sure you grab all the necessary data up-front, before calling the model builing code, or partially initializing the models and populating the rest later. Personally, I like to fully initialize my objects at once rather than doing it in phases. So, let's see how to cleanly grab the data upfront.

### Up-front initialization
Let's say this is the code calling the `GetModelAsync` method from above:

```csharp
var accounts = await accountRepository.GetAccountsAsync(userId, token);
var model = await modelBuilder.GetModelAsync(accounts, token);
return model;
```

What we want to do is make the `GetModelAsync` method synchronous, renaming it to `GetModel`. To do that, we need to grab all of the legal text up-front and pass it to `GetModel` as a lookup for later:

```csharp
var accounts = await accountRepository.GetAccountsAsync(userId, token);
var legalTextLookup = await GetLegalTextLookupAsync(accounts, token);
var model = modelBuilder.GetModel(accounts, legalTextLookup);  // now synchronous
return model;

// ..

private async Task<Dictionary<int, string>> GetLegalTextLookupAsync(IEnumerable<Account> accounts, CancellationToken token)
{
    var tasks = accounts.Select(a => GetLegalTextPairAsync(a, token)).ToArray();
    var results = await Task.WhenAll(tasks);
    var lookup = results.ToDictionary(p => p.AccountId, p => p.LegalText);
    return lookup;
}

private async Task<(int AccountId, string LegalText)> GetLegalTextPairAsync(Account account, CancellationToken token)
{
    return (account.AccountId, await GetLegalTextAsync(account, token));
}
```

If this looks complicated, it's because it is. Rather than just initializing an object as part of the normal flow, we are doing a bunch of up-front work to initialize a lookup. The challenge is we need to associate each account ID with the legal text. After we initialize our lookup, the original `GetAccountModelAsync` method looks like this:

```csharp
private AccountModel GetAccountModel(Account account, Dictionary<int, string> legalTextLookup)
{
    var model = new AccountModel()
    {
        AccountId = account.Id,
        Balance = account.Balance,
        // ...
        LegalText = legalTextLookup.GetValueOrDefault(account.Id)
    };
    return model;
}
```

Overall, we replaced a series of `async` methods in the model building code with another in the calling code - except now we have to pass around a lookup everywhere. We are also building the lookup by hitting the file system multiple times within a loop. This doesn't seem like much of an improvement!

### Partial initialization
An alternative is to initialize everything *but* the model properties requiring an asynchronous operation. In other words, don't initialize `LegalText` when initializing everything else. After `GetModel` returns, loop through the account models and set the `LegalText` property as a separate step.

```csharp
var accounts = await accountRepository.GetAccountsAsync(userId, token);
var model = modelBuilder.GetModel(accounts);
await SetLegalText(accounts, model, token);  // Need to pass the accounts *and* model
return model;

// ..

private Task SetLegalText(IEnumerable<Account> accounts, AccountSummaryModel model, CancellationToken token) 
{
    var query = from account in accounts
                join accountModel in model.Accounts on account.Id equals accountModel.AccountId
                select SetLegalText(account, accountModel, token);
    var tasks = query.ToArray();
    return Task.WhenAll(tasks);
}

private async Task SetLegalText(Account account, AccountModel model, CancellationToken token)
{
    var legalText = await GetLegalText(account, token);
    model.LegalText = legalText;
}
```

Rather than using a `Dictionary<int, string>` lookup and passing it around everywhere, this code uses LINQ's `join` operation to match up the original accounts with the models created by `GetModel`. Since all the work of populating `LegalText` is done in the second `SetLegalText` method, the method can simply return `Task`. I find this way less confusing than dealing with tuples, like we did in the lookup example. Note that this approach would still suffer from grabbing the same file several times. I've also encountered situations where there's no uniquely identifying field in the model to match on, like `AccountId` in this example, so you have to use a lookup instead.

## `Task` and `Lazy`
One of the interesting things about `Task` is that it acts like `Lazy` automatically. Once the async operation completes, `Result` will automatically and immediately return the same computed value each time. This will be very useful for grabbing the same file multiple times without needing to hit the file system each time. Consider this partial implementation for `GetLegalText`:

```csharp
private Task<string> GetLegalText(Account account, Cancellation token)
{
    string filePath = GetLegalTextFilePath(account);
    return File.ReadAllTextAsync(filePath, token);
}
```

We can avoid reading the same file each time by reusing the same `Task<string>` for matching paths, using a `Dictionary<string, Task<string>>`:

```csharp
private readonly Dictionary<string, Task<string>> fileLookup = new Dictionary<string, Task<string>>();

// ..

private Task<string> GetLegalText(Account account, Cancellation token)
{
    string filePath = GetLegalTextFilePath(account);
    if (!fileLookup.TryGetValue(filePath, out var task))
    {
        task = File.ReadAllTextAsync(filePath, token);
        fileLookup.Add(filePath, task);
    }
    return task;
}
```

This lookup ensures the same `Task<string>` is returned for a path. Several account models could be associated to the same `Task<string>`, so as soon as the file contents are read the legal text is immediately populated for each of them.

It's important to note we do not need to use a `ConcurrentDictionary` here. Even though further up the stack we are using `Task.WhenAll`, the tasks were created inside of synchronous code. In other words, the `Dictionary` is being accessed synchronously, even though reading the file is runnning asynchronously. Understanding that is fundamental to understanding asynchronous coding!

### Local function implementation
Personally, I do not like having a class-wide `Dictionary` when it is only being used in one method, since that extends its lifetime unnecessarily. I might instead choose to create the `Dictionary` in the first `SetLegalText` method and pass it to second `SetLegalText` method, but now we're back to passing lookups all over.

Another nifty trick comes from C#'s recent addition of local functions (although you could do this with lamdas, as well):

```csharp
private static Func<Account, CancellationToken, Task<string>> GetGetLegalTextAccessor()
{
    private var fileLookup = new Dictionary<string, Task<string>>();
    Task<string> GetLegalText(Account account, CancellationToken token)
    {
        string filePath = GetLegalTextFilePath(account);
        if (fileLookup.TryGetValue(filePath, out var result))
        {
            return result;
        }
        var task = File.ReadAllTextAsync(filePath, token);
        fileLookup.Add(filePath, task);
        return task;
    }
    return GetLegalText;
}
```

Now we can pass a function everywhere instead of a `Dictionary`. Is that an improvement? Not sure... However, you'll notice `GetGetLegalTextAccessor` is `static`. That means, I could call `GetGetLegalTextAccesor` inside of the containing class' constructor and store the `Func` as a backing field. At which point, it just becomes a convenient wrapper around a `Dictionary`. Personally, I think this is just too obscure.

Let's take this obscurity to the maximum! You can use these sort of wrapper methods multiple times over *to avoid passing around the wrapper methods*. Consider this implementation:

```csharp
private Task SetLegalText(IEnumerable<Account> accounts, AccountSummaryModel model, CancellationToken token) 
{
    var setLegalText = GetSetLegalTextAccessor()();
    var query = from account in accounts
                join accountModel in model.Accounts on account.Id equals accountModel.AccountId
                select setLegalText(account, accountModel, token);
    var tasks = query.ToArray();
    return Task.WhenAll(tasks);
}

private static Func<Account, AccountModel, CancellationToken, Task> GetSetLegalTextAccessor()
{
    var getLegalText = GetGetLegalTextAccessor()();
    async Task SetLegalText(Account account, AccountModel model, CancellationToken token)
    {
        var legalText = await getLegalText(account, token);
        model.LegalText = legalText;
    }
    return SetLegalText;
}
```

I think this might be easier to understand if there were better names for the accessor methods. Still, this is enough to give most people an aneurysm.

## Recurring theme - using lookups
There's a recurring theme here that I want to focus in on. Over the past several years, I've noticed that code can often be broken out and optimized by employing lookups or doing `join`s with LINQ after-the-fact (which also use lookups internally). As another example, I can eliminate hitting the database inside of a loop by building a SQL query including an IN filter (or using `Contains` in EF) and building a lookup from the results.

In the most literal sense, lookups are caches. I hope I've shown that caching is especially useful when working with asynchronous code. The primary challenge with using lookups is scoping their lifetimes, especially when used in conjuction with dependency injection. Most of the time, a lookup can only be populated after retrieving some data and then it is immmediately used thereafter, after which it is no longer necessary. This immediate use typically entails passing it as a parameter to one or [often] more methods.

The legal text lookup in my example is particularly nasty. Not only is a lookup employed to associate an account with the legal text, but a second lookup is needed to associate a file path to its contents. In both cases, the primary challenge was limiting the scope of the lookups.

## Using `this`
Let's consider the example for building account models again. Let's say our model builder class was called `AccountSummaryMapper`. Rather than pass a `Dictionary<int, string>` to the `GetModel` method, let's say the constructor accepts the lookup and stored it as a backing field.

```csharp
public class AccountSummaryMapper
{
    private readonly Dictionary<int, string> legalTextLookup;

    public AccountSummaryMapper(Dictionary<int, string> legalTextLookup)
    {
        this.legalTextLookup = legalTextLookup ?? throw new ArgumentNullException(nameof(legalTextLookup));
    }

    // ... rest
}
```

Now all the methods in `AccountSummaryMapper` would have access to the lookup via `this`.

The challenge in most modern software architectures is that classes like `AccountSummaryMapper` will have multiple dependencies, which will be injected via dependency injection. So there'd be some constructor arguments that are populated by the dependency injection framework and some that can only be populated after retrieving some data, which isn't allowed.

To get around this, we could make the lookup a property of `AccountSummaryMapper`:

```csharp
public class AccountSummaryMapper
{
    // .. primary dependencies

    public AccountSummaryMapper(/* Primary dependencies injected by the DI system */)
    {
        // .. initialize primary dependencies
    }

    public Dictionary<int, string> LegalTextLookup { get; set; } = new Dictionary<int, string>();
}
```

With this approach, we must make sure we always set `LegalTextLookup` before calling `GetModel`, which feels a bit sloppy. Used in conjuction with a `AccountSummaryMapperFactory`, we could hide some of this two-step initialization, at the cost of more obscurity:

```csharp
public class AccountSummaryMapperFactory
{
    // IServiceProvider is .NET's out-of-the-box IoC container.
    private readonly IServiceProvider serviceProvider;

    public AccountSummaryMapperFactory(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider ?? throw new ArgumentNullException(nameof(serviceProvider));
    }

    public AccountSummaryMapper GetMapper(Dictionary<int, string> lookup)
    {
        var mapper = serviceProvider.GetRequiredService<AccountSummaryMapper>();
        mapper.LegalTextLookup = lookup;
        return mapper;
    }
}
```

You could even justify moving the lookup building code into `AccountSummaryMapperFactory` and change `GetMapper` to:

```csharp
public async Task<AccountSummaryMapper> GetMapperAsync(IEnumerable<Account> accounts, CancellationToken) 
{
    var mapper = serviceProvider.GetRequiredService<AccountSummaryMapper>();
    mapper.LegalTextLookup = await GetLegalTextLookupAsync(accounts, token);
    return mapper;
}
```

At first this sounds great, but after a while this starts feeling a bit weird to me, to be honest. Consider what the calling code would look like:

```csharp
var accounts = await accountRepository.GetAccountsAsync(userId, token);
var mapper = await mapperFactory.GetMapperAsync(accounts, token);
var model = mapper.GetModel(accounts);
return model;
```

I create the mapper using `accounts` and then immediately pass `accounts` to the mapper's `GetModel` method... urghh??? Should I just store the accounts in the mapper, too, then?

Another perspective on this situation is that the legal text lookup's lifetime isn't necessarily tied to the accounts' lifetimes. There's no reason you couldn't reuse the same lookup to process a different batch of accounts later on. Rather than having a `LegalTextLookup` property on the mapper, simply have a method for updating the lookup.

```csharp
var accounts = await accountRepository.GetAccountsAsync(userId, token);
var mapper = mapperFactory.GetMapper();
var lookup = await GetLegalTextLookupAsync(accounts, token);
mapper.UpdateLegalTextLookup(lookup);
var model = mapper.GetModel(accounts);
return model;
```

It is easy to see this is getting out of hand. We're just flopping back and forth like a dying fish! At the end of the day, I'm fighting my own sense of code cleanliness - how do I limit the scope of my lookups without introducing excessive parameter passing, factory factory factories or other mind-numbing oddities in my code? I've yet to find an ideal approach and am often choosing the lesser of evils.

## Conclusion
Discussing these topics with my colleagues, the overwhelming response is "Who cares?!?!?!" Most of the time, the performance impacts are completely irrelevant for your run-of-the-mill applications. When database queries, file accesses and API calls are in the order of milliseconds, you can easily make dozens of calls within the 1-2 second attention span of most users. Most business users are conditioned to be happy with 10-30 second response times (yuck). A single request rarely deals with more than a few dozen, small records at a time, anyway.

If executing in parallel really doesn't affect performance that much, breaking out code to use lookups is just adding complexity for little or no gain. At which point, it's only personal preferences, like wanting to keep business logic and data access out of my mappers, that keeps me from just using `async` methods all throughout my code. One of the unexpected advantages of this separation, however, is I can often find more opportunties to parallelize and cache calls.

I don't think ignoring the problem is really working. A large portion of being adept at using tools like Entity Framework is knowing how to maximize your use of `Include` and `ThenInclude` to pre-populate navigation properties. This avoids having code littered with calls out to the database. Lazy loading has been all but abandoned because it has historically been abused by people ignoring (not understanding) the performance impacts. We only saw lazy loading come back with [EF Core 2.1](https://docs.microsoft.com/en-us/ef/core/querying/related-data#lazy-loading).

With databases, there's a clear route to improving performance. Most of the time this just means `Include`-ing everything you need up-front. Other optimizations go back to my earlier example of using `Contains` to build lookups. These sorts of optimizations aren't as readily available with REST APIs and the file system. Unless your API supports bulk operations, the best you can hope to do is parallelize and/or cache some of the results. If you've not understood the benefit of [GraphQL](https://graphql.org/) up-till-now, it's basically the equivalent of enabling `Include` for your REST API!

The point is, going through these sorts of exercises will ultimately pay off. While most of the time performance doesn't demand anything nearly as complicated as demonstrated in this article, it is important to know how to achieve it. You can do that by consolidating I/O (e.g., `Include`), parallelizing I/O (e.g., `Task.WhenAll`) and caching (e.g., `Task.Result`).

I'd love to hear from anyone who's found a better way to separate out asynchronous code from synchronous code and simplified exposing lookups relying on parameters or weird lifetimes.