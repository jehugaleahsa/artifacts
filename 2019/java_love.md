# I love java
I spent the last 10+ years using .NET, primarily working in C#. Because of that, it might seem like me recently going to a Java shop was a pretty big leap. However, it's been an especially awesome experience and I can honestly say that Java is a wonderful language and ecosystem to work in.

Without getting into too much of the nitty gritty, I hope I can share with you some of my recent discoveries that have made working in Java a real treat. This should be of particular interest to other people working in .NET who wonder if Java's worth their time.

## Common complaints
It's true that Java is a slow moving language. In recent years, however, Java has received various major language features in a very short time. Code written in Java 8 looks entirely different than code from previous releases. Java 8 introduced lambdas and streams, which bring some functional programming concepts to the language.

Java is still a very verbose language; however, you rarely encounter a block of code that is so obscure only the author understands it. In a way, the verbosity makes it self-documenting code. It's still possible to introduce obscurity using crazy stream-builders, bad inheritance practices and abusing static imports, but it's not something you'll encounter frequently.

A lot of people still think Java is too enterprise-y, with all the J2EE nonsense from 5+ years ago. Today, the ecosystem is a little more open-stack, with more companies utilizing Java and the JVM for its stability, cross-platform support and vast platform integration support. You don't even need to know what a "Bean" is anymore to be productive in Java.

Several years ago, a friend asked me what platform I would use to build an industry-strength, totally free system to build a business around. At the time, the Microsoft stack was still pretty much Windows and Visual Studio only, so the only two options in my mind were Node.js and Java. However, Node.js was pretty new at that point without promises, async/await or TypeScript to make it more manageable, so that really just left Java as the only option. At the time, Java didn't have as many nice features as it does today, so it was a bit of a downer that it was the best option. However, in last 5 years, both Node.js and Java have become both more viable and more exciting options. Microsoft's done a good job of making .NET Core an equally appealing option; it's great there's so many options for people looking for a totally free setup.

## Maven
When NuGet entered the scene, it literally changed the way I wrote .NET code. Suddenly, instead of needing to keep all my DLLs in a separate folder *in source control* and referencing them from my solution, I could just go search for a library out on the Internet and bring it into my project. It made me more willing to introduce dependencies, so I stopped reinventing the wheel and relying on copy'n'paste programming. It also made me more willing to investigate things like dependency injection. NuGet also made it easier to keep up-to-date.

Java has had this type of dependency management since, like, forever. Maven has been around for a long time and it's pretty much the de facto way of retrieving dependencies during build time. But Maven goes *waaay* beyond dependency management. It's like someone rolled MSBuild and NuGet into one, and then some. First of all, Maven dictates your project structure; it uses convention over configuration, which has several major benefits. Some of them being:
 
1) Every project across the entire Java landscape is going to have a similar structure.
2) You don't have spend any time configuring where resources or source files are.
3) The generated output (e.g., Jars) is going to have similar internal structure.
4) What you do customize can be inherited by child modules.

At first I didn't like that Maven uses XML configuration files (.pom). However, now that I'm used to it, I really like that IDEs can provide auto-completion and I don't have to lookup what the element/attributes are.

Maven's real strength is its use of plugins. You can execute plugins as part of different life-cycle events (e.g., compile, test, package, install, deploy, etc.). These plugins are downloaded as needed, so they just sort of wire themselves up and off they go. You can use them to generate class paths, spit out a directory full of your dependencies, copy files, run a shell script or even spin up a Docker container.

The next time I feel like playing around in C++, I might look into Maven's support for native languages. It'll be neat to see if Maven can outdo the myriad of less-optimal build tools out there today. 

## Annotation Processing
The next feature I'd like to mention is Java's support for compile-time annotation processing. This language feature is not something you are going to need everyday. However, it is especially powerful for automatically generating code.

