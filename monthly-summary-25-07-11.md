The D Language Foundation’s July 2025 monthly meeting took place on Friday the 11th and lasted about an hour. Walter was unable to attend.

## The Attendees

The following people attended:

* Rikki Cattermole
* Jonathan M. Davis
* Timon Gehr
* Dennis Korpel
* Átila Neves
* Razvan Nitu
* Vladimir Panteleev
* Mike Parker
* Steven Schveighoffer
* Nicholas Wilson

## The Summary

### New release process

Dennis said the situation with master vs. stable branches creates friction in several ways. There were constant PR debates over which branch to target. Rebasing was kind of tricky, so it often went wrong and made history messy. It also meant that once we commited to merging master into stable, we couldn't make anymore patch releases. For example, once he tagged 2.112, he couldn't go back and tag 2.111.1.

He had looked into how other open source projects handled it and decided on a proposal for an alternative branching system where all contributions would just go into master. Maintainers or GitHub Actions could then backport into release branches based on labels. So we'd create a branch for 2.112, then every fix that needed to go there after the feature freeze could be backported from master.

This wouldn't automatically prevent merge conflicts, but would make it less likely that stable and master would diverge if, for example, we forgot to merge stable back into master.

He had put all the details in a document and wanted to get feedback in the forums, but first wanted to check with us to see if there was anything else he needed to account for. This was going to affect a bunch of things and he didn't want to miss anything.

Rikki had two suggestions. First, we should reserve point releases for LTS versions only. Second, we should start squashing commits by default. Dennis noted we already had the option to squash commits. Rikki said there was a setting to make it the default. Dennis had no problem changing that.

Jonathan asked what the specific benefit of making it the default would be. Rikki said it was mostly for convenience. When people had multiple commits, the reviewer didn't have to worry about squashing them.

Nicholas said he did that already anyway. But if the PR contained orthoganol sets of stuff, he'd leave the commits as-is and do a rebase. It didn't bother him either way, but he suggested checking with Martin and Iain.

Dennis then raised the question of cadence. His proposal suggested a minor release every three months and an LTS release every year. The LTS would be restricted to critical fixes, like when macOS broke everything, or wrong code fixes. There had been some requests for an LTS, and though he thought all the betas and release candidates we had were nice, very few people actually tested them. All the regression reports tended to come in after the release was pushed to the Linux packages. We always released and then saw the consequences.

Timon thought we should keep in mind all the heavy refactoring going on over the past year or so. That would impact this negatively if it continued at that pace, so maybe there should be some coordination there.

Dennis said that was why he was talking about limiting LTS to critical fixes. Maintaining all the stable bug fixes would be too much work. In certain cases, like if an industry user wanted a specific non-critical bug fix, but didn't want to go through the upgrade process, then we could add it to the LTS. Rikki agreed.

Dennis had hoped Martin or Iain could weigh in durin the meeting, but it was a holiday week for them. He said he’d post the proposal to the forums to get more feedback.

