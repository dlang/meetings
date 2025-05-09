
[Forum Post](https://forum.dlang.org/post/uhcopuxrlabibmgrbqpe@forum.dlang.org)

The December meeting took place on the 3rd of the month at 15:00 UTC. The following people were present:

* Andrei Alexandrescu
* Walter Bright
* Ali Çehreli
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Robert Schadek

The meeting lasted around an hour and a half.

## The summary

### Dennis
Dennis opened by reporting that he had begun working on a DIP to use `@default` as a means of resetting attributes. He had a question about what it should affect: should it reset all attributes (including visibility attributes like `private`), or only function attributes? The unanimous consensus was that it should only affect function attributes. He has since [submitted the DIP for Draft Review](https://github.com/dlang/DIPs/pull/236).

### Razvan
__@property__

In his ongoing effort to resolve old Bugzilla issues, Razvan had encountered a number of old issues regarding `@property`. Walter and Andrei had said in the past that we shouldn't support the feature anymore, and it's often recommended that people not use it. But people do still use it. So what are we going to do about it? Should we attempt to fix these old issues? Should we deprecate `@property`?

Walter suggested one possible approach is just to leave it as is. Those who see a benefit in using it can continue to do so, and those who don't can avoid it. He hadn't looked into the issues surrounding `@property` in a long time and asked if this was a viable approach. As far as he understood, the only time `@property` has an effect is when you take the address of a function it annotates. Razvan said `@property` sometimes interacts with other features in unexpected ways and the spec says nothing about what it's supposed to do. Maybe we could just add something to the spec saying it shouldn't be used.

Andrei said that's not what a spec is supposed to do. A spec tells you what happens when you, e.g., move your hand like this. A spec doesn't give you advice. If we're keeping it, we should spec it out even if we then never touch it again. Guides will say whether or not to use it. But we can't just leave it hanging unspecified.

Ali noted that he didn't include it in his book other than to say using it is discouraged.

Razvan asked how he should handle these `@property` issues in Bugzilla. Find someone to fix them or just document the behavior? Walter said the latter. He also suggested adding a recommendation to avoid `@property` in the best practices section of the documentation.

Robert spoke up then to suggest deprecating `@property` and releasing a tool that removes it from a code base. Then we should apply that tool to create pull requests for all dub packages using `@property`, and then in a future release, we kill it. Anyone affected by the removal can then run the tool on their own code. He added that we should do this with any feature we decide to remove. This is the modern way of software development: you don't just break someone's code, you break their code and give them a tool to fix it.

Átila said that would work fine here except in cases where someone is taking the address of an `@property` function. We aren't going to be able to make a tool for that. Robert said that's true but do it anyway. The tool should tell them, "This doesn't work for this case. Sorry." He said that if we test the tool with all the D code we can find on GitHub, he'd bet beers at DConf that we'd find no more than ten instances of code that would break.

After more discussion in a similar vein, Razvan said what it comes down to is that this is a broken feature and we don't know how to fix it. We need to just deprecate it. We shouldn't be keeping broken features around if we aren't going to fix them. Robert agreed. A tool to remove it from code will handle most cases, and for those people whom it doesn't help we'll have to help them migrate.

There was then some discussion about whether or not `@property` is fixable. Dennis brought up the case of when the property is a callable (and linked the [docs for Adam Ruppe's `arsd.jsvar` module](http://arsd-official.dpldocs.info/arsd.jsvar.html) as an example of the problem manifesting; see Adam's comment in the example code and his notes near the bottom of the page). If we want to support this kind of type that can store callables, then we need some kind of fix. Walter said that's an ambiguity for which no one had been able to settle on a solution.

Andrei said one intent of `@property` is to be a replacement for a data member. That's a good goal. Any improvement of `@property` should serve that purpose. If there's an ambiguity, it should go in favor of that. Átila thought that made sense. If there's an ambiguity, then just pretend it's a field. If there's only one set of parentheses, you call the callable. If there are no parentheses, you call the callable.

Robert countered by saying we're trying to make D simpler, and `@property` makes it more complicated for very little benefit. Properties are functions and then attributes like `@safe` play into it. Simpler is better.

Walter said that when you have special cases for a feature like this, you have to ask, is it carrying its water? He thinks `@property` isn't. Átila argued that there is something to be said for what it enables when it comes to refactoring. It increases the plasticity of the code. Robert said that's fine for existing users, but it doesn't help new users. Kill it and make a tool.

To wrap this up, Walter said that on the list of problems that we have to deal with, `@property` is down towards the bottom. Unless this is something affecting a significant number of our users, it's not something we should spend time on. He's happy with just deprecating or ignoring it.

Razvan noted that simplifying the language is part of our vision, and this seems like a good candidate. Walter agreed. Dennis suggested going through DRuntime and Phobos to look at all instances of `@property` and seeing if they could be removed. Walter agreed.

__CTFE writeln__

Razvan next brought up [a PR to implement a `__ctfeWriteln` built-in](https://github.com/dlang/dmd/pull/12412). It was currently stalled and needed Walter's approval. Walter asked Razvan to email him about it. He subsequently approved it.

__Reducing Phobos template bloat with traits__

Razvan next noted that compile time was a recurring topic these days and [cited a recent thread](https://forum.dlang.org/post/ahhgsnvoimnpsxabjasv@forum.dlang.org). Template bloat in Phobos is often cited as one of the culprits. He brought up the `std.traits.fullyQualifiedName` template as a specific example. Every instantiation of it results in an additional 10 to 15 template instances. In this case, changing it to a `__traits` would be a big improvement. On the other hand, we've been reluctant to add new traits. What should the policy be when there is an obvious opportunity to reduce Phobos template bloat with a new trait?

Walter said that `__traits` is meant to be ugly. It's meant to be a sort of catchall for random stuff. Reducing template bloat for a commonly used template is a very worthwhile reason to add a trait.

Walter then noted that `isPointer` had come up in the same thread. He looked at its implementation and found it was trivial. Trivial stuff generally shouldn't go into Phobos. The library shouldn't be a mile wide and an inch deep of functions. He went through Phobos and removed all the uses of `isPointer`, though he left the template there for backward compatibility. He also did that with `implicitlyConvertible`. In both cases, he replaced the usage of the templates with the equivalent `is` expression. Átila noted that the templates are still necessary if you're static mapping or filtering, and Walter said that's why he didn't remove them.

Walter said that's one approach to getting rid of template bloat: flattening out the number of instantiations. There are a number of cases where Phobos templates forward to another template when they really didn't need to. Andrei said that's mostly historical, in some cases it was done to avoid bugs, and he agreed that flattening out instantiations is a good thing to do.

Andrei said that we should discourage bad practice as a matter of course, but the notion that we have template bloat because of abstraction is a bit of a problem. Consider that `__traits` is ugly because it was intended to discourage direct use and for use instead inside an abstraction layer. Now we're kind of changing the charter and saying you should use `__traits` because if you wrap it in a template it's going to produce code bloat. What's the deal?

Walter said the problem isn't wrapping `__traits` in templates. Templates should use `__traits`, not a whole bunch of wrapper templates. The problem is that we have to ensure that we aren't creating a nest of template instantiations because too many nested templates slow compilation. We want Phobos to serve as an example of how to do things, but it also needs to be performant. That means making some concessions to performance.

Robert said that doesn't solve his problem with compile times. He has a project that doesn't use much of Phobos, but compile times still suffer. The compiler is slow. Making Phobos a tiny bit faster doesn't change that. Changing the template implementations is like saying you can't use our screwdriver with big screws. Doing that gets you maybe from Level 30 to Level 29, but it doesn't solve the problem.

Walter said Robert was right about the compiler. But that's a big and difficult problem to solve. What we can do right now is reduce the nesting of templates.

Razvan sees these as two completely different problems. One is that the compiler is slow. The other is that Phobos has template implementations that are wasteful. We need to solve both. And even Walter's changes are only a minor dent in the Phobos bottleneck. He had previously dug into `hasUDA` and it was scary how many recursive expansions he found. And we don't provide the tools for the user to understand where the bottlenecks in their template usage are at.

Walter brought up the `-vtemplates` compiler switch for reporting template statistics which was added at Weka's request. Razvan said there had been some complaints raised that it doesn't provide enough information. Átila said what we're really missing here is compile-time profiling.

Andrei suggested a longer-term project we can consider. Most of the time when people modify their code, they aren't changing the template instantiation but the code around it. That goes to the idea of caching or pre-compiling the template so that further compilations don't take so long and only compiling the template when the instantiation has changed. This idea has come up a number of times over the years, and it may be time to bring it to the front burner. Átila brought up C++ 11's extern templates. Andrei said that's not what he meant. He's talking about a little database that the compiler creates to save the instantiations. It's not extremely difficult, but it's a big project. And he suspects it will transform the problem into a non-issue. He thinks it would be an excellent project.

(This discussion ended after that, with Razvan saying, "Sounds good to me". My impression is that we're going to explore the caching idea at some point.)

### Mathias
Mathias wanted to let us know about two things he was working on regarding dub.

The first involved dub's settings file, `settings.json`. As he put it, have you ever seen a program that asked you to write its settings using JSON? There had been [some favorable responses to the idea of moving to YAML](https://github.com/dlang/dub/issues/1832) from some core contributors a few years back. It just needed someone to do it. He asked if we were okay with the move. Átila said we probably shouldn't keep JSON, but wondered if YAML was the best choice. What about TOML? This sparked a minor bikeshedding discussion, but there was no major opposition to Mathias's plan. ([He has since opened a draft PR](https://github.com/dlang/dub/pull/2546). SÃ¶nke Ludwig wants to see a broader discussion of this before finalizing it, so I expect Mathias will ask for community feedback at some point.)

The second issue was dub's build cache. When you build something with dub, you end up with a large number of build artifacts in the working directory of the package being built. It's not just a few MBs, but hundreds. Different flags and compiler versions create separate directories, so it can quickly go into GBs. He had [an open pull request to change that behavior](https://github.com/dlang/dub/pull/2542) so that the artifacts go into a .dub/cache directory rather than the CWD (the PR has since been merged). That's the first step, getting output into a single location. The next step was to make it configurable for the users.

I asked Mathias who is the best POC (point of contact) for dub. Was it still SÃ¶nke? Was it Mathias or Jan Jurzitza? He said that would be him (Mathias). Several months ago he had gotten annoyed with the state of dub and had done a bunch of work on it. SÃ¶nke is around from time to time, but he has little time for it these days. Mathias usually leaves his PRs open for a while so someone can review them, and sends them to Jan, but he manages most of the PRs himself.

### Robert
__D frustrations__

Robert let us know about a programmer in Symmetry who had a project in D that was going to transition from a toy to production. The programmer ultimately decided to use C# on the production version. There were a number of little annoyances and pain points with D that led to that decision. Robert said that he had been with D so long that those little things don't hurt him anymore, but on reflection, they do hurt.

He noted that with the 2.101 release, Jan Jurzita's LSP (Language Server Protocol) implementation started crashing. Compile times keep getting slower. Why doesn't an LSP implementation come with DMD? Why don't we have a compiler daemon? Why aren't his build times sub one second? Why are we talking about adding `@live`, tagged enums, and all that stuff? He just wants a simple language that works out of the box with LSP that isn't too crazy. We're talking about how to fix memory safety, but it's such a complicated issue. We're trying to make the language simpler, but the combination of features being added is exploding in our faces.

When Robert uses D, he feels there's a simple, straightforward compiled language in there that knows what it is. Sometimes he feels he's the only one that sees that and that other people want more elaborate features. Átila agreed and mentioned his DConf '22 talk, saying we need to fix what's already there before we consider more features.

Robert feels like the way we write and build software with D is 15 years behind the curve. Compiling isn't really a batch job anymore. The compiler should be a daemon with built-in LSP that compiles files when they change. Andrei's earlier comments on caching templates would benefit from this. He knows it's a gigantic change, and would require a complete rewrite, but it's easier to start now than two years down the line. He knows he's not capable of doing it, and he doesn't have the time even if he were, but he thinks it needs to be done.

Though this frustrates him sometimes, he enjoys writing D every day and wants to do it for the rest of his life.

Walter asked if Robert could get the Symmetry programmer who moved on to C# to provide a list of the issues that caused him to give up on D. Robert said he would try. Walter said that the horse has left the barn with this programmer, but those issues are probably affecting other people. He noted the forum posts about the guy who dropped D for Jai. Walter had the impression that the person had filed multiple bugs, but it turned out he had filed three. Walter had been able to fix two of them. He then sent an email asking if there were other bugs to fix, and the person was very nice but couldn't come up with other bugs to fix. However, those bugs he'd fixed were important ones, and Walter hadn't been aware of them before. They were ABI problems with the Microsoft C compiler. The MS documentation was wrong about the ABI, so the code generation was wrong.

This was followed by a discussion about how to solve the small annoyances and issues people have that they find so frustrating. Walter said the big problems aren't so easy, but the small ones should be. He said we absolutely need to fix them, but we need to know what they are. He expressed frustration that he often has a hard time getting actionable items from people when it comes to the small stuff. That's why he's always asking for Bugzilla issues.

Robert understood, and he had seen the same. He reiterated that his Number One actionable item was adding an LSP implementation to the compiler. He then explained the benefits this could provide if the compiler were running as a daemon, with the example of incorporating tools to automatically change code in the editor when we make breaking changes to the language. He said Jan's work with serve-d was great, but this would probably be better and quicker if implemented natively in the compiler.

Razvan said the problem is that we don't support incremental compilation. He brought up past work done attempting on dmd as a library that encountered problems because of that. Robert said that's one reason why moving to a daemon would require a complete rewrite. Incremental compilation is a thing, and we should have it. He suggested everyone install Android Studio and play around with Flutter and Dart. The developer experience is like very good crack. Razvan wondered if SDC supports incremental compilation.

We ended this discussion with Walter being open to looking into this. Robert said he would email a link to an interesting talk about a compiler design that he thinks will lend itself to running as a daemon and supporting incremental compilation. He emphasized that he wants to still be discussing D with us 40 years from now and that all of his feedback, even the critical bits, should be taken as loving input.

__Bugzilla to GitHub migration__

I then asked Robert about the script for the Bugzilla to GitHub migration. He said he planned to set things up so that I could run it myself, but he needed to make the time to get there.

__JSON parser__

Robert had recently implemented a quick JSON file parser using `SumType`. He'd wanted to see if he could use `SumType` to represent HTML/XML/JSON/YAML/TOML. Apart from assigning `void` as a value to the sumtype, it worked in `@safe` and was very nice. The implementation was only around 150 lines. It took him only half an afternoon. `SumType` is really awesome, and we should really do something with it. `std.json` works, but could use a revamp. We don't have `std.xml` anymore. We don't have `std.yaml` or `std.html` or `std.toml`, etc., and those should be in Phobos. Átila agreed. We just need someone to write them.

### Ali
Ali reported that he had finished the new D program at work he had [told us about in the November meeting](https://forum.dlang.org/thread/citxnklerlvqmybyoaat@forum.dlang.org). It had uncovered a performance issue with `std.file.dirEntries`. As for the program, he was happy with the end result.

He said he'd used `std.parallelism.parallel` again and speculated he's probably among the people who've used it most. He said it helps tremendously. It's very simple and everything becomes very fast.

### Andrei
Andrei said he had been doing a fair amount of C++ coding at work, and it had given him an interesting perspective on things. One area in which the D language can do a lot of good is simplification. A simpler language is really useful. He'd noticed that some of the features in C++ were designed to fight against other features that were poorly designed or implemented, and that adds complexity.

He suggested that going forward we should consider removing limitations of existing features, making them simpler and more general. This should be a high-level goal, and new features should be targeted at that. Caching template instantiations, for example, would be a compiler addition that would be high impact toward that end. He'll let us know if he has more such ideas. Walter thinks that's a good goal. Andrei said the main thing is that we have things that don't work that should (like `@property`), and things that work that shouldn't. It's a bizarre situation, and people latch onto that.

That segued into a brief discussion about removing features. Átila said that we can't do that until we have something like "Editions" so that we can make non-breaking breaking changes.

### Átila
Átila said he had been trying to figure out what to talk about for DConf Online. He went back to something he was doing a long time ago that Andrei had talked to him about, which is a better, simpler way of doing reflection. The disparate APIs we have are complicated to use. And with all the talk about templates and compile times, why not do it using string mixins? He'd started working on a library called "mirror", but never finished it because the only way to know if the API was good and worked was to use it for something real. So he had been trying to write another Python wrapper library from scratch using only string mixins. It had been relatively easy, but there's no prior art in this space, so it's kind of hard to figure out how to do it. He thought it would be an interesting talk. He noted that the first thing he noticed was how fast the compile times were. (I think [his talk turned out to be interesting](https://youtu.be/OeCY1QotnRw)).

I brought up a problem I'd had years ago when I implemented a mixin-based OpenGL binding for Derelict that could be configured via string mixins as a free-function API or as a struct-wrapped API (e.g., `glClearColor` vs. `GL.clearColor`). Something was causing the struct configuration to take over 10 minutes to compile. I could never pin it down, so ultimately gave up. We have no good way to debug string mixins. (Atila and I talked a bit about this in [our DConf Online Q & A session](https://youtu.be/UsIO-b50OAc)). Átila said that debugging templates isn't a great user experience either.

Andrei said he and Walter had talked about this in the past, and he thinks there's an opportunity there. With string mixins there are two stages: one is creating the strings and the other is mixing them in. He speculates there's a cost to each side. His suspicion was that the creation of the strings was more expensive. Walter's idea back then, which Andrei thinks is worth revisiting, was that the compiler could easily create the reflection for a module/struct/class in one shot, rather than assembling it piece by piece. You ask for the reflection, and it gives you a struct whose fields are strings representing the info. Átila said that's what his library does, and Andrei suggested it would be more performant if it were in the compiler. Átila's intuition is that a bytecode interpreter for CTFE would have more impact on speed.

And anyway, just by mixing stuff in rather than using templates, Átila had found the performance improvement surprising. The challenging part had been deciding how to structure the code. With templates, everything is neat and tidy and clean. If he needs to manufacture a function out of the aether, he can put it in a template and just instantiate the template, whereas with mixins he has to generate the string for the function and then put it somewhere.

I noted that Átila's talk was a complement to Steven Schveighoffer's. (Steven's talk was [Model all the Things!](https://youtu.be/GFvh6Hbc-3k)).

### Walter
Walter had been working on a long-standing bug with `ModuleInfo` which had been around for 10 years. Reading the bug reports on it, no one seemed to understand how `ModuleInfo` actually works. Part of the problem was a documentation problem. He was going to see if he could unstick all of that. This was a barrier to people using D and it needed fixing.

He then said that he had noticed in discussions on HN and elsewhere a tectonic shift appears to be going on: C++ appears to be sinking. There seems to be a lot more negativity out there about it these days. He doesn't know how big this is, but it seems to be a major shift. People are realizing that there are intractable problems with C++, it's getting too complicated, they don't like the way code looks when writing C++, memory safety has come to the fore and C++ doesn't deal with it effectively, etc.

For us, that's good news. The bad news is that Rust, not D, is filling in that gap. He wants to keep plugging away at memory safety. That won't help us in the marketing department, but at least in the technical department, we can get it there.

Robert thinks Rust has won that game. We're the second person to the moon. Put `@safe` on top, disallow taking addresses of the stack, don't allow returning `ref`, and don't allow pointer arithmetic. That's as safe as we need to be. D's niche is on top of Rust and under TypeScript. That's where we need to be. That may not be the most popular opinion in the group, but he was alone in his room and no one could hurt him. He thinks C++ *has* been sinking, but it's probably going to keep sinking until he's dead and will never sink completely, but Rust will take that over. Rust is also taking over some of the web world because it compiles easily to web assembly.

Walter agreed that C++ will never disappear, but he thinks it will fade into the background. He appreciates Robert's opinion. It's something we need to think about.

Next, Walter said he needed to record [his DConf Online talk](https://youtu.be/RO7IJvxtwjQ) and send it to me.

Then he said something else he'd been doing lately was asking people to provide specific lists of problems they're having. Vague generalizations aren't useful. He needs action items that he can fix.

As always, he just has too much work to do. He has to prioritize what he's getting done. Looking ahead, he was thinking about a built-in sumtype. He wasn't sure about the implementation schedule for it, but a spec that is vetted and would work is a good thing to have. Don Allen had posted a number of problems with ImportC that he needed to look at. He was hoping to fix all the low-hanging fruit with it that he could.


## Conclusion
The next meeting was a Quarterly meeting, meaning industry reps were involved, and took place on January 13, at 14:00 UTC. "D frustrations" was a major topic of discussion in that meeting. I'll have the summary as soon as I can. Unfortunately, it's going to be less detailed than usual. I managed to botch the OBS Studio settings once again and ended up with no audio output. My mic came through, but that's all (this is the second time I've done that with a meeting, and I'm determined to make it the last). This means that I'm going to have to rely on help from the participants to cobble together a summary. I know several people enjoy reading these, and I feel terrible for such a silly mistake, so I'll try to gather as much detail as I can.

I would like to remind everyone to continue sending me your own frustrations with D and your wishlists for the future (social@dlang.org). We're going to use this information to help guide changes in processes and management and to mark a path forward.

I can't provide details at this time, but I can report that we have recently taken a big step toward our long-term goal of getting a handle on the chaos. Most of the people (everyone?) involved found it to be a positive experience. I think I can safely speak for them when I say we're looking forward to what comes next. At some point, hopefully next month, I'll be able to say more about it.
