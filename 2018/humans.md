# Battling Complexity
I found myself coming onto projects pretty frequently as a contractor. Unless I authored the project, coming onboard, I invariably felt lost and confused. I am not sure that confusion ever went away, even as I learned more about the project. If anything, I just acclimated to the insanity *enough* to make progress.

## Industry knowledge is the easy part
Believe it or not, the industry knowledge (or business logic) is one of the easier complexities to deal with. Without understanding the "why" of code, it seems riddled with bizarre cases and you really wish there were comments explaining the intention... comments that almost never exist, by the way. And good luck going to a business user and asking them why that special condition exists! Ask a programmer and they will likely say, "Oh yeah! You can't make that change directly because the user has to fill out a form because of some law." Like I said, though, this is the easier complexity to deal with.

## Two sides of complexity
In software, two types of complexity are two sides of the same coin: **how large** things are and **how many** things there are. Often, the solution to large classes is to create more, smaller helper classes. The solution to breaking up large methods is to create more, smaller helper methods. On the flip side, once you've started decomposing classes and methods up, opening up a folder containing 200 classes will baffle even the best programmers. 

One challenge is knowing which file to look at first. Further, it's not always obvious how to discern between fundamental classes (concepts) and helper classes (utilities).

## Patterns help a little
When you read enough books about software architecture, you can design your code in ways to reduce the impact of changes later on. So long as everyone on the team is aware of these design principles the structure of your code is self-explanatory. How often do you find your colleague's approach is 100% compatible with your own?

The unfortunate side-effect is it can drive a stake between senior and entry-level developers. Without careful hand-holding, putting an application in the hands of a junior developer is fraught with danger. Put in the hands of a senior, juniors just feel annoyed following all these "guidelines" without really understanding the benefits. And how often are seniors prone to over-engineering and "doing it all wrong" anyway? A sure give-away is when the senior can't explain "why" a practice is followed: "Just do it!"

Often, the whole reason a project falls on a junior to begin with is it is low priority, low risk and the seniors are too busy cranking out something else. Do we just leave entry-level developers alone and let them fail? Do you schedule recurring meetings to review their approach? For them to voice their concerns and questions? How self-observant are entry-level developers? Do they even realize when their making important design decisions? Probably not! The point is, patterns can help avoid rework later on but they come with their own mental overhead.

## Languages help a little
If you read the Gang of Four today, you might be surprised by the pattern catalog. Over time, many of the patterns have become "built-in" to newer languages. At first you might think this means developers are saved the overhead of reading up on design patterns from some old, dusty book. However, what you'll find is now developers just have more to learn about the programming language.

Instead of a quick-and-dirty manual for the language, you end up with a book that's 1,000 pages long and constantly being updated. One good thing about it is it means less boilerplate code and all that knowledge finds itself in one place... sorta. I guess, you still have to know "when" to use a language feature.

## Humans are limited creatures
The reality is most developers are not out there reading books on software architecture in their spare time. Most of the advancement we've made in the past couple decades comes down to one, simple thing: buy in.

Once in a while, we as developers "elect" someone as the go-to, defacto guru and blindly follow their every word. We don't do this across the board, of course. People working in .NET might choose one person and hard-core Java advocates choose another. In doing so, the two groups evolve in different directions and come to value different practices. That's where idioms are born.

A large percentage of developers out there have no tools available to them to fight off complexity. Even if they are conscious of code duplication, they don't know what to do to eliminate it (generics? base classes? high-order functions?). When they do tackle duplication, they manage to increase complexity rather than reduce it. Most examples of gold-plating are the direct result of someone trying to "reduce complexity".

## Concepts are easy
If you take the time to investigate Domain Driven Design (DDD), one of the core principles is defining a shared vocabulary among developers and industry specialists. This is often misunderstood as forcing developers to learn an entire industry just to write a few lines of code, but that's not the case at all.

Actually, here's the idea: the developers come in asking a lot of dumb questions. Specialists try to answer those questions and when there's a failure to communicate, someone takes the conversation up one level. You keep dumbing down the conversation until both sides can follow along again. 

The common and natural result of these conversations is either the developer or the specialist throws out a metaphor, "Oh... so it's like blah..." and whenever that topic comes up again, the specialist can just reiterate on that "blah" metaphor. More often than not, the specialist and the developers just start saying "blah" rather than the industry-specific term because it speeds up communication. The code starts having a lot of "blahs" in it. User stories and tests start being written in terms of "blahs", as well.