In .NET (and most languages), dependency injection is a runtime concept. When you ask for an instance of a class, the DI library will look to see what needs injected and recursively creates a graph of objects. The first time this happens be slow if the library has to build an internal cache using reflection and *even dangerous* if you neglected to provide a binding.

A couple years ago I started thinking about what it would take to make a compile-time dependency injection framework where any missing bindings would be caught at compile time. After a couple thought experiments, I realized this *could* be done with with a lot (*a lot, a lot*) of manual code. The only way to make this palatable would be if some sort of code generator could write most of that wiring code for you.

Tho and behold, that's exactly what [Dagger](https://google.github.io/dagger/) does! Using annotation processing, the library searches for annotations and generates a dependency injection container at compile time. What's cool is you can reference the generated container class and the compiler won't complain. It doesn't care so long as it exists by the time it is done compiling.

This intrigued me so I started looking for an excuse to play around with it. After a while, I finally had a great idea for a side project. Java has a lot of options when it comes to MVC and web frameworks, but I wasn't really too happy with most of them. However, after a lot of searching the Internet, I stumbled on a project called [Javalin](https://javalin.io/). Someone familiar with Node's Express would appreciate Javalin's simplistic REST middleware approach. It is very minimal, mapping routes to function calls.

You could be pretty productive with Javalin in its own right. However, having worked in ASP.NET Core, I wanted to see if I could use annotation processing to automate the creation of routes and automatically extract path parameters, query strings, JSON bodies etc. and simulate ASP.NET Core's `ActionResult`s for sending back a response. In other words, I wanted to see if I could build an MVC framework using Javalin.

Long story short... yes, yes I could!

## Enums
Anyone who's worked in Java for a while knows how amazing enums are. Unlike most other languages, enums in Java are not simply named numeric constants. Instead, Java enums are singletons, *actual objects*, that can have their own backing fields and methods. The only negative side to this flexibility are people making enums *too* smart.

## Simple Deployments
Maven takes care of a lot of the complexity surrounding build and deployment. However, at some point you still need to put your software out on a machine and kick it off. Historically this had been a bit of a pain point for .NET. You either needed to create an .exe or host your application another way. With Java, you generate a JAR file and run `java -jar /path/to/jar`. If you need to pass in configuration settings, you can use `-D` arguments.

The only real downside to running Java programs is needed to specify classpaths. A classpath indicates to the Java runtime where to look for classes. This is useful for swapping out an implementation, though, so it can be a very powerful feature. Fortunately, Maven can copy all your dependencies into a single folder for you and can even spit out a classpath string. Plus, most IDEs take care of generating the classpath when running/debugging locally so it's like you don't even notice.

Another positive note: I was able to really easily integrate spinning up a Docker container as part of my Maven process. In less than a day, I was able to configure one container to run Postgres and another to run my web app. Now I can run a suite of integration tests against them to get immediate feedback.

## Simple Development
Only a few years back, the idea of working outside of an IDE gave me nightmares. In the past 2 or 3 years, I find myself wanting to get away from IDEs altogether. A big part of that, I think, has been VSCode coming out. Now I appreciate the simplicity of working with just the command line, a file explorer and a text editor with basic syntax highlighting, auto-completion and little else. Java's made as many strides in recent years to free itself of bulky IDEs as C# has. This is especially good news for people like myself who find web development easier in VSCode.

I feel free.

## Conclusions
In recent years, I find myself working as much in the front-end as I do in the backend. I find myself more willing to sacrifice certain backend conveniences to be more productive, front-end wise. In that regard, the quality of my backend language doesn't dictate my overall satisfaction programming as much. In reality, I am happy with some rigorous type safety and convenient access to an MVC framework and my database.

It's been hard not to feel like I was falling behind with all the talk about automated deployments, containers and cloud in recent years. But here's the thing, most of these things are only relevant to a project once in a while. Once they're in-place, they rarely need maintenance whereas writing code is a daily activity that ultimately underpins the success of a project. Working in Java's been great; I was able to knock out a big chunk of these annoying "unknowns" with relatively little work on my part.
