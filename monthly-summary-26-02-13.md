The D Language Foundation's February 2026 monthly meeting took place on Friday the 13th and lasted about fifty minutes.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Martin Kinkelin
* Mathias Lang
* Mike Parker
* Átila Neves
* Robert Schadek
* Steven Schveighoffer
* Nicholas Wilson

## The Summary

### Complex numbers

Walter said he had given up on trying to remove complex number support from the backend. It was easier to implement it than it was to deprecate it.

What irritated him was that there were three soft complex number implementations in the D library. There were two in the runtime and one in Phobos. Why did we hae three implementations? He had tried to remove one of them, but that caused problems because there were dependencies on all three implementations. The whole thing was just madness. But he still needed to implement complex number support in the AArch64 code generator.

That looked to be not as difficult as he had expected. It was mostly a matter of going function by function and figuring out how to handle complex numbers in each one. The code generation was very different from x86, where complex numbers had been implemented using XMM registers or the x87. AArch64 had its own set of floating-point registers, and that was what he was using. He said it was proceeding nicely.

He hadn't heard much interest in complex numbers for a long time, so to gauge interest, he [had posted about them in the forums](https://forum.dlang.org/post/10lukk7$2ne$1@digitalmars.com). That brought out the people who cared about it, which told him there was interest after all.

Once complex number support was implemented, the best thing to do would probably be to undeprecate it. Complex numbers had been deprecated for a long time, but people still relied on them. His thought we should throw in the towel and undeprecate them.

Martin said that complex numbers had been a pain for him as a backend developer because they meant ABI calling convention trouble. There was always the special case of complex numbers with two real values inside. Even if we removed D's complex number support, we would still ideally need to handle complex numbers for C interop. So at least for C interop, the backend would still need to handle those special cases anyway.

From the user side, he thought a library solution might still be preferable. Some users with a mathematical or physics background would want complex numbers, but if the library version could be made easy to use, that might be good enough. He also noted that Iain had previously mentioned the library solution could be faster in some cases because it could use SIMD instructions.

### Exception handling

Over the previous month, Rikki had been researching an alternative exception handling mechanism. Instead of tables, hidden parameters, or similar machinery, the calling convention would change so that after a call instruction, a function pointer for cleanup would be embedded.

He said this had some good consequences. For example, the catch statement would occur below the throw statement in the call stack, which meant the exception and related data could be stack-allocated safely. But this approach also meant that non-D functions couldn't throw up the call stack, and it would heavily complicate the calling convention. He said he had reached the limit of what he could determine on his own about whether it was a viable option.

Walter said the difficulty was that D supported C++ exception handling and Windows exception handling. Coming up with a better exception handling mechanism was a good idea in theory, but if it was incompatible with C++ and Windows, then D would still have to support the old scheme as well. That would be a large complication. He said he would rather have no exception handling than have two exception handling mechanisms.

Martin agreed. C++ interop was very important. With LDC, on Linux, macOS, and Windows, you could throw C++ exceptions in D and catch them in D. As for stack-allocated exceptions, he wasn't interested. In his view, exceptions were for exceptional cases, so he didn't care much if the GC or some other allocation method was used at that point. Optimizing exception allocation was not interesting to him.

Steve agreed with Martin that stack-allocating exceptions didn't make much sense. You had to allocate anyway to generate the stack trace, and D already had ways to allocate exceptions with `malloc`, global storage, or other approaches without using the GC. As far as he could tell, the most difficult part of exceptions was that they made D hard to port to other platforms. Exception handling was a complex, very low-level feature that didn't exist cleanly everywhere. He said WebAssembly was one platform where this was one of the main reasons D didn't work well.

Martin said the main problem was D's support for exception chaining. That made things complicated. Without it, D could more directly rely on C++ exception handling routines. On POSIX, D had its own personality function. On Windows, LDC used Microsoft Visual C++ primitives, but it had to do some ugly work to handle D's behavior, particularly around exception chaining and the way errors override exceptions.

Walter said exception chaining was unique to D, had turned out to be a bad feature, and he would be happy to see it removed. I said it wasn't unique to D, because Java had it, and I thought the inspiration had come from Java. Mathias remembered the same thing. Walter said he didn't recall that, but either way, it didn't work very well. C++ had survived without it, and he didn't see why D should keep it.

I didn't know if anyone actually used exception chaining in D or if most people were even aware it existed. Steve said the main thing people did with exceptions was print them. Nobody cared whether there was a sub-exception or another sub-exception underneath.

