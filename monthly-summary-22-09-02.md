[Forum Post](https://forum.dlang.org/post/omshhtkrtkftsihntiaq@forum.dlang.org)

The D Language Foundation's monthly meeting for September 2022 took place on September 2nd at 14:00 UTC. The following were in attendance:

* Andrei Alexandrescu
* Walter Bright
* Iain Buclaw
* Ali Çehreli
* Max Haughton
* Martin Kinkelin
* Dennis Korpel
* Razvan Nitu
* Mike Parker
* Robert Schadek

### Iain
Over the preceding month, Iain had little going on with D due to a personal situation. Most of the work he'd done had been on the infrastructure project and not on the compiler. He noted that I had migrated the dlang.org and dlang.io DNS servers to Cloudflare, and that several records had failed to transfer across (we had some other hiccups, but Iain, Vladimir Panteleev, Petar Kirov, and Mathias Lang, provided invaluable assistance in resolving those). We had managed to recover most of them.

The dlang.io domain had been maintained mostly by volunteers from the community. When Iain went through the DNS records, he discovered that a number of them were pointing to documentation sites on servers that were no longer available. He cleaned that up for us.

He talked about how he had set up a bucket for docarchives.dlang.io and downloads.dlang.org on the Backblaze account. The DNS for the doc archives was now pointing to the new bucket (via a Cloudflare worker), and all of the documentation for 2.068 - 2.099 was available there. The DNS for the latter was still pointing to the AWS download site, as Martin Nowak was still packaging and uploading the 2.100.2 release. Once that was out, Iain planned to set up a Cloudflare worker to point downloads/download.dlang.org at our new Backblaze downloads bucket. Those are now live. He would like to eventually work out how to automatically deploy to our new setup via GitHub Actions.

For those who missed my mentions of this in the past, by moving to Backblaze + Cloudflare, we are paying only for storage. Both are part of the Bandwidth Alliance. As long as we have Cloudflare in front of our Backblaze buckets, [we have free data transfer](https://www.backblaze.com/blog/backblaze-and-cloudflare-partner-to-provide-free-data-transfer/).

I reminded him that at our DConf meeting he had complained about the state of our build system and how it needed to be cleaned up. At the time, Átila said something like, "Sounds like you're volunteering." He was joking, but there was also a serious question there. I asked Iain if he would be up to taking it on. He agreed to oversee it. I don't know what his plans are in terms of priorities, but I'm sure we'll hear more about this in the not-too-distant future.

### Robert
One of the problems we had in migrating the DNS records was an issues.dlang.org outage. That was a bit of a scare. Because of it, Robert ran one of the scripts he'd developed for the Bugzilla->GitHub migration to collect all of the Bugzilla issues as JSON files on his disk. As a next step, he needed to get the software in a state so that, with a GitHub access token, he can migrate the issues. He said the code was "about 95% there". He would need to run it one more time to make sure everything is right, then make the move.

One thing he'll need when the time comes is to put Bugzilla into read-only mode during the migration.

Martin reminded Robert that the compiler suggests posting to issues.dlang.org in certain error messages, so that will need to be updated as well. Robert said he would do so.

Iain noted that Vladimir had also been working on a way to backup all of the Bugzilla issues.

### Martin
Martin started by complimenting the DConf speakers. He had watched the livestreams and said the talks were at a "great level".

In recent weeks he'd had little time for D development of any kind. For LDC, he worked on bitfield support for the upcoming release for ImportC. He said it seemed ImportC was now able to compile zlib thanks to Walter's work, and if it really works on all platforms, then it's a nice acheivement that shows ImportC has potential. That said, he found that implementing bitfield support in LDC was the PITA he'd expected it to be. It means that now LDC has to use 24-bit and 48-bit integer types and such in their LLVM IR like clang does.

He encountered a frontend problem in the very first test cases where the bitfields end up with an 8-byte struct when it should be 4 bytes as with GCC and clang on Linux. He filed an issue for it. So there are still corner cases that need to be ironed out.

Other than that, Nicholas Wilson had been working on LLVM 15 support and had asked for help in the forums. It's not trivial work. But Martin's first priority was the D 2.101.0 release, primarily work involving the shift to the DMD/DRuntime monorepo. He'd not had time to follow through with that on stable, so it was going to take some time before the next LDC release.

Finally, he brought up again a persistent DMD issue that has been affecting him with his work at Symmetry. It's a problem with the reliance on the semantic order of the root modules being processed. In some cases, switching the order of root modules results in compilation errors. At first, it was specific to using `-allinst`, but now they're seeing an error related to `@safe` in one case, and linker errors in another. It can be triggered by just removing an unused import. He's found no workaround for it.

Max (also a Symmetry employee) told Martin he had worked out roughly what the problem was and suggested they get together for a few hours to work out a fix. Martin went on vacation for a couple of weeks shortly after the meeting, and they hadn't gotten together yet last I heard.

### Razvan
Razvan said that he and Dennis had been working on extracting some tasks from the vision document and [created some projects in our GitHub project planner](https://github.com/orgs/dlang/projects). He walked us through what they had done so far, said that it should be publicly viewable, and talked about how we could use it. The main goal is that it should serve as a one-stop-shop for contributors looking for impactful projects to work on. One thing they would like to use this for is to help in new contributors get started with some Bootcamp projects. He suggested that Walter and Átila could create some projects there for some of the things on their plates and maybe someone else can pick them up.

[He has since posted about it in the forums](https://forum.dlang.org/thread/ktfdceqpxpoplwzdpnkf@forum.dlang.org).

### Dennis
Dennis only had one remark: migrating our Bugzilla issues to GitHub will benefit our use of GitHub Projects.

### Max
Max said he had little to report but needed to get his TODO list sorted. He said he would like to help Nicholas with the LLVM 15 support for LDC. (In a subsequent meeting Max and I had, he said that of all the projects on his TODO list, the one for 8-bit floating point emulation was the one he could probably finish first and is working on that.)

He said that he and Steven Schveighoffer had found a bug in the way that DMD merges the types of pointers, e.g., it will merge `int*` and `void*` to `int*`, and that is now fixed. Fixing that also fixed a bug in the template argument matching code, so it was a two-birds-with-one-stone thing.

### Me
I let everyone know that I had finally received the footage from the camera in the back of the room at DConf. I'd already been granted access to the recorded versions of the talks, which are identical to what is in the livestream. Those were free, but Symmetry had to pay an additional fee for the camera footage (and a big thanks to them for doing so). I let everyone know I would start editing the videos the following week. (I'll be finished with day two in a few days; I'm working on Max's talk now).

Next, I gave an update on some boring admin stuff regarding reimbursements for DConf speakers.

Finally, I noted that I had spoken to Weka's Eyal Lotem at DConf. He said their biggest remaining sticking point is what happens after a move, which is what originally prompted [one of their employees to write DIP1014](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1014.md), which introduced `opPostMove` and was accepted. It was never implemented and now never will be, thanks to [the approval of copy constructors in DIP1018](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1018.md) intended to replace post blits. That eventually led Walter to introduce [DIP1040 to propose move constructors](https://github.com/dlang/DIPs/blob/master/DIPs/DIP1040.md), among other things. Max has since taken over responsibility for the DIP.

At DConf, I'd asked Eyal if he and/or his coworkers could take a good look at the DIP and let us know if that solves their problem. He agreed, and has since provided Max with feedback. I believe Max has a question or two yet to answer, but we're hoping the collaboration here will result in a stronger DIP for the next review round.

### Walter
Walter had begun going through all the DIP 1000 bugs. He and Dennis had submitted fixes for several of them, but they were all stalled due to failures in the Buildkite projects. A general problem he finds with the Buildkite projects: if something goes wrong, what are we supposed to do about it? He finds this quite frustrating.

Dennis noted that the DIP 1000 Buildkite failures were related to the preview switch. It's only a few projects that actually fail, others have deprecation warnings. He has been going through the failing projects and submitting PRs to them. It's taking time, as it depends on the maintainers merging the PRs and tagging new releases for code.dlang.org. A benefit of this is it has led him to find the causes of some DIP 1000 bugs.

Another problem was that std.logger was crashing in its unit tests, preventing the test suite from running [as described in Issue #23286](https://issues.dlang.org/show_bug.cgi?id=23286). Razvan noted this was only happening in 32-bit builds. Robert was the original author of std.experimental.logger, so he looked into it [and submitted a fix a few days later](https://github.com/dlang/phobos/pull/8557) that has since been merged.

Finally, Walter said that he finds that the test suite "just seems to stall on things", causing massive delays and interrupting his workflow. This led to a discussion about how Buildkite works, and through this we learned that Walter had never been set up with a Buildkite account, so he doesn't have an interface to restart builds.

Aside from his frustrations with the autotesters, he's happy about the current state of the DIP 1000 bugs. The issues he and Dennis have been closing have been fixable and they had whittled down the list to a small number. Overall, he thinks we're in good shape with it. He's also been working on ImportC, and he's happy with how things are progressing overall with the language.

### Ali
Ali informed us of his invitation from Mike Shah to speak at Northeastern University on September 30. Steven Schveighoffer was also planning to attend. The plan was a talk and then a Boston meetup. Steve had invited some people who used to attend the Boston meetups. Ali subsequently [posted a link to the recorded talk here in the Announce forum](https://forum.dlang.org/thread/thggap$1lqi$1@digitalmars.com). I noted that Mike Shah had submitted a DConf Online talk, and that [he has a D tutorial series on YouTube](https://www.youtube.com/playlist?list=PLvv0ScY6vfd9Fso-3cB4CGnSlW0E4btJV).

Ali had also been working on the hobby project that [he had talked about in his DConf '22 lightning talk](https://youtu.be/ksNGwLTe0Ps?t=20372). He expects he'll eventually post it on code.dlang.org, and that the discoveries he's made while working on it could make for a potential future DConf talk. He didn't submit anything to me for this year's DConf Online, so maybe we'll see something down the road. But he has since [announced his new `alid` package](https://forum.dlang.org/thread/tfmtfe$1a5b$1@digitalmars.com) in the dub repository.

### The next meeting
The next meeting was a quarterly meeting that took place on October 7. I plan to post the summary for that meeting on the last Friday of this month.

As always, if you have something you'd like us to put on the agenda of one of our meetings, please let me know and I'll see about arranging it (though I may recommend [posting in the forums first](https://forum.dlang.org/thread/vphguaninxedxopjkzdk@forum.dlang.org), depending on the topic).