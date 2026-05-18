The D Language Foundation's November 2025 monthly meeting took place on Friday the 14th and lasted about an hour and thirty-five minutes. I was unable to attend, so Razvan ran the meeting and Dennis recorded it.

## The Attendees

The following people attended:

* Walter Bright
* Iain Buclaw
* Rikki Cattermole
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Steven Schveighoffer
* Mike Shah
* Nicholas Wilson

## The Summary

### A D symposium at Yale

Mike Shah opened saying he wouldn't take too much of everyone's time. His pitch was to organize a D programming language symposium hosted at Yale University.

He said Steve and Ali had recently come to Yale to give a guest talk, which had gone very well. About 50 students showed up, and there were a lot of good conversations afterward. That convinced him there was real interest.

The basic idea was to hold a two-day event, possibly in April or May, while students were still on campus. One day would be for talks, and the second day would be more of a hackathon or workshop where speakers, students, and community members could work together on something productive and fun. Yale could provide the space and automatic recording at no cost, and Mike could handle the catering. The main logistical hurdle would be finding sponsorship to help fly speakers out.

Átila thought it was great. He wasn't sure what he could do, but if sponsorship could be found, he didn't see why we wouldn't support it.

Mike said the goal was not to compete with DConf. This would be spaced a few months before, and at this stage he was mainly trying to gather interest and see whether people would consider speaking if reimbursement were available.

Robert joked that this was preaching to the choir. He thought it was great that Mike was specifically reaching out to students because the language needed the next generation.

Mike said that if the group was willing to give him the blessing to go ahead, he would start working on it. There were people in Boston who might be able to help recruit, and he could reach out to Symmetry and other companies to see if anyone was interested in sponsoring. More than anything, he thought it would be good to get some visibility for D.

Walter asked where Yale was, and Mike explained that it was in New Haven, Connecticut, about a two-hour train ride from either New York or Boston. He was thinking about late April 2026, since that would put the event late enough in the semester for his students to have had two semesters of D programming. Walter thought it was an awesome idea and was all for it.

Rikki suggested that since it would be more local to the United States, people could make a road trip of it. That led to some joking about the scale of the U.S. 

On the topic of local gatherings, Walter brought up the D Coffee Haus he had been running for the last year and a half. It was a monthly meeting that averaged six to ten people, with some regulars and a rotating set of others. He said it had been really fun. Sometimes they didn't even talk about D. They talked about things like nuclear fusion, airplanes, and investments. It reminded him of the conversations he had in college, when students would sit around talking about computers, technology, and anything else with smart, well-educated people.

He recommended that others try setting up local D meetups if they had D people nearby. He said they had even invited C++ people, some of whom liked it so much that they started their own local version.

Mike said he and Steve were close enough geographically that they should probably stop making excuses and make something happen eventually. Steve said Mike lived in Rhode Island and he lived in Massachusetts, only about 20 or 30 minutes away. Walter asked whether there were other D people in the area. Steve said Andrei lived in Boston.

Razvan was concerned that the previous year's DConf had been hard to fill with talks, and that having the Yale event only a few months before DConf might split the limited pool of people willing to give talks. His view was that DConf was for the hardcore D community, often the same faces with some new ones, while the Yale event sounded more like a popularizing event. He suggested that the Yale talks should focus more on showing how cool D is, rather than going deep into D internals or other topics that might put off a wider audience.

Mike agreed that this was something to think about. The event could have a mix of material, especially since it would be at a university. There could be more back-to-basics talks. He had enough teaching material prepared that he could hand slides to someone and tell them, for example, to talk as much as they wanted about templates. Part of his secret plan was also to invite neighboring universities and get as many people as possible to try out a new programming language. He mentioned schools in the northeast such as MIT, Harvard, and Columbia, and thought students would at least come to hear about a language they hadn't used before.

Walter asked whether Mike's class and students were at Yale. Mike said yes, he was teaching a D course there now and would be teaching another one in the spring. That meant there would be a built-in student audience. He also mentioned that one of his students was working with Nicholas and Bruce, so he could encourage them to present as well. It wasn't the goal to make people produce two new talks in one year. He was fine with people recycling DConf talks or giving shorter versions.

