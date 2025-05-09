[Forum Post](https://forum.dlang.org/post/jzqabfjmdwveeatpdklj@forum.dlang.org)

The D Language Foundation's October 2022 meeting was a quarterly, meaning that several industry representatives attended. It took place via Jitsi Meet on October 7, 2022, at 14:00 UTC. The following people attended (those with DLF next to their names are either D Language Foundation board members, paid employees, or affiliated volunteers):

* Andrei Alexandrescu
* Mathis Beer (Funkwerk)
* Walter Bright (DLF)
* Iain Buclaw (DLF/GDC)
* Ali Çehreli (DLF/Mercedes Benz R & D North America)
* Max Haughton (DLF/Symmetry)
* Martin Kinkelin (DLF/LDC)
* Dennis Korpel (DLF)
* Mario Kröplin (Funkwerk)
* Mathias Lang (DLF/Symmetry)
* Robert Schadek (DLF/Symmetry)
* Bastiaan Veelo (SARC)

## Robert
Robert had nothing to report.

## Martin
Martin said he was very happy to see the recent work going on with dub (mostly courtesy of Mathias Lang and Jan Jurzitza, a.k.a. Webfreak) and thought it was looking great.

On the LDC side, he was currently working on merging an intermediate step of DRuntime and the D frontend from just before the dmd and druntime repositories were merged. That was the first step toward changing how they fold in DRuntime, D frontend, and DMD test suite changes into LDC.

He reported that Nicholas Wilson had received some outside help for catching up with accumulated changes in LLVM. He summarized some of the changes for us and noted that this was important as it lays the foundation for supporting LLVM 15 and 16, which come with breaking changes. He was not looking forward to doing it himself and was glad that these two had taken care of it.

Regarding ImportC, supporting it wasn't too bad at the moment, but he hoped that it didn't end up requiring too many special cases in the glue layer. Bitfield support, which [he had brought up in the previous meeting](https://forum.dlang.org/post/omshhtkrtkftsihntiaq@forum.dlang.org), was done. Now he had seen some test cases related to taking the address of a struct literal, which in D is allocated as a constant and in C is allocated on the stack. In DMD that was fixed by a glue layer change. It's this sort of thing he hopes remains minimal.

ImportC initializer lists are still in the AST, but another glue layer issue is with initializers of multi-dimensional static arrays in C. The initializer still has some C initializer lists that LDC doesn't support. So he thought those could be translated to D initializers so that we don't have to worry about C initializer lists. Another example he gave involves initializing static arrays with string literals when the literal is shorter in length than the array. This is something else that needs to be accounted for in the glue layer.

Martin is happy as long as the number of corner cases doesn't increase. He hopes we don't see the need for more with each release as ImportC issues are fixed. LDC is a D compiler. Being able to compile C is fine, but he sees the main use case as being able to import C headers to interoperate with C code compiled externally.

I asked him if there was an update about the DMD issue he has often brought up in these meetings in which switching the order of root imports can cause compilation errors. He said he had done a micro adaptation in DMD to fix a specific test case, but it opened a pandora's box of forward referencing issues. The main problem he found was that [attribute inference is skipped for aggregate methods when the aggregate itself hasn't been analyzed yet](https://issues.dlang.org/show_bug.cgi?id=23127). We need to figure out why. Attribute inference has other problems, so if we want to move to a scenario where attribute inference is enabled for everything, we'll need to work through these bugs.

## Bastiaan
Bastiaan reported that SARC has had problems with DMD crashing more often as their code base grows. He so far has been unable to get a reduced case demonstrating why and was unsure if he'll be able to. The good news is that everything compiles fine with LDC, so that's the route they're currently taking. He suggested that it could be that DMD is running out of stack space and that LDC simply allocates more of it. Martin said that he had recently increased the stack limit from 8 MB to 16 MB and that could indeed be the issue. ([The forthcoming 2.101.0 release of DMD](https://dlang.org/changelog/2.101.0.html#dmd.windows_stack_limit) includes the same stack limit increase.)

## Mathias Lang
In the previous meeting, [Mathias received the green light from Walter](https://forum.dlang.org/post/tpdbhhcqyxplxllsmlpm@forum.dlang.org) to move forward with a breaking change to `Throwable`'s internal `TraceInfo` interface. Following that, he had begun adding colors in the stack trace because he thought they were unreadable, and adding colors was not that much work. He had a proof-of-concept but still had a few things to work out.

He also updated us about Buildkite issues. It had been completely broken for a few days before the meeting. DLang Bot had gone down around October 1st, so nothing was being scheduled and Buildkite just hadn't been working. He had gotten it back on track the morning of the meeting. It was running off of his server, so it was a bit slower than before, but it works. And now we can update it. The problem before was that the list of dependencies was static: we had no control over it. For example, we had been stuck with LDC 1.29 because we couldn't update LLVM.

He thinks that Buildkite is one of the best CIs we have because it tests other people's code in practice and catches a lot of regressions. The direction forward is to have something that integrates with GitHub Actions. These days, pretty much everyone has Actions in their GitHub repositories, so if we could use their action definitions as the base for testing, that would be amazing. It's not trivial, but he thinks it's doable.

## Dennis
Dennis had begun implementing [DIP1035, "@system Variables"](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1035.md) and requested that Walter look it over. Walter [has since merged the PR](https://github.com/dlang/dmd/pull/14478), and Dennis has moved on to implementing the named parameters DIP (see [PR #14475](https://github.com/dlang/dmd/pull/14475) and [PR #14575](https://github.com/dlang/dmd/pull/14575)).

He had also stumbled upon the attribute inference cycle problem, where checking if a function call is safe might result in a pessimistic assumption that it's unsafe. He's looked into fixing it and he thinks it's possible to, in those cases, optimistically assume the function is safe. He hadn't yet seen a case where that breaks, but he wants to ensure that switching from the pessimistic approach to the optimistic approach doesn't create a big safety hole.

He then brought up [pragma startAddress](https://dlang.org/spec/pragma.html#startaddress). Iain had found a handful of uses of it in D GitHub projects, and it's only supported for the 32-bit Windows OMF target ([see the discussion in this PR thread](https://github.com/dlang/dmd/pull/14512)). It's not a burden to maintain, but given that one of the goals in the Vision Document is to simplify the language, this seems like a good candidate for deprecation and removal. Walter recalled that people often used it in 32-bit builds, and it's a supported extension in all the x86 C compilers. Mathias didn't think getting rid of it would gain us anything, and Walter agreed.

So then Dennis asked if we should spend time fixing bugs with this feature. Martin and Mathias said no. Walter suggested making sure there are issues for them in Bugzilla, but just marking them as low priority. That way they're on record if they become a priority for someone in the future.

## Iain
For the benefit of the industry folks who only join us in the quarterlies, Iain recapped some of the things he'd covered in the two previous monthly meetings, such as the move to Backblaze for downloads.dlang.org and docharchives.dlang.io, the transition to Cloudflare that gives us the benefit of free data transfer with Backblaze, and the general tidying up of the dlang.io namespace.

He then said that Martin Nowak will not be doing any more releases of the D language. Iain had been going through the build scripts that we currently have and figuring out what needs to be done to tailor them to run in GitHub Actions. It was taking much longer than he had anticipated. In the interim, he thought he could at least merge master into stable and get some 2.101.0 alpha builds set up by hand. (He then [announced the first beta on October 17](https://forum.dlang.org/thread/etvlqbomriskyeihzpuv@forum.dlang.org).)

Next, he gave us a summary of his experience at the GNU Cauldron where he attended a meetup of GCC/GDB maintainers. He reported that there are some really interesting things going on with GCC internals regarding the direction in which they're taking the compiler, including several things we're doing already. For example, they're adding options to automatically initialize all static variables to 0 or a bitmask. This sort of thing is good news for him, as he currently has to do all the memsets by hand in GDC. It's a win if the middle-end can do this for him. If you're interested in all the GCC internal changes that Iain was gushing about and how he can benefit from them in GDC, please ask him :-)

## Mario
Mario said he has no problems with D because he can hand them all over to Mathis. So he deferred to let Mathis talk.

## Mathis Beer
Mathis said the state of the compiler for Funkwerk was pretty good right now. He was unaware of any crippling compiler issues, though as always they'd been pushing into the dark corners to look for them via his experimental work.

Now that they have librebindable as a backstop (see [his DConf presentation about it](https://youtu.be/eGX_fxlig8I) if you haven't already), they'd been trying to do a lot of immutable stuff. Before, there were things they couldn't do with immutable that librebindable now enables. So now they're doing things like working with internal data types that had previously been "pseudo rvalue structs" and making them into fully immutable structs just to see where that breaks.

He's found it's broken with associative arrays. [Issue #22244 summarizes the problem](https://issues.dlang.org/show_bug.cgi?id=22244). This causes them to have to use a lot of casts, which he sees as a code smell.

If you're using associative arrays with ranges, you need `assocArray` to work, and it doesn't work with immutable values. He'd been trying to think of how to make it work so he could submit a PR to Phobos, but he had been unable to see a way forward short of pushing librebdindable into Phobos. In other words, how to allow `assocArray` to mutate the array, but prevent the array from ever being mutated again and disallow mutable views of it? He was looking for feedback.

Robert suggested this should be a "WONTFIX" because fixing it would create a hole. He doesn't think it's technically possible without having some feature that tracks the variable. Walter agrees.

Mathis said this raises the question of how immutable and associative arrays are ever supposed to work together. Robert thinks they won't, and suggested the best bet is to roll your own hashmap and move on with your life. Mathias Lang noted the same problem exists with const.

Mathis said Funkwerk have their own internal hashmap type, and they can work around the problem fine. He just feels like it's fighting the language a bit and brought it up because he wondered what would have to be relaxed or changed to make that less painful.

Robert mentioned head const, but insisted that we shouldn't go there. Andrei suggested that we need head const for C and C++ compatibility anyway. Walter said that ImportC doesn't support head const. His experience with C is that people generally mean const in C to be transitive. So in ImportC, he turned the head const cases into transitive const and it appears to work well.

Mathis said what we really need is head mutable. Walter said that's fine, it's head immutable that's not going to work. He's thought many times about adding head const, but it would be disruptive to D's type system, and that would likely introduce a long list of compiler bugs. I saw several heads nodding in agreement.

There was more discussion about the topic, but Walter didn't have an answer right now.

## Max
Max was having mic issues throughout the meeting, so he didn't make a report.

## Ali
Ali had nothing work-related, but he gave us a summary of the talk he gave at Northeastern University and the meetup they had after.

Other than that he said he had been working on his cached algorithm.

## Me
I gave everyone an update on my progress editing the DConf videos (I was a little over halfway through [Mike Shah's talk](https://youtu.be/nCIB8df7q2g) at the time), and that I would soon be publishing [a conversation Walter and I had recorded](https://youtu.be/G6b62HmsO6M).

I reported that we had raised a little over $26.00 at that point via the YouTube partner program. (Now I can report that in the month since we are closer to an estimated $40.00. Ads are currently disabled until we sort out issues with identity verification, but once they're enabled again I look forward to seeing that dollar value climb as we push out more content. I don't think there's any easier way to support D financially than by sitting through a few YouTube ads when watching our videos, so please help us out!).

I said that I'd been so focused on YouTube these days that I'd practically forgotten we have a blog. I was working with Ate Eskola to prepare [his subsequently published second blog post on DIP1000](https://dlang.org/blog/2022/10/08/dip1000-memory-safety-in-a-modern-systems-programming-language-part-2/), but I had no time to create blog content myself for these.

I then reported that Dennis had been preparing some YouTube tutorials for our channel and that I hoped to start publishing them by the end of the month. (The tutorial series is focused on contributing to DMD. Each video will be short and focused on a narrow topic. I reviewed the first video in the series a few days ago.)

Next, I informed everyone that I'd spoken with Dennis and Razvan about implementing DIPs that have been approved but not yet implemented (one result of which you can see above in my summary of what Dennis said, as well as [the project board they started for it](https://github.com/orgs/dlang/projects/16)). A loose, though likely unrealistic goal, is to get them all done by the end of the year.

Finally, I reported that Max was focused on finishing up floating point emulation. (I heard from him last week that a PR was imminent.)

## Andrei
Andrei opened with a request for financial support. It became clear recently how dependent we are upon Symmetry for funding. There's a short tail of small contributions from individuals, and there have been occasional donations from other companies, but without Symmetry we're stuck. So Andrei said he was coming "with his treasury hat in hand" to ask the industry reps to go back to their employers and ask about some kind of sponsorship. (I intend to do a write-up here in the forums about donations and funding sometime in the next couple of months.)

Next, he brought up the work that Walter and Dennis had been doing to squash DIP1000 bugs. He was curious if the industry folks were looking for a safer D language. He also thought we should consider a rebranding of "DIP1000" for the people outside of the small part of the D community who understands what that means. This prompted a discussion about the relationship between DIP1000 and `@live`, the number of Walter's DIP1000 PRs that are hung up on Buildkite failures, head const, C and C++ compatibility, Rust's marketing success, and more.

Some highlights:

* Mathis Beer's "completely personal opinion" as to why he isn't pushing to use `@live` at Funkwerk: he feels Rust has created a false impression that only liveness and borrow checking can guarantee safe code, but he believes that safe code is also guaranteed by copying, or trusting in a reference counter or a garbage collector. He says the borrow checker enables a tradeoff between performance and safety. That's important for people who chase new languages for performance, but all of Funkwerk's performance gains are algorithmic. They run on servers with 80GB of memory, so it's not the sort of micro-optimization where they need safety at the moment. That said, he thinks it's a good idea from a language perspective, a user perspective, and a marketing perspective
* Walter said that with a GC, you don't need a borrow checker, but having one allows you to guarantee no leaks when you aren't using the GC. So although these problems should be rare in D, they can happen. And he'd like to close that hole. Rust has successfully solved the idea of 100% memory safety, and to compete we should do that, too.
* Andrei talked about how Rust has very effectively managed to sell memory safety as the be-all-end-all of good system software, citing as one example a tweet by Microsoft Azure's CTO Mark Russinovich saying that it's time to give up C and C++ in favor of Rust. Since we're working on `@live` already, why not capitalize on their marketing? * Bastiaan said that SARC is interested in safety in principle, but in practice not for a long while. With their code base being transcompiled to D from Pascal, they rely on a lot of unsafe hacks. Removing those hacks and implementing `@safe` is many milestones away if they ever get to it.

Among all of this, there was also a discussion about D's opt-in approach with `@live` vs. a full program borrow checker. Dennis paraphrased previous forum comments from Adam Ruppe and Timon Gehr making the argument that you can't have 100% memory safety *and* an opt-in `@live`. Walter said Rust has unsafe code, too, and if you really want to you can write an entire program in Rust with unsafe code and disable the borrow checker. He thinks it's possible to have the borrow checker guarantees in `@live` functions and be fine with that.

Martin thinks we should be cautious in talking about 100% memory safety because of compiler issues that keep surfacing. Walter stressed that this is why the test suite is important. Every bug that is fixed goes into the test suite, and that's like a ratchet that only moves forward. Mathis Beer noted that the ratchet only goes one way if the compiler stands still; every new feature brings a multiplicative increase in the potential for corner cases. Walter agrees that's a potential problem.

## Walter
Walter asked me to ensure that we have our own copies of all of the videos we upload to YouTube. I said that I'd been using Google Drive through our dlang.org Workspace account to store the DConf videos, but I hadn't copied other things there yet. I'm going to look into whether our Backblaze account will end up being cheaper.

Next, he said he had been going through the bug list for ImportC. He started with the worst bugs and then worked his way down. There were still some PRs stuck because of Buildkite. He thinks we're in good shape with ImportC, is pleased with it, and believes it gives us a competitive advantage. Once those are down to a "dull roar", he'll go back to DIP1000 bugs. When those are at a dull roar, he plans to get back on `@live`.

One thing Walter would like to see from the community is people doing presentations about D outside of DConf. Everyone who has presented at DConf has something in the can for other conferences. Walter has presented at three programming conferences in the last year. Most of them will pay for your travel if you're speaking there. You've already got the presentation ready to go, so why not submit it?

He's thankful for the people who have joined him in defending D on Hackernews. He encourages those who read sites like HN and Reddit to politely and non-confrontationally point out D's strengths when the opportunity arises. Basing other languages turns people off, but you don't need to compare to other languages. Just say, "this is how we do it in D" without mentioning the other language. That's an effective form of evangelism that we need more of.

Ali said that he had talked to someone recently who, upon learning that Ali uses D, said he had become interested in D because of Walter's comments on Hacker News. He was impressed with Walter's demeanor and always followed his posts. Ali admitted that he has a hard time keeping his cool like Walter does when reading some of the comments he sees out there.

Walter talked about an embarrassing situation that had recently cropped up. Someone was having an issue with DMD crashing in random ways when it allocated. Walter was sure he was always checking for overflow when calling `malloc`, then he and Dennis grepped the code base and found a lot of allocations in DMD that did not check for overflow. He had just put out several PRs to fix that, so now when DMD runs out of memory it will always print an error message saying so rather than producing some other kind of random failure.

## The next meeting
The meeting closed out with some last-minute discussions about DConf Online (I'll post the schedule later this week) and Iain's experience with the DMD release process. We wrapped up at almost the one-hour-and-twenty-minute mark.

The next meeting is happening on Friday, November 4, at 14:00 UTC. This is a regular monthly meeting. If you have anything you'd like to bring to us, please let me know and I'll see about getting you into either this meeting or the next one, depending on your schedule.
