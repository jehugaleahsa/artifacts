# The Importance of Being Fast
In this article, I am going to talk about reality. I am going to explain how you can have everything you ever wanted as a software developer, but it has a particularly elusive prerequisite: being fast!

What does "being fast" mean? Some people might say it is *working smart*, being productive, avoiding rework, avoiding bugs, increasing code reuse, YAGNI, DRY, knowing command line, knowing IDE shortcuts, or just being able to type 150 words a minute on a keyboard. These are all correct.

## Unit testing
Some would argue that writing unit tests distinguishes the boys from the men, the girls from the women. Many development shops have strict code coverage requirements or even try to force TDD. If you have ever worked on a project that has unit testing, you might have come to the same conclusion as many other people: unit testing nearly doubles your development time.

The argument for unit testing is that it results in less time fixing bugs down the road and makes it easier to make changes without the **fear** that you are breaking something. I agree with these assessments and think unit testing (and integration testing) is well-worth the extra effort.

The thing is: it takes a lot of time. I find that when my tests fail, more than 50% of the time, it's a bug in my unit test, not in the code I'm testing. Coming up with good tests takes time. Test code needs refactored to avoid repeated code, which is often, and that takes a lot of time, too. You need time to figure out how to inject mocks, check the mocks and do so without rewriting 100 unit tests each time you add a new constructor argument. You need time to make sure unit tests are readable and meaningful to other coders. It takes time to set up your build environment to run your tests and notify the team when tests fail or coverage slips.

However, in my opinion, if you write 30 unit tests and just one of those tests reveals a bug in your code, it's well worth it! Unit testing also forces you to design your code in a way that makes it easy to test (*without using `static` everywhere*). You can almost design software at various scales just by consistently writing unit tests.

My experience has been that most developers write awful unit tests. In order to write 30 good tests, you need to be fast. You need time to think about common usage (happy path) but also error conditions. You need to constantly refactor test code to eliminate duplication and keep the test focused on what's being tested. If you are slow, there's just no way.

## Refactoring
When I look at most code bases, the first thing I usually notice is that classes and methods tend to be huge. The lifetime of variables is astronomical and the cyclomatic complexity is enough to induce vomiting. The most common refactoring, probably, is Extract Method. That probably means most people aren't refactoring their code.

When I am feeling negative, I contribute this to developers being bad at coding. My thought process is this: a developer who struggles with logic is going to waste a lot of time just getting the code to generate the desired output. Such a developer sees working code as the finish line. Expecting such a developer to then refactor their code is like asking a marathoner to run another 10 miles. Being less negative, more likely, developers who leave code in such a state just aren't as passionate as I am, or don't associate their self-worth with their code like I do, or don't fear the judgement of other developers like I do, or haven't read the same books I have and don't know no better.

Having philosophical debates: Should I break this code up? What's a better name for *this*? What order should I initialize these variables? Could I do this in a more functional way? Can I make this read-only/immutable/const? Does this belong here? ...and so on takes time. Again, in my opinion, these questions are always worth it in the long run, but in order to ask them you need **time**! The only way to have time is to be *fast*.

## Re-architect-ing and updates
Hate how your system is written? Come up with a way to reuse more code? Come up with a way to make everything faster? Want to swap our your dependency injection framework? Wanna go from .NET Framework to .NET Core? Java 5 to Java 11? Want to completely redo the UI? In most development shops, the answer is always "yes" or, more likely, "for the love of god, yes!". Also in most development shops, these sorts of improvements never happen. Awww...

My experience has been that for these types of large changes, it's better to ask for forgiveness than for permission. Throughout my career, I have had large rewrites in their own branches, maintaining them side-by-side with the production branch, often for months or even *years*. Some times I never got the chance to merge them into master, but usually it does happen... *eventually*.

Some people might read this and start thinking, "He's talking about massive changes." However, in a lot of shops, teams aren't even updating their dependencies. Some times dependency changes *are* huge, like going from Webpack 3 to 4, but other times it is just running `npm outdated` and bumping to the next minor version (aka, maybe new features but no breaking changes).

What I want to emphasize is that it *takes time* to update your dependencies to the latest version with a regular cadence. If you are maintaining multiple branches, it takes time to keep them in-sync. It takes time to experiment with improvements, developing reusable libraries or crafting UI components. If you spend all your time doing actual tickets, how will you ever have time for improvement? I can say from personal experience, working on side projects *for work* at home during the evenings and weekends quickly loses it's luster.