Walter said he had recycled talks himself, including at monthly C++ meetings, and saw no problem with doing that. He would tentatively like to attend and meet Mike's students.

Rikki said the same topic could be presented very differently depending on the audience. For a Yale-style talk, someone might present a null check or bounds check and emphasize that it could now be checked at compile time. A DConf version would be much more likely to explain how it was hooked into the compiler, how semantic analysis worked, and the theory behind it. Those were two wildly different talks for different audiences, and each audience could love the version aimed at them.

Steve asked whether Mike's students paid attention to DConf talks. Mike said he didn't think they had watched many. In the first semester, they were mostly getting everything from him in lectures. He had linked to a few keynotes and older DConf talks he thought were interesting, but the students wouldn't have seen too many. Steve thought recycling old talks would be fine, since there had been DConf talks going back to 2013.

Mike said he had another meeting, but he would be happy to join the next monthly meeting and report on progress. Razvan made one more suggestion before Mike left: if the audience was primarily students, the event could be framed more as a workshop or symposium, with talks on topics not covered in Mike's course, followed by hands-on workshops the next day. Mike liked that idea and said he would love it if speakers were willing to grab a handful of students and build something with them. He said they might market it as a D workshop, D symposium, or something like “come learn about the D language.”

(__UPDATE__: Mike [announced the D Language Symposium 2026](https://forum.dlang.org/post/mwlmmkokvjgqrhvtrupc@forum.dlang.org) in the forums in January. [It was a successful event](https://dlangsymposium.com/). He has posted [the talk videos on his YouTube channel](https://youtube.com/playlist?list=PLvv0ScY6vfd-7WWQl86RY6hL6mSO7YGCe).)

### Null check status

Rikki said the first item on his list was the status of the null check.

The feature had been approved months ago and had originally gone into OpenD. It now had [a DMD PR upstream](https://github.com/dlang/dmd/pull/22040). The good news was that most things were working, especially thanks to @limepoutine, who had managed to get function pointers and delegate/function pointers working. The PR was waiting on a review from Walter. Once Walter reviewed it, Rikki could turn the feature back on, take it out of draft, and merge it.

There was some bad news. Context pointers for delegates couldn't be checked because lazy parameters could have a null context pointer. He thought the solution there could be to disable copies of lazy parameters, but that would have to be done in an edition.

Another issue was intrinsics. Rikki said they had to be disabled entirely for null checks because their backend handling was too fragile. He didn't want to try to fix that. His proposal was that once Walter got to the point of needing intrinsics for ARM, Walter should try to fix the architecture rather than copying a bad x86 architecture over to ARM.

Walter didn't understand the problem at first and asked if Rikki meant things like `memcpy` intrinsics. Rikki clarified that he meant XMM/SIMD intrinsics. Walter said he hadn't attempted to worry about null checks with SIMD because there were no pointers in SIMD instructions. Rikki said the CI disagreed.

Walter still didn't understand, saying there were no pointer instructions in x86 SIMD instructions. Rikki said some of the intrinsics took pointers as arguments. Steve asked whether Rikki had a link to the failed CI. Rikki did not.

Rikki said what he had done was disable null checks around intrinsics to get around the error. Walter continued to say he didn't understand how null checking a pointer to a SIMD location would be any different from anything else. Rikki said Walter was thinking about values like `float4`, but there were more cases than that, including pointers to data and arrays. All he knew was that trying to do the null check caused failures, and he wasn't going to debug it further.

Átila said this would be very hard to resolve by talking about it without an example written down. Rikki said he would [link the code where he disabled the checks](https://github.com/dlang/dmd/pull/22040/changes#diff-54719c55eec4a6cfe3b7b7f4b5289d0b5a0b9d41e8589a023bba4d68921f8be0R5525) and then pass it to Walter, who could remove that code and investigate later. Steve clarified that disabling the code was just to get around the CI error. If the code were put back, the CI should reproduce the error and show what was going wrong. Rikki agreed, but wanted to get the feature reviewed and merged first, then worry about the backend issue later.

Walter still didn't understand the problem. Átila suggested this was probably better handled asynchronously. Rikki said the thing to do was to disable the workaround once the PR was merged and see what actually failed in the backend. Razvan suggested that Rikki provide a written example and perhaps start an email thread with Walter, since they weren't making progress in the meeting. Rikki agreed to provide the links and instructions to comment out the relevant block of code.

(__UPDATE__: The PR has since been merged.)

### macOS support

Rikki next moved to the status of macOS support. He had bumped macOS support from 13 to 14 with some help from Iain, and that had worked out. [Moving from 14 to 15](https://github.com/dlang/dmd/issues/21987), however, had not worked. He was using the macOS 15 Intel image and had concluded that DMD did not support macOS 15. He said it was basically DOA at the moment. He had tried with an LDC bootstrap, which got a bit further, but the CI still failed. Even with all the fixes on master, it didn't work.

Walter asked whether the problem was DMD generating x86-64 code that didn't work. Rikki said yes. Walter asked why. Rikki said he didn't have a MacBook to test on, so he couldn't debug it.

Steve said it was hard to find an Intel Mac that supported macOS 15. It had to be a very late Intel Mac, and most people had already switched to Apple Silicon. His own Intel Macs were too old to update. Walter asked whether newer Macs were no longer emulating x86. Rikki said the GitHub image was called `macos-15-x86`, but it was running on an M1 according to GitHub's documentation. It definitely should have been working.

Walter said x86 apps on Apple Silicon required Rosetta. Rikki said the setup did work, but not enough. It was definitely set up for x86. Steve said his own experience running x86 programs such as dub and DMD on an M1 was sketchy. The system would sometimes decide that a universal binary was in ARM mode and try to link ARM code with x86 code. He had mostly switched away from DMD on his Mac and used LDC instead.

Walter said that from a larger point of view, he had been generating programs with DMD that ran on his ARM Mac mini. He wasn't going to worry about DMD failing to generate x86 code for Mac because he didn't see much point in struggling with it. Apple didn't care about x86 anymore, and fighting with that was a waste of time. He would keep working on the ARM-generating version of DMD.

Rikki said that sounded like dropping x86 support for macOS with DMD, because as of macOS 15 it was dead. Walter said he understood that. Steve pointed out that there were still people using macOS 15 on x86 systems who used D, because he had people complaining in some of his projects about it not working. He usually told them to use LDC.

Walter asked whether no D programs worked, or whether there was simply no release. Steve said there was no release version of DMD that worked on macOS 15 because of a segfault issue. That issue involved Apple changing something internal, related to thread-local storage, where the system was using bits it hadn't used before. The way D was zeroing those bits caused segfaults. That had been fixed in master, but not released yet.

The discussion then got tangled because there were two issues being discussed. Steve reiterated that the issue he'd brought up had affected DMD and LDC when macOS 15 was released and had been fixed in master. There had not been a DMD release since then, so the current released DMD did not work. What Rikki was talking about was different. Even if the compiler and library could be built from master, the macOS 15 CI still didn't pass well enough to use that image for release testing.

Rikki said the one idea they had was that x87 appeared to have gone away on macOS 15, but not macOS 14, even on the same hardware. Someone with macOS 15 would need to test it. He had pinged Luna, who had a relevant machine, but she wasn't able to help much at the moment.

Steve tried to clarify the CI situation. GitHub had been running tests on macOS 13. They had bumped to 14. If they tried to bump to 15, CI failed. So they could build on macOS 14 and possibly run on macOS 15, but they could not build and test on the macOS 15 GitHub image. Rikki added that they didn't actually know whether a compiler built on 14 and moved to 15 would pass all tests. Some cases might work, and others might not.

Walter asked whether anyone had a macOS 15 machine they could use to test DMD and see why an executable was failing. Átila said he definitely did not. Robert was on macOS 14. Steve was on macOS 26 on an M4 Mac, but not on an x86 system. Rikki explained that the image in question was an Intel image running on M1, with x86 system libraries. Steve said if the issue was only running an x86 image emulated on an M1, then he didn't think we needed to care about it. Rikki cautioned that it might also apply to regular Intel macOS 15 systems.

Walter asked whether the compiler crashed, the linker failed, or the generated code crashed. Rikki wasn't sure anymore because it had been a while since he investigated. He offered to create a new PR bumping to macOS 15 Intel and ping Walter on it so Walter could see exactly what was failing. Walter said he would at least like to know whether it was something trivial, such as a wrong switch or the linker trying to combine x86 and ARM code.

Steve found what he thought was [the last time macOS 15 had been enabled in a PR](https://github.com/dlang/dmd/pull/21985). Walter saw an undefined identifier error and said that was something that could be found. Rikki thought that particular issue involved an incompatibility between the backend and something in the frontend where x87 wasn't supported, and the configuration between the frontend and backend was mismatched. He had filed a bug report for that, but he wasn't sure whether it had been using macOS 15 or macOS 15 Intel. The distinction mattered.

Walter was frustrated by the shifting explanations. Rikki said the issue Steve had linked was one problem associated with macOS 15, but not necessarily the same issue he had originally meant. Steve agreed that they now knew that particular log was not the issue Rikki had been talking about. The next step was for Rikki to create a new PR with the macOS 15 bump so they could get current logs.

Átila said that in the future, this kind of thing should probably be handled asynchronously first and then discussed in the meeting. It wasn't efficient to try to diagnose this in real time without links to CI failures. Walter again asked whether anyone had a macOS 15 Mac. When the answer was no, he said we were dead in the water because no one had the machine. Rikki said all he could do was report that he had run into the problem.

Rikki mentioned that his own Mac was from 2014, and joked that we could use his Macintosh Classic if we wanted, but that was Mac 68k machine and he hadn't booted it yet. Steve said Luna had an actual Intel MacBook Pro with macOS 15, but again, she was not in a position to work on it at the moment. Átila said one of the users with such a machine would have to debug it. Steve thought the first step was still to reproduce the CI failure. Rikki said if the failure involved something like missing x87 instructions, we would need a debugger on a real machine. Átila reiterated that it wasn't useful to talk about what might be happening without either a machine or CI results. Steve agreed that step one should be to make the failure visible.

Rikki said he would create the bump PR, ping Walter, and then probably leave it to Walter, because that was as far as he could go.

### Bundles

Rikki moved on to bundles. Robert asked him to explain what he meant by "bundles", because Robert had several different concepts in mind.

Rikki said he would get to that, but wanted to begin with the motivation. For starters, at least for DMD, DRuntime had to be split from Phobos in terms of the binary. We couldn't keep putting both into the same binary because of symbol limits in some cases. We were also introducing BaseD, which had to live somewhere and would increase the symbol count. It would need to exist as object files, not just as part of DRuntime. Phobos v3 would also not be a single binary. It would have a set of shared libraries built on top of it for things such as coroutines, web servers, and other functionality. The current situation with compiler switches and how D shipped things wasn't good enough for that.

Átila asked what problem he was trying to solve. Rikki said the issue was linking and imports: how to link against BaseD, how to link against Phobos v3, and how to handle the fact that these pieces would be separate binaries. Walter said we probably needed Adam Wilson to talk about that. Rikki disagreed, saying Adam hadn't been thinking about it. Walter said Phobos v3 didn't exist yet. Átila agreed and said he still didn't understand what problem Rikki was trying to fix right now.

Rikki said Phobos v3 couldn't exist if we couldn't link against it. Átila responded that on November 14, 2025, nobody was trying to link to it. There was nothing to link to yet. Rikki reiterated that it couldn't exist without solving this first. Átila said that when we got to releasing it, they could solve the problem if we had one, but Adam wasn't even present and he didn't understand why we were talking about it now.

Rikki agreed to move on. Átila quoted himself, saying, "Never do tomorrow what can be left until the day after tomorrow, because by then you might not have to do it anyway." Rikki said this was definitely a case where we would have to do something.

Robert said he didn't know that yet. He noted that Rikki hadn't run a shell script or otherwise tried to build and link Phobos v3 to show that the linking failed. Unless the problem had materialized, there was no measured problem. Rikki might be right that there would be something to fix, but there might also not be. Robert said Rikki was trying to build a bridge when there was no water to build a bridge over. Walter added that it seemed like trying to build a launchpad for a rocket without knowing what the rocket would be.

### Class monitors and `synchronized`

Rikki's next item was the class monitor. He said the current situation was dire in terms of unsafe casts and hard-coding between the compiler and the runtime. He said Steve knew about it because they had been trying to figure out a way around it for months.

One reason this mattered was custom root classes. When D eventually switched to safe by default, he didn't think `Object` was going to come along in the same way it existed now. The monitor was one of the first steps. Currently, it was special. It hooked into the compiler, the compiler synchronized on it and initialized it, and the relationship between the compiler and DRuntime was hard-coded. Rikki thought we could move the monitor to be a field of the root class, simplifying the compiler somewhat, though it would change the class layout.

We could also loosen things so that the monitor worked with structs, pointers to structs, classes, and more than one interface. We could provide static methods for monitor entry, monitor exit, and perhaps destruction. That would allow custom monitors rather than being hard-coded to `_d_monitor`. If it worked, we could even have monitors in C++ classes, and eventually in a custom root class.

Átila asked why we would want a monitor in a C++ class. Rikki said it would verify that the approach worked. Átila asked who was asking for this. Rikki said if we wanted custom root classes, we had to solve it. Átila said that was begging the question. Walter said D already allowed C++ classes that didn't have object monitors. Rikki said yes, current C++ classes didn't have them. The proposal would allow them to have one and would also allow custom root classes to have them. Walter said COM support in D already allowed classes without monitors. He didn't get what problem was being solved.

Steve said the problems he had with the monitor were that it wasn't just the synchronization primitive. Other things were stuck in there too, including signals and slots, and it was all hidden under the hood with allocations and odd behavior. It seemed like there should just be a language hook for `synchronized`. There was no reason the monitor had to be a special part of the class. He didn't know the right solution, but when writing the new GC, one thing he still didn't support was synchronizing on an object. If an object was synchronized on, it gained a monitor and also gained a destructor if it didn't have one before. His current GC had no way to do that.

Átila said his proposal would be to remove the monitor and also remove `synchronized` from the language in a future edition. Steve said that would work, too. Rikki said `synchronized` definitely needed to stay. Átila disagreed.

Walter said the monitor came from Java. It was there so the `synchronized` keyword could work. He thought signals and slots had been a technique that was hot 20 years ago but no longer was. Steve said he had at least one bug report against his GC about signals and slots not working, so at least one person still used them.

Walter asked whether `synchronized` was an obsolete feature because it wasn't how modern synchronization worked anymore. Átila said he agreed 100%. Rikki said people did use locks through `synchronized`, and that it was a valid way to write such code. Steve said having a scoped locked area was not horrible. Átila said there were ways to do that without `synchronized`.

Rikki argued that what `synchronized` provided was compiler understanding that a block was synchronized and therefore couldn't be yielded within. He would rather the language make that safe than have people work around it with something like `core.sync.mutex`, accidentally yield while holding a lock, and end up with a deadlock they couldn't figure out.

Átila again suggested removing `synchronized` and the monitor in a future edition. Rikki asked for a counterproposal. How should we tell the compiler that a block was synchronized? Átila didn't know if that was useful and asked Rikki to send code showing what he wanted to do. Steve said all of the GC used locks without `synchronized`, and the compiler never complained about yielding. He wasn't sure what Rikki meant.

Rikki explained that he was talking about stackless coroutines. He had added to the DIP that if a `synchronized` statement existed, yielding inside it would not be allowed. The concern was that people would write synchronized statements, then yield inside them and create deadlocks.

Átila said this was not the best way to talk about blocks of code without looking at the code or the proposal. Steve asked whether other languages had this restriction. Rikki suggested checking C# quickly. Walter said Adam was an expert in C# and should probably be part of the discussion. Átila again said he wanted to see code before forming an opinion.

Rikki described the scenario. Inside a coroutine, you synchronize on something, do work that requires the lock, and then accidentally yield. You cannot hold the lock across the yield. Átila asked why the coroutine was holding a lock anyway. Steve said he didn't see how this was fundamentally different from a thread holding a lock and being put to sleep by the kernel. Rikki said that was exactly the point. He wanted the language to model and prevent it.

Robert said he didn't understand the actual problem. His understanding was that if a coroutine or thread entered a synchronized block and then got interrupted, when it woke up again, it would still have the lock. He considered that the correct semantics. Rikki said that was right, except for the part where it might never wake up. Robert said that sounded like solving deadlocks in the wrong way. Taking resources away from other threads or coroutines was not how people normally avoided deadlocks, except in a few academic languages that had failed horribly. If that was the proposed direction, he would strongly oppose it.

Rikki clarified that he was not proposing taking the lock away. The problem was that if one coroutine went to sleep while holding a lock, another coroutine might try to acquire that lock, fail, and prevent the original coroutine from ever waking up. Steve pointed out that internally, when entering a synchronized block, a coroutine would have to sleep until the lock was available anyway. Rikki said some languages and runtimes allowed yielding while waiting for a lock, but people had found that it caused deadlocks.

Steve said deadlocks would happen if synchronized code wasn't written correctly, and that was a fact of life. Átila mentioned Pony as a language without deadlocks because it didn't have locks, though it had a complicated type system to support that model.

Walter said yielding inside a synchronized block screamed bug to him. Rikki agreed and said it shouldn't be allowed. Walter said the language did not know what a yield was because D didn't have a yield feature. Átila noted that with stackless coroutines, the compiler would know. Walter said stackless coroutines were a different synchronization issue from `synchronized` on a monitor. They were just incompatible with each other.

Átila summarized the oddness of the discussion. They were talking about a feature that didn't exist yet, that might cause problems with `synchronized`, and that somehow related to the class monitor. He again proposed removing the monitor and `synchronized` in an edition. Robert said he would put his name on that. Walter said he thought `synchronized` was obsolete. Steve thought they would get pushback and probably hear good reasons why people used it. Walter said that was reasonable. Every time he tried to eliminate something, somebody had built their entire store around it.

The rough outcome was that we should look at removing `synchronized`, and Rikki should provide concrete examples or precedents from other languages if he wanted to argue for retaining it or enforcing restrictions around coroutine yields.

### DIP 1000

Átila brought up something he'd been investigating with DIP 1000. He'd looked again at his idea of making `scope` explicit and tried compiling the runtime with it. After about ten edits adding `scope` everywhere, he realized the approach was not going to work if the runtime itself required that much annotation, let alone every D project in existence.

He took a step back and returned to the original issue that had made him question DIP 1000: a vibe.d pull request that had required adding `scope` even though the code wasn't doing anything bad. With a modern DMD, he could no longer reproduce that issue. He wasn't sure what had changed in the DIP 1000 checks, but it no longer seemed like such a big deal.

He then tested how many dub projects failed to build with DIP 1000 turned on. There were about 2,600 dub projects. Of those, 1,720 built with a plain `dub build`, excluding abandoned projects that didn't build with current DMD anyway. He had to use DMD 2.110 instead of 2.111 because 2.111 compiled for 40 minutes straight on two projects, and he filed an issue for that. Out of the 1,720 projects that built normally, only 84 failed with DIP 1000 turned on. That was a tiny number.

He wondered whether we should revisit enabling DIP 1000 deprecation messages and eventually turning it on by default.

Walter said that was good news. He didn't think we should remove DIP 1000 as an option because he still had hope for it. His `@live` analysis relied on DIP 1000 because without `scope`, it couldn't tell what happened to pointers passed to other functions.

Átila said the original concern had been that transitioning would be too much work for people. But now, only about 4.9% of the projects on code.dlang.org that already built failed when DIP 1000 was turned on. Robert said vibe serialization was an important project, but if there were only about 84 failures, we could look at how easy they were to fix. The failing projects could also act as a guide for where DIP 1000 itself still needed work.

Átila said 84 was not completely insane to investigate or submit PRs for. Robert said this basically proved Átila's earlier point that sometimes not doing something for a while solved the problem. What had once looked like a large transition might now be a small one.

Walter speculated that people might already have modified their code to be DIP 1000-compatible. A large part of the confusion around DIP 1000 involved the `scope` status of the `this` reference in member functions. One improvement he wanted to make in D was to make the `this` parameter explicit. If it were explicit, it would be much clearer where `scope` and `return` belonged, instead of making people guess. He thought that would clear up about 70% of the confusion with DIP 1000.

Átila agreed, saying that Python had `self`, but Rust was a better comparison because it had attributes on `self`. Walter agreed and said D should do something like that. Steve didn't like the idea of not having an implicit `this` pointer. He found it reasonable that attributes applied to `this` rather than to some explicit parameter. Walter said copying C++ there had been a bad idea. Dennis thanked him. Átila said implicit `this` was fine if you didn't have attributes or storage classes, but not when you did.

Walter said making `this` explicit could also eliminate the need for the `static` keyword on member functions. If a member function didn't take an explicit `this`, then it would effectively be static. Steve pointed out that this would affect existing code that assumed an implicit `this` parameter without the `static` keyword. Átila said that would have to be handled in an edition. Walter said they should implement it fairly soon as an option, giving people a long adaptation period before even thinking about making it required.

Returning to DIP 1000, Átila asked whether anyone objected to him looking at the failing projects one by one, maybe submitting PRs, and then turning on deprecation messages again. In the past, it had been a massive pain, but now it didn't look that way anymore, probably due to a combination of DMD changes and projects adapting.

Walter thought that would be impractical until there was explicit `this` syntax. Robert suggested that before Átila started working through the list, he should send an email with the list of failing projects. Some might belong to people in the meeting, and they might be able to fix their own projects.

Átila said he had used Codex to help with this work. He already had code to pull packages from dub and build them, then had Codex help write an app that took the list, accepted DMD arguments, and ran the dub builds again. He said he didn't write one line of that himself, though he did have to tell it multiple times what he wanted. Walter joked that he loved it when people said something “just works.” Átila said he wasn't insane. He had tried it first on three projects. Two passed with DIP 1000 on, and vibe serialization failed, exactly as expected, so he was confident enough to run it across the full set.

Walter complimented Dennis for doing a lot of work to make DIP 1000 work better and said the progress Átila was seeing might be a direct result of Dennis's work. Átila agreed that was probably true. He said he'd been confused when he could no longer reproduce the old problem, even after copying more context from the original issue. Dennis said scope inference had gotten better, so something that used to fail might work now.

### The bootstrap compiler

Walter said he couldn't use DIP 1000 and other newer features in DMD because of the bootstrap compiler. Walter asked whether there was any hope of moving to a more modern DMD as the bootstrap compiler.

Dennis asked why he couldn't use DIP 1000, since even the bootstrap compiler supported `scope` and `return` keywords, and there were already some scope annotations in DMD. There was also a CI check that built DMD with DIP 1000.

Walter said this wasn't only about DIP 1000. Every time he wanted to use a new D feature in the DMD source code, it failed because the bootstrap compiler didn't support it. It was frustrating to create shiny new features and then not be able to use them in the compiler.

Dennis asked whether Walter was thinking of anything besides bitfields. He knew Walter had talked at length about those, but aside from bitfields, Dennis didn't often feel the need to use fancy features in the compiler. He preferred the compiler code to stay relatively simple rather than using lots of templates, mixins, and fancy constructs. Walter didn't regard bitfields as fancy, but he couldn't immediately think of another specific feature.

Steve mentioned shortened methods and named parameters as examples of newer features he used.

### DMD on macOS ARM64

Walter said the macOS ARM64 support was taking a stupid amount of time, far more than he had anticipated, and he found that extremely frustrating. He said it was his own fault.

Dennis said he had seen Walter working a lot on varargs, which was a pretty niche thing, and asked whether the more basic things were working now. Walter said the basic stuff was working. The problem with varargs was that Apple had created its own version that it did not document. He kept running into crashes, then having to figure out that Apple was doing something different from the standard.

Exception handling was another area he hadn't yet attempted. Debug info and stack unwinding were also problems because Apple had gone its own way again and invented another undocumented approach to stack unwinding information. All of that was making it hard for him to compile DRuntime with it.

He didn't know how much time he should keep spending on it, or whether he should set the project aside for a bit and work on other DMD features, such as the explicit `this` argument for member functions. He said he should come up with a DIP for that and that it shouldn't be hard to implement. Dennis thought it sounded mostly like a parser change.

## Conclusion

The next monthly meeting was scheduled for December 12th. If you anything you'd like to bring to us in a monthly meeting, please let me know.
