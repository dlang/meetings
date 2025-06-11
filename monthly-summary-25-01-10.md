The D Language Foundation's January 2025 monthly meeting took place on Friday, the 10th, and lasted approximately two hours.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Ali Çehreli
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Adam Wilson

## The Summary

### 2025 Objectives

Razvan thought it would be good to have some poster projects to work toward in 2025. He was wondering if there were any projects to add to the list of those he was already monitoring.

Rikki said that was why he'd brought up target dates in [the December meeting](https://forum.dlang.org/post/uvqbluehjkljthuhzsjm@forum.dlang.org).

Adam said he'd like to see Phobos v3 ranges finished, though that wasn't his project.

Razvan said we didn't need to decide this now. He suggested a planning session to discuss what we'd like to see and choose from the list.

I reminded everyone that we had a sorted list of projects we had put together in a planning session last year. I said I could email the list for review, then we could go over it and revise it in a meeting. I recalled that collections had floated to the top and allocators were tied to that.

Adam said Rikki had put forward a PR for allocators in Phobos v3. A long discussion had ended with everyone agreeing that we didn't actually want to build allocators, but rather an interface for allocators.

Átila asked how it related to Paul Backus's work. Adam said Paul had suggested the question ought to be "should we have allocator-aware containers" instead of "should we have allocators".

Átila said Paul was working on a new interface, which was why he'd asked. Adam thought Paul wanted to be involved in the interface design process. Átila said he probably should be. Adam said we should have a conversation about what we wanted to do regarding allocator solutions.

Rikki said Paul had given up on his allocator project because DIP 1000 was dead. Átila said he'd been talking to Paul about DIP 1000 recently trying to figure out what we needed to do to move forward. We should see where that went before declaring it dead.

### Google Summer of Code projects

Razvan and Teodor Dutu had been preparing our GSoC application. They got in touch with the maintainers of another project that had put forward successful applications for three or four consecutive years. They compared those successful applications with our failed ones so we could copy the strategy.

Their intuition was that our project ideas repository was stale. We'd been using the same repo for a few years. The last commit was eight or nine months ago. That didn't look very good. They wanted to create a new repo dedicated to GSoC.

They were at the point where they needed projects. The projects we chose should be important to the community and something the students could work on for three months. 

We also needed mentors. The application period was scheduled to start on January 22nd. He asked us to think about it. If we had any tasks in our ongoing projects that we were putting off or needed help with, then it might be good for a student to work on. It didn't have to be anything complicated.

I added that I had learned from the GSoC mailing list last year that they put a lot of weight on the presence of project time estimates. Project descriptions should always include a good time estimate.

Razvan said he could make sure the projects all had estimates. The important thing for now was to get some projects and mentors. He thought the community should be involved in that. I suggested he post an announcement in the forums.

I asked if everyone was good with holding a planning session next Friday. There were no objections.

(__UPDATE__: In our planning session, we discussed potential GSoC projects. We also agreed that the projects Razvan was currently tracking are what we need to focus on. We can look at more once editions are sorted. We were later accepted into GSoC and ended up with two projects: Translate DRuntime Hooks to Templates, assigned to Albert Guiman, mentored by Teodor Dutu; JSON Library for D, assigned to Fei, mentored by Adam Wilson.)

### Feedback on memory safety in kernels

Rikki reported that someone in the Discord had declared that D's story wasn't good enough for kernel development. In some cases, it was basic things like being able to hook `new` and `delete` into `malloc` and `free`. There were no examples of that. For that, we could undeprecate `delete` and make it a compiler hook that DRuntime didn't implement. There were other cases, too.

When using C, you had access to static analysis capable of doing escape analysis and type state analysis that had been available for 20 years. This might seem optional, but it wasn't. You could see that by just asking yourself one question: when was the last time you had a kernel crash that wasn't related to user or driver error? For him, it was 20 years ago when this stuff was coming into play.

Átila wanted to know what `new` and `delete` had to do with kernel development. Rikki said you wouldn't have the GC for it, and people wanted to have a nice syntax for memory management.

After a back-and-forth about how it was done in C, Átila repeated his question. Rikki said that `new` had compiler hooks already, you just had to write them, but there was no example of how. Since `delete` was deprecated, you couldn't just write your own.

Walter said you'd just use `free` instead of `delete`. He didn't understand why using `malloc` and `free` was a problem compared to `new` and `delete`, especially if `new` and `delete` were just shells around `malloc` and `free`.

