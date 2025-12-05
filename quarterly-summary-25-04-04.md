[Forum Post](https://forum.dlang.org/post/ljcazyvqlmvnonmhherh@forum.dlang.org)

The D Language Foundation's quarterly meeting for April 2025 took place on Friday the 4th at 15:00 UTC. It lasted about an hour and twenty minutes.

Our quarterly meetings are where representatives from businesses big and small can bring us their most pressing D issues, status reports on their use of D, and so on.

## The Attendees

The following people attended the meeting:

* Mathis Beer (Funkwerk)
* Walter Bright (DLF)
* Luís Ferreira (Weka)
* Martin Kinkelin (Symmetry)
* Dennis Korpel (DLF/SARC)
* Mathias Lang (DLF/Symmetry)
* Átila Neves (DLF/Symmetry)
* Mike Parker (DLF)
* Carsten Rasmussen (Decard)
* Robert Schadek (DLF/Symmetry)
* Bastiaan Veelo (SARC)
* Ilya Yanok (Weka)

## The Summary

### Ilya
Ilya said his focus at Weka was on improving the developer experience. To that end, he'd realized they were spending a lot of compiler cycles compiling code that was then thrown away when linking. [He'd posted a DIP idea](https://forum.dlang.org/post/veqgighfeexaxrgpwnuj@forum.dlang.org) for an annotation or pragma to mark functions intended only for use at compile time.

Walter asked if his concern was long compile times, which Ilya affirmed. Walter then asked if they were using separate compilation. Ilya said that they were compiling by package during development but compiled everything all at once for release builds. They were aware that part of the problem was redoing static semantic analysis on templates, and compiling everything together helped with that. But they had found through benchmarking that a lot of time was spent in the back end. Luís noted this was specifically regarding LDC and not DMD.

Walter asked why not then use DMD for development and LDC for production. A lot of people did that. Luís said they needed the ability to define custom sections, which LDC allowed and DMD didn't. They also had a tracing system that heavily used LDC stuff. DMD would be an option if that were implemented in the its back end. But for a release build they took something like an hour.

I asked Ilya if he wanted to talk about the proposal. He said the idea was super simple: attach an attribute to a function or function template. A stupid implementation would just skip code generation for that function. He'd tried that out and it wasn't so nice, as it ended up leading to cryptic linking errors. So he'd gone further and implemented a simple static analysis on top of it, such that marked functions could only be called during CTFE or from other marked functions.

There were some complications in making something like `map` work with such functions. With a straightforward implementation, it wouldn't be allowed. So you'd need some sort of propagation through the templates to make it work with templated functions.

In the forum thread, there'd been discussion about just using `assert(__ctfe)`. He'd tried to explain many times that it wasn't possible because `__ctfe` assert was evaluated at run time and this was a compile-time thing. He said that, in a sense, `assert` could be infinitely more precise, and it could happen that at run time it never triggered. But at compile time, we had to approximate. We couldn't just simulate execution.

Átila said he preferred an `in` contract instead of an assert because you could see that at compile time. It was kind of part of the function signature. 

Ilya agreed that was better in that, on the technical side, it was easier to get the contract than the assert because you didn't have to go into the body. But from its semantic meaning, contracts were still evaluated at run time. There were some examples in the discussion thread, like taking the address of a function with such a contract and putting it in an array, and then taking that value back from the array and executing it. You could do that inside CTFE, so it would work, but it would be super hard for the compiler to see that it could actually skip code gen for it.

Átila agreed, but said that `in(__ctfe)` was a special case. Ilya said you could special case it, but then you'd have a special kind of semantics for that specific contract. Átila argued it was only that one.  `__ctfe` was already special anyway. Special cases in general were bad, but this one might not be terrible.

Dennis noted it would be a breaking change. Ilya agreed it was technically a breaking change, though he thought people never used that kind of contract, so it wouldn't actually break anything. Dennis had an example in the thread of a valid program that would break with the change. It would be rare, but it could happen and was something to keep in mind.

I said that would mean we'd have to put it behind an edition, but wondered if we'd really need to do that with an attribute? Átila said if there was a chance we'd break things at compile time, then yes, it should be behind an edition. He proposed we look through GitHub to see how often that kind of code turned up.

Luís reminded us that the spec said contracts were removed in release builds. This would have to be an exception like `assert(0)`. Átila said if this idea went forward, that's what would happen.

Luís asked if it would be a special case for a contract with `__ctfe` or a contract that at compile time would get a zero folded variable. Átila thought it should just be `__ctfe` for now. He didn't think it was weird to special case it, because CTFE was so fundamental to the language and that variable was already special. Luís said he understood.

Átila said that, should this go forward, we should do the least amount possible to make it work. Luís was worried about breakage and would need to see examples to form an opinion on which approach was best.

Martin said that, out of curiosity, he'd just done a search for `in(__ctfe)` across all of GitHub and turned up four instances, all of them in 
Átila's code in the tardy project and Symmetry's autowrap. He'd expected to see none at all.

I asked what the next step should be. Should Ilya move forward with the DIP or not? I thought we should resolve that in this meeting. Did it have a chance to go all the way with an attribute, or was it preferable to take the contract approach?

Átila said he wouldn't reject it outright.

Walter said that `assert(!__ctfe)` had been brought up in the forum thread, where people had said it required semantic analysis. He said it didn't require semantic analysis. The compiler could just look at the first declaration in the function and see if it matched `assert(!__ctfe)`. No language changes were required.

Átila said the contract was even easier. It didn't need to look at the function body. Walter said he wasn't sure how it interacted with contracts, but he remembered discussing the "no runtime thing".

Martin said if we were just talking about skipping code gen for some functions annotated with whatever syntax, their bodies needed to be analyzed anyway. If they were code gen candidates, they needed to be semantically analyzed in Sema3. It was later that code gen would be skipped. So it didn't matter if this feature was part of the body. The body would be analyzed first anyway. Suppressing code gen for an already analyzed function was not a blocker for having it in the body. He thought it would be a bit more elegant as a contract, but he didn't have a preference.

Átila thought it was good to have either way. At this point, we were just bikeshedding.

Ilya said the main problem with the `assert` approach wasn't about looking into the function bodies. The main problem was that existing functions that had this assert as an `in` contract. Skipping code gen for those would be a breaking change. Ditto for `in(__ctfe)`. He and Dennis had posted examples. Átila said that was a good point.

Martin had just seen those examples for the first time, and wanted to clarify that the issue here was that code that compiled before could have linker errors with the change. Ilya confirmed.

Átila reiterated that this was a good idea, however it was implemented. We shouldn't be wasting time generating functions that were never called at run time.

Dennis asked if the backend could just see that a function started with `assert(0)` and recognize that it didn't need to be analyzed. Walter said yes. Martin said if you were using it during CTFE, you'd need the analysis.

He wondered about the concern over breaking changes. It was easy to come up with these examples, but was this really a thing in real-world code? Was it one person or a thousand projects? We could just put it behind an edition.

He brought up a problematic case of taking the address of a function with `assert(__ctfe)` in it and storing it somewhere to call later. Ilya said another problematic case was calling such a function in a branch. Walter said the compiler, upon seeing the `if(__ctfe)` or something like it, wouldn't generate the code for the false branch. Ilya said it wouldn't always be that simple. It could be any condition.

Walter said the compiler was pretty good at eliminating code that was clearly never going to be executed. You could have a complex conditional and it would not generate code for the CTFE-only branch. Ilya said that was true, some dead code elimination would kick in and delete some code, but that was already happening in the back end. After you spent some cycles in the glue code and then some other back-end code... he would like to avoid that.

Another thing was that it could not delete the unused code in 100% of cases. It could be super complicated and potentially depend on something at run time. There would always be a situation where a program could get an input that would never trigger a function to be executed outside of CTFE, but the compiler wouldn't be able to see it.

Walter said if you put an `assert(__ctfe)` in there, no matter where you put it or how complicated it was, the compiler was going to delete the code that followed it during code gen. If anyone could find a case where that wasn't true, he'd love to see it.

Mathis Beer said if it was really just going to be putting `assert(__ctfe)` at the start of a function, then maybe replace it with a stub function. He was unaware of any case offhand where that shouldn't work.

Martin said we wouldn't need to change anything at all, based on what Walter had just said. So if there was an `assert(__ctfe)` at the start of the function, the whole function body would be optimized out to an `assert(0)` at run time.

Walter suggested trying it out: write some functions, put `assert(__ctfe)` at the start, and check the generated code. Everything that was guarded with the assert should disappear. 

Luís said the symbol would still be generated. Dennis noted that the issue was about compile times in LDC's back end.

Walter said the compiler would figure it out early on. It does trivial optimizations, and one of those was pruning out everything preceded by `if(0)`. It just got deleted. Átila asked if that was in the DMD backend. Walter said it was. Átila replied that LLVM probably didn't do that, hence the issue.

Mathis said that LLVM did that, but there was a bunch of stuff that happened before that point where it had to generate the intermediate representation before it noticed that kind of thing. 

Walter said DMD would generate the intermediate code, but then it would delete it.

Átila thought the contract still sounded like the easiest thing since it didn't require a language change. And since his code was the only thing that turned up on GitHub already using it, there would be almost no breakage. Mathis warned about underestimating the amount of private code out there that might break. Even then, Átila thought the amount of private code using it would still be small given how it didn't show up on GitHub.

I asked Ilya if he was okay with the contract approach. He said he was. The purist in him said contracts should be run time and attributes compile time, and so this should be an attribute. But from an implementation perspective, there was no difference. 

Átila noted that contracts were available at compile time anyway.

Luís thought as long as it worked for Weka's use case, then it would be fine.

I told Ilya he was free to post an updated DIP in the Development forum at any time.

### Luís
Luís pointed us at [a GC bug Ilya had reported](https://github.com/dlang/dmd/issues/20917). They used a lot of `__traits(compiles)` in their code and had found that `@nogc` analysis wasn't doing well with it. He also brought up [an older issue](https://github.com/dlang/dmd/issues/17584) that he thought might be a trivial fix. There were other issues he needed to track down and would bring to us later.

__UPDATE__: The first issue has since been fixed.

### Interlude

There was a brief side discussion prompted by Maritin asking if Weka had done any testing with LDC's "link once template switch".  Weka were still on an older version of LDC, but were slowly upgrading their code base to work with newer versions.

Before moving on, I announced that we should thank Weka for their support. They had approved funding for DConf and a new paid staffer. Nicholas Wilson would fill the Pull Request & Issue Manager role that Razvan vacated when we made him Project Coordinator, and I would soon announce it publicly.

### Bastiaan

Bastiaan reminded us that SARC had faced difficulties getting the 32-bit performance of their D port on par with their old code, and they'd been unable to use LDC on 32-bit Windows. They'd intended to move it all over to 64-bit upon finishing the D port.

They'd encountered a numerical bug at one point in DMD-optimized code. That had since been fixed, but because they'd been under quite a bit of stress to get the port over the finish line, they decided they couldn't spend too much time validating the numerical results in 32-bits, so they decided to go ahead and move to 64-bits in parallel with porting to D.

The 64-bit port was complete and they were now using 64-bit LDC. They were testing it alongside the 32-bit code. They wanted to get rid of the 32-bit stuff completely at some point, but some systems were still running 32-bit Windows. They were using non-optimized builds for that.

That process had gone quicker than he'd expected. Unfortunately, he was seeing some heavier use of the heap in D compered to the original Pascal. That was causing some problems in the 32-bit builds sometimes.

Walter asked why they were still using 32-bit systems. Bastiaan said there were some very old systems onboard some ships that still used a 32-bit OS.

Walter said he wasn't working on 32-bit code gen anymore. Bastiaan understood that. The problem was just that their code wasn't 64-bit compatible because they were doing a lot of tricks with pointer to integer conversions. It was really messy old Pascal code that couldn't do pointer arithmetic. So they had to covert to integers for that. Pascal didn't know anything about `size_t`, so they had to introduce that and change it in the right spots. The problem wasn't about porting D from 32- to 64-bits, it was about interfacing with the parts of their
code that were still in Pascal.

He said one interesting case was that Pascal worked with a stack-based string. You used a very large string on the stack, did your manipulation, and it was gone on function exit. In the D port, that became a heap allocation. He was looking to eliminate some of that and was surprised to learn that `Appender` was incapable of using a stack array.

Luckily, he found a package that Robert maintained that did what he wanted, but he was still surprised. He was seeing a factor of about 1.5 increase in memory usage in 32-bits. He was trying to hunt that down. He believed there was some fragmentation going on, too.

He said they were all in on the testing phase now, so they were getting closer and closer to the end.

I said it had been a long journey and I was happy to hear it was almost over. Bastiaan said everyone was looking forward to it. One of his colleagues was already using D for a new module, and they had some other small modules up as well. He said Dennis was working on the base of an output feature based on a tree. That was also in D.

Martin asked if the Pascal code was using a GC. He was wondering if the memory usage was a GC vs. GC comparison. Bastiaan said no. He wondered if the D GC was keeping pointers alive for too long.

Martin said it could. When it decided there was no need to run, then it wouldn't. They'd had problems like that in the past at Symmetry. The new GC that was coming was going to make things better in every way. They'd seen a reduction of about 60% or so in the average case, which was quite good. He asked if Bastiaan had experimented with the precise GC on 32-bit DMD to prevent false pointers from keeping stuff alive.

Bastiaan said yes, they were using the precise GC. 

I couldn't recall if the new GC supported 32-bit. Átila said it did not. It was hiding information in the the extra bits afforded by 64-bit.

Luís thought precise was also not a thing under the new GC. Bastiaan thought that was really only useful for 32-bits. Martin said it was also a speed issue and talked a bit about the implementation.

### Mathis Beer

Mathis said the only thing new he had to report was that Funkwerk had tried the new GC. He was so hyped about it. He genuinely believed that if D had had that GC 20 years ago, it would be in a different place today. It was so good. He expected it was going to cut their memory usage in half across the board.

The main reason was that they were heavily pushing toward having a mutation-free design with immutable data structures and algorithms. That put an unavoidable load on the GC that they were always trying to manage. The'd have to manage it a lot less with the new GC, and that was awesome.

Luís was very interested in checking it out, but being on a custom runtime and an older LDC version might make it difficult. Mathis said it would be very hard not to notice an improvement with it. And it was being actively ported to DMD. 

### Carsten

Carsten said Decard were still working on their network. They were transpiling WASM into BetterC and it worked okay. They were testing it at the moment, but it was working fine for them.

### Dennis

Dennis had one minor issue at SARC to tell us about. They were supporting both 32- and 64-bit and were using a binary serialization package. The issue was that when you had a struct with `size_t`, it would be incompatible. The 32-bit version saw a `uint` and the 64-bit version saw a `ulong`. So on 32-bit, they were serializing it as 64-bits.

He wondered if anyone had any other ideas for how to deal with `size_t` and introspection.

Mathias Lang suggested not using `size_t`. Dennis said that would then require casts when indexing an array with `ulong` or converting an array size to `uint`. The code would be littered with casts every time an array index was used or stored.

Mathias suggested if it was a network protocol, they'd probably need a fixed size anyway, so `uint` or `ulong`. Dennis said it wasn't a network protocol. It was just serializing to disk and reading it back.

Carsten said he'd had a similar problem trying to compile to a 16-bit AVR. Doing a `foreach` on an array, things like that, would break when you tried to compile it. You couldn't use arrays in BetterC on 16-bit systems. He thought it was a fixable problem, but he didn't look into it.

Dennis said D didn't officially support anything less than 32-bits, so Carsten would have to work around it some other way.  Carsten said he accepted that, but everything else actually worked in BetterC as long as you didn't use arrays. But that was the loss of a big advantage over C.

Walter said it would be nice if Carsten could write something up about how to use 16-bit DMD. That would be fun. Carsten said he'd think about writing up what he'd done trying to compile for the Arduino.

I asked if anyone had any other advice for Dennis. Walter suggested doing something like this:

```d
struct size_t2 { size_t s; alias this = s; }
```

Dennis said it wasn't very usable. You couldn't assign an integer to it:

```d
size_t2 i = 0;
Error: cannot implicitly convert expression `0` of type `int` to `size_t2`
```

Átila suggested adding a constructor, then it could be `i = size_t2(0)`.

Dennis said the whole point was to avoid making things difficult for their inexperienced D colleagues. The programmers at SARC were used to Pascal. If they had to know all of these `alias this` quirks, then he'd rather just not use it.

Walter didn't think there was a magic solution to this. Dennis said that was why their current solution was just to add an attribute to `size_t` fields.

### Martin

Martin said they had tested 2.111 at Symmetry. One blocker that they'd hit for one of their main projects was a compiler bug related to semantic analysis order that may have been there for ages. Iain hadn't been able to confirm if it was a regression. So probably very old compiler versions would also fail to compile it. He'd [reported an issue for it](https://github.com/dlang/dmd/issues/21137).

Martin had run Dustmite on it and fixed the test. Unfortunately, the bug appeared in real world code and that wasn't fixed. So he'd had to do another Dustmite reduction.

Nothing had changed in their code. It appeared to be related to a change in Phobos to make `Appender` more efficient that ended up uncovering an existing compiler bug with semantic analysis ordering.

They'd had multiple problems where switching the order of modules on the command line broke things. He thought one of the Weka issues Luís reported, which was all about inconsistent attribute inference, might be related in some way. 

You were only going to see issues like this on bigger projects. Normal people with smaller projects would probably never see this. But as a hardcore D programmer with a large code base, this was the kind of bug you didn't want to see. They were difficult to track down. He'd probably have to hack the compiler with some print statements just to figure out the different semantic analysis ordering, what to change or what the correct order was for it to work and how he could get there. Right now he had no idea.

That seemed to be the only problem so far.

I noted that this kind of issue with semantic analysis order had been with us a long time. This wasn't the first time it had come up. I asked how we could go about fixing it once and for all. Was there a project we could set someone up to investigate? Contract work perhaps? Did we need to have Walter spend a month digging into it?

Martin said he honestly didn't know. It could be a tiny little trivial fix, like a semantic call missing somewhere, or a little check missing somewhere. He thought what would really help would be some kind of logging framework where we could figure out the semantic passes, how they were done and in which order they were done. Something like LDC had for code gen, which was very helpful in some cases when solving LDC code gen problems. It was easily thousands and thousands of lines with all the information you needed.

Maybe LDC's `-ftime-trace` would be enough if he set the granularity to zero to ensure that everything was printed. But for hard bugs like this, having some form of extra output to track the problem down should be the first step. Ideally, it shouldn't hurt the runtime performance of a normal DMD build where you weren't interested in those features. And ideally you wouldn't need a separate compiler build just for that. You'd opt in and pay the runtime price just for that extra output.

Walter said that the general problem with attribute inference was that if the graph had cycles in it, then it depended on which cycle was done first. It shouldn't be a problem as long as it was a tree structure, but as soon as you started having cycles, we didn't really have a general solution. 

Generally in attribute inference and data flow analysis, that problem was solved using data flow equations. But the compiler didn't do any data flow equations for inference. Trying to solve it without some sort of data flow equations was going to be like stepping on a ripple in a carpet. All it did was pop up somewhere else. So until someone came up with an algorithm for dealing with cycles in the flow graph, he didn't think we could solve this problem.

It was the same problem with forward references. For example, take a struct that has a `static if` that adds another field. If the `static if` was conditioned on the size of the struct, then it was compiling in a field that changed the size of the struct and you ended up with this insoluble chicken or egg problem. He thought the problem with `static if` was never resolved, and the inference engine there had the same problem.

What you had to do was to sit down with a pencil and a piece of paper and ask, how can I figure this out with just some simple examples of inference? He thought Dennis had done some work on that earlier with the attribute inference. Maybe we had the defaults set up wrong if there was a cycle in it.

Mathias didn't think it was a general solution, but it seemed at the moment that attribute inference was pessimistic rather than optimistic. So when we couldn't infer something was safe, it was by definition unsafe. If we could take the opposite approach where we opted out of safety and assumed that safe by default, he thought that would solve some of the problems we were having.

That was the approach he wanted to use with recursion, because we also had this problem with auto functions. When they recursed, you didn't have any attributes on them.

Regardless, Walter didn't think the problem could be solved by printing out thousands of lines of which order it tried to infer things.

Martin said the problem in his test case wasn't about inference. He'd only brought it up because that was the issue Luís had mentioned, and he thought it might be related. In fixing the initial issue in three test cases, all three were happy. But the runtime code wasn't. He kept getting errors about `@safe` inference. 

He said the whole semantic thing was so brittle. He didn't think in Symmetry's specific case it was about data flow analysis, but something to do with semantic analysis order. You could compile `b.d`, but you couldn't compile a module that imported that module. That was just ridiculous. We needed to be able to fix this.

Walter asked some questions about the details of the issue, which Martin answered.

Dennis said he'd tackled this before, and again recently with the 2.111.0 release. And every time he got into the fundamental problem that DMD did semantic analysis by recursively calling functions that made specific assumptions, such as that you started analyzing at the end of the function, you finished that symbol, and it couldn't be interrupted. If you recursively entered yourself again, it was cyclic and it would give up.

He'd heard that Timon used some kind of scheduler in his front end that allowed you to interrupt semantic analysis, continue somewhere else, and then come back. That was a fundamental architecture thing. Given the size of DMD, changing the way semantic analysis was initiated and processed would be a huge undertaking. It would require some serious design work and restarting from the ground up. That was why it was so hard. He'd been looking for an easier way out but had come up with nothing so far.

Martin fully agreed. He recalled that Amaury had given a lightning talk about his SDC, Snazzy D Compiler, at a past DConf, and had mentioned that he was also using some kind of fiber structure for semantic analysis. Whenever he reached a point where he couldn't progress any further, he just yielded and let another thing be analyzed so that at some point he could hopefully get to a place where everything could be analyzed, or at least to a more deterministic state than we currently had.

### Walter

Walter said that, as he'd announced in the forums, the first major test file for AArch64 actually compiled and ran now. He said he must have fixed 100 bugs in getting that test file to compile. He was pretty happy about it. It was good progress.

The ARM code generator was making progress, though it was frustratingly slow. He'd had a lot of problems like generating code with the operands reversed and things like that. He was knocking these things down one by one and it had started working. It was still a long way from being a functional compiler, but it was working.

Martin congratulated him. Upon seeing Walter's announcement, he'd updated his PR for doing the first ARM64 experiments via GitHub actions on the macOS ARM64 runners. It didn't work there for now. He was hoping that just doing a "hello world" with the `puts` function from C would work, but it didn't. It would be great if Walter could get it working on macOS and not just on Linux.

Walter said he'd have to buy a Mac to test it, but the Pi was a nice little machine. It was easy to use. What he was doing right now was the exact same thing he'd have to do get the Mac version to work, so he wasn't worried about it yet.

He said he'd love to get the ARM code generation added to the tester. Martin said the reason he'd chose the macOS ARM64 test runners for the first experiments was that macOS came with a built-in x86 emulator. So we could generate a native x86 DMD and have it produce ARM64 code, then run that code natively on that machine. This was something we couldn't do on Linux.

Walter said that was a very good point he hadn't thought of. It would make development faster if he could do it all on one machine. Right now he was compiling it on Linux then ssh'ing over to the ARM to link and run it.

(__UPDATE__: He has since picked up a new Mac Mini.)

### Conclusion

Our next meeting was a monthly meeting that took place the following Friday, April 11th. The next quarterly meeting was on July 4th.

If you are running or working for a business using D, large or small, and would like to join our quarterly meetings periodically or regularly to share your problems or experiences, please let me know.