The D Language Foundation's April 2025 monthly meeting took place on Friday the 11th and lasted about an hour and ten minutes. This was Nicholas Wilson's first meeting in his PR & Issue Manager role.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Timon Gehr
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Robert Schadek
* Steven Schveighoffer
* Adam Wilson
* Nicholas Wilson

## The Summary

### Exception clean up routines and language interop

Rikki wanted to know if the cleanup routines for D exceptions were called from C++ and vice versa. Walter said that happened everywhere except on 64-bit Windows because Microsoft's exception unwinding mechanism was only vaguely documented and completely impenetrable. He hadn't wanted to spend countless hours trying to figure out how it worked. So we used our own unwinding  mechanism on Windows. We used the dwarf/elf unwinding scheme everywhere else. It could catch C++ exceptions, but only if they were throwing pointers to classes. D didn't support catching values like C++ did.

Martin said this didn't hold for LDC. It used the Visual C++ personality function, but it required a hack to work. Rainer Schütze had implemented a great hack checking the assembly instructions around a specific address to figure out which version of the MSVC library was in play because LDC needed to suppress the terminate handler in case an exception was thrown during the unwind. That was allowed in D with exception chaining, but not in C++. It should work on both 32-bit and 64-bit as far as he knew. All the other systems used a single, simple model. Everything was much more complicated with Microsoft.

Martin's recommendation would was to come up with a test for this to figure out the behavior of the compilers and the support on all of the targets. There were some DRuntime exception integration tests that verified we could catch. They were only run on Linux as far as he knew, but they should work on more systems. This was done by compiling C++ code with the system compiler, throwing some D exceptions derived from `std::exception`, then catching them and checking that the strings were what we expected. Rainer had done some work on that a while back. We were basically compatible everywhere except for DMD on Windows.

Rikki said his question wasn't about throwing and catching, but about making sure the cleanup routines were getting called. He asked again if it was a compiler bug when it didn't match the system toolchain.

Walter said it wasn't a compiler bug. He could not figure out Microsoft's exception handling system on 64-bit Windows, so he was using something completely different. We conformed on other platforms.

Rikki said that in that case, his take was that it was implementation-defined. He was asking because he'd been talking with Manu about exception handling with fibers in relation to the work being done on Phobos v3. Manu needed to know if was a guarantee from the spec or not.

I noted that the spec said that objects in C++ stack frames were not guaranteed to be destroyed when the stack was unwound because of a D exception and vice versa. Rikki said he was double checking because there'd been a lot of work on exceptions the past 10 years and he didn't know if the spec was up to date.

Martin thought it *should* be part of the spec. At least that it worked on POSIX. Assuming the cleanup handlers were properly called, he didn't see a reason to keep it implementation-defined. That would then require us to add tests to catch regressions or to know when it didn't work on new platforms. On POSIX, anyway. On Windows, things might be more complicated.

Rikki said he'd be happy for it to be considered a compiler bug that was a WONTFIX. He just needed to know if it was guaranteed or not. Jonathan said it sounded like it should be, but in practice it couldn't be right now because Windows was such a mess. We had to say it was implementation-defined regardless of what we might like to do. If we couldn't make it work like we wanted on Windows, then we didn't have much choice.

Martin said we did. It would just be a DMD special case. Jonathan replied that we could also say this was an open issue that we weren't planning on fixing. But if someone came along and decided to implement in DMD what LDC was doing, for instance, then we could make it work that way. He didn't think there was a problem with that.

Walter said he'd be fine with someone picking up the flag to figure out how it was supposed to work on Windows.

Steve said we should be cautious about removing the "implementation-defined" label. D could be ported to some other platform where it didn't work. He thought we should just leave it implementation-defined and then document what was in the implementations.

Walter said that if you designed a crucial part of D that relied on interoperability between C++ and D as far as exception handling, there would be problems.

### Memory corruption in DMD

Dennis said Walter had recently added a test which was compiling on ARM but failed sometimes on x86, but in very specific scenarios: only on Windows and only when the compiler was compiled with a host LDC 1.40 or the bootstrap DMD. When he removed the MS-COFF void initialization, it seemed to disappear with the LDC build, but the bootstrap DMD build still failed. 

There seemed to be some really weird stuff going on. Maybe it was the padding bytes in a struct or a use after free. It was non-deterministic. Sometimes you needed to run the test 100 times to fail. He wondered if anyone had any ideas how to track it down.

Walter asked how Dennis had decided it was related to MS-COFF.  Dennis said that as he was bisecting it, he'd removed some void initialization, then ran the compiler 100 times to see if the issue was still there. It was when he removed the MS-COFF code that it worked on LDC, but not DMD. It seemed to be dependent on the stack layouts the compiler happened to assign to all the locals. Then DMD ran that test 100 times. Sometimes it generated a slightly different executable with a slightly different stack frame. It seemed like the register allocator or something had a non-deterministic edge case.

