The D Language Foundation’s quarterly meeting for April 2026 took place on Friday the 3rd. It lasted approximately fifty minutes.

Our quarterly meetings are where representatives from businesses big and small can bring us their most pressing D issues, status reports on their use of D, and so on.

## The Attendees

The following people attended the meeting:

* Walter Bright (DLF)
* Martin Kinkelin (LDC/Symmetry)
* Dennis Korpel (DLF/SARC)
* Mathias Lang (DLF/Symmetry)
* Átila Neves (DLF/Symmetry)
* Razvan Nitu (DLF)
* Mike Parker (DLF)
* Guillaume Piolat (Auburn Sounds)
* Bastiaan Veelo (SARC)
* Nicholas Wilson (DLF)

## The Summary

### Guillaume

Guillaume said he had been making a SIMD library for a few years. It was used primarily by people developing consumer software.

There was an issue with either LDC or DUB related to the function ABI. It was an esoteric issue where one function built in one translation unit with AVX enabled could be called from another function that had not been compiled with AVX support, then the register would be missing data. Two different users had reported the problem to him.

He believed the issue was at the intersection of LDC and DUB. It made using the latest AVX and AVX2 instruction sets in D more troublesome than it should have been.

Nicholas asked Guillaume for a reproduction of the build script so that he could determine whether the problem was in DUB or LDC. Guillaume had already [linked to a GitHub issue containing a reproduction](https://github.com/dlang/dub/issues/3080). Nicholas said he would take a look at it soon.

Other than that issue, Guillaume was very happy with D. He was using LDC 1.41 and had encountered no problems on any of the three desktop operating systems. He hadn't yet moved to LDC 1.42 because upgrading required a bit of validation.

He had also been doing some ARM64 benchmarking. In some cases, integer operations performed better than floating-point operations, which made fixed-point arithmetic more interesting.

(__UPDATE__: [A fix was submitted to DUB the next day](https://github.com/dlang/dub/pull/3111). Nicholas merged it a couple of days later.)

### Dennis

Dennis had no new work problems to report.

I asked him how SARC’s D port was performing. He said he was still working on making the switch. The code he was writing was only for the D version, but the Pascal version still had to be maintained. It was annoying because the testing was really slow.

### Mathias

Mathias said he hadn't been doing much programming lately.

### Átila

Átila said he didn't have much to report. He at least got to so some coding, but it was mostly in Python these days. He was working in D once a week.

He was going to talk about some of the stuff he'd been doing [at the D Symposium](https://dlangsymposium.com/), and that was agentic coding. It had helped a lot with DRuntime. He still had some PRs that were red in CI that he needed to get back to. He was also trying to review building DRuntime with `-preview=nosharedaccess`.

### Razvan

Razvan had nothing to report.

### Walter

Walter had moved from cross-compiling his AArch64 implementation to running natively on a Mac. He'd gotten the runtime built and was now attempting to link a single program. The linker was complaining because he was giving it the wrong arguments, so he had to figure out the correct command line for the linker.

He was making progress inch by inch. There were a thousand small things to deal with, and he had to juggle multiple compilers to get things compiling on the Mac. There was the last working DMD release on the Mac, the current one he was building, and the AArch64 build of the compiler. He managed to regularly confuse himself about which compiler he was running and which platform he was targeting. He was slowly getting there.

He had also completed his presentation for the symposium. His computer had crashed and caused him to lose half of it, but he had been able to rebuild it from from his notes. He'd decided to be more bombastic about D. A long time ago, some of the C++ people had gotten mad at him for being critical of C++, so he had toned that way down. Now that seemed like the wrong thing to have done. He was toning things up now. He didn't care if people complained that he was bashing C++.

The presentation was called 'Elegant D'. It had a lot of examples in C, C++, and D to demonstrate how things worked in D and how much easier it was to write code in D. He had considered including Zig and Rust, but since he had never written a program in either language, he thought he would likely embarrass himself by making inaccurate statements. He'd decided to stick to criticizing C and C++.

I asked if he'd thought about his DConf presentation. He was considering either a revised version of 'Elegant D', a talk about the AArch64 code generator, or more of an overview about why D was a better language to program in.

Átila said that if it made Walter feel better, he had bashed C++ while speaking at CppCon once. At one point, one of the attendees had accused him of being on a covert mission to convert everyone to D. Átila had replied that was being very overt.

Walter said one of the things in his presentation was using `= void` to avoid default initialization. Someone had sent him a link to a C++ proposal to do the same thing. They had come up with every ugly, complicated way under the sun to do it. And they did it by inventing weird new keywords and stuff. His response was to ask why they didn't just copy D. Somebody had said you shouldn't have `= void` because `void` didn't mean "do not default initialize". Walter had argued that of course it did. The word "void" meant "empty". It meant nothing was there. It was the perfect keyword for it and it was already there.

He had also been looking at how C++ did CTFE and was confused. He had read the descriptions of the keywords `constexpr` and `consteval` and wondered how they managed to do it wrong every time. Why did they have two keywords to do it? C++ was a powerful language, but it was utterly lacking in taste. They took ideas from D and turned them into ugly things.

Átila said the C++ contracts proposal explicitly did not support custom error messages in contract assertions, while leaving open the possibility of supporting them later. In the meantime, `contract_assert(predicate && "error message") would work and the compiler would print the error message out when the assertion failed. Of course, string decayed to a pointer, so you could do that because the short circuit worked.

### Bastiaan

Bastiaan didn't have much to report, but he thought he would need to try profiling SARC's code again. Whenever he had attempted to use profiling in the past, the executable had crashed, possibly because of stack space exhaustion. Since they had moved to 64-bit in the meantime, he hoped that might work better.

Dennis asked if he was talking about the `-profile` switch, and Bastiaan said yes. For 64-bit builds, they were using LDC.

Walter asked why they would use anything other than 64-bit. Bastiaan said they had been 32-bit until now, and he thought it was because of old hardware on ships. Now the idea was to fully optimize the 64-bit build and not optimize the 32-bit build. In practice, he thought they could probably do away with 32-bit, but he wasn't completely sure.

One reason they still used DMD in development was its faster compilation speed. They hadn't been using 64-bit DMD because their use of `alloca` was incompatible with exception handling on 64-bit Windows. He noted that we'd discussed this previously, and Walter had said the exception handling mechanism on 64-bit Windows was undocumented and that he'd given up trying to understand it.

Walter said DMD on 64-bit Windows used its own exception handling mechanism, but he had no idea how it would be affected by `alloca`. Dennis posted a link to [the exact line of code in the backend](https://github.com/dlang/dmd/blob/2298fced3c39bfbea4e02c0bfc99812975de2362/compiler/src/dmd/backend/eh.d#L50) that generated the error message "cannot mix alloca and exception handling". Walter said he would look at it.

Bastiaan supposed it would be a lot of trouble to get that to work, but they were good with using LDC. That said, DMD’s quick compilation was nice to have during development, and that was when they used 32 bits.

He said they used `alloca` to allocate variable-length arrays on the stack and avoid the global lock in the garbage collector. He supposed poor multithreaded performance might not matter much during development, but then they'd have to use conditional compilation or some other things. He wondered whether the new garbage collector might eventually make all of this obsolete.

### Martin

Martin said he didn't have much to report. It was just more of the same.

They'd recently added support for LLVM 22 to LDC. The next step was to merge the stable DMD front end to bump to DMD 2.113. He hoped it would go smoothly since the last release wasn't too long ago.

At Symmetry, they were using LDC 1.42 with the latest changes and the new garbage collector enabled by default. They had encountered no significant problems.

They had also seen an average run-time improvement of approximately five percent simply by moving from LDC 1.41 to 1.42, without changing their code. They weren't completely certain where the improvement came from.

At first, he thought it could be due to a small off-by-one error that had been fixed in the new garbage collector. That error could cause extra allocations. But then he recalled that they had seen the performance improvement before the fix. It really seemed to be something in the compiler.

He was thinking it might have been something to do with the associative arrays being templates now. Not having to go through `TypeInfo` and all the indirections could give the optimizer more to work with. He was hoping other people would see some improvements and chime in to confirm their numbers. But he thought that was really nice.

Another good thing was the new built-in `__traits(needsDestruction)`. It allowed LDC and the runtime to eliminate extremely ugly workarounds that had been necessary because of semantic analysis ordering and forward reference problems. The compiler could now perform the additional aggregate semantic analysis for structs and classes to check if there was a compiler-generated destructor and/or a user-defined one.

There was a comment in the runtime saying to replace all of the old `hasElaborateDestructor` logic with the new trait. He'd done a quick grep and it was terrible. Most of those checks just did it for structs and ignored static arrays of structs. We desperately needed a `destruct` function, a little helper somewhere, which would take care of stuff like this. Give it a reference to some object and it would handle destruction. For a struct, it would call the destructor. For a static array, it would go through the array and destroy all of them, handling any exceptions when destroying one of them. It was just a matter of handling old things that had never been fixed properly. There was always work to do, but we all knew that.

Other than that, he had seen contributions to LDC from people interested in using ImportC on exotic platforms. One contributor had fixed ImportC built-ins after testing three C libraries on Android. By adding a small number of missing built-ins, he'd been able to interoperate with those three libraries on Android.

Another contributor was using either ImportC or WebAssembly, he wasn't certain which. He just thought it was interesting that people were using this stuff on more complicated platforms where you could only cross-compile. There was no native compilation for WebAssembly, obviously, on Android. There could be, but there were some weird issues that prevented providing LDC binaries. They'd put some effort into investigating it recently, but had to give up.

He was pleased that the DMD release process now was in a proper state. He didn't know what had been done, but it was nice that the master and stable branches were being handled better.

### Me

I gave on update on some discussions I'd had over the past couple of months about funding and about an informal proposal I'd received to create an industry workgroup. My sense was that this would go beyond what we were doing in the quarterly meetings and be more focused on actual development, and not only the problems that the industry people were having. It was just an idea at this stage.

I reminded everyone that I was looking forward to receiving their DConf talk submissions and would be watching my inbox. Also, June would arrive quickly, and that was when I needed to announce the next Symmetry Autumn of Code. We needed to begin thinking about potential projects.

### Addendum

As we were getting ready to wrap, Walter talked about how he'd been improving some of his old D-Man doodles using AI to make them look more professional. He'd begun replacing the ones he'd uploaded to the website.

During this discussion, I asked if he'd replaced the image used for 404s. It's the "assembly required" image used on [the Inline Assembler page](https://dlang.org/spec/iasm.html) of the documentation. He said he had. That prompted Martin to ask about Walter's plans for inline assembly on AArch64.

Martin wanted to know whether Walter intended to create something similar to DMD’s x86 inline-assembly syntax or use the GCC-compatible syntax supported by GDC and LDC. He said the GCC syntax was architecture-independent at the language level. It used strings for the instructions and required the author to declare the inputs, outputs, and clobbered registers explicitly. That made it easier to adapt code through string mixins when register names or calling conventions were different across platforms. It could also allow functions containing inline assembly to be inlined.

DMD’s current x86 inline assembler understood the instructions itself, including which registers they read or modified. One consequence was that the compiler had to disable some optimizations for functions containing inline assembly. He said that if Walter created a separate AArch64 assembly language in the same style, it would probably remain DMD-specific. Martin did not intend to implement another architecture-specific assembly parser in LDC.

Walter said his plan was to follow the same approach as DMD’s Intel inline assembler. The Intel implementation matched the syntax used in Intel’s documentation and knew from its tables which registers each instruction read and modified. The AArch64 implementation would likewise use the syntax from the AArch64 specification and track register use automatically.

He had nothing but bad things to say about the GCC approach. The programmer had to specify the inputs, outputs, and affected registers manually. He understood why it was portable across architectures, but he didn't like using it. He preferred an assembler where an instruction from the processor manual could be typed exactly as documented. His AArch64 disassembler already worked that way, and the inline assembler would match the specification as well.

He said the AArch64 inline assembler was vastly simpler than DMD’s x86 implementation. He hadn't written the original x86 version, which was clever but nightmarish code that he wouldn't inflict on anyone. Its main advantage was that it worked.

Martin understood the reasoning but noted that users who wanted their inline assembly to work across DMD, LDC, and GDC would face limitations. Much of the inline assembly in DRuntime and Phobos existed to compensate for DMD code-generation performance, so perhaps AArch64 would need less of it. The remaining low-level runtime functions would still require a small amount, particularly for bare-metal use.

Walter noted that inline assembly use had declined dramatically over the past decade. There were only about five or six instances in the runtime. Since most users would need only a few instructions, he didn't consider portability of the syntax a high priority.

## Conclusion

We delayed our April monthly meeting by a week to April 17th since several folks were heading to Mike Shah's D Symposium. Our next quarterly meeting was set for the first Friday in July.

If you're running or working for a business using D, large or small, and would like to join our quarterly meetings periodically or regularly to share your problems or experiences, please let me know.