There was some more back and forth, then Rikki said to think about placement `new` and `destroy`. People didn't just use structs. They wanted to use classes in kernels, too. Walter noted we did have a PR for placement `new`. Rikki said that helped, but we were talking about a custom runtime. The request wasn't about DRuntime.

Walter said it could be done with the placement `new` PR. It gave `new` an extra parameter to specify where the object would be instantiated. It didn't allocate anything. It was a replacement for the `emplace` template.

Átila said no one was complaining about C as a language for writing kernels, and D could do whatever C could. He didn't understand what the issue was. Rikki said that didn't mean they didn't want a better experience. More importantly, we didn't have the tooling C had. That's what this was about.

I asked how this was different from the other kernel projects in D? There was [a talk about using D for kernel development at DConf '23](https://youtu.be/VL8F7rnrCCA) where I didn't recall this coming up, and there had been a few people doing it as a hobby. Was this a problem specific to how this person wanted to do things, or was it a more general issue?

Rikki said in terms of memory safety, the person just wanted to be close to what C could do. Átila said it already was.

I asked why it hadn't come up with other kernel projects. Rikki said it was because the person had experience working on Linux, and the bar was a lot higher when working on actual, production-level kernels.

Walter asked what feature C had for kernel development that D didn't have. Rikki said it wasn't about kernels or the standard, it was about tooling. The tooling for C had escape analysis and type state analysis. We didn't have enough of that stuff.

Walter said it was news to him that GCC had type state analysis as part of the C compiler. Rikki said it was a vanilla feature. They even had things like AddressSanitizer working in the kernel. Átila said he still didn't understand the issue. He'd used AddressSanitizer in D multiple times.

Rikki said it wasn't just GCC. Visual Studio had stuff, too. Everyone had had stuff like type state analysis for 20 years. He posted a few links in the chat:

https://en.m.wikipedia.org/wiki/Sparse
https://lwn.net/Articles/689907/
https://devblogs.microsoft.com/cppblog/improved-null-pointer-dereference-detection-in-visual-studio-2022-version-17-0-preview-4/

Walter said he knew about AddressSanitizer but hadn't heard of any mention of this stuff being built into the compilers. Rikki said that though we had AddressSanitizer, we didn't have the range of tools available in C. We weren't meeting the bare minimum of what was expected for production development.

Átila asked what Riki proposed. Rikki said that for now he just wanted to establish that what we had in DIP 1000 was not escape analysis compared to what C had and that type state analysis in the form of nullability was considered a standard thing. He just wanted to make us aware of it and wasn't saying that we had to make any decisions about it today.

Walter asked if Visual Studio was just tracking null pointers. Rikki said that was the primary thing they exposed, but he was sure they analyzed uninitialized memory and all the other stuff, too.

Adam said he knew a little bit about this because he used Visual Studio every day. They came out with a bunch of new static analysis tools for C++ in 2022, but they'd had it since 2017 as far as he was aware.

Visual Studio had quite a bit more than what Rikki had mentioned. The `/analyze` switch had about 20 different options of things you could do. Most of them were related to how you dumped the analysis, but you could even have custom analysis with plugins.

Resharper, a tool he used regularly with C#, had recently released a new tool for C++. It was famous for refactorings and analyses of numerous things. Your standard C and C++ developer in the Microsoft ecosystem had had these things for at least eight years now.

Átila said he'd just skimmed through some of Rikki's links and agreed that static analysis was a good idea. He still didn't see what this had to do with kernel development or `new` and `delete`.

Rikki said that Linux required static analysis. Átila said static analysis was for all development and wasn't particular to kernels. Rikki agreed but said this was an example of where it had been used for 20 years. Átila said it had been used in many other things as well. He'd lost track of how many projects on which he'd had static analysis turned on.

Rikki wasn't discounting that. He just wanted to give a clear example to show this stuff was standard. Átila said it would have been better to start with "other languages have static analysis, it would be good for us, too."

Rikki said that wasn't what he'd wanted to focus on. He'd brought it up before. Here, he'd just wanted to let us know that someone had found that D's story wasn't good enough for kernel development.

At the moment, `new` was fine, but `delete` wasn't. The static analysis stuff wasn't fine. Since Walter was looking into DIP 1000, this stuff should influence him to recognize that we were nowhere near what C had.

I noted that on the other hand, we had the Multiplix project saying D was fine for kernel development. If we were going to talk about static analysis tools, that was one thing, but we shouldn't tie it to a discussion about kernel development in D.

Rikki said his next agenda item was escape analysis. This was just a use case. I asked if anyone had anything else on this issue and no one did.

### -verrors=context as default has exposed inconsistent AST node locations

Dennis said that with a recent PR, error messages by default now displayed the line number and a little caret showing where the error was supposed to be. In checking the error messages, he'd seen the caret was sometimes in a suspicious position.

He'd also found that the locations of the AST nodes weren't always consistent. You often wouldn't notice as usually, all you cared about was the line number. You would see, for example, that for a function call, the caret pointed to the opening paren, but for other expressions, it pointed at the start of the expression, which is what he would expect.

He wondered if we could make it consistent such that, for example, the location of AST nodes was always the first character that was parsed as part of that node. Were there other situations where we really wanted the caret pointing at a specific point somewhere in the middle of an expression?

Walter thought it should be on a case-by-case basis. He didn't think we could establish a general rule for it.

Dennis said that a parser generator like Tree-sitter always had a start and end location for each production rule that it directly derive from the grammar. It was from the first character being parsed to the last character being parsed. We didn't currently store the end location, but for the location we did store, we could just say it was the start location. That would at least make it more reliable.

Walter said we didn't store end locations because they blew up memory consumption. If Dennis wanted to change what the start location was, how it was stored, or which one was picked, he was fine with that. Dennis said the end location could be derived from the start location by reparsing it from that point.

Rikki said the caret should point at the operator in unary and binary expressions. Otherwise, it should be the start. Dennis asked why.

Rikki asked where you would put the caret for `a.b.c`. Átila said at whichever dot it applied to. Rikki said yes, at the operator. Walter thought it did that already. Dennis did, too. Rikki said that was why just changing it to always be at the start wasn't quite right. It should be case by case.

Steve asked if we could just store the size instead of the end. Wouldn't it normally be small? Walter said it normally was, but it could easily be 30,000 characters off. There was always someone who'd write code like that. And then you'd have to store that offset for every AST node. That was a big memory consumption problem.

Storing the column was a similar problem. D didn't used to store it because of memory. The more details you stored in every AST node, the more the memory exploded. It all looked fine with small examples, but there was always somebody with a 60,000-character line.

He said Dennis should just fix any error message he found where the caret was in the wrong position. Dennis said it wasn't necessarily wrong. It was kind of subjective.

Walter thought it was pointing at the operator because expressions could get long and complicated. If you were pointing at the beginning of the expression, you wouldn't know where in your complicated expression the problem was. He thought pointing at the operator was the most general solution.

Átila said that was just for binary and unary expressions. He said Dennis was proposing to always put it at the start. We could make an exception for operators, but what about everything else? He didn't think it would work in practice.

Walter didn't think it would work, either, because of long, complicated expressions. It just wouldn't be useful. Átila said he'd have to see examples. Rikki said we were all proposing the same thing: a mixed model. There was some confusion then about what was being proposed.

Dennis gave the example of `a.b.c()`. If there was an error in `c()`, the caret would currently point at the opening paren. He thought that was weird. Rikki said the identifier should get the caret in the case of function calls, not the opening paren. Átila agreed.

Steve asked what should happen if it was an expression rather than an identifier. Átila and Walter said that was a good point. I said it sounded like we were talking about a case-by-case basis.

Steve asked if it were possible to somehow identify the expression one way, like underlining it, and then have the caret point at the operator. Walter thought that would require storing three locations. Steve said it wouldn't. The operator knew where it and its operands were already. Rikki said you could walk the tree and get the other positions, but he didn't think you needed to do anything like that.

I asked what GCC and Clang did. Dennis thought most compilers had a start and end range and had carets under the whole expression that was relevant.

Walter said he could see marking three locations: the beginning, the operator, and the end. He thought that would be better, but then it was three locations you had to keep track of. Dennis said you could compute them if you had the start location.

Rikki said another way to do it would be to draw an AST tree and just give everything. Walter said he dumped AST trees for debugging purposes. Beyond a certain level of complexity, they became very difficult for humans to read. Dumping the AST tree wouldn't be helpful, especially given that the compiler rewrote a lot of those AST nodes.

Dennis [posted a link in the chat](https://github.com/royalpinto007/dmd/blob/3fa3274065ec3650681251df9fd80fc5dfceb678/compiler/test/compilable/diag20916.d#) showing an actual example of what the caret looked like on call expressions. He thought it should point to the start of the function name rather than the opening paren. Rikki thought everyone was on board with that.

Walter said all of the function calls there were identifier function calls. What if it was a more complicated expression? Dennis said it should still go at the beginning. Rikki asked if Walter had an example of a complicated expression.

Walter posted the following in chat: `(a.(b.((c)().e.f)))()`. He said the problem was with `f`. What were you going to point at? Rikki said `f`. Walter said that wasn't where the AST was. It was to the left and would start with `a`. The expression that was the function to call started with `a`, not `f`.

Rikki said if you were generating an error, wouldn't it be on `f`? Walter said you were then talking about walking down the AST because the tree wouldn't be on `f`. This led to a discussion to clarify that the identifier for the function call wasn't `f`, but was actually the entire expression.

Martin noted in the chat that with GCC, the caret was in the middle of an underlined expression. He posted this example:

```
<source>: In function 'void foo(int, int)':
<source>:2:14: error: return-statement with a value, in function returning
'void' [-fpermissive]
 2 |     return a + b;
 |            ~~^~~
Compiler returned: 1
```

Steve said that was exactly what he was thinking of. Dennis noted that would require expanding the error interface so that it was passing two locations instead of one and asked if that would be a problem.

Walter asked where you'd store the two locations. Dennis said he wasn't thinking about memory now. He was just thinking that GCC and Clang had a better caret. He would worry about the implementation details later. The first course of action was that the error interface needed more information than a single location, whether it was three locations or a location and two offsets.

Walter suggested changing `Loc` so that there'd be no need to change all the code. Dennis said that `Loc` would then get bigger and explode the AST size. Steve suggested having an overload that took a single location and one that took three. Dennis said that could work.

Dennis thought we'd spent enough time on this. He said he'd look further into it, hack at the code a bit to see how far he got, and then update us later.

### Rikki's DIP updates

__Matching types__

Rikki reported he'd met with Walter and Átila about his [matching types DIP](https://github.com/dlang/DIPs/blob/master/DIPs/DIP1048.md). The result was that Átila was going to put together a counter-proposal for Rikki's value-type exceptions (the matching types DIP was one of a series of related DIPs Rikki was working on, including value-type exceptions). He said that Átila hadn't quite understood why Rikki had spent three or four years getting to this design.

Átila said that after that meeting, he'd gone back and looked at Herb Sutter's proposal again ([direct PDF link](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0709r4.pdf)). He understood it a bit better now and said he wouldn't copy it for D. That meant thinking about it.

Rikki said he wasn't surprised. Walter said it was good information.

Átila said he kept forgetting that the core idea of the proposal was essentially one actual exception type. It had two pointers in it so that it was always the same size. There was no dynamic type information. You always caught the same type from the point of view of the type system. But it was also geared toward error codes. One example that kept coming up was that it could support both POSIX and Windows error codes with the same type with two pointers.

Átila thought he agreed with them that catching exceptions by type hadn't seemed to be useful. He needed to think about it some more.

Rikki said to keep in mind that it had to work in BetterC. It couldn't be tied to DRuntime. Átila asked why, and Rikki said it didn't need to be. The more we could avoid that, the better.

Átila said it should work for "BetterD" and not be hamstrung by C. Rikki said it was his motivation, anyway. Átila didn't think it should be a requirement.

__Stackless coroutines__

Next on Rikki's list was his [stackless coroutines DIP](https://github.com/dlang/DIPs/blob/master/DIPs/1NNN-RAC.md). He would assume everything was good if Walter and Átila didn't comment [in the forum thread](https://forum.dlang.org/post/asbfsajrtasfqewzbsce@forum.dlang.org).

Rikki said Adam had been happy with the design. It was about as good as they could get it without experimenting with an implementation. Even if it was accepted, he didn't expect it to get turned on until we had multi-threaded safety and a bunch of other stuff sorted.

(__UPDATE__: I merged the coroutines DIP in April and started revising it with Rikki's help, but then I put it aside for a while. I recently asked Rikki to meet with me so we can work through it in real-time. We'll do that as soon as he's available.)

__Escape analysis__

Next was escape analysis. Rikki said he had the functionality "kind of sorted out". It was clear we didn't need value tracking for garbage-collected memory, as that was a huge usability issue.

Walter said that DIP 1000 *was* escape analysis and asked what the basic idea was behind Rikki's proposal and how it differed. Rikki said it described the full escape set. Walter asked how that could be done without annotations like `return` and `scope`. Rikki said you could use those as well.

With his proposal, you would use `@escape` with an output identifier and an optional relationship strength. Walter noted that everyone complained that DIP 1000 was complicated. This sounded like a more complicated scheme.

Rikki said that DIP 1000 got a lot of things wrong. For example, relationship strength was determined by attribute order. The strongest relationship strength was related to protecting the reference, not the variable. He didn't fully understand the semantics of that, but there was a difference between those two things.

DIP 1000 also didn't understand garbage collection. Átila said garbage collection wasn't `scope`. Rikki said it didn't do value checking. You literally couldn't escape garbage-collected memory in its model.

Dennis said being able to escape memory was the point of garbage collection. Rikki said the problem was that you effectively had typed pointers there. Then like C++ managed memory, you required attributes to enforce the input.

Walter said if Rikki's proposal required more annotations, he didn't see how we could sell it given how everyone complained about DIP 1000 annotations. Rikki said it used inference. Walter said that would require data flow analysis across the whole program.

Rikki said it would still be local like DIP 1000. Walter said DIP 1000 couldn't always infer. You had to annotate some things. Inference meant running the analysis across all the code on every compile. Rikki said it wouldn't need to run on every compile if you output it to a `.di` file, but we couldn't do that currently because it was running before semantic.

Walter said that `.di` generation was a good idea, but had been a failure in practice because people didn't really use it. Rikki said there were reasons for that and we were dealing with some of them. Walter said the reason was that it was inconvenient. It was so much easier to just import `foo.d` instead of creating a `.di` file.

Rikki said that build managers could deal with that. Átila said he'd thought about doing that, but one thing that had stopped him was inlining. Rikki said we had LTO.

Walter said that without really understanding the proposal, it sounded like it was more complicated than DIP 1000 and required either more annotations or a massive slowdown in compilation speed.

Rikki said it was similar to DIP 1000 in being a forward-only parse, but it was a couple of notches more expensive because of things like value tracking. It did tracking similar to how DIP 1000 already worked.

Walter said the `@live` feature did DFA. It was a computationally expensive operation, exponentially so in some cases. If we turned that on for normal compilation, compilation speeds would be the same as when the optimizer was turned on, and that was dramatically and exponentially slower. He did all his builds with the optimizer turned off because it was too slow otherwise, and that was because the optimizer did DFA. This was why he'd resisted features that required it.

Inferring DIP 1000 characteristics was a dramatic slowdown, which was why the compiler only inferred attributes on things it absolutely had to, like templates and `auto return` functions. If you wanted to infer attributes generally, then you'd have a dramatic increase in compile times.

Átila thought there was something to the idea of `.di` files. Walter said we'd then be changing how building things worked. Átila agreed but said if you had a build system that did it automatically, you could turn on inference and only pay the price when you changed the default.

Walter said that wouldn't work because functions called other functions. If you looked at any non-trivial project, every module imported every other. That meant you couldn't get a stable `.di` file without recompiling everything. You couldn't do it module by module. Rikki said it was more like a dub package that you recompiled. It was a different level of granularity than what the compiler understood.

Adam wondered if we could make these features--`@live`, DIP 1000, all this DFA stuff--explicitly opt-in forever. If the compiler encountered something like that and you hadn't turned the switch on, it would put out an error message like, "You need to turn this thing on." Then we could say, "You can still have fast compile times, you just can't use this feature."

In C++, if he went in and started turning on all the static analysis stuff, his compile times would go to the moon. Instead of saying that we wanted to do everything possible fast, why not just do what other compilers did and say, "You can turn that on, but you're hosing your compile times."

When he shipped something off to CI, he didn't care how long the build took. He was turning on everything, all the optimizations, because he could afford the time there. He asked if that made sense.

Walter said it did make sense. One potential issue that came to mind: he wasn't sure you could optionally turn things on and off to do all this inference, because it changed the semantics. He wasn't sure if that was correct, but he had carefully designed the DFA for `@live` so that it wasn't transitive. It only happened in the `@live` function. He did that so that you could then cherry-pick which functions you turned it on for.

Adam said this came back to `.di` files. He'd been a huge advocate of them for ImportC and thought they'd been pretty successful there. That was a feature we should keep. But if we could also use it for this so that we weren't doing the inference on every compile...

He was just spitballing, but there was a meta point here that our tooling sucked. That aside, could we say "turn this feature on and you'll get a bunch of .di files"? He thought Rikki was right in that you'd only do it once for packages.

He noted that C# and NuGet had this concept that you downloaded a package and it would just sit there. You didn't have to build the whole thing all the time because it was locked to a git commit. It wasn't going to change unless you changed it at build time. That might be something to consider.

Walter said that doing that required managing the `.di` files. One of the things he liked about D was you could just put the files on the command line and they would get built.

Adam thought that for this whole feature set, we were saying, "If you do this, you're doing advanced stuff, and you'll have to step in to intervene." Look at what the C++ guys had to deal with. If they came over and saw this, they'd say, "That's all I have to do? Yay!"

Átila asked why someone would have to intervene. Just make dub spit out `.di` files for the dependencies. Adam agreed.

Rikki's plan was always going to have a compiler switch to turn off DFA. He said that DIP 1000 was DFA, but it was a forward-only parse and straight from the AST into the hooks. That was very, very cheap. Unfortunately, that was how his mind worked. He couldn't go implement it. But after his work on a Semantic 4 version of it, he was confident this was the way to go and that it did handle all the transitive stuff. It was quite cheap.

He said we could turn the dial up just a little bit on the cost and get even better trade-offs in RAM, if not in time. It was cheap as a forward-only parse because you didn't have things like exponential loops.

Walter said DIP 1000 really did only one thing: it made sure pointers to the stack didn't escape. That was all. He asked what additional protections Rikki's idea offered.

Rikki said his DFA wasn't in the mangling. It was more of a lint. `scope` was part of the mangling, as was `@live`. It also extended the protection beyond the stack to any object.

Átila said that everything that wasn't on the stack lived forever. Rikki said that things like reference counted objects didn't live forever. You borrowed from one and then you had to deal with escape analysis to protect the owner and make sure it outlived the borrow. Then you didn't mutate the owner, so you killed off the borrow.

Walter said that was the same thing. If he created an object on the stack, he didn't want a reference to it escaping. That meant making sure no references lived longer than the owner. That was what DIP 1000 did. But its ultimate purpose was to prevent stack pointers from escaping.

DIP 1000 didn't do null tracking. It could be extended to do it. He asked if Rikki's type state analysis checked for null pointer dereferences. Rikki said that would be part of it. Walter said that would then be an advantage. That was the kind of thing he was asking about. What advantages did it have over DIP 1000? What was the ultimate goal? If it was adding null checking to DIP 1000, that was a benefit to the user.

Rikki said he could also do features like isolated because of the value tracking. Walter asked what that did for the user. Átila said isolated was good. For example, you could send something to another thread and that was fine because nothing could access it from your side. There was no other reference to it. It wasn't the only use case.

Martin took us back to attribute inference, saying it was unstable at the moment. They were having issues at Symmetry in the form of undefined symbols because inferred attributes depended on the order of semantic analysis. So if you passed `a.b` then `c.d` on the command line, it worked, but switch it around to `c.d` then `a.b` and you had undefined symbols. If we extended that to the whole program or even large subsets without explicit attributes, it wouldn't work because of these issues with recursion and cycles in the graph.

As for the build system using `.di` files for build-cache artifacts, it could work, but he stressed that it wouldn't work for templates. The `.di` files had to include all the templates, so nothing would be sped up there. All meaty D code was going to be templates.

Walter said Martin was right on both points. The reason inference had its problems was that it didn't do DFA. If it did, it could account for loops and recursions. The simplistic inference that was there did the best it could without dealing with those.

Rikki said that analysis didn't need to iterate to work with loops. Like with type state analysis, all you needed to know was the inherent type state.

Walter said the presence of `goto` in the language torpedoed any simplistic loop analysis of these things. You could always concoct a case with gotos that defeated it unless you did a proper DFA, which was what the intermediate code optimizer did. It used DFA equations and such. No matter how complicated the loop or what kind of goto rat nest you had in it, it would work successfully.

The problem with doing loop-based analysis was that it would only appear to work. There would always be someone who would write something to break it. Walter didn't want to do an analysis based on simple loop things because he knew he would get reports that it didn't work in certain cases. Trying to kluge fix it wouldn't work.

You had to have a general-purpose DFA, which was what `@live` did. No matter what kind of hell your flow graph was, it would figure it out. But doing proper DFA took memory and time. There was no shortcutting it.

At this point, I reminded everyone that this segment was supposed to be about Rikki's DIP updates. We still had more agenda items to get to. I asked Rikki if there was anything Walter and Átila could look at right now. He said he needed to finish up the version he was working on and get it into the Development forum. Then we could discuss it in more depth.

(__UPDATE__: Walter is willing to explore adding a DFA implementation if it doesn't have a big impact on compile times and memory consumption, but is skeptical it can be done. Rikki is working on an implementation to test against those constraints.)

### __module

Dennis said some users of ImportC had issues integrating C with D because of attributes, so we'd fixed it with extensions. There was still a major issue with ImportC when you had C files and D files with the same name. Try to import them from two different libraries, and you'd end up with conflicts. Try to put them in a folder to differentiate them, and the compiler would complain about the C file lacking a module declaration.

We already had a precedent to add `__import` to C files, so he thought it would be a good idea to add `__module`. He had a PR implementation that was about 20 lines of code. He asked if we had any thoughts on that.

Átila said, "No." Allowing D attributes in C code with extensions was a hack. This `__module` thing was a hack. We should stop doing hacks and fix it once and for all. We could do that by allowing the inclusion of C headers in D code. If we kept going down this road, there was going to be another D feature we needed in ImportC and a hack to add it, and then another, and so on. We should stop it now and just use D instead of approximating our dialect of C to D.

Walter said he agreed with Átila that kluge after kluge led to madness, but he was still very reluctant to embed C code in a D source file. Átila said you wouldn't be doing that. The C code was hidden in the header. He was saying, just include the header, parse it, and then meld the ASTs together. It was the only way to fix all these issues.

He'd run into the module issue Dennis had mentioned. He put things in packages because he liked organizing code well for multiple reasons. He'd tried ImportC on GNU lightning last year and the first thing that happened was the compiler said it couldn't find a module. And because it was C code, he didn't have any workarounds when the macros didn't work. No satic foreach, no mixins, no templates.

Steve said he'd encountered this, too. He'd tried organizing his C code, but what he'd ended up doing was to put it all in one folder and assume it was in the same package, which wasn't great.

He thought we would need to have a way at some point to import C headers rather than putting them in a C file and importing that. The problem was that C and D were completely different. In C, when you defined things more than one time, they just became one thing. But in D, if you had module A that wanted to import C's `stdio` and module B that wanted to import C's `stdio`, then you now had two different lists of symbols. You had two different versions of `stdout`. He didn't think it was a viable solution.

If there were a way to include C files and stuff them in a special package structure, then maybe we could get that to work. The `__module` thing seemed fine to him. He understood that these were hacks upon hacks, but he didn't know if that was such a bad thing given that ImportC itself was a hack.

Adam said Steve was exactly correct. He'd started writing a tool that would read all the different headers in a C file, then call ImportC on them with a module name so that he could get around the symbol clashes.

To Walter, Adam noted that when we used `.di` files, we were already admitting that we used header files. We just called them `.di` instead of `.h`.

Walter said the reason importing C headers didn't work with ImportC was because of a disagreement he'd had with Iain regarding how to handle C headers and D source files with the same root name and directory. It had broken one of Iain's builds. They hadn't reached a consensus on it. How could you disambiguate it?

Átila said the way to disambiguate was `include` vs. `import`. Mathias seconded that.

Rikki said we already had a solution here that was applied to D modules: just read the fully-qualified name out of the file name. No parse hacks were necessary. Walter and Átila said that was an interesting idea. Walter said he couldn't think of a fundamental problem with it at the moment.

Steve said that regarding import order, the problem wasn't that you had C files and D files living together, it was that you had import directories. Import directories for C had the same kind of package structure as the D files. Which one to pick? Currently, it picked the D files first, but it did that per import directory. So if you had a package structure underneath, it would go through the C and D files in this directory, and then the C and D files in this other directory. So in effect, the C files in one package could override the D files in the next package.

Walter said there was no solution for that. Steve said there was a solution: two different flags for importing C and D files. [He'd suggested it in a PR discussion](https://github.com/dlang/dmd/pull/14864#issuecomment-1430650577), but it never went anywhere. Walter thought that had already been implemented. Steve didn't think it had.

Átila said, "Or... include." Steve said that also worked.

Martin said that however it came about that we needed C files to include C headers, he thought it was very good. Ideally, you'd put all the headers you needed in a single C module. Then all of those and everything else they brought in, system headers or whatever, would only go to the preprocessor once. Then you'd end up with unique symbols, all in that one module. And if `__module` were implemented, you could customize it with that. Getting rid of duplicate symbols like that was really impactful on compile times.

He then went into some implementation details and there was some discussion about the C preprocessor and how DPP handled things.

To bring the discussion to an end, I said it looked like the consensus regarding Dennis's proposed `__module` declaration was a "no".

Átila reiterated that he didn't want to be in a situation where we implemented hack after hack just to bring D features to C. The first hack was to add attributes, but we already had attributes in D. We could have just done `@attribute: include c.h`. We already had module declarations. What was the next thing we already had that we were going to add?

Dennis said there were only a finite number of bridge features. It wasn't like we needed exceptions in C. He said that Walter had designed ImportC such that a C file was just a different way to write a D module. It used the same `import` because Walter had explicitly not wanted a separate include. If we were going to stick with that design and import C modules like D modules, then we needed a way to add module declarations to C files.

Átila said he understood, but it was more complicated than that. He'd used DPP and ImportC on actual C headers. There were always workarounds needed because some macros were untranslatable. With DPP, because it was a D file, he could do a static foreach or a mixin and the problem went away. He couldn't do that with ImportC. He offered to show Dennis what he'd had to do for GNU lightning. He said he'd had to keep repeating himself over and over and it was a pain. There were no tools to get around the limitations.

Dennis said he was curious to see it. I said that sounded like the next step. Átila said he'd been trying to convince Walter for a year now.

Walter said he still needed to see a piece of code. He knew Átila had sent him some long, complicated source files, but he didn't know what he was supposed to be looking at in long, complicated files like that. He needed to see something in a few lines. Átila said he had it in his calendar to "send Walter Python examples" on Monday.

The discussion went on a little while longer, going back to things brought up earlier, touching on implementation details, and so on. Finally, I asked Dennis if we could close this segment out.

Dennis said this was an issue that many people using ImportC ran into, and he'd seen a few saying they really hoped it got fixed. He didn't want us to be sitting on this for another year. He wasn't married to the `__module` solution or the `include` solution. He just wanted us to have some kind of solution.

Walter said writing a C wrapper file and changing the module name should work. Átila said it wouldn't because you didn't have packages. It would disambiguate the file names, but he didn't want to call them both `foo`. He wanted to call them `foo.bar.baz` because he wasn't a heathen who put everything in the top-level namespace.

Steve said to consider that you had two dub packages. Each of them wanted to import `stdio` from C. What would you name them? Walter said you'd name them `stdio`. Steve said you'd then have conflicts in the linker when you merged them because they both defined the same thing.

Walter said you'd have the same problem if you pulled in two things with the same name in C, and that didn't seem to be killing people. Átila said people used prefixes on all their names on purpose so that nothing ever clashed. Walter said, "Exactly!". When people had conflicts in C like that, they changed the names or used the preprocessor to change the names. He didn't see why this was a killer problem.

Martin said the problem here was just about duplicate structs. All the rest didn't matter. The way it should be handled for the package scenario Steve brought up was that each package should have its own version of the C world, just including the headers that they needed, including `stdio.h` in both cases and not exposing anything in the public API. That was very important.

Ali said that the way C++ got out of this multiple definition problem was with [the One Definition Rule](https://en.wikipedia.org/wiki/One_Definition_Rule). The onus was on the programmer.

Martin said the module system complicated that in D. And with C module names determined by the file name, if every project had its own thing called `my_c_world`, now you had to disambiguate the module name. That was quite common. They'd already run into this at Symmetry in using ImportC on a couple of projects. You needed to come up with a unique module name, and that was just embarrassing. We needed a solution for module declarations.

After a little more back and forth, we decided to table this discussion here given that we were coming up on the two-hour mark.

### Enhancement requests

Dennis said he wanted to let us know that there had been a few enhancement requests posted on GitHub rather than in the DIP Ideas forum. He suggested Walter take a look at them to decide which ones needed a DIP and which didn't.

* [https://github.com/dlang/dmd/issues/20624](https://github.com/dlang/dmd/issues/20624)
* [https://github.com/dlang/dmd/issues/20644](https://github.com/dlang/dmd/issues/20644)
* [https://github.com/dlang/dmd/issues/20645](https://github.com/dlang/dmd/issues/20645)
* [https://github.com/dlang/dmd/pull/20658](https://github.com/dlang/dmd/pull/20658)

Átila said he was a "no" on the shortened switch, but the others would probably need DIPs. Walter said he would take a look at them.

## Conclusion

We held our next meeting on February 7th, 2025.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
