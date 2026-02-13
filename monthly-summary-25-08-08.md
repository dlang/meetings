The D Language Foundation's August 2025 monthly meeting took place on Thursday, the 7th, and lasted about an hour and twenty minutes.

## The Attendees

The following people attended:

* Rikki Cattermole
* Jonathan M. Davis
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Nicholas Wilson

## The Summary

Before we got underway, I had some admin details to discuss related to SAOC projects and mentors, and reviewing DConf slides. Once that was done, I noted that I'd only received two agenda items, one from Nicholas and one from Walter, then I handed it off to Nicholas.

### A new undeaD tag

Nicholas reported that Nick Treleaven needed a new release of undeaD to fix something in Phobos. He wasn't entirely clear on the process for undeaD releases.

I asked if undeaD even had releases. Jonathan thought the whole point of undeaD was just that it was there and we weren't really doing anything with it anymore.

I checked, and the last release tag was 1.1.8, created three years ago by Petar Kirov. Mathias volunteered to create version 1.2.0 and call it a day, assuming the CI passed first. Nicholas said he was happy to do it if nobody else did, though he wasn't sure he had the privileges yet. I told him he should have full privileges as the PR & Issue Manager and would make sure he had them if he didn't.

__UPDATE__: _Mathias tagged 1.2.0, and Nicholas now has all the privileges._

### Merging the Bitfields DIP

Walter said [the Bitfields DIP](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1051.md) had been approved, and there was a PR to pull it. The bugs that mattered most had been fixed, so it was time to merge it and get it over with.