Walter said the MS-COFF code was originally written in C, then simply ported to D. It still used a lot of `strcpy` and `strcat` and wasn't doing any overflow checking on them. It would be better in general to use D arrays for it so we'd have bounds checking. Regardless of whether that uncovered the problem or not, we needed to modernize the code to get rid of the C string madness.

He asked if the bad code was always the same or if it varied. Dennis thought it was always the same, but he hadn't looked into it thoroughly. Walter said when he saw bad code generation and didn't know where or why it was happening, he would add an assert to the compiler to test if the bad code was generated. That had helped him find scores of instances. Who was generating the bad instruction? Instead of failing at run time, fail at compile time. That could help work backwards.

Another possibility to consider was that it could be a bug in the bootstrap compiler. Dennis was also thinking that, but because it also failed with LDC, that would then be two different back ends generating a corrupted DMD in the same way. The sounded kind of coincidental.

Martin said he wouldn't say so. We could expect these kinds of problems to be more likely with LDC than DMD. The optimizations done on an optimized build were in a completely different universe, so it was exploiting much more potential. He wouldn't say these problems were related, especially if was really consistently only happened with a specific host DMD compiler version and not with the recent ones.

Assuming the problem was with void-initialized stack variables, he wasn't sure if the LDC memory sanitizer could help. Dennis said he'd tried it. It gave some false positives for the GC, but he hadn't found the cause.

Steve asked if it was the same type of problem with the code generated by LDC and DMD in terms of the stack layout. Dennis thought it was the same. Steve didn't know if this was related, but he'd found a code gen bug when porting the GC over from SDC. He didn't know how to narrow that down. He was at a complete loss when it came to figuring out what should be generated and what wasn't. It would be awesome to have some kind of tutorial on how to diagnose things like that.

Dennis thought Walter's assert suggestion was pretty good. He was hampered a bit by his unfamiliarity with the back end, but he could try to figure it out. He didn't know how much time he wanted to spend on it. It was really nasty, but it was also a very edge case with the bootstrap compiler.

Walter said he'd wanted to upgrade the bootstrap compiler for a long time, but Iain was against doing it without a very good reason.

Rikki asked what happened when turning off debug info for the built compiler and the test case. Dennis didn't think he'd tried it on the compiler. He'd tried the failing test both with and without `-g` and it had failed either way. He could at least see in the disassembly which function the corrupted code had. Rikki said he'd had a case where `-g` caused corruption of some kind. Dennis said that was weird. He thought `-g` was a completely separate step. Rikki said it should be. If he remembered correctly, it was off by one number throughout, so it sounded very similar.

Walter said that some time back, he'd updated the compiler so that it didn't have to be running on Windows to generate a Windows binary. With `-os=Windows`, it would generate Windows binaries. That opened the possibility of running the address sanitizer on Linux while the compiler was trying to compile the Windows version of the test case.

Dennis said he'd tried that, but it had complained that he had to remove the position-independent code flags, and that he had to specify a Microsoft C runtime to link with. Walter said it didn't need to be linked. Dennis only had to run the sanitizer to see if it detected an invalid access. He didn't even need to check to see if the generated code was bad. Just run Valgrind and the address sanitizer on it and compile only.

Dennis thought he now had some ideas now on how to continue. 

Nicholas asked if Dennis had been able to reduce the test case and if he knew which functions were causing it to happen. Dennis said he knew which function, but if he removed everything after it, the problem didn't happen anymore. It was a real heisenbug. When you tried to narrow it down, it disappeared. It was really annoying.

### Templated runtime hooks broke DMD's custom behavior of new

Dennis said the compiler had a custom bump pointer allocator that tried to override the runtime hook when the `new` operator was used. It would do this unless the `-lowmem` switch was passed. Now that the runtime hooks were templated, the linker override no longer worked. Once we started building releases with a new enough LDC host compiler, it would start using `lowmem` by default and doing GC allocations.

There were three things we could do: just accept using `lowmem` by default; find a new way to override the `new` operator; or replace all uses of `new` with a custom template that could use either the bump pointer allocator or the GC. He asked what everyone thought of these options.

Martin said that `lowmem` didn't just mean it wasn't using the custom overrides, it also meant it was using the GC, but collection was still disabled. We were allocating with the GC instead of `malloc`.  When he'd done some tests some years ago with Rainer, they were seeing a performance improvement with `lowmem`. So this was mainly to address performance issues. The bump pointer allocator was supposed to be more performant than the GC, but in those tests years ago, the GC was faster. He had no idea why. He threw some possible reasons at us, but his point was that this was mainly a performance issue. He noted that, on the other hand, if the compiler still had a need for the overrides, we should probably give users a way to override the runtime hooks, too, so they could customize the templates with whatever allocator they wanted.