__UPDATE__: He posted [his request for feedback](https://forum.dlang.org/post/ckfscuzcbkqfqxrdaicg@forum.dlang.org) on July 15th.

### zlib dependency problems

Steve said he'd been working on the Rayib D binding. As part of that, he maintained an install script that unpacked the correct prebuilt Raylib library so users didn't have to guess what to download. He'd gotten a few reports from people trying it out saying the install script didn't work. Usually, it was a link error against libz. We were missing the `inflateEnd` symbol or something like that. This always puzzled him because we had libz compiled into Phobos.

Luna, who was doing some work on macOS, gave him a binary to try for something she was working on and it had a missing libz dependency. She told him she had to have a special libz dependency because of the way the compiler worked, but he thought that didn't make a lot of sense.

He'd also learned that Vladimir's ae library had an extra libz dependency if you included the compresison stuff.

The source of this problem was that Linux systems had a policy of stripping out libz dependencies from Phobos becaue they already had a libz, and you should be depending on that instead. As such, they'd been mucking with the build so that it didn't include the libz C code that we compiled and instead linked against the system libz library.

This was okay as long as you were using Phobos symbols that dependend on libz. With dynamic linking, you depended on Phobos, Phobos depended on libz, and it just worked. But if you used any of those zlib C functions directly, which he was doing in his install script via iopipe, then it wouldn't link. You ended up with the situation that if you installed the blessed compiler from the operating system, it didn't build code that normally built with our compiler releases.

He wanted to try to figure out what to do about it. His first thought was that we should make it standard practice to link against libz, since Phobos linked against it anyway. We should also consider removing it completely from Phobos v3. It had been there since D1, and it was the only C package we included like that. We had `std.net.curl`, but curl was loaded dynamically.

He brought this up because it had been bothering him for two years and he'd had no idea where people were finding a compiler that didn't have the proper build. It turned out it was just `apt install ldc`.

Átila agreed in principle that we should be linking with whatever was on the system. Steve asked if we should be telling these package maintainers to just add `-lz` into the compiler configuration? He thought that was the right answer. Átila wondered what Martin and Walter would say about it, but said it all made sense in principle.

Jonathan said that if there was no technical reason why we needed to include it in Phobos instead of linking externally, then we should just switch to using `-lz`. I noted that we couldn't expect it to be installed on Windows.

Rikki said that on POSIX, we definitely should not be shipping our own copy of zlib. On Windows, the situation was different. Jonathan thought that might be why Walter had included it, since he'd been working on Windows initially.

Steve said he’d see what he could do on the packaging side. Nicholas shuggested that Adam Wilson should be looped in on this for the Phobos v3 stuff.

### Vladimir’s industry feedback

Vladimir had been unable to join us for the quarterly meeting the week before, so he said he effectively had “two meetings worth” of feedback for us.

He described his current work as building a CRM-like system, including writing a GraphQL client library. That was a lot of business logic: rules, reporting, numbers, and all that stuff.

__The good__

He gave named arguments a huge thumbs up, saying they were a major boon for readability in business logic for both structs and functions. It replaced the builder pattern; that was gone. It replaced the flag template from `std.typecons`; that was gone. It was very nice, very readable. It was a huge thing he wished he could have had sooner, but he was glad it was there now.

He was thankful for the work to keep compilation times low, and especially that we had better tooling now. Using `-ftimetrace` on DMD was very cool. He’d used it on a few occasions to track down the root cause of some issues, including one dealing with a GraphQL-generated D file that was a megabyte in size. They'd run into some problems, but he could narrow them down so the compilation times were still very reasonable.

He'd seen some issues with error message locations. They may seem trivial, like the caret being one character to the left of where it should be, but they were very helpful and integrated into the editor. So if you were making a big refactoring, you could just jump to the next error and fix it, jump to the next and fix it. He was thankful those had been getting fixed.

Finally, he said he’d been using dub, and though historically he hadn’t been a dub user, he could see visible progress and found it much less annoying than it was years ago. He was thankful for that.

__The bad__

About once a month, he hit an issue that completely blocked him. Sometimes it was an ICE, sometimes a linker error. In a large application that was generating a lot of code, even understanding the problem could be time-consuming. He said it usually took him a day to either reduce it or even understand the cause and find a workaround.

He said he used to try reporting these bugs. He'd even created Dustmite to make reporting them more efficient. But experience had shown that if these bugs were easy to fix, they likely would have been fixed, so he expected they weren't easy to fix.

Producing a clean, reproducible test case, especially for a project that wasn't open source, was a serious time investment. The cost/benefit just didn't work out when he had to justify the time to the people paying for it, so he hadn't been doing it every time.

He noted that his team had switched to using LDC exclusively because that was what was used in production. He wasn't using DMD at all. One problem was that DMD's memory usage was higher than LDC's, so it ran out of memory compiling a program where LDC didn't. Second, there were some DMD-specific ICEs in its backend that he assumed didn't exist in LDC.

He'd had trouble with recursive attribute inference and assumed it was a familiar problem to the people in the meeting. (__NOTE__: It was. Martin has brought that up on several occasions.) Things either didn't compile or didn't link because the mangling was different. These were a big pain to deal with.

He'd been able to deal with what he'd found so far, but there was also the effect that he was being asked how he spent his time. So when he occasionally reported that he'd spent a day working around a problem that came from the D compiler, he'd get questions about why they were using D when developers in the same organziation working with a different language didn't run into stability issues.

Of course, the time saved from using D was less visible and less quantifiable. But these problems that he occasionally ran into were seen as a risk. So the sentiment about using D had been lukewarm in his corner of the organization. There was some movement toward considering other languages, TypeScript in particular since they were already using it on the front end.

__Exception types__

Vladimir had a wish list item, which he explicitly noted Walter would disagree with were he present.

Typically, when we talked about unexpected failures, like assertion failures, the argument was generally that the program had entered an invalid state and could not continue. He could agree with that for the typical definition of the program, but--

Rikki interrupted to say that this had been his first agenda item, but we'd put it aside given that Walter was absent. Vladimir said he'd keep it short in that case.

Essentially, he would like to have an error boundary so that we could somehow just assume that the program didn't have a global state. However, there were tasks that had their own state, and it was okay to terminate those tasks. So we could have an error boundary say, okay, an assertion was thrown, now throw away that fiber or whatever. Then we return an error page which we assume will work even if an assertion has been thrown.

There were only very few actually unrecoverable errors when looking at it from this perspective. And if the GC's state was corrupted, we had an optional exception for one and another exception type for the other.

Átila didn't think that was true. If an assertion failed, you didn't actually know what happened. It could be anything, from the things Vladimir had just said to cosmic rays through your hard drive.

Vladimir agreed, and said that was a problem. Without further information, he would just like to be able to assume that it was something local and could be handled. Átila said it couldn't be handled by definition.

Vladimir argued that definition wasn’t useful and should be changed. C#, Java, and pretty much every modern language could do it. D could not. Jonathan noted that bounds-checking errors were in the same boat.

He said there were people who argued that unwinding was just for your ability to report and you weren't going to try to recover, and people who argued that it was isolated enough at least some of the time that it was worth attempting to recover even if youd didn't necessarily want to do so in all cases.

Ultimately, if the errors did all the unwinding and cleanup correctly, the developer could choose to how to handle it. Right now, we had this weird halfway thing where Walter and several others argued it should just be killed. But we had this whole stack unwinding mechanism in place, which was just dumb if we weren't going to unwind properly. There had been arguments for making it configurable.

Rikki said it got worse. In 2018, the unwinding logic was changed in a way that prevented all finally statements from running all cleanup. It was an optimization the front end shouldn't be doing. He was going to recommend reverting that and making Walter's approach opt-in.

Átila said one of the big problems with that was that asserts were used by default in unit tests. How could you write a framework without catching `AssertError`? He thought out of bounds errors were a bad example, as you should probably always crash immediately in that case.

Rikki said those weren't the hard cases. The hard case was contracts. Átila agreed, saying they could be turned off.

Jonathan said that was the same with assertions in general. The biggest problem with contracts was inheritance because of all the muck it did to expand or tighten contracts. The language was forced to catch those assertions and then potentially not stop the program because, oh, this one was relaxed so we weren't supposed to skip it. So you couldn't really skip cleanup with `AssertErrors` at a minimum regardless of the rest.

He said we could make it configurable. Átila suggested we could make asserts and contracts not use `AssertError`. Jonathan said stuff relied on it working that way. Rikki said the problem wasn't that they threw `AssertError`, it was that they threw `Error`.

Átila's point was that he saw no reason we couldn't change them to use `Exception` instead of `Error`. Rikki said that as long as it was throwing `Error`, we weren't going to fix it without breaking the language, and that was worse than breaking user code. I noted that it couldn't be fixed without convincing Walter anyway.

Átila thought there was a conflation there, with asserts throwing `AssertError`. Normally, if you asserted something, it should never happen. If it did, then it should crash now because who knew how you got to that state. The problem was that we were also using them for unit tests and contracts. Rikki agreed and said we were over using asserts.

Timon said that an out of bounds error could just be a small off-by-one in a non-critical component. Getting an out-of-bounds error meant it had been caught. You could always argue that maybe the process had been compromised, but then maybe the assertion only passed because the process had been compromised. There was no information either way.

Átila asked how you'd be getting off-by-one errors in the language. How would you get there? Timon said maybe an intern wrote the non-ciritcal component. It was given to the intern because it was non-critical. And now it's taken down production. You could argue that all the requests should just run in a separate process on CGI. But he assumed that for some people, that wasn't the most performant option.

Other languages allowed you to do this. So basically you had the option of not using D, then you could get good performance without risking production going down for some stupid issue. Or you could use D and then you had to choose between isolating every request in its own process and risking that production would go down. And when people asked why, you'd have to say it was because you used D.

Átila thought the bounds thing was important. The most likely thing that happened was that the process was compromised. Again, how did you get there?

Nicholas noted that unit tests used a different assert than the regular assert, so we could configure that.

Jonathan said there were plenty of cases where he'd explicitly thrown `AssertError` all over the place because that was what you had to do as soon as you created a helper. Changing that so it wouldn't use `AssertError` would break things.

Rikki said that with assert, it shouldn't matter which function it was. It could be in a unit test or not. It was still going to work with the unit test runner as far as he was aware. It was all configurable, but it was just an assert. There shouldn't be any super codegen for unit tests.

Átila said he could argue that asserts in a unit test block should throw a `UnitTestException` and asserts in a contract should throw a `ContractException` instead. We knew so much about where exactly these things were, so should we overload assert? He didn't know.

At that point, I stepped in and said we weren’t going to resolve it in the remaining time, and suggested we schedule a dedicated meeting rather than trying to cram it into a monthly. We settled on Friday, July 25th, and I said I’d mark the calendar and add Vladimir to the invite list.

__UPDATE__: In the subsequent meeting, Walter approved the addition of a compiler switch to configure whether a thrown `Error` gets cleaned up in a finally block.

__Null pointer exceptions__

Vladimir said it would be nice if we had software null pointer checking that threw a normal exception instead of segfaulting. It would be okay if it were only in safe code. Null pointer exceptins were a solved problem in other languages and they were a pain in the behind to deal with in D.

Átila asked why they were a pain. Vladmir said you couldn't easily distinguish between a segfault from a null pointer or some kind of crazy memory corruption. You wanted to be able to catch one and let the other crash.

Átila wanted both to crash. And when that happened, which was rare, he just fired up gdb on the last core dump. Vladimir said that gdb didn't run in containers in production. Átila understood.

Rikki said he'd like some opt-in read barriers for null deref checks and throw an error there, or have a handler in DRuntime, whatever. This was prompted by segfaults he was getting for DMD in the CI that he couldn't debug.

Átila noted that would have performance problems. Running code on every deref woult be expensive. Rikki said it was no different than a bounds check.

Jonathan said we'd have to test it. OpenD had apparently implemented this, but he didn't know what kind of performance hit they'd seen. The only way to know exactly what would happen would be to implement it.

Rikki reiterated that he was suggesting this as opt-in. The point was to be able to turn it on for DMD in the CI, and users could turn it on for their stuff when they wanted.

Nicholas said that `etc.linux.memoryerror` could catch null dererences. Vladimir said it didn't work on LDC. Nicholas was surprised and said we should bring this up with Martin.

Dennis noted that `memoryerror` was Linux-only. He said there was also an alternative assertion handler which worked with both null dereference and stack overflow, but it wasn't catchable; it would print a stack trace and abort. It wasn't enabled by default because of limitations related to fibers, threads, and debug.

We talked a little about what the implementation of the null checks might look like. Then Timon pointed out that one risk of running optional options in CI that weren't running by default was that you might not catch an issue that only happened if the flag was off. Rikki had no concerns about it as long as it was easy to turn on temporarily while debugging.

## Memset hooks

Rikki said the memset hooks were not templated, and LDC had more of them than DMD, with the result that BetterC was broken.

Nicholas said he could look into that. He asked why LDC was doing that instead of using LLVM intrinsics. Rikki said they were type-specific memsets, usually for initializtion, and he wasn’t sure why.

Nicholas said he remembered seeing them somewhere. We could definitely template them. 

Rikki added that the bug reports were for both LDC and DMD, but he didn't have a link. It was something that had come up in Discord.

Nicholas suggested it might be something for the GSOC student who was working on the templated runtime hooks project. Razvan said he would pass it on.

### Getting feedback from industry users

Rikki thought we should have some way to get feedback from industry users regarding language and compiler changes. Not all of them paid attention to the forums, GitHub discussions, etc.

Átila noted that we had quarterly meetings with industry reps for exactly that reason. Anyone who wanted to bring us feedback could attend. Rikki asked if we talked about planned features there. Átila said we sometimes did.

The discussion went around for a while, talking about the various ways we could get feedback from people who didn't pay attention to any of our normal methods of communication, and whether a heavily moderated 'Production' forum or mailing list would be helpful. Ultimately, Jonathan said the root of the issue was that anyone who wasn't paying attention to the existing sources of communication was unlikely to pay attention to anything new we came up with.

Átila noted how sometimes people would show up at DConf or in the forums and tell us they'd been using D in their projects for years and we'd never heard of them.

### Conclusion

Our next monthly meeting was on August 8th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
