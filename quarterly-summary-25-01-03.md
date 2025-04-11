[Forum Post](https://forum.dlang.org/post/faxtztrhckkzknwrcuhn@forum.dlang.org)

The D Language Foundation's quarterly meeting for January 2025 took place on Friday the 3rd at 15:00 UTC. It lasted about 35 minutes.

Our quarterly meetings are where representatives from businesses big and small can bring us their most pressing D issues, status reports on their use of D, and so on.

## The Attendees

The following people attended the meeting:

* Walter Bright (DLF)
* Luís Ferreira (Weka)
* Dennis Korpel (DLF/SARC)
* Mathias Lang (DLF/Symmetry)
* Átila Neves (DLF/Symmetry)
* Vladimir Panteleev
* Mike Parker (DLF)
* Carsten Rasmussen (Decard)
* Robert Schadek (DLF/Symmetry)

## The Summary

### Carsten

Carsten said that at Decard they were just continuing with work, building out their network, and so on.

As a personal project, he'd been playing around with BetterC on embedded systems. He'd encountered an issue with D's `size_t`: it didn't fit the 16-bit words on AVR, which caused problems with `.length`. It broke when trying to compile. He hadn't looked deeply into it.

I said I'd never heard of AVR. He said it was an old 16- or 8-bit processor. It was still used a lot. Arduino used it, for example, and LLVM had a backend for it.

He said it worked if you didn't use `.length` on arrays or didn't use `foreach` and such. Then you could actually compile with BetterC, but you lost the advantages of BetterC over C.

He explained that AVR only had 8-bit registers. It could map them to 16-bits, but it couldn't do it with `size_t` because it was 32-bits. He'd given up and switched to RISC-V because he didn't want to spend the time on it.

I asked if they were still doing WASM stuff at work. He said they were. They were building a transpiler for WASM to BetterC because it was easier for them to use it like that.

### Vladimir

Vladimir said he'd had problems with a couple of codegen bugs, but he'd found workarounds since the last meeting. Aside from that, everything was okay.

### Luís

__Issues__

Luís said Weka had fallen into four issues, two of which were related to codegen:

* [Lamdbas were being mangled incorrectly](https://github.com/dlang/dmd/issues/20234), which caused the resulting codegen to use the wrong mangle.
* [There was an issue with `__traits(compiles)`](https://github.com/dlang/dmd/issues/20545) guessing the wrong branch and generating the wrong code.

He'd uncovered another issue while rewriting their tracing system. It traced every function entry and function call. The new system was behaving differently than the old one. He found that [out parameters didn't call the destructor when it was assigned](https://github.com/dlang/dmd/issues/19109).

There was a discussion thread on the issue. Razvan had said there that the spec wasn't clear whether out parameters should call the destructor. Luís thought they should, but it was something we should discuss. He said out parameters influenced the reference counting of some objects. With this issue, reference counting was buggy.

Another issue was [a problem with typed variadic arguments and destructors](https://github.com/dlang/dmd/issues/17822). He said we could see it in two different ways: either the destructor was called twice or the postblit wasn't called. When an object in a variadic argument had a custom postblit, it wasn't called.

He said someone at Weka was trying to solve it, but the good news was they'd found workarounds for everything. The `__traits(compiles)` issue was very hard to debug. They'd had to run DustMite on their code and wait a few days to find it.

Luís said he'd submitted a PR for another issue, though he didn't have a link handy since it was on his personal account and not his work account. There were some cases when a template failed to instantiate in which the errors weren't propagated into a supplemental error message. It ended up saying the template failed to instantiate, but not where. If that were clearer in every case, those errors would be easier to debug.

Dennis asked if the lambda mangling issue was manifesting with separate compilation. Luís said he wasn't the one who had reported it and wasn't that familiar with it, but he thought that yes, it was two compilation units. Dennis said that had been [fixed in a recent PR](https://github.com/dlang/dmd/pull/17102), at least the reduced example.

(__UPDATE__: Dennis has since told me the following: 

> It's a partial fix. It fixes the most common cases, but there's still cases where the location isn't unique (e.g. because of mixin) where it falls back on the old unstable logic.

)

Luís said another thing on their side was that they were using an old compiler. So there would be cases where old issues were solved for later compiler versions. They were in the process of updating the compiler to the latest version. He knew that Johan had run into an issue upgrading to the latest beta, but he didn't know what it was. They were on LDC 1.30 and wanted to upgrade to 1.40, which was a lot of versions.

Walter said we had move constructors mostly implemented now. Once they were solid, he thought it best to abandon postblits.

Luís said they used postblits a lot. The spec said they should be using copy constructors instead, but they weren't, so that was an inconsistency. But he thought the problem would be solved if there were just one way to do it. So he agreed that move constructors would probably solve it.

Walter said he had no plans to improve postblits. I noted that Weka's use case was one of the motivations for move constructors anyway.

Regarding out parameters, Luís asked if there was any consensus on whether they should call destructors. Átila said his instinct was to say they should, but he didn't know what kind of second-order effects could come from it. Walter said he wasn't familiar with the code that caused the issue, so he couldn't say for sure. He'd have to look into it. Luís said if we decided they shouldn't, then we should either warn the user or put it in the spec. 

Walter asked if Luís was saying that the destructor wasn't called before the object was reinitialized. Luís said he was. Walter said that wasn't how out parameters were intended to work. They were supposed to be given uninitialized objects.

Luís said if we went in that direction, we should enforce that the object was uninitialized and make it clear in the spec. Walter agreed the compiler should detect the error, but out parameters shouldn't be passed objects that expected to be destructed before they were reinitialized.

Luís said he understood. He thought they mostly had that issue in their RPC infrastructure. In cases where they upgraded and downgraded between versions, if they changed the out parameters to ref or something, then the person who wrote the code was probably assuming some kidn of behavior that wasn't happening. It wasn't something he approved of. He'd found it when he created some tests.

I was surprised that it wasn't in the spec. Luís reiterated that Razvan had mentioned in the issue that it wasn't clear. Átila said the spec said only that the parameter was initialized with `x.init` upon invocation. Walter said initialization implied it was uninitialized to begin with.

Luís thought that in `@safe` code, we should warn if something that wasn't void initialized was passed to an out parameter. Walter said the problem with doing that in the general case was that it required data flow analysis. Luís said we could still catch some cases, even if others were silently allowed. We could mitigate some issues.

Walter understood the point behind that. He worried that if we only covered some cases, people would assume it would always work. They'd pass something it couldn't detect and wonder why they had a problem.

It was the same sort of thing with statically checking for null pointer dereferences. People wrote examples saying the compiler should track whether a pointer was null. He thought the compiler probably should, but if we implemented it, someone would have cases where it wasn't checked because of complex flow control. That would be very unexpected behavior for them. That was why he'd been reluctant to implement it. It gave a false sense of security.

Luís said it was something they'd considered implementing a linting rule for. It was probably easy, though, to at least give a warning about using out parameters in specific situations.

I said the docs should be explicit that anything requiring destruction shouldn't be passed to an out parameter. It was all too easy to reuse an instance for that. If it required destruction, you would have to handle it manually.

Átila agreed and thought we should explicitly say that destructors would not be called. Walter and Luís agreed.

__The new GC__

Luís said he'd spoken with Amaury at DConf about the new GC implementation regarding ARM. The new GC assumed a page size of 4K, but ARM had a different page size,  which could cause higher-than-expected memory usage on ARM systems. He wanted to follow up on the implementation's progress.

I said no one at the meeting was up to speed on that. It would have to be Steve or Amaury. I said I could put him in touch with them and he could ask them (and I did).

Luís said Weka had considered forking the GC to have some extra checks. This was a problem for them with scaling in terms of memory, basically scaling and the number of nodes they had on the same machine. Sometimes the GC didn't collect when they wanted.

Mostly this was their problem and not a GC problem, but if the GC had better interfaces for debugging it would be easier for them to understand the issue. For example, one of the things they did was to have some calls to the GC for a small allocation or a large allocation. That wasn't parameterized in the GC, so they needed to change it in their compiler fork.

They thought of forking the GC to have traces for debugging when there was a problem. He emphasized this was almost certainly due to their misuse of the GC in some way. He understood the new GC had some improvements for that. It was something they wanted to have.

### Dennis

SARC had been upgrading from 32-bit executables to 64-bit. Everything had been mostly okay. They'd had some trouble with 32-bit precompiled external libraries. Now they were trying to bind it all together with Microsoft's COM servers, which was a hassle, but nothing to do with D.

Everything was going pretty well.

### Robert

Robert had nothing related to work, but he said someone had let him know that he'd overlooked the Installer issues in the Bugzilla to GitHub migration. Those were now done. Unless anyone found anything else he'd missed, that should be all of them.

Walter said he'd noticed that there seemed to be a one-off between the issue numbers and the PR numbers. He was getting confused when switching between them as to whether he was dealing with the PR number or the issue number.

Robert said that it depended on the repository and, to a degree, on the project itself. They were just incremental numbers that happened to be lining up too close for Walter's comfort. He could create a couple of dummy issues and delete them. Aside from that, there was nothing we could do about it.

Walter asked if Robert was talking about new issues and PRs rather than the migrated ones. Robert said it was true even for the migrated ones. However, he'd used a two-stage API to create the issues from the migration, so they wouldn't necessarily be incremental.

He said the issues were created in parallel, and then another API pulled on them when they were done. And because GitHub was a highly distributed system at this point, they weren't necessarily processed in the order they'd been submitted. So there could be some reordering now for certain issues, though the difference wouldn't be large.

Vladimir said Robert was talking about migrated issues. He asked if Walter was talking about migrated ones or new ones. Walter said new ones. Now all of the issues and PRs were mismatched.

Vladimir explained that GitHub had a single numbering system for both issues and PRs. So if you took a URL to an issue and replaced the number with a PR number, GitHub would take you to the PR, because it knew that number referred to a PR and not an issue. GitLab had it the way you might expect, with issues and merge requests having different numbering systems, but on GitHub, they used one system. 

He said the only fix would be to create a separate repository just for the issues, but he didn't expect we'd want to do that. Walter said we definitely did not.

Walter noted that the instructions when submitting a PR still referred to setting things up for Bugzilla instead of with GitHub issues. Vladimir said there was an open PR to fix that. The tests were still failing.

Robert said someone had left a note in Bugzilla that the issues had been moved. He wasn't sure who that was. Vladimir raised his hand, and Robert thanked him.

Walter thanked Robert for putting in the work to get the migration done. He thought it was going to be great.

Luís gave Robert a "big thanks". He said it was much better. Weka used GitHub for their issues, and when they referenced issues now it should all be tracked in the same place.

I asked if we still wanted to do what we'd discussed before and make the Bugzilla read-only. Dennis and Luís said it had been done already. Vladimir said you could still comment on things, as it was the only thing that couldn't be turned off, but you couldn't file new issues.

Walter asked if we could make the Bugzilla pages static instead of continuing to generate them. Vladimir said we'd have to involve Brad Roberts (he manages the server), but we could do it.

Walter said that would make the pages faster and also easier to archive. The Bugzilla issues were a treasure trove of things. If we made them static, then it became simple and cheap to host them and they couldn't be modified.

Luís said LLVM did exactly that when they migrated to GitHub. He thought you could even download the archive.

Walter suggested it would be possible to write a program using wget to pull the pages down and store them statically. I said that would make it easier to move them to our own server, too, when we got to that point. Luís said that since it was just plain text it should be very small.

Vladimir noted they were also mirrored to the forums. Walter said he had [a static version of the newsgroup web pages](https://www.digitalmars.com/d/archives/digitalmars/D/). He ran the bot on it once a week to refresh it.

### Me

I'd told everyone in the [December monthly meeting](https://forum.dlang.org/post/uvqbluehjkljthuhzsjm@forum.dlang.org) about the DConf dates of August 19-22. I repeated it here for the industry folks.

We'd wanted to go with September, but it was too expensive this year, so it wasn't a possibility. We'd thought about December, but that was no good for Symmetry. So we'd settled on August. I was waiting for the contract to be signed. It was virtually locked in, but I didn't want to announce it until it was finalized.

Vladimir asked where it was happening. I told him it was in London again, at CodeNode. As long as Symmetry was hosting it, it would continue to be in London.

## Conclusion
Our January Monthly meeting took place the following Friday, January 10th. Our next quarterly happened on Friday, April 4th, 2025.

If you are running or working for a business using D, large or small, and would like to join our quarterly meetings periodically or regularly to share your problems or experiences, please let me know.