Steve said he had no idea how the overriding of the `new` operator worked, but he wondered why we weren't instead using a GC interface that implemented the bump pointer. Then there'd be no need to override these low-level functions. For example, we had a GC interface that just used `malloc`. It was a very similar thing.

Martin said that back in the day, every `new something` was lowered to the non-templated runtime functions. Overriding those meant not worrying about keeping track of which GC implementation was active, or having to resolve all the virtual functions. It avoided all that overhead. We should do some checks to see if any of this still paid off or if we should just kill all that advanced functionality.

Rikki said the easy way to do this was by changing it at the GC API level as Steve had suggested. But he wanted to point out that implementing the bump pointer allocator this way required a global variable to turn it on. It could be guarded by a `static if`. Then if it were too old, who cared? You then just got the GC. That gave a nice, backwards-compatible way to do it, and we'd get the performance in the future.

Dennis agreed we didn't need the override to work with old bootstrap compilers. He wasn't sure what the preferred option should be now. He hadn't explored using the GC interface yet.

Martin said he should start with testing. Something that multiple people could run on their machines to gather some performance numbers just to see if the default approach still paid off. Then see how a custom GC implementation or the `malloc` implementation worked out. Maybe the C `malloc` was fine. We weren't confined to using the one that came with the C runtime. For example, the previous LDC used mimalloc from Microsoft. He recalled that Johan had done some experiments and found up to 20% improvement in compile times just by switching to that allocator. Dennis said that mimalloc was awesome.

### ImportC module statements