## Comments and documentation
If you have already started a project and are now thinking about going back and adding doc comments, you might want to think again.

First of all, this is a major undertaking even on a "smallish" open source project. On a real-world project (that's months or years old) it will soon feel like the most monotonous, never-ended task you ever undertook.

Secondly, any useful insight you might have had while writing the code will likely be lost by now. You will find yourself just typing the function definition, but in English. For example:

```java
/**
 * Divides the first value by the second value.
 * @param dividend The value being divided.
 * @param divisor The value to divide by.
 * @return The quotient, from dividing the dividend by the divisor.
 * @throws IllegalArgumentException if divisor is 0.
 * @implNote This code returns a double which might not have enough
 * precision to store the result if the dividend or divisor is 
 * sufficiently large.
 */
public double divide(long dividend, long divisor) {...}
```

Someone reading this code will probably just find this comment more annoying than useful and skip over it. The only useful information is the little note at the bottom pointing out precision issues - the rest is probably obvious.

You'll have an even harder time writing in-line comments explaining how the code works. Was there a website explaining how the code works? Good luck finding it again! Was there a bug ticket linked to this code change? Have fun searching for that one! Did *so and so* say to do this because of reason XYZ? Who was it again? What was the reason again? Too late! That information's lost forever!

I rarely have the attitude that something *just isn't worth it*, but this might be one of those cases. If you are vigilant from the very beginning, documentation and comments are a wonderful thing. Even if you don't do this in your normal code, providing API documentation via OpenAPI is critical. If you are writing an open source project, good comments on the public-facing interface is just being responsible.

The thing is... good documentation takes time. Good comments take even longer. Just as code frequently changes over time, the documentation and comments also need to be kept up-to-date. And at some point, you'll find yourself wondering where the code is hiding amongst all the comments... you might need to spend time cleaning them up!

## The Catch-22 of development
So basically, every technique that's ever been proposed for making better code takes time. Is there any surprise, then, that these suggestions never seem to get adopted by the next generation of developers? Is it a surprise dozens (if not hundreds) of books have been written to convince management that these types of effort are worth it (financially speaking)?

Many of these time-consuming tasks are difficult to start adopting on an existing code base or, being more honest, would feel like slapping lipstick on a pig. Can you really introduce unit testing on a system that doesn't support it? Can you really add meaningful documentation or comments? Will a new library or framework upgrade really solve your problems?

The reality is, developers who are fast will probably start tackling these types of problems on their own time without being asked (or use that time to hunt for a new job). If they are really fast, they will find slivers of time between tickets.

If this motivates you to do anything, it should be to put a little more time in up-front the next time you start a project. Do more research and make a list of desired practices. Make sure you are doing things right from the beginning.

## Getting *fast!!!*
The problem with a lot of classic software productivity books is that they focus on tools, process management or certain skill sets developers should learn to make themselves more productive. What no one ever seems to suggest is that most developers should just focus on if/else statements and loops.

Most of your time as a developer is spent reading other people's code (or your own code). If you can improve how fast you read code, you will be significantly more productive. Somewhat comically, authors since forever have been suggesting making code *readable*, not that developers get better at reading code. I don't think one is implied by the other and, in fact, I think it could be used as an excuse: "This code looks awful; I don't want to stare at it too long so I am not going to familiarize myself with how it works." And code quality is subjective and often based on incorrect assumptions about what the code is doing.

For example, I saw code like this in Java one time:

```java
final int[] sum = new int[] { 0 };
final int[] count = new int[] { 0 };
collection.stream().forEachOrdered(x -> {
    sum[0] += x;
    ++count[0];
    // A bunch of other work
});
```

When I saw this, my gut reaction was that this code was poorly written. Why would anyone create single-element arrays just to add up some integers? After trying to change it, I immediately realized Java doesn't allow you to reference non-`final` variables within lambdas (or anonymous classes) and using an array like this is considered standard practice. In fact, this is why Apache Commons has the `MutableInteger` class, just to avoid this weird looking pattern.

Then I thought about writing it using a simple for-loop and even that was a bad idea. It turned out the original developer actually knew what they were doing. If they had done anything to improve the code, it would have been adding a comment. So in reality, it was my own lack of Java knowledge that was the problem, not the code!

Frequently, the code you are looking at now is not what it looked like originally. Looking through source control history, you might see that a rather simple one-liner kept growing until it became the current monstrosity it is today. Without that time-traveling insight, most production code is going to look ragged.

I will continue by arguing that many developers could benefit from staring at ugly code more closely. It's important to understand *why* you think the code is ugly in the first place. It's important to work out ways you could improve it, especially while avoiding introducing any bugs. Could you make a series of small changes that are guaranteed to not introduce bugs? How does one make such a guarantee? There are actually areas of computer science that are dedicated to the study of code equivalence.

For example, you can say these two blocks of code are the same, and it can be proven mathematically:

```c
int x;
if (some_boolean) {
    x = 1;
} else {
    x = 0;
}
```

```c
int x;
if (!some_boolean) {
    x = 0;
} else {
    x = 1;
}
```

Some hotshot C developer might look at this and think, "Ha! I can do it even better like this!"

```c
int x = some_boolean;
```

But those developers would be wrong! In C, there's no such thing as a boolean type, so booleans are often stored using integral types. That means anything other than `0` is true, so now `x` could be something other than `1` or `0`. The equivalent code would be: 

```c
int x = !!some_boolean
```

You can use conditional equivalence like this to carefully refactor code without risk of breaking anything, assuming you don't make silly mistakes like the one above.

### Can you read code?
A productive developer looks at the code below and immediately sees what it is doing:

```java
int a = 0;
for (int i = 0; i < values.size(); ++i) {
    a += values.get(i);
}
```

Be honest with yourself: if you need time to figure out what this snippet does, you probably need to go back and focus on the fundamentals. Don't feel bad; "senior developers" with decades of experience often can't read this fluently. The reality is that most developers go their entire careers not getting comfortable with the basics. Other guys spend a weekend messing around implements "knight tour" or "8 queens" and attain enlightenment before leaving Programming 101.

For those of you who are fluent at reading code, even if you work in Perl or Haskell or C# or C or Fortran or whatever, you will be able to read that code with no problem... the language doesn't influence how well it reads very much. Those fluent at reading code will finish looking at this code and just have this to say: "It should use better variable names".

If you are one of those developers who get caught up with it being `++i` instead of `i++`, be honest with yourself and go read a book covering the syntax of your language in more detail... understand the difference and when to use one over the other. The next time you call code "bad", make sure you have the authority to make such a claim.

## Beyond if/else and loops
The original author of a system rarely is the one working on it today. The unfortunate reality is that software changes hands quite frequently. By the time the code gets to the current team, large-scale changes are limited to small subsets of the codebase. As a second- or third-generation developer, you may find you are intimately familiar with some parts of the system and a complete stranger elsewhere.

As daunting as it may seem, finding time to delve into other parts of the system is critical to being faster. A lot of the choices you make depend on you knowing what options are available to you. In a large system, there might be ways to achieve the desired effect without having to write much new code. In other cases it feels like you are trying to put a round peg in a square hole and when this happens you're more likely to introduce duplication. In other words, rather than addressing the square hole and adjusting it to fit multiple shapes, you just drill a new hole. That new hole might overlook logic that addressed previous bugs or captured subtle business logic. It might also be 10x the code you'd have needed to just adjust the other code!

When a project is young, the original author has insight into what each piece of code is doing (how it fits into the overall picture) and immediately knows the best alternative. Often, the best alternative *today* is a short-term solution until a more robust solution can be implemented (technical debt), except it never actually happened. Once the original author "moves on", it's up to the new guys to attain that insight. I can say from experience this is nearly impossible on large systems, especially when there are multiple original authors each responsible for their own little piece.

Still, as a developer who is striving to become faster, you need to take the time to get more familiar. As you learn more about the application, the user-base and the industry, hopefully things will start to click. Gaining familiarity like this takes a long time (6+ months at least). Only developers who are fast enough to burn through their tickets will have time for this type of investigation, though. The only hope is in the meantime your tickets are focused enough that you aren't needing to make major architectural decisions right away. But, hey, that's another brute force way to learn a system!

## Conclusions
I think it would be easy to argue that the larger the task, aka., the more "involved" it is, familiarity will make you faster. However, for most small tasks, simply reading code fluently will make you faster. A large part of any development task is figuring out where to make your changes and that involves reading a lot of code. I would argue that most professional developers today, despite their years of experience, could stand to be better at reading code.  I think it's more important to point out that *everyone* has room for improvement and should strive to improve their ability to read code.