Nicholas noted there were some outstanding library bugs related to [printing bitfields with writeln](https://github.com/dlang/phobos/issues/10840), but in terms of language features, there was nothing blocking the merge.

Dennis disagreed. He thought it would be nice to fix the writeln issue first. The situation to avoid was people reading that bitfields worked, trying them out, then having writeln break and thinking the feature was half-baked. That was an old complaint about D, and he thought we should take it seriously. It was one of the most basic things. If that didn't work, people would consider the feature completely broken from the get-go. Átila agreed.

Walter thought it was strange that it wouldn't work because the result was an integer. Why couldn't writeln handle an integer? Dennis explained that writeln saw the fields were overlapping, like a union, and didn't know how to print unions because it couldn't know which union member was active. In existing code without special checking, it would see the fields like a union of every bitfield.

Jonathan said it sounded like writeln had protection against reading all the values in a union because that was bad news. When it detected an overlap, it barfed. That same checking would detect bitfields and barf for the same reason. Presumably, whatever check was there for overlap would have to be altered to recognize bitfields and print them properly.

Walter pointed out that you didn't actually pass a bitfield anywhere because the compiler converted it to an integer value. Jonathan said writeln had to do some level of introspection on all the members. It sounded like it was checking offsets to make sure there was no overlap because it didn't want to read multiple members from a union. Bitfields would look the same even though they weren't.

Jonathan thought it was probably an easy fix if you could find the piece of code doing it. One of us would just have to take the time to track it down and fix it. Dennis noted that Iain had already posted a diff with a lazy fix with the caveat that it was incomplete because it didn't account for every case. 

Dennis thought a more general problem was that whenever you wanted to generically iterate over the fields of a struct, and you just iterated over its `.tupleof`, you wanted to assume that it was safe to access every member. Currently, any code doing that had to take special consideration for unions, because they appeared as just members of the tuple. If you didn't check `.offsetof` and `.sizeof`, you'd get weird things with overlap. He often wrote that sort of code, assuming no unions.

He wondered if the compiler or library could provide a kind of `.tupleof` that merged all overlaps into one special thing. Walter said there were attributes to pick apart a bitfield. Dennis argued that every call site would have to remember that and take it into account that when just iterating over a `.tupleof` you had to check the offset and the bitfields offset and the `.sizeof`... You had to do this whole algebra to figure out what you were actually looking at. Walter said he understood the problem now.

Rikki proposed three new traits: one to get all methods, a second to get all fields that didn't overlap, and a third to get all fields that did overlap. Dennis asked why those were needed. Rikki said it was all about taking an existing type and getting only what you needed out of it. Right now, it was more expensive than it needed to be. If you only needed a few fields, you could just get those and do it more cheaply than now. That would solve this problem because you could just ask for only the fields that didn't overlap. No runtime checking or anything.

Dennis said that sometimes you did want everything. Like, writeln wanted to go over all the normal and overlapping members in one go. If you had two traits, that would be two for loops, and you'd have to position things, try to figure out where the overlapped fields were in relation to the normal ones.

Jonathan expanded on that, saying that `.tupleof` gave them in order. If you had to have separate ones, then they wouldn't be in order in the same way. He didn't know what a good solution would be.

Walter thought maybe we could just make it a compile-time error to call writeln on a union. Átila said that was probably too draconian. Rikki noted that writeln had a fallback for unions, where it just printed all the fields. The problem was that the fallback didn't understand bitfields.

Rikki suggested we put them order-wise in an alias sequence of alias sequences. Nicholas said you'd still have to intercalate them.

Jonathan said the alias sequences would all just flatten out. You couldn't have any depth to them. And arrays wouldn't work because you needed them to be aliases. Rikki disagreed, saying they would be strings, not symbols. Jonathan said that `.tupleof` gave you aliases. `allMembers` gave you strings. Rikki said this would replace `allMembers`. `.tupleof` was just an alternative way to do it.

Jonathan said the issue here was `.tupleof`, not `allMembers`. You already had to do all kinds of funky things with members. You had to filter out all the different possibilities, like static this and that, enum, whatever. `.tupleof` just gave you what was directly in the struct. That was mostly simple, but unions screwed with it because that gave you overlapping fields where you might need to ignore some of them depending on the context, or you might actually need to see all of them to, for example, list them all out, but still be aware of the fact that they were overlapping. With bitfields, you potentially didn't care, as they were actually separate even if they overlapped. He didn't know how easy it would be to abstract that in a way that you just didn't care.

There was a bit more back and forth about anonymous unions, writeln's implementation, potential fixes, etc. Ultimately, given that Iain's fix was there, the consensus was to incorporate it even if it wasn't perfect.

Walter said Adam would be rewriting much of this stuff for Phobos v3 anyway. Jonathan said we were nowhere near the point where we were ready to rewrite `writeln`, but it would have to be fixed at some point.

I asked if Walter wanted to wait on merging the bitfields PR. He said yes, at least until the writeln issue was fixed. There was no big rush. Dennis said there were two PRs needed: one for Phobos and one for DRuntime. Both of them would benefit from having the writeln issue resolved. Walter agreed to put the merge on hold until then.

__UPDATE__: _The PR was eventually merged and bitfields were released in 2.112._

### Editions

I opened the floor to anyone with anything else to discuss. Razvan asked what the situation with editions was looking like. He'd understood that Rikki had been working on the implementation, but had learned that wasn't the case.

Walter said Dennis had done some work to detect the edition in the module declaration, then had a couple of cases where it actually tested the edition. That seemed to be working fine. Walter had made a change to pick the edition off the command line and some other stuff related to that. The only thing left to implement was the edition checks for individual features.

I asked if the DIP was ready to be merged. Átila said he'd submitted the PR a couple of weeks ago. I said I'd take a look at it over the weekend.

Walter said we should decide if the first edition was going to be '2024' or '2025'. Átila did think that was important right now.

Razvan asked what the next step was. I said it was on me now. I'd go through the DIP to revise it. Any major changes would need Átila's approval as the author. Then once that was done, I'd submit it to Walter, and it would be on him to give the thumbs up.

Walter said once it was approved, we could start getting rid of crap in the language. Or at least declare some of it obsolete. Átila agreed.

Walter said he'd love to have the docs blocking out obsolete stuff and showing what was enabled in each edition. Razvan didn't think things would get simpler from a complexity perspective. Walter said they would from language point of view. Getting rid of cruft was of major importance. Maybe we should have separate doc pages for old editions.

Jonathan said there were multiple ways to run into it, depending on what you were looking at. In some cases, projects like this would have separate pages: a current one and then different ones for older versions. In some cases, they'd say 'this changed in this version or that version' throughout the docs.

I suggested we have a page for each edition listing the changes it brings. Then, in the rest of the main spec, we could have links to the edition pages where relevant. So if there were any changes to a specific feature, or if a feature were added in an edition, that edition page would be linked at that point in the docs.

Walter said that would work, but obsolete features shouldn't be in the main documentation so that users wouldn't be tempted to use them. He thought that would simplify the spec.

Rikki said that, as a simple solution, we could use the HTML tag that can hide or show a block. We could chuck obsolete stuff into one of those and hide it by default. Then we didn't have to completely overhaul the spec.

Walter said that sounded good.

__UPDATE__: _[The DIP was subsequently revised and accepted](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1052.md)._

### Tangeant: bitfields and bootstrapping

Martin asked if bitfields were going behind an edition. Walter said they didn't need to as they were purely additive. Jonathan noted they would break the introspection stuff in some cases. We could argue that it was minimal enough that we didn't care, but adding bitfields did change something.

Rikki said one clean split to decide if something should go behind an edition was declarations. Any new declaration like bitfields would go behind an edition. Otherwise, statements, traits, etc. would go into the base language.

Átila didn't think we needed an edition for something that was purely additive, that didn't exist before, only when we were deleting or changing things. Or adding things that broke something. Walter said editions were for breaking changes.

Jonathan agreed. The issue was that adding things sometimes would break things. In terms of type introspection, that was kind of true in general. Changes of any kind would risk breaking it. If we were super worried about that, then we'd have to make an edition for every change, and that was something we didn't want to be doing. He didn't know if it was a big enough deal with bitfields to care, but he could argue it was borderline. We could go either way on it. The core question in general should be, is this change going to break things if we introduce it without an edition?

Martin noted that introspection changes might break serialization frameworks. Bitfields in particular. They made a difference from a compiler perspective as well. For LDC, it meant changing the whole structure. We could no longer assume that every member had a unique, distinct address. You couldn't address members on a byte level now.

Rikki said we had to consider encouraging people to upgrade to newer editions and accepting breaking changes. New features were a way to do that. Átila agreed. I said that bitfields weren't really a marquee feature that would motivate people to upgrade. Walter was the one who wanted them most. Átila agreed again.

Martin noted we'd need to wait for years to use them in the compiler code base because of the bootstrap compilers. Walter lamented that it was frustrating being unable to use newer D features in the compiler source because of the bootstrap compiler. He said Iain was dependent on the bootstrap and asked if Martin was, too.

Martin said it wasn't for him. It was for all the people maintaining packages. They needed to be able to build the new stuff, so they needed a D compiler to exist in the first place. He'd been told that in most cases, they didn't use the version from the previous release. They did it all from scratch. That was why most of the time they depended on GDC, the old 2.076 version, because it was still completely based on C++. They started with that, because they only needed a C++ compiler, then they could start building newer versions up to the latest bootstrap version.

So we couldn't just arbitrarily say, 'Oh, I want to use bitfields now.' We'd have to bump up the boostrap compiler version, adding an extra step to the process for the maintainers, which could be quite tedious.

Walter said that meant we were stuck with it long term. Martin said we weren't stuck. There just needed to be a good enough argument to bump up the bootstrap version. We had to be really careful about it.

Rikki said a permanent fix to the bootstrap compiler problem would be dumping D out to C++. DMD was an excellent code base for it. Martin said then we'd be no better than Zig or whatever that started out transpiling to C and building with a C compiler.

Átila asked if Rikki was suggesting doing this just for the sake of the bootstrap compiler. Rikki said it was just for DMD so that we could create an updated bootstrap. Martin said no one needed that. Package maintainers were using GDC or LDC, never DMD.

Mathias said we didn't have this sort of default for nothing. He'd done the work to bootstrap on a new distribution, Alpine Linux, and he'd been very glad to have GDC available at the time. We couldn't use the shiniest D features in DMD, but he didn't think that was the end of the world. The tradeoff was worth it because it made it so much easier, so much more appealing, for people outside of the community to work for it and maintain it.

Martin said converting the D codebase to C++ wouldn't automagically make it able to target completely different targets. We'd still be stuck with it only targeting x86, and that was completely insufficient. Walter noted he was working on ARM support. Martin said it was going to take time and would perform very badly in the beginning. And anyway, there were other relevant architectures that DMD did not support.

Rikki said he wasn't talking about the DMD backend, only the frontend. Literally, just copy it over to a new target with the LDC glue code, and you had an LDC. You didn't need a D compiler to compile it.

Walter said the compiler could convert D code to C code, but it wasn't capable of converting it to C++ automatically. You could do it by hand, but who would want to make that effort?

We spent some more time discussing why transpiling the D frontend to C++ was not a good idea (huge project, bugs, test suite, and so on).

Getting back to the bootstrap version, I suggested it would come down to Walter getting together with Iain to determine what would be a good milestone at which we could upgrade. 

Walter suggested we could go back to the bootstrap version and add bitfields to it. He then said he could see us all shaking our heads, and that we obviously hated the idea. Átila said he didn't hate bitfields, but Walter was the only one who cared about them.

Rikki said he didn't see this ending well for us long term without dumping to C++. Martin disagreed. Clang was definitely written in C++. GCC was, too, nowadays. He doubted they came up with an idea to transpile to C and fix all their bootstrapping problems that way. Átila said they did not do that.

Martin speculated that Iain's answer to the bootstrapping thing would be aligned with the release timeline of GCC. If we looked at the currently supported major version of GCC, that should tell us which frontend version was important. Mathias said the latest GCC version was 12.5 in July and asked which frontend version it targeted. Martin thought it was 2.110.

Jonathan said there were two approaches to bootstrapping. One was to start with the oldest version and make subsequent builds through some set of versions until you arrived at the newest, or you had some sort of cross-compilation solution. He thought cross-compilation would be ideal. Otherwise, we'd have to go too far back eventually.

Martin said cross-compilation would have been his first suggestion as well, but from what he'd been told by package maintainers, they didn't work that way. They started on native systems and needed that to work. They had a two-step process: they started with 2.076, then built the current bootstrap. So if we added in bitfields, that would be a third step.

Jonathan said he'd be afraid that eventually there'd be so many issues cropping up, we'd be unable to build the old version anymore. Because it was C++, it should continue to be buildable in theory, but assuming you could continue to build something that was going to become 10, 15, 20 years old, eventually that became questionable. At which point, cross-compilation would be better. If we had a clean way to do that, maybe we could get people to do it more easily.

I said we weren't going to solve it in this meeting and suggested we move on.

### The proposed new release branches

Martin said he wanted to better understand [Dennis’s proposal to get rid of the stable branch](https://forum.dlang.org/post/ckfscuzcbkqfqxrdaicg@forum.dlang.org) and move to release branches with backports. He asked how backporting would work in practice. Considering all the potential merge conflicts that something like this would entail, would it be a manual process? Would we have some automated machinery in place to take care of easy merge conflicts?

He gave an example of opening a PR for the master branch that would need to be backported to 2.109, 2.110, and 2.111. So that was four commits to at least four branches. Did we have an idea of how to tackle it?

Dennis said that some cases would require manual merging, but the idea wasn't that we would maintain 2.109, 2.110, and 2.111 with continuous backports. It was just for the rare cases when there was a critical bug to fix, like something breaks with a new macOS release, or industry users are being hampered by something and can't upgrade yet for some reason.

In theory, we would still be releasing at the same patch cadence. The branch structure would just be different so that we didn't have to continuously choose between master and stable and merge them back and forth. It was currently a huge source of confusion. Which branch to target depended on the time and the kind of bug fix, and people had differing opinions.

Martin asked how it would be different. We'd been working on 2.112 for half a year now. Stable still hadn't been merged into master. So if he wanted to pick something in 2.111, like the performance regression for the array append in DRuntime, he'd target stable, the latest. So what difference would the release branches make?

Jonathan said that everything would go into master. Beyond that, it was just a question of whether and where you backported it. Martin got that, but who would do it?

Dennis said it would be the pull request managers. Or if the PR author had his own idea, he could do it himself. But a normal contributor who didn't know about any of this stuff could just target master. If we thought it should target a prior release, we could help them out and wouldn't have to ask them to rebase. That was the sort of thing people new to contributing always messed up.

Martin said the thing that mattered most to him was the latency. Most of the time when he was working on the DMD stable branch, it was because he noticed a problem in LDC. Maybe he merged a new frontend and noticed some issues. It was important for him that this stuff landed quickly in the branch he was interested in. Currently, he could target stable. As soon as it was merged, he knew it was up to date, and he could merge it into LDC. He just didn't want to be waiting weeks for something to get merged. He worried that it would be more work for the PR managers now.

Dennis said he didn't have all the answers up front. He was considering keeping stable as a pointer to the latest. He didn't know how easy it would be to maintain, but the stable branch might remain.

The most important issues to solve were the constant confusion, the divergence between master and stable where we forgot to merge stable back into master in time, and weird conflicts and issues that sometimes cropped up. For example, in this release cycle, he once attempted to build 2.112 and then couldn't make a patch for 2.111 again because stable had been merged into master, so there was no branch to build 2.111.

Martin said that was no problem for LDC. Everything was based on master. There was no stable branch. Then again, they only had a couple of contributors, so it was no problem. This was where tags would come in handy. You could always create a new branch for an old release based on the latest tag.

One last thing Martin wondered about was that associating issues fixed in a certain time frame with a specific release was kind of hard. If it was master, then at what point was the master stable? Dennis said his idea for that was to use milestones. Martin said he could then use milestones to say he wanted a fix backported to a certain version.

Rikki suggested that instead of backporting, Martin could do the opposite and target the old branch, then have a tag to put it into master. If he only had to go one release back, that seemed like it would work better for him.

Martin said that was what we currently had with stable. Periodically, we had these merges of stable into master, and that was automatically forwarded.

Mathias asked Dennis if he was interested in hearing about the release process that Sociomantic had. Dennis said he was open to it. He wasn't experienced with the release process and was still new to it. Iain was still taking it slow, so until someone else stepped up, Dennis was figuring things out and listening to feedback. If things didn't work, he wasn't trying to force one approach or another. He was suggesting ways to solve existing problems.

Regarding milestones, he wasn't thinking that every issue should actively target a release. But occasionally, if, for example, Walter said, 'Hey, I want bitfields in by 2.112,' then we could make a milestone for that. And then we could prioritize the issues we needed to fix before bitfields could be released and make sure they were ready for 2.112. Right now, it wasn't clear where we were with cases like that. Sometimes we had a train of PRs working on one thing. If a release started in the middle of that, then we'd end up with a half-baked release of that feature.

Martin asked how the changelog generator worked these days. He had no idea how it had worked with the old Bugzilla system. Did we select the version in some way?

Dennis said it was based on time. It collected all the issues from a period and concatenated them into the changelog. Martin said that, in that case, just working with the master branch as the main branch to target PRs made sense.

Martin then veered off into a suggestion about automating the CI artifacts for the release process, and he and Dennis discussed some of those details for a few minutes.

### Fast DFA update

Rikki said the fast DFA he'd been working on had caught its first production bug. Earlier in the day, he'd put his big 100k code base through it, running at 4.2 seconds. That was the same amount of time as without the DFA enabled.

Martin asked how he'd done that. Rikki said it was optimized. Walter said he should post about it in the newsgroup. Rikki said he would as soon as it was ready. There were still some bugs to work out. He was hopeful it could be merged in the next month. It hadn't been reporting any false positives. If any did come up, he could fix them.

It was running at the right speed. He'd caught two segfaults related to memory management. He thought one of them might be due to memory fragmentation. He was wondering if we should give DMD a new memory allocator for Windows to replace malloc.

Dennis asked if he'd tried running it on DMD's code, because there were plenty of segfaults in there. Rikki said he had it on the CI at the moment. It was currently ignoring the flag, but when building DMD with it, it was passing. He wasn't all that surprised by that, as the people working on DMD were generally highly skilled.

Dennis said there were still many issues with null dereferences. Was Rikki saying they were all gone? Rikk said no, the fast DFA was not a 100% solution. It wasn't handling indirections. It was so fast because it was just doing a small subset of things.

## Conclusion

We held our next monthly meeting on September 6th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
