# The Meta-Programmer
15 to 20 years ago, developers had to prove themselves by performing pointer arithmetic. It's seems somewhat dated now to read old articles where the authors brought this distinguishing feature of quality developers up. Pointers in themselves aren't super difficult; however, pointers to pointers and collections of pointers quickly baffle the mind! More to the point, pointers required developers to be able to look at a problem from various levels of abstraction. The unfortunate side to all this was that 15 to 20 years ago this meant that very few people could pursue a career as a full-out developer. As languages evolved and the concept of "references" were introduced (and garbage collection), managing pointers and properly cleaning up after yourself became less common. In fact, I think it was this one innovation that led to an influx of developers who could be productive. That's good right?

## What do people struggle with today?
I have worked on a lot of projects in my career. Some projects are pretty straight-forward; you simply pull data from a database, allow the user to view it and edit it, sending back the changes to the database. In most young, modest systems, their whole purpose is to simply replace an Excel spreadsheet and often the final output of the system is just another Excel spreadsheet someone uses to **manually** perform the "next step" in the business process.

Other systems I have worked on have been more complicated. For the more common case, applications are more complicated when you, the developer, can never get a solid yes/no to a question. To compensate for the indecisiveness, you simply add a config setting or a settings entry in the database. Then you turn around and implement both (or more) solutions so, even if someone changes their mind later (*and they will*), you don't have to redo everything. The funny consequence of *protecting* your architecture is you end up damaging it by making it *too complicated*. But I am more interesting in the less common case why systems are complicated.

Some of the systems I have enjoyed working on more than any others are systems that are driven by metadata. What does that mean? Well, as complicated as your home-grown internal application is, it usually only serves your internal processes and if you ever thought about turning it around and selling it as a stand-alone product, you would probably either have no idea how to make it reusable/marketable to other companies or you would effectively need to rewrite it from scratch. Comically or tragically, I have witnessed several such attempts throughout my career. In many of those cases, a company will correctly build a new system from scratch and then spend an enormous amount of time trying to implement/port their old system using the new one, only to find out supporting that level of customization means a level of development a small company simply cannot afford to implement.

Things like truly universal CRMs/CMSs/reporting suites/etc. are examples of platforms that have managed to be marketable. Even these often gain enormous hatred from the software development community. Who wants to work at a company for 2+ years only to one day be forced to learn SharePoint or Salesforce. I absolutely abhorred working with Crystal Reports or SSRS - as powerful as these tools were they more often than not made seemingly simple things extremely hard. And 90% of my users just exported the fancy-looking report to an Excel spreadsheet anyway, so what's the point? And that's a general trend with all of these universal platforms: they provide solutions to 70-95% of your problems and you have to hire a small army to implement the remaining. I used to have this theory that the majority of customizations were the remnants from when the company was still small and doing things *their* way. I feel like as companies mature, they just concede and train people how to use these sorts of systems as out-of-the-box as possible, even if that means it's less than ideal and retraining everyone.

The army of developers who build internal-only applications, write software to connect incompatible systems (ETL or whatever) and build customizations on top of other platforms are probably the same developers who once would have struggled with pointers. Myself, I have been in this situation several times. These sorts of projects were painful for me - they were more a matter of getting through all the typing and less about overcoming truly challenging hurdles. One nice benefit to these types of systems is that they usually involve talking to knowledge experts.

## What happened to the old pointer wrangler?
I am more interested in the developer who can write the early version of a CMS system that actually gains traction. I am interested in the guy who allows you to drag-n-drop form elements to customize your screen's layout in the CRM. I am more interested in the asset tracking application that allows you to add new, arbitrary fields to your assets and even provide basic validation. I think a lot of developers either a) can't understand why these are a different class of problem or b) would fail utterly to provide a viable implementation. Now, I have enough experience in the industry to realize that a shaky and poor implementation can be pushed and marketed long enough and hard enough that eventually the company has enough developers behind the product to save it before it goes under. I'm obviously not talking about those types of walking nightmares, but actual successful, well-planned, well-executed implementations undertaken by a small group of people.

Where I work now, Pinnacle 21, the software is unlike software you are likely to encounter anywhere else. The way I like to think about it is that, in most software systems, there is a pretty close relationship between the number of buttons and knobs in your users interface and the amount of functionality in the backend, almost one-to-one. However, at Pinnacle, the UI is very limited in terms of do-dads, buttons and knobs. Instead, most of the functionality is driven by the data you pass to the validator and the metadata describing what that data *should* look like, provided by another organization. There's very little you absolutely must do in order for a validation to be run in our system - sure - garbage will result in a lot of validation issues but, hey, that's why you using it in the first place... to find issues!

The reality is the software we develop at Pinnacle is just one example of a class of more advanced software systems. The common theme with these classes of systems is that in terms of features they are typically pretty lite; however, the number of different problems they can solve is enormous! A long time ago, many organizations would spend a vast amount of money to purchase and then upkeep mainframe systems and then spend even more money hiring specialist to write software on those systems. But then a couple smart guys write the first spreadsheet software and, even as modest as those early versions were, spreadsheets enabled anyone with a cheap desktop computer to track and analyze their data. Undoubtedly, spreadsheets have had the single greatest impact on business and probably on society as a whole.

## Conclusions
My theory is that as programming languages evolve, more and more people will be able to enter the programming profession, until the point where the average employee at a sit-down company will be writing small programs to do their daily jobs. I also think tools like Excel spreadsheets will add more features for building small UIs, reports, grabbing and exporting data to other platforms, collaboration, etc. What that means is programming as a profession will probably eventually go back to being performed by the small minority of developers who are clever enough to wrangle pointers and program systems driven by metadata.