Rikki suggested adding a new hook to the error module in DRuntime. If an exception chain were detected, the runtime could call that hook and terminate the program if there was nothing useful to do. In that case, the program would clearly be in a state where it couldn't even do cleanup. Walter said he would post in the newsgroup about whether exception chaining could be removed, not even deprecated, just turned off and removed.

I asked for the consensus on Rikki's exception handling idea. Rikki clarified that it wasn't a proposal, but research into whether the approach was even an option. Steve and Mathias both said no. Rikki said that meant if we wanted a different error handling mechanism for something like Phobos v3, it would have to be more involved for users. It wouldn't be something where the language handled everything invisibly. There would be some kind of cost on the happy path, and users would have to write code explicitly.

Mathias asked what problem Rikki was trying to solve. Rikki said it was more like result types with optional errors. Mathias said that if the problem was exceptions using the GC, he didn't think that was a problem. Rikki said that wasn't necessarily the problem, but it was a signal that the current mechanism was more costly than what he had in mind.

I asked whether this was related to an earlier discussion about different categories of errors and exceptions. Rikki said no. This was part of his value-type exception work, which had turned out to be impossible. This mechanism had looked like it might be the holy grail, but apparently it wasn't right for D for other reasons, even if it might theoretically work.

### The DMD 2.112 release

Martin brought up the 2.112 release. Since there hadn't been a proper beta, it was clear that 2.112.0 wouldn't be regression-free. There had then been a strange 2.112.1 point release that wasn't announced and didn't appear on the downloads page. He thought it wasn't even on dlang.org, though there was a GitHub release page with a few artifacts. That release came only two weeks after 2.112.0, so only a few regression fixes had made it in.

He said regression fixes needed time. Users needed to try the new version, encounter breakage in real code bases, investigate the issues, and then fixes had to be prepared. The number of required adaptations wasn't small, especially since the timeframe for the 2.112 release had been long and quite a few breaking changes had made it in. One interesting example was that the associative array implementation was now fully templated in DRuntime with proper attribute checks. That had caused several adaptations on Symmetry's side, including regression fixes. The process of analyzing, finding, and fixing issues was still going to take some time.

He said another point release would be needed once things were in halfway proper shape. He'd spent the previous week or two testing the current state of the latest stable branch with LDC against Symmetry's main projects. Those projects had always been an interesting testbed for compiler upgrades because they were never regression-free. At first, the build success rate was only about 20%. Out of roughly 350 build targets, around 80 failed due to compile errors. With current `dmd-stable` and a future cherry-pick from `dmd-master`, he had gotten the compile errors squelched, and was now dealing with two undefined symbols related to `RTInfoImpl`.

He had also checked the 2.112 regressions on GitHub and found one major issue that he wanted to bring up because he thought it needed Walter's attention. It was related to the backend and [an array performance regression Steve had filed](https://github.com/dlang/dmd/issues/21976) in October.

In Steve's test case, two arrays of 1,000 integers were compared a large number of times. On Martin's older AMD Threadripper machine, the performance degradation was about 80 times slower. Something that had taken around 35 milliseconds with DMD 2.111 now took around 2.9 seconds with 2.112. Steve had seen similar results on his machine.