The reason this is such a good approach to software is that it matches the way people learn. We learn by relating new ideas to other ideas we're already familiar with. That's why the word "sandwich" is frequently used in contexts unrelated to food or "clockwise" or "put a pin in that". So long as everyone comes upon a concept in the same way, that concept is likely to share common meaning. You're building a language.

## Concepts instead of names
People are really good at naming things. Once we name something, we don't need to think about it in as great of detail anymore. Naming them was claimed to be the biggest benefit to design patterns. Hey, now you say "adapter" and every other developer immediately understands your intent. Great! Strangely, we still only have 20-30 well-accepted design pattern names, after this much time. Why?

For one, if there were many more design patterns then no one could keep them all straight. They also need to be general-purpose enough to reach a wide audience. More importantly, it turns out most people have not read Design Patterns and, because no one read it, it's really not clear what we mean when we say "adapter" or "flyweight" or "bridge". The by-the-book definition of some of the patterns doesn't even match the "common" accepted meaning anymore! Comparing wikipedia to the book, it's clear design patterns are evolving over time. It's not just the approach that's changing - it's the *very goal itself*.

The fatal flaw of naming things is we stop thinking about what those names mean. A name is an abstraction. Once that abstraction's takes place in our minds, we lose our understanding. It reminds me of asking a 7-year-old to define a word.

That's why it is so important for teams to take the time when on-boarding to focus on the concepts rather than terms. Furthermore, concepts need to be at the core of your codebase. It's really important to make concepts first-class citizens in your code and figure out why they are being clouded behind implementation details. Start with the concept and the goal, then try to show how the code *accomplishes that goal*.

Frequently, concepts lose visibility because we become so accustomed to naming classes with certain prefixes and suffixes. We end up with "factories" and "controllers" and "managers". Then we throw on the database design and architectural concepts, too, so we end up with "UserRoleRepositoryFactoryService". And it's easy to be distracted with resource management, exception handling, logging and other non-essential essentials.

## Can we really do this?
I really debate whether we, as a species, can write software. It's a good question to [keep asking ourselves](https://blog.codinghorror.com/why-im-the-best-programmer-in-the-world/). If computers could write computer programs, I wonder if they would use classes and methods, like we do. Or would they be able to just mung together the code needed to execute the desired functionality? Would they need to worry about maintenance if they could just reconstruct the same code from scratch, new features included, every time? Are programming languages simply trying to compensate for our limited brain capacity?

As an aside, I had a horrible experience with Verizon recently. When I ordered a new phone, they also sent me the shipping materials for my old phone, to trade it in. They neglected to include the shipping label, however. I went to the website to reorder the shipping materials but nothing ever arrived. I finally contacted support and he told me the website to reorder shipping materials wasn't going to work because there was a delivery receipt on file. Seems to me reordering shipping materials would be indication that *they didn't arrive* or something else went wrong. I'm confused thinking of the user story that website *does* address. Happy ending, sorta... it turns out another Verizon employee stuck the shipping label to the *inside of the box*, so I had it all along.

The moral of that story is we can't even properly define our user stories half the time. When we can, we lack the technical expertise to adequately implement a solution. Developers are often put in positions where they lack the authority/access to support the core functionality. If we can't even put a sticker inside of a box properly, what hope do we have as a species to construct enormous software systems? I'm astounded we were able to get the first computers working at all! How did we ever make it to the moon?!?

## What's missing here?
Obviously, we are capable of achieving incredible things. Why do we succeed some times and not others? Personally, I think a lot of it comes down to the way to define success. For most programmers, success is simply a matter of guaranteeing a paycheck for as long as possible. We, as an industry, seem to discourage this notion that developers should like and focus on money. As if our projects are interesting and challenging enough and we are truly passionate, we will stop worrying about paying the bills and just get swept up in our work. Baloney and hogwash!

It's quite the opposite, in fact. When we have enough financial security, we can stop focusing on money and direct our energies elsewhere. Now think about foreigners who have money and deportation to worry about! Or if you're employees are going through a divorce! The reality is there are a lot of serious *and trivial* thoughts plaguing most people. It takes a certain insanity for people to just "shut off" those thoughts to focus on work. It's like stopping to tie your shoe when you're escaping from a burning building!

I don't feel like most companies are willing to invest in their employees anymore. At the same time, I feel like we're becoming a country consisting mostly of independent consultants. I think companies need to start investing in their employees more and reiterate what a great loss in investment it would be if their employees were to leave. Make it clear that there will be a job there for them for as long as they are willing to put in the effort. Honestly, shaking off this feeling of impermanence will help people to focus and develop some loyalty. Maybe it's not that simple. Maybe it's because people just go home and watch TV now. Who knows?

