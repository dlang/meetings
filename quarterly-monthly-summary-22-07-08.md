[Forum Post](https://forum.dlang.org/post/lxfildvecircypoabain@forum.dlang.org)

This summary is quite a bit overdue. Sorry for the delay.

The July 8 D Language Foundation meeting was one of our quarterly meetings. In the first part of these meetings, representatives from industry join us to provide us with updates, notify us of issues they're experiencing, and provide us with feedback and ideas. The second part is the typical monthly meeting addressing foundation business. The industry reps are always invited to stick around for it and usually do.

For this meeting, we had two new faces join us: Paul and Robert Toth of Ucora. [They let us know back in June](https://forum.dlang.org/post/oxapxadxxkrujnhfeoqr@forum.dlang.org) that the company has been using D for years. We invited them to join us for our quarterlies, and they accepted. We hope we can help resolve any D issues they may face, and we welcome their feedback on the D user experience.

## The meeting
The meeting took place on July 8 at 14:00 UTC. The following people attended:

* Walter Bright (DLF)
* Iain Buclaw (DLF/GDC)
* Ali Çehreli (DLF/Mercedes Benz R & D North America)
* Max Haughton (DLF/Symmetry)
* Martin Kinkelin (DLF/LDC)
* Dennis Korpel (DLF)
* Mario Kröplin (Funkwerk)
* Mathias Lang (DLF/Symmetry)
* Razvan Nitu (DLF)
* Robert Schadek (Symmetry)
* Paul Toth (Ucora)
* Robert Toth (Ucora)
* Bastiaan Veelo (SARC)
* Joseph Rushton Wakeling (Frequenz)

### Mario
Funkwerk had a problem [with Issue #23234](https://issues.dlang.org/show_bug.cgi?id=23234). There was a little discussion about the issue, and that Funkwerk has a workaround for now, but Razvan put it on his list of priority issues and subsequently resolved it.

Mario said they sometimes run into issues like this when they update the compiler after a long time, or when Mathis when they try something weird (i.e., when Mathis has ideas and they find new areas in the compiler that don't work as expected). On the other hand, they've been using D for well over a decade. They have their own set of libraries and the solutions to their problems are usually straightforward. D has been a big help in their work.

### Bastiaan
SARC has marked a major milestone in that their 500KLOC Extended Pascal codebase has been completely transcompiled to D (if you aren't aware of this project, you might enjoy Bastiaan's talks from DConf 2017, [Extending Pegged to Parse Another Programming Language](https://youtu.be/NoAJziYZ4qs), and DConf 2019, [Transcompilation into D](https://youtu.be/HvunD0ZJqiA)). Now they are increasing their testing and working out other steps they need to take before going into production.

One of those steps is internationalization. For that, they're using GNU getext, and a couple of weeks before the meeting had released [their D package for interfacing it on dub](https://code.dlang.org/packages/gettext). He said they would soon be releasing a package for string extraction, and that with that we'd have comprehensive support for gettext in D. He said everything is looking good, though they do have a couple of nasty bugs for which he hopes to find a solution.

### Robert S.
Here I can quote Robert in full:

> Not much to report. The compiler's too slow. Other than that, D is the best language.

### Joe
And I can quote Joe in full following Robert:

> What he said I think. Nothing to add from my side.

### Paul & Robert T.
Robert noted that one of their new programmers has had a hard time getting up to speed with D via the existing documentation.

Ucora have been working with D for about 7 years, and their primary product is written in D, so they are heavily invested. Paul's interest is in the accessibility of D to the wider programming community. He was pleased with the response to [the post he made in June](https://forum.dlang.org/thread/xpykboajqfwpbpzgflru@forum.dlang.org) with a suggestion for the D documentation. Being new to the community, for now he's just observing, but they are interested in helping out in any way they can.

Bastiaan noted that in Paul's forum thread he requested that Ucora be added to the Orgs Using D page. [Max had opened a PR](https://github.com/dlang/dlang.org/pull/3326), but then got busy with other things (and has since been unable to build the website). The PR hasn't yet been merged, but I'll see about making that happen as soon as I publish this summary.

Paul said that for his own purposes, the documentation is well-written and he can find what he needs. But there is an initial hurdle to get over to be able to use the documentation effectively. New programmers rely on a programming language's manual when learning the language, so he's interested in ways of making D's documentation more accessible for new users. Enabling comments on documentation pages was the only concrete suggestion he had now. Walter suggested this could be done by creating a forum thread for each page in the documentation where people can post comments.

I noted that we have previously discussed overhauling the website and that a complete evaluation of the UX should be part of that. This is something that will fall under the purview of the ecosystem management team when it's in place.

We then discussed the topic of the user experience of documentation which touched on the experience with programmers at Ucora (in using the documentation and why they aren't using some of the more powerful features of D), comments on PHP doc pages, the D Tour, different kinds of documentation (reference vs. tutorial vs. how-to guides), Ali's book, ways to delineate them, and more.

Some good ideas were brought forward. Overhauling dlang.org and the D UX as a whole is something we want to get done as soon as we can after we get our ecosystem services sorted.

### Mathias
Mathias had a few things to bring up.

The first was [Issue 23164](https://issues.dlang.org/show_bug.cgi?id=23164), which he encountered when preparing his DConf talk. In short, it looks like dmd is moving structs that have copy constructors. This has resulted in portions of our `core.stdcpp` being disabled on different platforms. But LDC and GDC work as expected. Walter said the first problem is that the example has an internal pointer, which is noted in the D spec as something that shouldn't be done. He said the second thing is he doesn't think dmd is moving structs, even though the spec says it can. He thinks what's happening is that one of the copy constructors is passing an rvalue, causing a copy onto the stack and not updating the internal pointers. But Mathias said it's working as expected with LDC and GDC, and Andrei had said in the past prohibiting internal struct pointers is untenable. (This has all since been noted in the Bugzilla issue).

Ultimately, Walter suggested the example be further reduced to determine exactly where the problem is. There's too much going on with it right now. Mathias said he would try to do that.

Iain noted that GDC's behavior can be attributed to an internal flag attached to non-trivial structs that prevent it from being copied around, the compiler will crash instead. That means this kind of struct is always passed and returned by reference.

The second thing Mathias brought up is that he had been working on improvements to dub to support more use cases and output better error messages. He encountered a blocker when he had to enable DIP 1000 on some code and ended up with memory corruption. He hadn't yet submitted an issue for it at the time (and I don't know if he has submitted one since).

The third thing was the merger of the DMD and DRuntime repositories into a monorepo, which was something he and others had scheduled for the next day. Mathias warned that there might be some disruptions in CI and such as a result. He asked if anyone had any objections to the move. No one did. The merger was subsequently carried out. It did cause a few hiccups, but Mathias, Iain, and whomever else was involved handled them all (as far as I know).

Next, he said that we have a problem with some preview/revert switches (and it arose specifically with `-preview=in`). DRuntime and Phobos need to be compiled with a certain set of flags. If the flags used when compiling user code don't match those flags, we have a problem. We want a period where the user may or may not be using a certain preview, but if that preview changes the ABI, it doesn't work. One idea he had for this is a new `pragma` that could be put at the top of a module to specify which flags it needs to be compiled with, but it's not very elegant. He was hoping multiple brains could come up with something better. This led to some discussion about whether the pragma would work (Walter doesn't think so).

Walter said the only way he sees around this is that preview features shouldn't change the ABI, but one thing that could help is moving to header-only libraries. Martin suggested this happens with DRuntime and Phobos because they're the only pre-compiled libraries we have in the ecosystem. If they were dub projects, the problem goes away. In the end, no solution presented itself in this meeting, so we deferred this to a future monthly meeting.

Next, Mathias brought up the two ways to define C++ namespaces in D, one using an identifier and the other using a string. The former came first, with the latter being added later to address deficiencies in that approach. Mathias had encountered problems in his work with C++ interop using the identifier version and has since ignored it completely, relying on the string version. Plus, having two alternatives is confusing to people with no experience in D and C++ interop. He would like to deprecate the identifier option. Walter recalls past discussions about this, but can't remember the details about the tradeoffs. So he had no answer at the time and suggested they talk about this at DConf (I don't know if they did).

Finally, he talked about how `core.stdcpp` is compiled into DRuntime and is dependent on a specific C++ runtime. Initially, it was added to DRuntime on the premise that it's equivalent to the C bindings already there. The problem is that the C bindings don't cause any symbols to be generated, but the C++ bindings do. This can result in an ABI mismatch. Mathias thinks they should be out of the runtime. Walter agreed.

### Martin
Martin still has some annoying issues with semantic ordering and cycles. Apart from that, he had been waiting for the 2.100.1 release of DMD, and if it didn't happen in the next week or so he would go ahead and push out a new release of LDC anyway ([which he did on July 20](https://forum.dlang.org/thread/ieblpdhflpdjjtkalogh@forum.dlang.org)).

He said the move to a monorepo for DMD and DRuntime provided the opportunity to undo some mistakes made with LDC's repo in the past by moving it also to a monorepo. It also presented some challenges, but most of them had been cleared up. He expected everything to be fine. It would simplify things for LDC in the future and he was looking forward to the merger.

### Iain
Iain has done a lot of internal GDC work which mostly benefits him and anyone using GDC as their main compiler. He had been doing a bit of tech support for Bruce Carneal related to SIMD. That has resulted in some new SIMD intrinsics. He had also moved about 600 lines of code out of the compiler and into the library.

Upstream, he had added float and double precision implementations of the log family functions in `std.math`. This has the potential to break compilation for projects passing integrals to these functions. Additionally, `logb` is now pure.

On infrastructure, he had seen no pushback on the monorepo from anyone heavily involved with the DMD and DRruntime repos. The only thing left to test was whether the pipelines would be okay after the change. There were no issues running the test suites locally.

On DMD releases, Martin Nowak has less and less time to do them. Iain was in the process of taking over. The intent is to take move the specifics away from one person (e.g., having secret keys on one person's laptop) and set things up on GitHub runners so that anyone on the DMD core team can trigger a release. He still had blockers from making that happen. The first was that he did not have access to the downloads.dlang.org server. The two people with the AWS credentials had been unresponsive (though I believe that has since changed). The second is that he has no certificates with which to do code signing for the Windows binaries.

Since the meeting, Iain and I have set up a new server for the archived downloads. We've moved away from AWS and are now using Backblaze (with free bandwidth, thanks to the Bandwith Alliance and our use of Cloudflare). All 235.2 GB of DMD downloads have been moved. Iain can correct me if I'm wrong, but I believe this is active right now, so anything you download from downloads.dlang.org will come from there. As for the code-signing certificate, we're trying to decide on an option that's best for us. In the meantime, the last I heard, Martin Nowak was going to put out a release of 2.100.1 without signing the Windows executable.

### Me
I told everyone I'd not had an update from Vladimir Panteleev or Petar Kirov regarding our migration of ecosystem services to foundation control. I'd created accounts with Hertzner, Netlify, and Cloudflare, and granted Petar admin access to them. Petar had since become busy, and I anticipated we'd have to wait until after DConf for any progress. That turned out to be the case.

We've since migrated our DNS to Cloudflare (which some of you may have noticed a couple of weeks ago when we had outages for some subdomains) and I've granted Cloudflare access to more administrators (Iain, Vladimir, and Mathias). We hadn't originally intended for the downloads to be the first service migrated---that was dictated by circumstances---but now that it's done we can look at migrating the next thing. I'm not sure at the moment what that will be, but I'll announce it here when I know.

### Razvan
Razvan had nothing to report.

### Dennis
Dennis brought up a discussion about reducing the size of `object.d` [that came up in a PR thread](https://github.com/dlang/druntime/pull/3860#issuecomment-1170139760). Specifically, moving things into separate modules, then making `object.d` a list of public imports would make it easier to maintain a custom `object.d`.

Walter said that the bigger `object.d` gets, the slower it compiles, but splitting it into multiple modules would make it slower. Compilation speed is less dependent on file size and more dependent on the number of files. We may be expecting too much from `object.d`, but he had no answer for now. We need to be more parsimonious about adding things to it.

Martin had no magic answer either but said he's always seen `object.d` as the foundation of DRuntime. Any pay-as-you-go runtime would need to start from there. But he noted that most of the recent size increase is because of the project converting DRuntime hooks to templates. He suggested those imports could be removed from `object.d`, and the compiler can insert imports explicitly where they are used so that they aren't part of the compilation for modules that don't use those hooks.

Next, Dennis talked about the focus of the PR that brought about the discussion: that symbols like `destroy` are part of the global namespace (the PR added the ability to disable `destroy` via a special version identifier). Should we be concerned about this? The consensus was that with D's built-in ability to differentiate symbols (FQN, static imports), this isn't a problem. Iain closed the PR after the meeting, noting that we agreed something needs to be done about `object.d`, "but not this".

The third thing Dennis brought up was a recent thread about private-to-the-module, and how some people would prefer that it be private-to-the-class instead. He doesn't think the feature should change, but he did [put together an implementation to try it out](https://github.com/dlang/dmd/pull/14238). He asked if this was going to be dead on arrival. The consensus was "yes", with Martin noting that what he liked most was Dennis's "thumbs down" for his own pull request. We did have a short discussion about this, with Martin noting that this isn't a problem if you're properly encapsulating your types. It's only if you're used to older codebases or other languages that it may seem to be a problem, but in D we don't need to worry about it. Dennis subsequently closed the PR.

As part of that discussion, I raised the issue that came up in the aforementioned forum thread: we treat the private API of the module as part of the class/struct's private API, but we don't do the same for the public API. Meaning, calls through the public API of a class always trigger its invariant, but calls through a module's public API do not. This means it's safe for the class to modify its internal state through its private API, but not for the module to do so (similarly, a module's public API does not obtain the lock of a `synchronized` class). I think it's something most programmers don't consider, and we should think of some way to address it. Though, we didn't do that in this meeting :-)

### Max
Max said he had several PRs he needed to finish. For one of them, he had implemented `with` expressions to play with in a branch, something he and John Colvin are fond of. He had also opened a newCTFE branch from Stefan Koch's old work. He and Nicholas Wilson had been going through to get rid of problematic code. His opinion was that we should get rid of its warts, refactor it, and upstream it. The preview flag was enabled so people could play with it, but it will revert to the old engine in cases where it fails.

Martin wanted to know more about the revival, as he was under the impression that Stefan had given up on it because it didn't provide the performance gains he had expected. Max estimated it should generally be between 5 and 10 times faster, but that it can be much faster in situations where the old engine gets bogged down. In contrast, [the JIT in SDC](https://github.com/snazzy-d/SDC) (which uses the LLVM JIT API) can be up to 50 times faster. But for Max, performance isn't as important for newCTFE considering that the code is easier to work with and it uses less memory.

Martin then brought up the fact that newCTFE had been restricted to using 64-bit doubles and asked if that's still the case. Max affirmed that's so, and Martin said that won't do. Max suggested that since we now have ImportC, we could just use it to compile x87 soft floats, assuming the license is compatible. Martin said we should focus on x87 too much. It's interesting for DMD as it's x86 only. GCC/GDC uses soft float emulation across platforms, and LDC uses the host platform's `real`. We have to decide if we want an abstract precision across all platforms like GDC is using, or try to emulate the target precision on any host. What's important is that newCTFE must be able to cope with arbitrary `real_t` types as the frontend currently can. Max thinks it's doable with some refactoring.

### Walter
Walter said he had lost a lot of time in the preceding few weeks trying to debug failures in the test suite that can't be reproduced. He had been complaining about the lack of information in the test suite logs. He cited a specific example where running the test online showed a failure in a file, but gave no other error messages about that file, but then running the test suite locally resulted in a useful error message in the log. He suspected it might be a `stdout` vs `stderr` issue. It's a general problem he has. Further, the tests put out log files that are thousands of lines long, making it difficult to pinpoint the actual problem. Sometimes, an error just disappears on a subsequent run and he doesn't always understand why. He had noticed that it sometimes happens after he rebases a PR. He noted that the autotester rebases automatically, so the other tests should, too.

Iain noted that the need to rebase should disappear once DMD and DRuntime were merged, as those two repos being out of sync was usually the culprit.

Another issue driving him nuts was running `cpp` on Mac for ImportC. The problem is that `cpp` on the autotester fails on some of the Mac system header files. So he was wondering if anyone knew which C preprocessor he should be invoking on Mac. Mathias noted that the autotester's Mac version is probably ancient, but suggested he invoke clang with the switch to use the C preprocessor rather than invoking the preprocessor directly. Walter said that was a good idea.

Next, he said he had made a lot of progress on getting ImportC to compile the C files in DRuntime. It was working Phobos except for the Posix files. Every Posix platform was failing differently, but he hadn't figured it out yet.

He said that other than that, things seem to be going rather well.

## The next meeting
Given the lateness of this summary, the next meeting has already happened. It was a monthly meeting that took place on August 4 during the DConf Hackathon. I'll publish a summary of that in the next couple of days.

The next, next meeting is happening Friday, September 2, at 14:00 UTC. If you'd like to bring something to us, please let me know and I'll see about putting you on the agenda.