Martin said the cause was [one of the templated hook changes that made it into 2.112](https://github.com/dlang/dmd/pull/21513). Previously, array comparisons used `__equals` or similar runtime logic. The runtime had logic to check whether the element types of two arrays were compatible for `memcmp`. If the arrays were trivially comparable, such as arrays of integers, the runtime would call the C library `memcmp`, which was heavily optimized.

In 2.112, that logic had been moved back to the compiler. The backend now saw an equality expression for the arrays rather than a lowered `__equals` call. DMD's middle end forwarded that as a memory comparison to the backend. But the DMD backend didn't emit a libc `memcmp` call. Instead, on x86, it used an inline implementation based on the `rep` instruction, which was much slower on Martin's machine than glibc's optimized `memcmp`.

Martin said the proper fix was probably for the backend to check the `memcmp` size. If the size was static and not too small, maybe 16 bytes or larger, then the backend should prefer a libc `memcmp` call instead of the inline `rep` instruction. From a user perspective, this was a regression because array comparisons could become dramatically slower. From a backend perspective, it wasn't a regression in existing backend behavior so much as a frontend change that now caused this backend path to be used.

Walter asked why the pull request that pushed the logic to the backend couldn't simply be reverted. Martin said he doubted the change was cleanly separated as its own commit. It was part of a pull request whose title was about getting rid of an array equality function, but as part of that work, the logic moved from the runtime to the compiler. He didn't see a necessity to revert the whole thing, but a partial revert of the logic shift might be possible if it was cleanly separated.

Steve said the reason for the 2.112.1 release was that 2.112.0 segfaulted on macOS. It had the same kind of problem seen before, where the wrong or older compiler version was used to build itself. In this case, DMD, dub, and everything else crashed. That was why there had been a rush to get out a point release. He wasn't sure why it hadn't been officially announced, since fixing a release that didn't work at all on macOS was a valid reason for a point release. He added that a 2.112.2 release could also happen for regressions.

On the array comparison regression, Steve noted that the backend operation was probably called something like `OPmemcmp`, which could have fooled someone into thinking it would lower to a `memcmp` call. Walter said that on x86, it lowered to string instructions, while on AArch64 it actually lowered to a call to `memcmp`.

Martin said the performance varied greatly by CPU and platform. Razvan had reportedly seen a speedup on Windows with a newer Ryzen laptop, where the `rep` instruction took around 100 milliseconds. On Martin's older Threadripper, it took almost three seconds. On his Intel laptop, it took around half a second. The `rep` instruction had apparently been optimized greatly in recent CPUs, but still wasn't in the same league as glibc's `memcmp`.

Walter pointed out that one problem with calling the C runtime's `memcpy` or `memcmp` was that it trashed registers. There were tradeoffs. If the compiler inlined the code, there was no function call and no need to follow the function call protocol or assume that registers would be trashed.

Martin agreed, which was why he suggested a size threshold. If the size was known and small, the inline implementation should remain. That was essentially what LLVM did for LDC. LDC emitted an LLVM `memcmp` intrinsic, and LLVM decided whether to inline based on the size and CPU features. For tiny comparisons of one byte, four bytes, or other small sizes, inline code made sense. But for unknown or larger sizes, calling `memcmp` could pay off hugely, especially on older CPUs.

Rikki noted that if DMD used an old C runtime on Windows, then performance would depend on that old runtime. He said DMD could work without MSVC installed. Martin clarified that the old Visual Studio 2013 runtime was used only as a fallback. If a user had an activated MSVC environment, DMD would use the selected MSVC version and libraries.

### D blog, intern, and DConf '26 updates

I had a couple of updates. First, Jared Hanson was working on migrating the D blog from WordPress to a GitHub site. Jared was one of the authors of the tuple unpacking DIP. He planned to write a couple of posts himself. He also had some guest posts lined up. I said we expected to have the blog active again soon. (__UPDATE__: Check it out at [blog.dlang.org](https://blog.dlang.org/)).

Second, we had an intern lined up. Fei, who had been Steve's Google Summer of Code student and one of Ben Jones's students, was still in the United States looking for a job. In the meantime, he was going to work with us for free, and Nicholas would be his supervisor. We would need to find work for him, since he needed to work at least twenty hours a week.

Rikki asked whether we could give Fei a book, specifically _Unicode Demystified_. He had some work for someone to do on `std.uni` that he didn't want to do himself. He said it needed to be done before modules were copied over to Phobos v3, and it was important for Turkish users. I said Nicholas would need to work that out with Fei and see if it was something he was willing and able to work on.

Next, I gave an update on DConf '26. CodeNode was still holding the dates for us. The contract negotiations between Symmetry and Brightspace, our event planners, had started recently. I hoped they would be finalized before mid-March.

Finally, I reported that I was trying to secure some unallocated funding this year. Every year, our funding dollars were always preallocated for things like salaries and DConf speaker reimbursements. I wanted to go beyond that this year so that we could have more room to spend for other things, like contract work, and maintain a cushion in the bank.

### More regressions

Martin brought up another issue he had encountered while migrating toward 2.112. A seemingly innocent change in Phobos, involving a template constraint for `std.container.rbtree`, had temporarily blocked Symmetry from migrating.

The constraint checked that a user-provided comparison predicate was a binary function taking two values of the expected type and returning a boolean. It had been changed to support `const ref` versions of the predicate. To make that work, the code needed lvalues, so Vladimir had come up with the lambda notation for it.

That change badly broke attribute inference in one complicated case at Symmetry. `@safe` and `pure` were no longer inferred from the user-provided predicate, and some internal functions used by `RBTree` were affected. Martin had worked around it in `phobos-stable` by using `std.traits.lvalueOf` in the template constraint instead of the lambda. This worked where the lambda did not.

He thought this could be an interesting test case. He hoped it could help solve the issue. The surprising part was that the change had only wrapped an existing call in a small non-templated lambda with an explicitly spelled-out type, but that had been enough to cause the inference problems.

Steve mentioned another regression that he and Martin had worked on at Symmetry. Since D 2.106, the GC or array runtime had apparently treated `void` arrays as not containing pointers. If a `void` array was allocated, the GC thought there were no pointers in it. Steve was surprised it had taken so long to find, because they were seeing objects collected that shouldn't have been collected. He said the fix was simple and should probably go in before the next release. If anyone saw random segfaults, he suggested checking for `void` array allocation.

Walter said he was glad they found it. Random crashes were the worst kind of language problem.

Martin said the reason was clear, but the question was how to fix it. The `TypeInfo` for `void` said it had pointers, which was how things had worked before 2.106 and before the templated implementation. The template now used `hasIndirections`, and it was unclear what that should do with `void`. The question was whether to fix it in `hasIndirections`, though Steve had pointed out that it had been used correctly in this specific array allocation case and could be used incorrectly elsewhere.

Jonathan said part of the problem was that `void` was an utterly bizarre case. `void` by itself was not really a meaningful type in the usual sense, and yet it sort of was. A `void*` had pointers, and a `void[]` could potentially contain pointers. Anything dealing with `void` by itself was a minefield, so it wasn't terribly surprising that something had gone wrong.

Going back to lambdas and template constraints, he said there were many small issues around whether a type allowed copying, whether `auto ref` was needed, and similar details. In general, the best solution was probably to provide enough test types. With Phobos v3 ranges, he wanted to provide many types to test against, because otherwise it was easy to write template constraints that worked for arrays or copyable types but failed for non-copyable types or other corner cases.

### Anonymous union traits

Jonathan said Walter had added a trait for checking whether something was a bitfield. He wanted to know whether it would be difficult to add a similar way to ask whether a field was part of an anonymous union.

Walter suggested rephrasing the question as whether any two fields overlapped. Jonathan said that was part of the problem. In some cases, that might be the real question, but if you wanted to filter out all overlapping fields, having to compare fields against each other made templates much more complicated. It would be simpler if each field could be queried individually, as with bitfields. As far as he could tell, there was no way to ask whether something was in an anonymous union, and that complicated some traits.

Walter said a trait could be added, but it could also be done from user code. You could iterate through all the members, look at their sizes and offsets, and see if they overlapped. That was what the compiler did internally when building initializers and doing some memory safety checks.

Jonathan said it could be done, but it became much more complex, especially if the compiler already knew how to calculate the answer. Walter clarified that the compiler didn't store the answer as a flag. It could calculate it, but it didn't already have it sitting around. He said it could be done.

Martin said that sounded like a job for an intern.

### Bugzilla backup

I closed with an update on `issues.dlang.org`. It had been reported as down a while back. I had emailed Brad Roberts, the long-time maintainer, because Vladimir didn't have a backup of the database. Brad hadn't responded, but the site had seemed to be back up.

Vladimir had still had difficulty connecting to it. Someone had recently set up a monitoring service for some of our servers, and it showed Bugzilla as being down for 16 hours, even though it had been consistently accessible to me. Because of the uncertainty, Vladimir downloaded the database and set up [a backup on our Hetzner server](https://bugzilla-archive.dlang.org/). So if we lost `issues.dlang.org` again, we had the full Bugzilla database. It wasn't in a Bugzilla instance, but the data was there.

Steve said he had also experienced slowness when the site first came back. After I posted in Discord that it was back up, he'd tried to open it and it took about a minute to load the first page. I said that was strange because it had been snappy and consistently available for me all day, even while Vladimir was telling me it was down for him.

Jonathan said the site had already had problems before it disappeared. For a while, he had had a terrible time getting access to it, even when it was still there. When he checked recently, it loaded quickly, but it had definitely had issues for a while. He noted that several of our services had problems caused by bots, and that might have been part of it.

I said that, either way, we now had a backup on our server thanks to Vladimir, so we were protected.

## Conclusion

Our next monthly meeting took place on March 13th.

If you have anything you'd like to bring to us in a monthly meeting, please let me know.