Dennis said that in our [January 2025 meeting](https://forum.dlang.org/post/ohspwyjwkccolhroqdjy@forum.dlang.org), he and Walter had expressed different opinions on whether we should have `include`, or `mixinC` or something in ImportC. He wondered if there were any updates on that discussion.

Walter supposed it was a workaround and it seemed rather harmless, so we might as well approve it.

Átila thought it was a workaround that didn't need to exist. Dennis said there wasn't a better option right now. Átila said we'd probably need another workaround if we kept doing this. Dennis asked what we'd need another one for. Átila replied that there would probably be another thing that came up which D had and C didn't.

Martin said this was a specific special case. It really solved a big problem. Dennis's proposal was pretty simple. He wasn't a big fan of it, but he didn't think we needed to be backwards compatible here. So enabling this workaround now probably wouldn't prevent a better or more elegant solution in the future.

Walter said he thought Dennis had done a nice bit of work with it. He liked simple things like that. So he approved it.

### macOS 15.4

Martin wanted us all to be aware that macOS 15.4 had had broken everything. Apple had changed the dynamic linker protocol in some way that meant most compiled D executables didn't work on macOS 15.4. There was no way to fix this sort of thing before a new macOS release because Apple didn't release the sources for the dynamic linker until a month after a release. Sönke Ludwig had a fix he'd been testing and it worked on his machine and a few others. It was a simple one-liner.

This was urgent, but the problem right now was that there was no way to bootstrap a compiler on a macOS 15.4 machine, because all the compilers needed to the fix. The runtime, which was linked into the compiler itself, was broken. This was a big issue. We needed new preview compilers out as soon as possible. He was going to put out an LDC beta with a fix within a week, so we'd then have at least one host compiler version which could then be used to compile all the other D executables to make that work.

We'd had a similar issue with macOS 10.15 when they removed a private API function that we used. That was the whole reason for this mess now, as we then had to implement all the details ourselves. That problem just meant you couldn't compile any D code on macOS 10.15. This current problem meant that D executables couldn't run on macOS 15.4. 

He said we could thank Apple for all of this. The issue had come up during a beta test. Sönke had reported to Apple that all D applications were broken with the update and gotten no response.

### code.dlang.org outage and a broken dub

Another problem Martin wanted to report was that code.dlang.org was down because it was being attacked or flooded by crawlers, or AI training, or whatever. On top of this, dub either wasn't handling timeouts, or it didn't have a timeout for very slow transfers. He wasn't sure what the problem was, but he was getting lots of connection resets and connection timeouts. All of the dub processes just seemed to hang.

Dub seemed to be completely unusable at the moment. It also wasn't falling back to the fallback registry. That meant we had problems with DMD CI frequently failing. As part of one of the tests, we'd built a little test extractor tool as a single-file dub project that we needed to build with dub. Lots of those jobs were now timing out. They had the same problem on LDC CI where dub was used to build reggae. They had the same problem at Symmetry for some projects. Basically, dub was broken for almost everyone. A huge problem.

Steve asked if we needed to use the dub registry to build those pieces? You could specify in dub to not even look at the registry. Martin said we didn't want to have to hack all of the existing CI scripts to overcome this situation. 

He had no idea how to fix it. Ideally, it would be handled by Cloudflare. Maybe there should be a revision in dub itself so that we check that there's a timeout mechanism or a working fallback to the other registry to make this less problematic. Then at least the CI runners wouldn't be running for 60 minutes only to timeout because at minute 20 a dub build was triggered that timed out after 40 minutes without saying anything. We should instead break after one or two minutes and report a network connection problem. At the moment, these dub builds were just hanging. For anyone who didn't see that there was a hidden code.dlang.org dependency, that was a problem.

I said we'd had a problem before with some of the services on dlang.org when Cloudflare's proxy handling was enabled. It had caused things to break. I would check to see if any of those settings had changed. Martin said it had just come back online. It had been broken just before the meeting, and had been up and down for several hours.

Mathias agreed that we needed a timeout. He also thought we should get rid of the registry and use the same approach as cargo, Nix, and Homebrew, which was to have an index hosted on GitHub. That would allow us to have the index locally. This was an approach used by tons of tools, and it just worked. Sebastiaan Koppe had implemented it for his mirror years ago. He asked if there were any objections to adding it to dub.

Jonathan said it was all the better for us, as then GitHub had to deal with being spiked and not us directly. They had much better resources. Martin said we were relying on GitHub anyway, at least for most packages. It was just the index that we depended on the registry for. He agreed that was a problem. 

I told everyone I had just verified that code.dlang.org was proxied, so Cloudflare should have been handling this already. Martin said it definitely wasn't, and it was now timing out again.

Adam said someone had posted an article about what was going on. Cloudflare would normally be handling it, but it was an AI-related thing. They were being very duplicitous about their user agent strings. So Cloudflare had to develop a whole new tool to sink these guys. So we would have to turn on the new tool in Cloudflare.

While I figured out how to turn on the new tools in Cloudflare, the discussion veered off onto the details of the article and how crazy AI had gotten. I think I had to adjust the Cloudflare settings in the following days, but it wasn't long before everything got back to normal.

### FreeBSD 14 CI

Jonathan reported that the test suite wasn't working with FreeBSD 14. He had a bug report about it somewhere and it wasn't super high priority, but it needed to be remembered at some point. The problem was that FreeBSD 14 used assembly code in at least one header that we were using for some ImportC tests, and DMD couldn't handle that. 

Martin said they'd seen this kind of ImportC issue on other systems as well, especially with inline assembly. The work around they'd used was to import the C file rather than compiling it. In the more general case, maybe problematic code could be replaced with `assert(0)` on importing or after automatically analyzing it later.

He noted that if this changed in some specific FreeBSD 14 header, a very important one, then presumably it would be a problem for everyone on FreeBSD 14 trying to compile a C file which happened to include one of the standard headers where this function had now been added.

Jonathan believed it was in `stdlib.h`. As he recalled, they'd fixed something that had been screwed up with `qsort` and had changed the signature. It was using some assembly and a macro telling it to use different mangling for old code.

Martin thought Walter had added a special case to recognize trivial ASM statements. And it was even tested. As he recalled, it had occurred on Mac a year ago or so. But it looked like on FreeBSD with that specific syntax, it wasn't working.

Walter said he'd found that those wacky compiler extensions in the header files were almost always protected by a macro. He constantly ran into these. When he did, he'd put a macro in `importc.h` so that it wouldn't follow the branch with the nutball extension. 

He said Jonathan could look for the macro that was wrapping the inline assembly. If was protected by `#ifdef`, then he could define a macro in `importc.h` that took the path through the header which avoided the inline assembly or the extension we didn't support.

Jonathan said that might work in this case. This was something that had come up before with FreeBSD when they fixed things. It would continue to come up. He believed they had a macro specifically for this to make it look clean.

If it was a straightforward fix, that would be great. But FreeBSD 13 was coming up on end of life next year. We were going to need to upgrade our CI at some point to FreeBSD 14. He'd been using it just fine, so it seemed to work. It was just failing the test suite because of this issue. 

### Misc.

I gave an update about [our new store](https://store.dlang.org/). I'd migrated some of our existing stuff over from the old one and the new one was a lot easier to use on the backend. 

I also let everyone know that Weka was sponsoring one night of BeerConf at DConf and had been working with our event planner to arrange something. For the first time, we were going to have food included. On past sponsored nights, we'd not included food because we were worried about running over our minimum spend too soon. With the higher minimum spends the past couple of years, that wasn't a problem. We'd excluded food and pricier drinks for the sponsored night in 2024 and never got close to using the whole minimum spend.

Adam was having a problem with the Google Summer of Code UI. He was a mentor and could see the proposals, but couldn't click anything. He was wondering how to get in to see them to evaluate them. I said he should email Razvan, as he was running the show. He'd only just left the meeting a short time before.

## Conclusion

We held our next monthly meeting on May 10th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
