The D Language Foundation's March 2025 monthly meeting took place on Friday the 14th and lasted about an hour and twenty minutes.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Martin Kinkelin
* Dennis Korpel
* Razvan Nitu
* Mike Parker
* Robert Schadek

## The Summary

### Data Flow Analysis requirements

Rikki had put together [a document describing features he would expect enhanced DFA](https://gist.github.com/rikkimax/e99b60ca6c739acdddbf0de7d4d0a134) in D could enable and the problems it could solve. Because I had missed his email with the agenda items, no one had the opportunity to read it before the meeting. Rikki suggested Walter take some time to read it in the coming weeks. He needed to know what to do next. Once he heard something from Walter, he'd make the document public.

Walter said Rikki could go ahead and make it public now. There was nothing secret about it. Rikki said he just wanted it to go through Walter first before he released the final version.

I asked Rikki if he wanted to talk about it or move on to the next item. He was okay with moving on.

(__UPDATE__: Walter has since reviewed the document. As I noted [in the January summary](https://forum.dlang.org/post/ohspwyjwkccolhroqdjy@forum.dlang.org), he's skeptical it can be done without a negative impact on compile times and/or memory consumption. Rikki is working on an implementation to find out.)

### PR for external import path switch

Rikki had [a PR to add a new compiler switch](https://github.com/dlang/dmd/pull/20899). He said Martin might want to review it and let him know if it needed any changes. Otherwise, it was green and could be merged.

Martin said he'd glanced at it and it didn't look too interesting to him because it was just adding the switch. He hadn't thought about the field Rikki had added. He did command line handling specifically for LDC, so it had more LDC options rather than the homegrown DMD options. He wouldn't be able to use everything Rikki was adding for that.

He noted Rikki had done some stuff in the glue layer, in `e2ir.d`. Martin said he couldn't use that part. He'd have to implement it separately for LDC. The problem was time. At the moment, he was mainly focused on several interesting things happening to the LDC repositories. For example, getting D 2.111 merged after six months of work, and reviewing contributions like some PowerPC stuff and some Windows ARM64 stuff from Rainer Schütze, who needed to get VisualD working on a native ARM64 Visual Studio.

He said Rikki shouldn't wait on him. Rainer had done the `e2ir` stuff, so he would be the ideal person to review those changes. Rikki thought Martin should look at it since he would have to bring it to LDC anyway.

Martin said the problem was getting his priorities right. This feature didn't have any priority for him because Rikki was the only one asking for it. He thought it was a good idea, but he had so many other things that were more important to him at the moment that he couldn't promise when he'd get to it.

Rikki said that because he'd added a struct for import path information, Martin was going to have to update his command-line handling code anyway. Martin said he'd done that a couple of days ago. But as for this PR, he'd only had a glance. It looked okay, but he hadn't looked at the tests or done a proper review.

Rikki said he just needed to know the next step if Martin wasn't giving the thumbs up on it. Martin said he couldn't say much about it yet. It depended on him implementing and testing it in LDC. He asked if the feature was tested. Rikki said that it created a `.di` file for one of the existing tests so that it became external and he could test it with the switch. That worked.

Martin said that didn't sound too bad. Since it was targeting master, it was something for D 2.212. He asked if Rikki was personally blocked by this. Rikki said he was building a build manager for it.

Martin was surprised to hear that: "A build manager? So now we have dub, reggae, redub, and another solution?" He said he would have a real look at the tests and the changes outside of `e2ir.d` and if it all looked good, we could merge it.

### Bitfields sanity check

Rikki had [implemented a simple bitfields sanity check](https://github.com/dlang/dmd/pull/20848) because [Walter's attempt had failed](https://github.com/dlang/dmd/pull/20840#issuecomment-2647233181). He offered to turn bitfields on if it was merged.

Walter noted that he'd already commented a lot in that thread. He still felt all this fear about bitfields was unjustified. All the effort to flag things that may not be portable was just a nuisance. He was sorry that Rikki had spent the time on it. Nobody was going to want to add those annotations to make it pass a sanity check because it wasn't insane.

Rikki said we should prove that. Removing the check after it was added wouldn't break anything. Walter said it would make bitfields ugly for no useful purpose. Rikki argued we'd be proving Walter's point, and we could remove it later in that case without breaking code. But if we didn't add it and later found that we needed it, then we'd break code.

Martin said he wasn't sure he understood. He'd just had a glance at it. It looked like it just added new errors in case the bitfields might not be portable. For example, if the offsets diverged across platforms. He asked what the point was.

He saw it the same as Walter did. He'd been against extra checks like that from the beginning. They didn't buy us anything. If there was anything to prove, just look at C. They used bitfields with their compilers and platforms without any extra checks like this, and it all was fine.

Walter said C had been around for 40 years and no one had ever proposed a fix for this issue. He'd searched extensively and the only thing he could find about it was Linus Torvalds replying to complaints about it by saying it wasn't a problem.

If anyone did encounter a problem with it, they could just use explicit shifts and masks for that particular case. It was a theoretical problem, not one that was worth bothering with. It made the language uglier to add all these errors for perfectly good code. The use cases where a problem might arise were fairly unusual. He didn't want to make the language uglier with spurious errors for those rare cases.

Martin added that we already had a solution for portable bitfields in Phobos. It was ugly, but if you cared about portable bitfields, use that one.

Walter agreed. The solutions were obvious. If you didn't like the bitfields, you didn't have to use them. They were an optional feature. But when you did use them, they could make your code significantly nicer and more readable. But if you were afraid of those rare issues, just use `std.bitmanip` or shifting and masking.

There really was no downside to this. He reminded everyone again that struct field alignment changed between C compilers. Who was complaining about that? Nobody. D did the same thing and there wasn't a single complaint about it.

Rikki noted we supported adjusting alignments for packing struct fields. D's bitfield implementation didn't support that.

Walter emphasized he was talking about non-bitfield fields. The layout wasn't portable for the same reasons and nobody cared. Rikki said the programmer could set the alignment and get it packed to whatever they wanted.

Martin said it wasn't about packing. He said that the alignment of `double` on 32-bit Windows was 8. That was the only 32-bit platform using 8 rather than 4. Nobody was complaining about any portability issues across 32-bit Windows, Linux, and Mac.

Walter said bitfields would clean up the DMD source code quite a bit once we were able to use them. The code would look a lot nicer. One of the main ideas behind D was to be able to write nice-looking code. They were already there in the compiler and they worked. Just turn them on.

Dennis said the implementation currently worked with C, but it didn't always work with all D features. Field initializers weren't supported. `const` fields weren't supported. Binding it to a reference wouldn't mask it, so that was a bug. It also didn't yet generate correct debug information. So there were still a lot of holes in the implementation.

Walter said you couldn't make `const` bitfields. They weren't allowed in C. The reason was that the implementation had to write to the field, and if the field was `const`, it couldn't.

Dennis said that was the point of `const`, you could read it but not write to it. Walter asked how you could create it then. You couldn't initialize it because it was `const`. Jonathan said you could initialize one bit in there, but the rest of it was screwed.

Walter said `const` bitfields couldn't work. As for debug info, he hadn't put any time into it yet because he couldn't get anyone to merge the PR. Martin said that people could explicitly opt-in to bitfields now with the preview switch. Walter said that meant nobody was using it.

Martin felt it didn't have to be a perfect solution to be merged or to be usable for most use cases. First, he didn't expect many people to use it. Second, as long as it worked for the trivial cases, that was fine. Ideally, we'd have some pretty good coverage mimicking how we expected it to be used in practice. He didn't know anything about the existing tests, so maybe they were fine.

Walter said the existing tests were pretty thorough. They had uncovered bugs in the initial implementation. He'd fixed them and thought we were in good shape.

Walter said that people had told him they'd approve the pull request to take bitfields out of preview if he added this or that, so he'd added this or that and it didn't get approved. It kept coming back to the same old complaint about differences between C compiler bitfield implementations.

Dennis thought there had been a decision made at the last meeting. Walter said no one had approved the pull request. Dennis recalled the deal was that Walter would add checks for when people did stupid things. Walter said yes, but after looking into it, he changed his mind because it would be too ugly.

Dennis asked if we were then going to decide to let it go without the checks. Once we committed to that, then we should fix all the bugs and make it proper for release. He understood why Walter didn't want to do the work to cover all the bases if no one wanted to pull it, but if we agreed to move it forward, then we should fix all the remaining issues.

Walter was unaware of any issues other than the missing debug info. He'd fix it, but he didn't think it was a high priority. Dennis said it was a high priority. He added that there was [a bug in the issue tracker](https://github.com/dlang/dmd/issues/20365) showing that you could pass them to ref variables and it would just pass the unmasked, full bitfields field. It should probably be an error because you couldn't take a pointer to a bitfield. Walter said he hadn't been aware of that issue.

Rikki said that Átila should be looped in, as he'd had some thoughts about it in the last meeting. Walter said Átila had wanted the checks. Jonathan reminded us that Átila had thought the checks were a good idea because he'd had colleagues in Cisco who'd done stupid things with bitfields.

Jonathan thought the checks were a good idea in principle, but the more he looked at it, the more he thought that if people were going to be stupid with it, then they were going to be stupid with it and you couldn't really stop them.

Martin agreed. As he saw it, you wouldn't be able to compile any C code with ImportC.

Jonathan reminded us that the idea was to enable the checks in normal D code such that you got a compilation error if you did something stupid with the layout of a bitfield. You could then use `extern(C)` to turn off the error. ImportC would still work then, as it could just make the bitfields `extern(C)`. As he understood it, Walter ran into an issue with putting `extern(C):` at the top of the file. It couldn't work with struct fields, which meant you could only get it to work by putting the `extern(C)` directly on the bitfield.

I recalled it [had been a semantic issue](https://github.com/dlang/dmd/pull/20840#issuecomment-2647233181) in that you didn't have the `extern(C)` information when you needed it. Walter said that in order to know if `extern(C)` was in effect, you needed to have semantic information. The issue was that you needed to know if it was in effect before the semantic pass was run.

Rikki said his PR did the checks in Semantic 3, and it was pretty simple. Was this the offset we expected? If so, great. If not, that was an error.

Walter said that wasn't quite what he'd been thinking originally. His idea was to give the error if the layout differed between an `extern(D)` and an `extern(C)` bitfield. He didn't think Rikki's patch predicted all the cases where they might differ.

Rikki said it wasn't meant to predict. It was target-specific, so it sometimes worked. Walter agreed it would sometimes work, but it overlooked other cases, so it didn't thoroughly solve the problem. Rikki said it did if you threw CI at it, which was what we should be recommending anyway.

Jonathan said that if that was the sort of thing required for this to do what it needed to, then it wasn't going to do its job. The whole problem was people who didn't understand this well enough, who would be using bitfields with a specific layout and not understanding that it wouldn't work in reality. People like that wouldn't be doing the checks they needed to do in CI and would just shoot themselves in the foot.

Unless the checks were straightforward and clearly caught all the stuff they needed to, he didn't think they would help in practice. If they required the user to be diligent to make them work properly, then they wouldn't be catching anything anyway.

Rikki said we should table this discussion until Átila was here. Anything we could do to mitigate this, including D-Scanner, wasn't going to work, so it was kind of a now-or-never thing. Walter said D-Scanner didn't do semantic analysis, so it couldn't work anyway. Rikki said it could be doing it soon. He'd recently submitted a PR to fix an issue that was blocking the replacement of libdparse with DMD as a library in D-Scanner.

Walter said if someone wanted to write a linting tool that did this or added it to D-Scanner, that was fine with him. Rikki said it was fine with him, too, except he didn't think it was going to work at that level.

This took us into a discussion about linters and how people either turned them off or made unnecessary code changes just to get them to shut up. Then we talked about DMD warnings, how Walter had never wanted them but we'd ended up with them anyway, how Dennis thought Iain had removed a bunch of them, and how Jonathan would like to get rid of them entirely.

At this point, I asked if the decision on Rikki's PR for adding bitfield checks in Semantic 3 was a thumbs down. Walter said he was sorry about that, as he hated to say no when Rikki had put the work into it.

I said if this ever did make it in, it was going to be the most thoroughly discussed, kicked, and beaten feature we'd had yet. Jonathan said that was funny, as he thought very few people would even use it.

(__UPDATE__: The Bitfields DIP [has since been submitted and approved](https://forum.dlang.org/post/wstbhxovohvwokldbdeo@forum.dlang.org).

### BetterC switch splitting

Rikki said he'd started splitting the BetterC switch into its constituent flags. Not at the CLI level, but internally.

The first one was [allowing `ModuleInfo` to be disabled](https://github.com/dlang/dmd/pull/20947) on a per-module basis. The idea was you'd have a preview switch that would allow you to disable it for an import path or individual modules. It allowed you to interplay between libraries more cleanly without linker errors.

He said that if you wanted to see why he thought this was necessary, all you had to do was look at any of the dub recipes of any of the BindBC libraries. They had to use a BetterC configuration and a full D one. Why was that? BetterC was just a subset of D. All it was missing was some code generation features that it never used. It was obnoxious that we had to do that. Having a way to tell the compiler not to emit references to stuff would be useful.

Dennis asked how this would allow the configurations to change. Rikki said that you just wouldn't have them. You'd just enable BetterC and you'd be done. The build manager would give you the prefix switch before all your files and import paths.

Walter said he was looking at the PR and couldn't see any description of how it worked or what the user needed to know to use it properly. Rikki said there weren't any switches currently, just some refactoring under the hood. So nothing should be breaking.

Walter asked how it was based on import paths. What would the user see with this? Rikki explained that this would allow you to tell the compiler a module didn't have a `ModuleInfo` instance even if it did.

Martin said this feature already exited in LDC and had for a long time in the form of a pragma, so you didn't need to tamper with any command-line options for any dependent projects, because it was directly in the module. That meant if the module was compiled, it wouldn't generate the `ModuleInfo`. If it was just imported, then it didn't mean that the reference was skipped because the reference was lazy anyway. It depended on whether there were any module constructor cycles or ordering issues.

Martin had assumed Rikki was going to talk about adopting what LDC and GDC had done. GDC had started this. It still had the BetterC switch, but it also had four or five individual switches for `ModuleInfo`, `TypeInfo`, that sort of thing, so that you could use it for regular D code as well. Just skipping the `TypeInfo` or whatever in specific cases. LDC adopted it not too long ago.

That's what Martin had thought this was going to be about. But Rikki's PR was just adding extra fields to stuff. We already had the checks for the type information. So there was at least one internal flag already for emitting `TypeInfo`s. We did check if there was a `TypeInfo` symbol in `core.object` and didn't emit anything if that check was missing. So he didn't think we needed to have any extra flags in the AST to make this work.

Ideally, we should start with these command-line switches in order to get rid of the blunt `-betterC` option. At least give it the option to make it more fine-grained. Then maybe Rikki could come up with solutions to solve this specific problem he had.

Rikki said there was [a bug report in the LDC issue tracker](https://github.com/ldc-developers/ldc/issues/4779) about C++ classes needing `TypeInfo`. Martin said the C++ classes only needed the `TypeInfo` when they were being called from D code. That had nothing to do with what we were talking about here. Rikki said it actually did because it should be nulled out if it didn't have `TypeInfo`, which was what the prefix switch he was going to add would do.

Walter said if LDC had a simple solution for this, then we should look at that. Martin said the point was that nobody really used it. There was one occurrence where it was needed in LDC DRuntime. In other cases, it was just a minor optimization. It avoided polluting your object file with `ModuleInfo`s when you didn't need them because you had no unit tests or module constructors or destructors, so the compiler could skip them. But nobody was using it. It was just for very advanced cases internally, like for LDC or LDC DRuntime.

He thought there was another option in the form of a pragma for no `TypeInfo`s for all aggregates in a specific module which, again, was hardly used. Either way, he expected there would be little demand for this stuff. It was all specifically for crappy BetterC stuff.

I said that I couldn't remember why I'd originally added those `BetterC` configurations to BindBC. I couldn't recall if there'd been an actual problem. Rikki said there definitely was. Without them, the compiler was going to emit those references to `ModuleInfo` and `TypeInfo` that didn't exist in a BetterC program.

I thought that if the configuration wasn't there and the library wasn't using any DRuntime features, then it shouldn't matter if you were using it in a full D or BetterC program. If you were using DUB to compile your BetterC program, didn't the switch get applied to your dependencies?

Jonathan didn't think it worked that way because then the client's flags would override the library's flags. I said that may have been why I'd added the configurations. The `ModuleInfo` would be generated without the BetterC flag.

Martin said that was the heart of the issue. He thought we were slowly getting somewhere. It was about where these symbols were referenced from and when they were needed.

He took us back to the `TypeInfo` problem and gave us an example scenario when compiling a D project that depended on a BetterC library. The library would obviously have been compiled without any `TypeInfo`. If the D project used one of the library types in a situation that required the `TypeInfo`, like putting something into an associative array, LDC would emit the `TypeInfo` on the fly whenever it was needed.

DMD worked differently. It would try to reuse the `TypeInfo` that it would expect to find compiled into the library. But in this case, since the library was BetterC, there would be no `TypeInfo`. It was similar for `ModuleInfo`, but a bit more complicated.

Walter said that since we already had a working approach in LDC, someone should make a pull request to implement it in DMD. Rikki said LDC wasn't working for him. That was the problem. Walter didn't understand why it wasn't and asked Martin if he understood.

Martin said that if he recalled correctly, the issue was specifically about `TypeInfo` not being lazily generated for `extern(C++)` classes. `TypeInfo` was lazily generated for structs, but not for classes. You normally needed the `TypeInfo` for D classes anyway. It wasn't optional, it needed to be there. And there was no special case for `extern(C++)` classes. So again, this was only an issue when using libraries compiled with BetterC or extra switches in a full D program in situations where you were doing something that needed the `TypeInfo`, like putting a class in an associative array or something.

Jonathan said that sounded like it ideally should be fixed up in LDC so that it was consistent and then DMD could be changed to do the same thing. Martin said it wasn't as simple as it sounded. Jonathan said that could be the case, but ideally whatever we ended up with would be the same across the board.

Walter said he hadn't anticipated mixing BetterC code with D code. Jonathan noted that a lot of people used BetterC in ways it had never been intended to be used. He didn't think many people were using it to incrementally port C code to D. Many were instead using it because they just didn't like the GC. I added that some were using it because they liked C and they just genuinely wanted a better C.

Rikki said that from his perspective, it was all about a pay-as-you-go runtime. If you didn't need something, you could turn it off. When you did need it, you could turn it on again and it would work. Needing separate configurations as we did now was obnoxious. If he wasn't the one to fix it, that was fine, but someone should work on it.

Martin said this was a DUB problem. We had similar conflicts because of the inheritance of flags. For example, if you needed to enable DIP 1008 in your program, it needed to be inherited in your dependencies to work properly, otherwise, you'd see weird errors when importing a module. But in this case, the BetterC problem in the BindBC example, if you just had a way to say, "Compile the library with BetterC, but don't pass any other flags down", then we wouldn't have this problem.

Walter said it sounded like LDC's approach of emitting `TypeInfo` only when needed was the way to go. Martin explained a bit about the `pragma` to skip emitting `TypeInfo` or `ModuleInfo` for a specific module, and a bit of history on why lazy emission was implemented.

I thought Átila should be present for any final decision we made on this, as he was a big proponent of a pay-as-you-go DRuntime. Jonathan thought we had a general agreement on that, just not on the details of how to do it. I thought Átila should be part of that discussion.

Walter said he'd love to see a PR from Martin to implement lazy emission for DMD. Martin shook his head "no". I speculated that it could be a SAOC project. Walter said that could be interesting. I suggested that someone who understood the details could write it up.

Walter asked if Martin could submit a bug report that linked to the bits of LDC that implemented it. Martin gave a rundown of the implementation details. He said he could probably come up with a simple description, but that we shouldn't expect too many pointers to LDC code, as that was quite specific to LDC. He'd put it on his TODO list. Walter thanked him.

### A couple of DIPs

I noted that we were finished with the agenda items and had only just gone past an hour. I said that Rikki had wanted everyone to know he had [a PR for his coroutines DIP](https://github.com/dlang/DIPs/pull/251) and was ready to move it forward. (Also see [the DIP Development thread](https://forum.dlang.org/post/asbfsajrtasfqewzbsce@forum.dlang.org)).

(__UPDATE__: I merged the PR on April 2nd and assigned it the number 1050. I started working with Rikki to revise it but ended up putting it aside for a while. We'll pick it up again and submit it to Walter and Átila before the end of this month.)

Walter asked if I'd had a response to his feedback about Quirin Schroll's [Primary Type Syntax DIP](https://github.com/dlang/DIPs/blob/master/DIPs/DIP1049.md). I said I had sent Walter an email asking for some clarifications before I forwarded the feedback to Quirin.

Dennis asked if this was the DIP we had discussed in previous meetings and if it was going to be accepted. I said it was the same DIP and that Walter and Átila had conditionally accepted it. The current email discussion with Quirin was about the changes they requested and whether he was willing to accept them. Walter explained he was primarily concerned with Quirin's approach in the parser to disambiguating types vs. identifiers within parentheses and some potential issues that arose from that.

I told Walter I expected he and Quirin would need to have a chat to get onto the same page.

(__UPDATE__: I scheduled a meeting where Walter and Quirin hashed it all out. Quirin agreed to implement Walter's suggested disambiguation approach in the parser and see how it worked out. He said it would probably be a while before he could get back to us with the results.)

### Miscellaneous

__DConf registrations__

I said I was waiting for word from Weka about my funding request. Part of it would be allocated to DConf. Everything was more costly this year. Even though Symmetry had given us a generous budget, we'd be unable to avoid going just a little bit over. That meant there was no room in the budget for speaker reimbursements.

If we got what we needed from Weka, we'd be in a better position to cover all the reimbursements, and I'd be able to reduce the registration rate.

(__UPDATE__: Weka did approve the funding level we needed and [I was able to launch registrations](https://forum.dlang.org/post/mcqialzvnlianturflqz@forum.dlang.org) at a lower rate this year.)

__DMD ARM64 backend__

Walter said he was making progress on the AArch64 code generator. He was having fun going through the test suite. It would die there, he'd look at it, fix it, and then it would start successfully compiling a whole new section of code.

He knew the initial version was going to generate rather poor code. His emphasis was on generating code that worked, not on optimizing it. At some point, he'd need to spin up a CI for an AArch64 for DMD. He thought Dennis would know how to do that. Dennis said he had no idea.

Martin said it was easy. GitHub Actions had supported it for a month or so. We would have an easy solution for Linux AArch64 as a native machine for running CI tests. For a start, we could use LDC to generate a native working AArch64 compiler and then run the DMD test suite and everything with it. It should be doable in five YAML lines. He could take care of it. 

Walter thanked Martin. He hoped to have a working compiler by the summer.

__GSOC applications generated with AI__

I said that although Dennis wasn't old enough yet to have Old Man Rants, he'd had one in an email about the AI-generated applications we were receiving for GSOC. I wondered if we had a policy for that.

There'd been a discussion about it on the GSOC organizations mailing list. Some organizations were setting policies for it. In some cases, they were choosing to reject anything that was obviously written by AI. There had been a flood of them.

Robert said we should reject them. If you had to use an AI to dumb down your fluff into a bullet point thing you could have written in 10 minutes, then what was the point? He wouldn't even ask them to rewrite it. He'd done that in other contexts, telling people he wanted to hear from them and not AI. But in this case, we should just reject them.

Walter thought that if someone was too lazy to write up a proposal, then they weren't going to implement anything anyway.

Razvan didn't think this was something we should care about. He had been surprised by the amount of interest we'd had. He'd personally received around 20 emails from different people about our proposed projects. Some had also contacted him on Discord. One guy had flooded the zone with an email, a forum post that mentioned Razvan's name, and a DM on Discord.

He hadn't thought too much about whether they were written with AI or not. The important thing was that we wanted the students to do some work before accepting them. Typically, the way GSOC worked was that you accepted students from whom you'd had prior engagement.

So if they put in some PRs and we could see that they knew how to get the code base compiled and actually do stuff, then we could select them that way. If they were using AI to apply, maybe it was because they weren't confident with their English ability and were using it to polish their writing. He didn't see a problem with that.

Walter said he'd leave it up to Razvan and Dennis. They were running it, so they could decide how they wanted to do it.

I said someone had emailed me and I sent them on to Razvan. I'd also noticed an uptick in Discord activity. Rikki said there'd been quite a bit of active moderation on Discord. He'd even had to disable one of the spam protections.

Jonathan said that most of the people coming into Discord for the GSOC stuff were just saying, "Hey, I'm interested, where do I go?". He didn't think they were really engaging with anything in D at the moment.

I said I'd noticed at least one on Discord who had been working on the templated hook stuff. Jonathan hadn't realized that was one of the GSOC people. I said I'd also recently received an email out of the blue from someone wanting to apply for the Community Relations Assistant role that Max had filled for a year. I'd replied that he was a few years too late.

__GitHub changelog generator__

Dennis asked Robert if the GitHub changelog generator was usable yet (there had been some problems with it after we migrated our issues database from Bugzilla to GitHub). He'd seen that it failed CI, but it may have been the CI being unreliable as it often was.

Robert thought it was. He was only aware of one thing that was missing. There were two macros defined that turned an ID into a GitHub URL similar to the one on Bugzilla. Because there were different issue trackers for different projects, there needed to be multiple macros. He wasn't sure where to place them, but they should be trivial one-liners.

Jonathan said most of the Ddoc macros lived in the dlang.org repository. He'd have to dig into it to see where. Robert said he'd looked into it and asked Iain about the location, but he didn't recall if Iain had replied yet.

One other thing to be aware of was that the GitHub API didn't distinguish between issues and pull requests. He wasn't sure if it was the right way to fix it, but the generator now checked the JSON for something that only existed on pull requests to filter them out. If that wasn't working, someone should scream at him.

__Publishing a paper__

Razvan reported that Teodor Dutu needed some publications for his PhD. The two of them were actively working on a paper in which Teo would present the work he'd been doing on templating DRuntime. The narrative was that it was enabling the use of a more modern language on embedded systems.

There was a test suite that was being used on embedded systems. Razvan and Teo were going to use a GC that was based on `malloc`. It meant manually freeing, but you'd still have a runtime that was smaller than 16MB, the typical size of DRuntime. They should be able to save more space with the hooks.

They were getting a student to port the code from C to D. If that worked out and they got some good results, they might be able to publish the paper at a good conference.

__Compiler daemon__

Robert reminded us how in the past he'd pushed Walter about running the compiler as a server that ran in the background, storing the AST and intermediate results. He said the TypeScript people were now doing something very similar. He linked to a specific timecode in [a video of an interview with Anders Hejlsberg on YouTube](https://youtu.be/10qowKUW82U?t=2691). He suggested that Walter watch it.

He said the interesting thing was that now with basically infinite amounts of RAM, you could cache quite a few bits that became interesting not just for increasing compiler speed, but for code completion.

### Conclusion

This brought us to the end of the meeting. Our next one was a quarterly that we held on April 4th. Our next monthly was on April 11th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
