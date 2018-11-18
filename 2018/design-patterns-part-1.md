# Design Pattern Ruminations - Part 1: Creational Patterns
It's been over 10 years since I read [Design Patterns](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612). Now that I'm going through it a second time, it's interesting that many of the patterns have come to mean something very different to me from what's actually described in the book. I thought I would collect my thoughts as I went through the book again. I hope you can find this useful, too.

## Abstract Factory
It took me several years to fully understand the difference between [Abstract Factory](https://en.wikipedia.org/wiki/Abstract_factory_pattern) and [Factory Method](#factory-method). Abstract Factory creates instances of families of classes, not just subclasses within the same hierarchy.

A recent example where I used Abstract Factory was to query several CRM platforms, each with their own set of repository interfaces. The set of repositories was determined at runtime based on the user's login ID, which had the platform identifier encoded in it.

The big thing that's changed since I first read Design Patterns is my use of dependency injection. Now, I would pass the "injector" to the abstract factory class and use it to instantiate the correct concrete classes. Otherwise, I might do away with the abstract factory class entirely and just configure the DI engine to call a method that looks at a config setting (or whatever) and picks the appropriate classes.

The book does make a good point about using a [singleton](#singleton) to avoid repeatedly recalculating which family of classes to use. However, using dependency injection, this is easily managed with lifetimes.

## Builder
Over the years, this [design pattern](https://en.wikipedia.org/wiki/Builder_pattern) has taken on a whole new meaning for me. The book talks about generating different objects by navigating through an object structure and calling methods on a builder. The "builder" can be any subclass in a builder hierarchy that reacts to each method call differently. Each builder subclass generates a different end-product.

However, this reminds me too much of the [Visitor](https://en.wikipedia.org/wiki/Visitor_pattern) pattern. The only difference is there is no double dispatch. Generally, I use the visitor pattern to "build" things.

Before reading this chapter again, my notion of the builder pattern was completely different. Rather than a hierarchy of builders, there is just one builder class. It provides a slew of methods for specifying the behavior of the object being built and then a "CreateThing" method for constructing the final object.

The pattern above can come in many varieties. A common approach is have the builder methods return `this` so method calls can be chained. Or, once chained, the author can choose whether calls just modify the original builder or create a new, immutable builder. This often depends on the usage. As an example, my [ComparerExtensions](https://github.com/jehugaleahsa/ComparerExtensions) project uses this "builder pattern" to construct .NET comparers and the builders are immutable.

## Factory Method
The authors (i.e., GoF) seem to discourage the use of [factory method](https://en.wikipedia.org/wiki/Factory_method_pattern). Their observation is you are creating a factory subclass for each subclass you want to create. At which point, you're really implementing the strategy pattern, where the strategy creates something rather than performs a different algorithm.

However, the authors mention another variant of factory method, where you simply pass a parameter to a method (static or otherwise) that creates the correct subclass. This is what I think of when people mention factory method.

In fact, in [Refactoring](https://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672), Martin Fowler points out you can replace repeated `switch`/`if/else` blocks with inheritance: [Replace Conditional with Polymorphism](http://wiki.c2.com/?ReplaceConditionalWithPolymorphism). You end up with just a single `switch`/`if/else` for creating the correct subclass rather than having them spread out all over your code. Hey, you just built a factory method. Yay!

Early on in my career, I used statics much more than I do now (almost not at all now), so I would have used static factory methods. These days, I create separate factory classes with instance methods. Curiously, these factory classes can grow into builders as they need to become more complicated.

With DI, you can register methods to call to create your objects. This is great for creating different objects based on configuration settings, but not necessarily when the flow/logic is controlling what gets created. For example, how do you create an `Even` or `Odd` object depending on what number a user types in? It's a last second decision. In which case, it's better to create a factory class that accepts an "injector" with a method accepting the value:

```csharp
public class NumberFactory 
{
    private readonly IServiceProvider serviceProvider;

    public NumberFactory(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
    }

    public Number GetNumber(int value)
    {
        bool isOdd = (value & 1) == 1;
        Type numberType = isOdd ? typeof(Odd) : typeof(Even);
        return (Number)serviceProvider.GetService(numberType);
    }
}
```

You could get away with just `new`-ing up `Odd` and `Even` in this example. However, in real applications, `Even` and `Odd` could have their own set of dependencies. Using an injector in the factory avoids also passing every dependency to the factory. Should `Even` or `Odd` ever change, the factory doesn't need to change, as well.

## Prototype
The [prototype pattern](https://en.wikipedia.org/wiki/Prototype_pattern) just feels a little too much like a special case to me. Unless working with immutable objects, you almost always need to deep clone, which can be a serious coding exercise. Even in ECMAScript 6, with array and object [destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), deep cloning an object is still a challenge.

An alternative is to simply store the steps required to reconstruct an object using a series of setter functions or using a builder. Or just serialize/de-serialize, something like `JSON.stringify`/`JSON.parse`.

Some of the examples given in the book, like constructing reusable circuits, would be hard to do without prototyping. That said, I have a hard time thinking of other examples where they make sense. Even in the circuit example, such reusable components would probably need serialized so they'd be available the next time the program ran. In which case, it's probably easier to just de-serialize each time a copy is needed, even if it is slower.

## Singleton
Basically, you shouldn't use [singleton](https://en.wikipedia.org/wiki/Singleton_pattern) anymore. If you need to limit the number of times an object gets constructed, it's better to use an object pool or use dependency injection lifetimes. Perhaps singleton still makes sense in applications where you're representing a physical device, like a device driver, and you need synchronization.

I think it is funny that during interviews, whenever design patterns come up, this is still the most cited pattern, even though it more an anti-pattern now.

## Next Steps
At the time when Design Patterns was released, developers were still keeping development reference tomes on their desks. Books like this were invaluable because it was the combined knowledge and experience of experts from all over the field. You rarely encounter books like this anymore; instead, you just get the informed opinion of one or two people. Instead, you do a google search and do your best to glean the best approach.

I have several colleagues who refuse to read this book, mostly due to its age. I must admit, the Smalltalk examples are a bit cryptic and feel dated. Personally, I wanted to re-read it because I vaguely remembered not groking the first part of the book. Actually, it turns out there are only two small chapters before you get into the design pattern catalog. It's great that it gets right into the meat of the content!

Next, I will start going through the structural patterns. Hopefully, the last 10 years will have brought some insight into them, as well. Stay tuned...