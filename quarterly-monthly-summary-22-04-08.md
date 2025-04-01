[Forum Post](https://forum.dlang.org/post/ntnxtskafgoalcramqry@forum.dlang.org)

## Quarterly Meeting Summary
The quarterly meeting for April, 2022, took place on April 8 at 14:00 UTC. Quarterly meetings include representatives from companies using D in production.

__D Language Foundation__
* Iain Buclaw (GDC)
* Ali Çehreli
* Max Haughton
* Martin Kinkelin (LDC)
* Dennis Korpel
* Átila Neves
* Razvan Nitu
* Mike Parker

__Industry Reps__
* Mario Kröplin -- Funkwerk
* Robert Schadek -- Symmetry
* Bastiaan Veelo -- SARC
* Joseph Wakeling -- Frequenz

Our quarterly meetings are normally rather long (2.5 to 3 hours is not uncommon), but this time we finished in a little under an hour. And that's following on a *three-hour* monthly meeting from March (those are usually 1.5 to 2 hours).

### Me
I opened the meeting with a request of the foundation people that we schedule another meeting in April to decide on how to approach transferring management of several of the D ecosystem services to the D Language Foundation. I let them know that I had received resource requirements from the maintainers of several services. Those present agreed.

I subsequently contacted several interested parties and we set up the meeting for April 29, at 14:00 UTC. You'll find a brief summary of that meeting at the bottom of this post.

### Mario
Mario informed us that Funkwerk had released a new open-source tool [that they call cÅgitÅ](https://github.com/funkwerk/cogito). It "analyzes D code and calculates its cognitive complexity." This is a measure of how difficult your code is to understand. He explicitly mentioned that they are using the D frontend for this. They've been using libdparse in their ecosystem, and this is the first time they've used the D frontend in a project. It's possible it will replace libdparse so that they don't have to keep up parallel maintenance.

I noted that Razvan has two students working on defining an interface for dmd as a library to replace libdparse. Razvan said he would be interested in any issues they faced during the development of the tool, as it could help him better define the interface. Mario said he could connect Razvan with the developer so that they can discuss it.

Mario also mentioned that Mathis Beer is having "the usual problems" in his work because he is always "in the dark areas of the D compiler", but they have nothing causing them any major issues that need immediate attention.

### Bastiaan
Bastiaan wanted to make sure we don't forget the dub documentation site at dub.pm in the server meeting.

He said his company has no major issues at the moment. They are making good progress with their Extended Pascal to D transpilation project (if you haven't heard of that yet, please check out his DConf 2017 talk, [Extending Pegged to Parse Another Programming Language](https://youtu.be/NoAJziYZ4qs), and his DConf 2019 talk, [Transcompilation into D](https://youtu.be/HvunD0ZJqiA)). He noted that they are using both dub and the `-i` dmd compiler option as part of their build process. They aren't quite sure what the best build options are for their project, and plan to experiment to figure it out.

### Joe
Joe said the D code on his company's project is running fine; it's the Python they're having problems with.

### Robert
The persistent problem with Symmetry's codebase is slow compilation, but Robert had no new problems to report. He said he had recently found time to work on the Bugzilla to GitHub migration script. And he closed by saying, "Other than that, D is the best language. Period."

### Martin
Martin had released LDC 1.29.0 (based on D 2.099.1; the beta was based on D 2.099.0) just a few hours before and it was looking good. He said that D 2.099.1 is very minor compared to previous minor version releases of D, so he didn't expect too much trouble from that.

He said that for Symmetry he had run the main project and its test suite with the latest LDC and DMD on Linux and Windows, and it was looking good so far. Everything was green.

### Iain
Iain had been waylaid by covid for most of the preceding month, so he didn't get much done in terms of development, but he did get quite a few things done on the admin side.

One thing he did was to update [docarchives.dlang.io](https://docarchives.dlang.io/). It hadn't been updated since version 2.081. He rebuilt every release from to 2.082 to 2.098. He then reached out to Seb Wilzbach to take over administration of the server (and on a related note, Seb has since transferred the management of the dlang.io domain name to the D Language Foundation; thanks Seb!). As I was preparing to publish this, the site still hadn't been updated. The last piece was for Iain to gain write access to the docarchives.dlang.io repository. And now that's done, so the site should be updated not long after I publish this post.

Iain went through the release process with Martin Nowak so that he could learn how it's done. He didn't yet have the secrets to upload to the S3 servers, but he was planning to upload the 2.100.0 beta once he got them (and [that was announced on April 22](https://forum.dlang.org/thread/ayztzvqvpcbknbshpflv@forum.dlang.org)).

GDC was in sync with the most recent master merge of 2.099.1 at the time of our meeting. The branching of GCC 12 was imminent. His plan was to switch to the stable D frontend branch in time for the GCC 12 release. He has subsequently announced in our #gdc Slack channel that the sync is done and GCC 12 is expected on Friday. So this weekend you should be able to use a version of GDC that is current with the latest D language version. (Please give a big, big "thank you" to Iain for the effort he put in these past months and years to make that happen.)

He had been working on ImportC with Walter. A big focus was what to do about the preprocessor. Walter was all-in on forking (calling out to the "system" preprocessor, e.g., cpp on Linux), but Iain wasn't convinced that's the best approach. He has so many different toolchains set up that if the D compiler calls out to cpp, it won't be the correct preprocessor for a given toolchain he's targeting. The preprocessor is actually linked into GDC as a library, although it isn't being used. It might be an idea for GDC to try to use that, but that's something for the future.

Iain had also implemented all the functions in the `core.int128` module as intrinsics. On x86, he gets the exact same codegen as per native. He was planning to do more testing before putting that in.

### Razvan
As noted in [my summary from our meeting in March](https://forum.dlang.org/thread/zfmkiqwkkexeaohjsuru@forum.dlang.org), Razvan had recently been looking into reviving the "metadata" (mutable) DIP (see [his forum discussion titled "Rebooting the `__metadata`/`__mutable` discussion"](https://forum.dlang.org/thread/cmtdqpfmnodcciivlfzl@forum.dlang.org). He had been reading a lot of material and was getting pretty close to submitting a new version of the DIP. He gave us a summary of the original and why it stalled. His forum post links in turn to [another discussion from 2019](https://forum.dlang.org/thread/3f1f9263-88d8-fbf7-a8c5-b3a2a5224ce0@erdani.org) where the original DIP was discussed. Anyone interested in the details can get them there, but the gist of it is that at the time it wasn't clear how to overcome Timon's objections about the interaction of `__metadata` with function purity.

He was looking for input on his recent thoughts on overcoming the problem with purity. There was a bit of discussion about compiler optimizations for strongly pure functions (DMD used to do some, but Dennis had disabled them due to a bug; Iain couldn't recall if GDC does; Martin said LDC does not, as LLVM provides no mechanism for it), and some discussion about how the monitor field of immutable class instances is already considered mutable. I'll leave it to Razvan to explain his idea and what, if anything, he took from the discussion.

### Dennis
Dennis brought up a PR to add a reusable bitfield utility to dmd and use it to replace the 'FunctionFlag' enum with a `BitFields` struct. This was prompted by the built-in vs. library bitfields discussion in our March meeting, in which Andrei suggested Walter try a library solution and see how it pans out. It was subsequently merged.

He said the most egregious DIP 1000 bugs had been fixed. There were still some "iffy" things with nested functions, but there should be nothing like the miscompilation of pure functions that corrupt memory. He noted that Átila had resumed work on making DIP 1000 the default by issuing deprecation warnings. He mentioned one problem that had recently come up in a forum discussion and PR thread in the form of transitive scope. He didn't know the plan for that and would like to discuss that and other DIP 1000 design decisions with Walter. Walter was absent from the meeting, but Átila said the meeting wasn't the place to discuss it anyway. He was planning to call Walter to discuss it anyway and suggested Dennis join the call. I know they have since been in contact (if not all at the same time) to discuss this, but I don't know if anything has been resolved yet.

He said that the dmd test suite is pretty slow, and sometimes it randomly fails. He finds it frustrating and hopes someone can do something about it.

Finally, he wanted to make sure everyone (in the meeting and outside) remembers that he and Razvan are here to fix bugs. Looking through issues by himself, it's hard to tell which are most important to people. So if anyone has an issue that they need or want to be fixed, let Dennis and/or Razvan know. If they can't fix it themselves, they'll do their best to find someone who can.

(On a related note, please remember #dbugfix here in the forums and on Twitter.)

### Ali
At work, Ali hasn't had time to code in D lately. Instead, he's been required to assist a couple of people who developed a tool in Rust. That led to a couple of observations.

His first observation was that Rust has a "tuple window" thing that is equivalent in functionality to `std.range.slide`. A big difference is that in Rust, the generator function is only called once and its results are cached. In D, the generator is called multiple times and there is no caching. This led him to experiment with implementing a cached range algorithm that would allow you to do something like `map.cached.slide`, but he was unable to get it to work efficiently. On the plus side, the tuple window cache in Rust is limited to 4 elements (though it's scheduled to be increased to 12), which in turn limits the size of the window. `slide` in D is not limited in this way.

Ali had to work next to a Rust programmer for this. Ali uses Emacs, but he noticed his coworker was heavily reliant on tooling, particularly Visual Studio with rust-analyzer installed, and was unproductive without it. This led Ali to make a New Year's Resolution to start working with Visual Studio Code with D.

### Max
Max reported he had [opened a PR with some preliminary work](https://github.com/dlang/dmd/pull/13964) for calling out to the preprocessor in ImportC. This is an alternative to Walter's approach that calls out to a specific preprocessor. Max's approach is to look through a list of available preprocessors and rank them based on their ability to support the target. So, for example, if you're on Linux and want to compile for BSD but don't have a cross-compiler set up, it will tell you rather than failing blindly.

Max also reported he had [opened a PR to merge](https://github.com/dlang/dmd/pull/13965) LDC's time trace into DMD. Dennis said he finds LDC's time trace very useful. It had already helped him find some problem code in DMD that was exponentially slow. He expects that having it in DMD will help find other bottlenecks and may prove particularly useful in solving the slow compile times in template-heavy code like Symmetry's. Martin has analyzed Symmetry's code with time trace in LDC and found some wins. He also said the biggest problem with Symmetry's code isn't that dmd is slow, but parallelizability. Moving to Reggae for builds has helped with that.

Max had also rebased the branch where he was working on implementing named arguments. He said it's working for normal functions, but he's had trouble getting it working right with templates.

He also has [a PR that implements with expressions](https://github.com/dlang/dmd/pull/13947). He said he put it behind a preview flag, but he doesn't mind waiting to push a DIP through for it.

Finally, Max bumped the version of lld that ships with DMD, as the shipped version was about five releases behind the latest.

### Átila
Átila is on a mission to "clean house" and "make all these things that were supposed to be temporary go away."

One such thing is DIP 1000. He's been trying to get to a point where we can make it the default. Before that happens, people should pay attention to deprecation messages and fix up their code.

Another thing on his list is making `-preview=nosharedaccess` the default, but [it's blocked on Issue #22626](https://issues.dlang.org/show_bug.cgi?id=22626). He doesn't know how to fix that and isn't sure who does.

He also wants to get allocators and the logger out of `std.experimental`, as they've been there long enough.

He would rather be working on figuring out how we can make breaking changes without breaking anything, but these "temporary" things are annoying and need to be fixed first.

## Server Meeting Summary
The server meeting took place on April 29 at 14:00 UTC. The following people attended:

__D Language Foundation__
* Walter Bright
* Iain Buclaw
* Ali Ã‡ehreli
* Max Haughton
* Dennis Korpel
* Átila Neves
* Razvan Nitu
* Mike Parker

__Community__
* Jan Jurzitza
* Petar Kirov
* Jan Knepper
* Mathias Lang
* Vladimir Panteleev

In this meeting, we covered the requirements of several D services and discussed options for hosting and management. The goal is to establish a system in which multiple admins can be available to respond to outages and other problems, but only a handful are able to modify the servers; one into which we can migrate services one at a time and scale out as needed. In the end, we settled on an approach using Terraform, Cloudflare, and a VPS provider.

Vladimir and Petar volunteered to oversee the effort. They'll pick a VPS provider, determine which services to focus on first, and basically take the seed of a plan we agreed on in the meeting and grow it into an action plan that we can follow. I asked them to give us three options in terms of budget: the ideal, the bare minimum, and a middle road. We will have a baseline of $150/month available to pay for hosting. I don't know just yet how much we'll have beyond that initially, but I expect it will be a little more.

They have set themselves a deadline of one month (so the end of this month) to get their plans together, at which time they'll come back to a meeting and we'll pick the path we'll follow. At that point, it should just be a matter of making it happen.

## The Next Meeting
Our next meeting is a monthly DLF meeting. It's happening on May 6 at 14:00 UTC. I currently do not have anything major on the agenda. If there's something you'd like to bring to the foundation and you can attend the meeting, please let me know and I'll make room for you.