[Forum Post](https://forum.dlang.org/post/tpdbhhcqyxplxllsmlpm@forum.dlang.org)

Under normal circumstances, I would have scheduled our August meeting on Friday, August 5th, but as most of us were in London for DConf, August 4th, the day of the Hackathon, was a perfect day for it. So in the afternoon, those of us physically present gathered in the space behind the DConf stage, with Iain joining us remotely, and got down to business.

This was only the second opportunity we've had to hold one of these meetings in person since we started them, the last being a quarterly meeting at DConf 2019. We really enjoyed this one, and we ended up in pockets of conversation that went on after the meeting finished, which allowed for some participants to further discuss topics that arose in the meeting, among other things. I'm looking forward to our next chance for a face-to-face meeting.

## Particpants

* Walter Bright
* Iain Buclaw
* Max Haughton
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Robert Schadek

### Me
I opened by informing everyone that Robert will be joining all of our monthly meetings going forward. He has been representing Symmetry at our quarterly meetings (since the beginning, IIRC), and I have brought him in to the monthly meetings now and again. He told me during the conference that he was willing to join us every month.

I then announced that I had nothing  on the agenda, and I turned the floor over to Iain.

### Iain
Iain started with an update on the merger of the dmd and druntime repositories, saying that it had gone mostly well, and that he, Mathias, Martin, and Vladimir had handled the problems that came up. He said the main takeaway was that our build infrastructure is a mess. There are too many makefiles and scripts all over the place, many of which do the exact same thing. He thinks it would be good to put them in a single repository that everything else feeds off of, but anything is better than what we have now. Átila agreed.

This led to a discussion about build.d vs. the makefiles. Max noted that building dmd with `dmd -i` almost works, and there's just one thing in the backend preventing it. Ultimately, we decided that we need a someone to act as a build manager to focus on streamlining our build system. Átila jokingly nominated Iain for this, since he brought it up. (When I brought this up with Iain in our September meeting, he had forgotten about it, but agreed in principle to oversee a building system revamp. He and I will discuss this more later.)

Next, Iain said the release scripts were the next things in his sights. Martin Nowak had previously taken him through the process of packaging a dmd release, but at this point there were two things he didn't have: a valid code-signing certificate and access to the S3 buckets for downloads.dlang.org. The code-signing certificate we had from DigiCert was good for three years, but had expired the year before, and now was causing problems in signing the Windows release. EV code-signing certs are expensive, and also have a 2FA dependency on a hardware token. Guillaume pointed suggested OV certs in a subsequent forum discussion, as they are cheaper and easier to use, but unfortunately, Iain has since discovered that the situation is set to change in a couple of months. Apparently, Microsoft's new policy will result in some form of 2FA for all code-signing certs, not just EV. I'm not at all versed on the details, or the pros and cons, so I'll leave it to someone else to answer any questions.

As for downloads.dlang.org, he noted that I had set up a Backblaze account where we intended to transfer all of the files. He and I also explained how we'll have free bandwidth with Backblaze by using CloudFlare in front, as they are both part of the Bandwidth Alliance. He also noted that docharchives.dlang.io had been down for a while, and he was going to move it to a Backblaze bucket. As I mentioned in [the summary of the July meeting](https://forum.dlang.org/post/lxfildvecircypoabain@forum.dlang.org) that I published last week, he has since migrated the download files to Backblaze. Iain then clarified in a reply that downloads.dlang.org still goes to the S3 bucket for now, but that docarchives.dlang.io is operational again from a Backblaze bucket.

As for the releases, Iain plans to try and get an environment set up through GitHub Actions in which anyone on the core team to trigger the packaging of a release, so that we are no longer reliant on one person to handle it.

### Dennis
Dennis had nothing for us at this meeting.

### Max
Max said he had nothing either, but Walter asked about his 80-bit floating point emulation work. Max said he had implemented multiplication and addition without correct rounding. Walter suggested he look at Walter's half-float implementation for inspiration. Max thought he should consult Knuth, but Walter thinks that's not necessary if he uses the half-float implementation as a reference. But they both agreed there's not a lot of information available on the topic.

I mentioned to Max that I had spoken to Eyal Lotem of Weka about the move constructor DIP ([DIP 1040, "Copying, Moving, and Forwarding"](https://github.com/dlang/DIPs/blob/master/DIPs/DIP1043.md), for which Max is the POC). Eyal told me this is the last big problem Weka has, so I asked him to review the DIP and provide us with some feedback before I move it to the next review round. Max said he had also spoken with Eyal and was particularly interested in Weka's thoughts on the semantics involving "last use". I had intended to email Eyal about this later, but I didn't get around to it until I started writing this summary.

Max then remembered something he'd wanted to mention: people are using [the Dicsussions page in the dmd repository](https://github.com/dlang/dmd/discussions) to report dmd bugs. He suggested this shows that there's no argument for using Bugzilla. Robert concurred, and then launched into his turn...

### Robert
Robert updated us on the current state of his scripts for migrating Bugzilla issues to GitHub. He believed he had found everybody who has both Bugzilla and GitHub accounts, and all that remained there was to `@` tag them. He wanted to find a way to do it without spamming everyone with GitHub notifications. He believed the way to do that was to have the issues private during the move, then make them public after. He would have to put the script on a VM and let it run, as GitHub limits how many issues can be created in an hour, then requires a period of sleep. He would need an admin to create keys for him to use with the GitHub repositories, then he could pull the trigger when he's ready.

He them said that once all of the issues are migrated, we should activate the GitHub's project planner with a focus on identifying issues that new contributors can handle and that we should try to respond quickly to their PRs, and do what we can to onboard them. He noted how he is a contributor today because he was able to get a few commits into a core repository early on and was motivated to keep going, so we should do everything we can to ease the entry process for new contributors. There was some more discussion about this, but in the end it's going to be in the hands of the PR Managers (Razvan and Dennis).

Mathias noted how most of the issues on Bugzilla are for dmd. He suggested the phobos should be the first project to have the issues migrated. Robert said he's been using Phobos as a test case, so once it's where he'll be starting anyway.

Mathias then asked how Robert handled categorization (for example, the "x86" category on Bugzilla). Robert said part of the program looks at all the Bugzilla categories and sorts them into different color-coded GitHub labels. Mathias said he had also done some work on equivalent GitHub labels and they're already in the GitHub organization. They agreed to get together later to finalize a list of labels, though they did briefly delve into some of the details here.

### Átila
Átila hadn't been aware of all the work that had recently been going into dub (mostly from Mathias and Jan Jurzitza, a.k.a. Webfreak). He was happy to hear about it, but slightly annoyed that he hadn't known. Other than that, he had nothing for us.

This led to a tangential discussion about new ways to recognize and reward contributors. I won't summarize that here, but it's something we're looking at.

### Mathias
Mathias asked Walter if he could make a breaking change to `Throwable`. That class has an internal interface, `TraceInfo`, which has two `opApply` implementations. Both of them take intended to take `char` arrays to generate text output, but Mathias said that's not ideal. Internally, DRuntime has a struct that contains more info, and it would be better to provide the user with this struct rather than a string. The user can then format the output however they want it.

The problem is that changing the interface is a breaking change. Most D users probably don't even know the interface exists, and if they do they probably aren't using it. If anyone is using it though, it would be Weka. So if he talked to Weka and got the thumbs up from them, would the change be allowed? Walter and Átila both saw no problem with it.

### Walter
Walter brought up the idea of having a forum page for each page of the documentation, something Paul from Ucora had brought up in the forums and that we had discussed in our quarterly meeting in July. It seems the idea had stuck in his mind.

Now that he had ImportC working well, he wanted to spend some time knocking out DIP 1000 issues. This would be his priority for now. This led to discussion of some issues Átila and Dennis had found. Átila still needed to reduce his, but the one Dennis referenced was already in Bugzilla.

From here we wandered off into a discussion of random topics such as the benefit of having an in-person meeting, speculation about next year's DConf, the coronavirus, and more. There was a bit of technical topics discussed, but nothing that needs coverage here. In otherwords, I didn't ever "gavel" the meeting closed as I usually do in our virtual meetings, but we were done. And we enjoyed hanging out a while longer before we wandered off to the main room.

## The next meeting
For the second time in a row, I'm publishing a meeting summary after the next meeting has already taken place. It was a regular monthly meeting that we held at 14:00 UTC on September 2. I expect to be back on track with this one, so you should see a summary for it published in the last week of this month. The next meeting as I write is a quarterly meeting which we should be having on October 7 at 14:00 UTC.