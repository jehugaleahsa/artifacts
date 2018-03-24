# What I've Been Up To
Early last year, I spent most of my free time learning Angular 2+ and all the new technologies I needed to use it. This year, I haven't felt as much need to learn anything specific. I have been brushing up on [Identity Server 4](https://identityserver4.readthedocs.io/en/release/) and some .NET Core stuff - nothing too complicated or time consuming. So what have I been doing?

For some reason, in January, I felt I'd better brush up on some math. I really wish I could remember why... Anyway, I have a minor in Mathematics from a small school, which is nothing to brag about, but I was starting to notice myself forgetting certain rules, like whether to add or multiply exponents, what's `sine` vs `cosine` and so on. In general, I've never felt very strong with math even though I've always excelled at it.

Around the same time, I decided to indulge the part of me that's interested in physics. So, I started watching [physics lectures from MIT](https://www.youtube.com/watch?v=wWnfJ0-xXRE&list=PLyQSN7X0ro203puVhQsmCj9qhlFQ-As8e) and am now watching 8.02x (electricity and magnetism). As soon as I started watching, it became *extremely* obvious  that I needed to brush up on my math.

After a little research, I bought an e-book on Amazon that covered algebra at a slight higher level than you learn in school, [Elements of Algebra](https://www.amazon.com/Elements-Algebra-Leonhard-Euler-ebook/dp/B00W1VZSFS/ref=mt_kindle?_encoding=UTF8&me=), written by Euler. I'm about a quarter of the way through it and I feel like I've recovered anything I'd learned in school.

## Another diversion
When Euler started explaining polynomials, I couldn't help but pick up all the rules he was describing. As a programmr, I hear rules describing math and think "requirements". I wanted to see if I could implement univariate and multivariate polynomial arithmetic using code, as I talked about in my [previous blog](https://gist.github.com/jehugaleahsa/e694ca41673b71b180e76dbe7094743a). Unfortunately, most of today's popular programming languages simply don't make it easy to express these types of operations. I got into Haskell, hoping to play with something new but also because it promised to be a good platform for this type of thing.

I've since come to terms with the fact that Haskell is nothing more than a temporary diversion and not likely something I'm going to ever work in. If anything, it was fun to learn a new syntax, delve into functional programming again and do some basic proof of concepts.

## Linked-in Learning is awful
My work has been trying to supply learning material, which is cool I guess. For some groups, I am sure these online course are okay. If anything, they are a quick way to cover the introductory material for a topic. However, no 2-5 hour series of videos are going to teach you very much. I blew through several courses on C, TypeScript, CSS and Discrete Mathematics and felt like I hadn't learned a thing.

Truth is, I already had a pretty strong foothold on those topics -- enough to know when the instructor made an obvious mistake or when they all together skipped over an interesting topic. Since taking these courses is required and *I had to take something*, for my next course I decided I would pick something I wasn't already familiar with, so I went with OpenGL.

I think as early as my first computer, I've always been interested in graphics. Video games, animation, 3D art -- it's all good. Back then, I didn't know anyone who programmed. It wasn't until college that I wrote my first program. Before then, the best I could do was Adobe Flash and [Blender3D](https://www.blender.org/). It feels like I wasted so much of my life not knowing how to program.

I think I chose OpenGL because I had stumbled on an interesting [WebGL tutorial](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Tutorial) at the beginning of the year. Ultimately, it wasn't a great first-time tutorial because it didn't explain what was going on very well. I think that just made me more interested.

The Linked-in Learning course I found was much better at breaking things down. It squeezed a few more concepts into my head and I could really start messing around with the code. I got about 70% through the tutorials before I realized I was completely lost. That's when I found this [tutorial](https://learnopengl.com/). I just finished getting through the "Getting Started" pages and have my own C++ code running his examples. I also finished up the rest of the Linked-in Learning course this week. All it's made me realize is that I'm a baby when it comes to this stuff.

It took a lot of wind out of my sails when I learned taking these online courses were manditory -- I was already trying to learn several things on my own. Then to find they are so poorly made and low in content. It didn't help my motivation to learn, after I'd already completed 5 courses, that I had to take *specific course* in order to get credit. Needless to say, I'm disappointed; I was really hoping this was going to be a great learning medium and I love that my company is trying to provide free learning resources.

## I love C++
One nice thing about playing around with OpenGL is it's been an opportunity to work in C++ again. My day job's all C# and JavaScript, for-loops and if-statements. I have to admit, C++ is a complex language but it's also very rewarding to get something working. I also enjoyed running circles around the C++ code found in the tutorials. With C++11's move semantics, `std::function` and lambdas you can write much nicer code. Even then, I'm surprised how few programmers seem to know what this line does in C++:

```C++
Widget widget;
```

In C# or Java, that line essentially does *nothing*. It just declares a variable. But in C++, that line can do **a whole lot!!!** In C++, there's less need to allocate objects on the heap; you can allocate more on the stack, which is typically much faster. So you aren't calling `new` everywhere and managing pointers to the free store. Instead, that line simply instructs the compiler to allocate space on the stack and to initialize it using the class's constructor. That constructor can then kick off a whole slew of other actions.

For the same reason, I cringe when I see this:

```C++
glm::mat4 myMatrix;
myMatrix = glm::translate(glm::mat4{1.0f}, glm::vec3{1.0f, 0.5f, 2.0f});
```

This innocent code is wasteful because it initializes a 4X4 matrix and then immediately reassigns it. In a best case scenario, the return value from `glm::translate` will use move semantics to swap out the return value with `myMatrix`. The beauty of C++ is you can fill your code up with subtle, wasteful operations like this, but it's so fast it probably won't matter.

I remember when I first started reading K&R's [C Programming Language](https://www.amazon.com/Programming-Language-Brian-W-Kernighan-ebook/dp/B009ZUZ9FW), I was baffled by a similar statement:

```C
struct Widget widget;
```

Unlike C++, this doesn't call a constructor, it simply sets aside space on the stack. You also have to include the keyword `struct` to indicate `Widget` is a struct and not some other type. I think the first time I saw this, I was looking at some code making a POSIX TCP/IP connection. Seeing simply `struct Socket socket;` followed by code manipulating it kind of melted my brain the first time.

It scares me how many programmers can make their way without knowing these sorts of basics. I think graphics is one of those areas where a lot of programmers get their start. "I want to make video games" is a pretty common entry-point for a lot of us, including myself. In my case, I had to wait so long to write my first program that seeing that `Hello, World!!!` pop up on my console left me speechless. By the time I was done with Programming 101, the idea of making games was so low on my list. I enjoyed programming for the sake of programming.

## Coming full-circle
I think it's only fitting that I should come around full-circle after 10+ years. Every year I try to think what area I'd like to improve on -- where am I the weakest and what should I focus on learning? Last year Angular quashed that. This year, I'm glad I am focusing on topics outside just programming and computers. I am going to keep reading my Euler book, keep watching physics lectures and learning OpenGL.