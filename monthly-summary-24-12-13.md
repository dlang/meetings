[Forum Post](https://forum.dlang.org/post/uvqbluehjkljthuhzsjm@forum.dlang.org)

The D Language Foundation's December 2024 monthly meeting took place on Friday the 13th and lasted around an hour and a half.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Ali Çehreli
* Jonathan M. Davis
* Max Haughton
* Martin Kinkelin
* Dennis Korpel
* Átila Neves
* Razvan Nitu
* Mike Parker
* Robert Schadek
* Steven Schveighoffer
* Adam Wilson

## The Summary

### DConf

I reported that we had almost finalized the DConf stuff with the event planner. The preliminary costing put us right at the budget Symmetry had given us. I was waiting for all of that to be finalized so I could make the preliminary announcement. (It was a few weeks until [I was able to do that](https://forum.dlang.org/thread/ighxvsogmzycjlonarsu@forum.dlang.org).)

I asked everyone if we wanted to have DConf Online in 2025. Átila asked if we needed it anymore. We'd started it because of the pandemic. I said I'd thought at the time that we'd continue it and have both DConf and DConf Online every year. But in 2024, we didn't have enough talks for two days of the Online version, so we only did one. I thought maybe we should still do it.

Steve said it almost seemed like we were trying to create talks just to have the online conference instead of having the conference for people with talks they wanted to share. It seemed more like work. Átila thought it had increased the quality of talks at DConf.

I said Steve had a good point. I had to push people for talks last time because I only received one unprompted submission. No one objected to skipping it this year. If there was demand for it later, we could try again.

### 2025 targets

Rikki noted that in the November meeting, I had brought up the need for us to be more professional. He thought we weren't looking good to the community right now and that we weren't meeting their expectations.

He'd considered things we were already committed to and thought we should set some target dates to complete them. If something was going wrong, we could find out about it, have it tracked by Razvan, and solve problems as they came up.

First, we had taken so much time on Editions that people thought it wasn't happening. Átila said he'd spoken to Razvan about it the day before. Rikki asked if there was any reason it hadn't been posted in the Development forum yet.

Átila said there were some things he needed to add to the document. He and Timon were in an ongoing discussion about. He needed to resolve that before finalizing the DIP. Other than that, it was 99% done. The real problem was figuring out who was going to implement it.

Rikki recommended the end of January as a potential target date. He thought Martin should be the one to implement it. He knew Martin had said he'd work on adding the external import path switch, but hadn't done it yet. That touched on the same area of the compiler, and if it were implemented incorrectly, there would be some problems with packages and things like that.

Martin said he'd told Rikki he'd implement the external path switch, but he hadn't said when. And no one else had requested it.

Rikki said the reason he was thinking of this was because the changes required for the external import paths and Editions would be similar. He thought Martin would prefer doing it because it would be a problem if they were done incorrectly.

Martin said it would be LDC-specific, so for that, yes, he was the number one choice. But his TODO list was huge. If more people were looking forward to it, his motivation to prioritize it would be higher.

Rikki said he could implement the switches in DMD because he knew about the simple stuff. Martin said it should be 50 or so code line changes. Rikki said he'd give it a go if Martin hadn't gotten to it by the end of January. Martin asked that no one give him any deadlines. He was reluctant to have deadlines on open-source projects.

Átila said he'd gone back through the comments on the DIP and there were fewer than he'd remembered. Most of them were from Timon, some of which Steve had replied to, but there were three primary concerns still outstanding.

Átila said Timon was concerned about safety bugs that we couldn't fix because they would entail breaking changes. We could fix them in another edition, but the concern was with the new edition calling into an old edition with safety bugs. Átila thought we'd have to live with that because he didn't know what else we could do.

The second thing was related to the command line and specifying editions for different modules. Átila thought that would entail a switch like `-mv`, but for editions rather than module names.

The third one was about having different APIs depending on the caller's edition. A library, for example, might want to have different declarations for its functions depending on who was calling into it and the caller's edition. A potential solution had recently come up in the form of a 'caller edition' trait.

I jumped in to say that because this was a DIP that Átila was authoring, approval required that we discuss it in a meeting before submitting it to Walter. We'd probably end up having a special session for it rather than doing it in a monthly. We could schedule that when the draft was ready. And the draft had to go through the Development forum first anyway.

Rikki next brought up tuples. He noted Timon wasn't in the meeting. Because Timon had been a lone wolf with tuples, Rikki thought it would be great if we could get some DIPs out of him by the end of 2025. He said Timon had an idea for tuple unpacking, but hadn't written a DIP yet.

Rikki's third target was related to AAs. He said Steve had already committed to removing the `TypeInfo` as part of the GC work. Due to some awful AA bugs, Rikki would like to see that kind of removal extended to AAs and the current implementation replaced with a templated one. He thought that was in line with what Steve was already working on.

Steve thought we needed to redo AA hooks, anyway. Rikki said the hooks themselves had to be converted to templates. Steve agreed and said there were a couple of hooks he'd thought were templated but actually were not.

Rikki said he'd like to see the AA compiler hooks moved into the same module as the implementation. Then anyone doing custom runtimes could copy and paste the entire module.

Razvan noted that Teodor Dutu had been looking into that, but was now busy working on his PhD. He didn't know how much time Teo had, but Razvan could contact him to see if he could look into it.

Next, Rikki said he expected we'd be getting to sum types soon. He said coroutines were his responsibility. He hoped to have a DIP in the queue in January.

(__UPDATE__: Rikki eventually submitted [the coroutines DIP](https://forum.dlang.org/post/asbfsajrtasfqewzbsce@forum.dlang.org), and I have since merged it into the repository. At the time of this post, he and I are working on revising it.)

Next, he said, was the mess with DIP 1000. The community chatter was that it could never be turned on with Phobos v3. We needed to find a way to resolve that either by removing it or finding a path forward on escape analysis as a whole. We had to make some decisions on this in 2025.

Átila thought the way forward was Robert's idea of making things safe by default by limiting what you could do. There'd been a discussion about it in the forums. There'd been some pushback on it, but he couldn't recall what that was.

Walter said he'd gotten a PR merged for `-preview=safer`. It implemented all the safety checks by default, but without them being transitive. He was curious because he'd heard nothing about it.

Adam said a lot of that discussion had been happening in the Discord. Átila remembered proposing something and Paul Backus had said something that made sense, but he couldn't recall what the objection had been. The idea was about moving DIP 1000 to `@trusted`. Adam thought that might have been in Discord.

Átila said he could email Paul. He remembered thinking that could be the way forward. Because then we'd get memory safety if people really wanted to do things that required `scope`. They wouldn't have to deal with that as we had to now. It had been a disaster trying to turn it on by default.

Walter said the safer preview didn't turn on DIP 1000. It was a bit of a modest step. He'd love to turn it on for compiler builds, but there was the bootstrapping problem. We couldn't seem to move to a more modern version of the compiler for bootstrapping.

I noted that we weren't going to settle this here and asked that we stick to Rikki's agenda item. We had several other agenda items to get through. If this other stuff needed further discussion, we could set aside some time for it later.

Rikki had no further targets to mention, but he emphasized he'd really like to see us finalize a decision on DIP 1000 in 2025 because he really wanted to do escape analysis.

### Enabling bitfields in D

Walter said [the bitfields thing](https://github.com/dlang/dmd/pull/16084) had been around for a long time. He'd implemented the suggested improvements, the extra traits and such. He didn't see any barriers to getting it adopted.

He'd been looking at the DMD source code recently and thinking again that a part of it would be much nicer with bitfields instead of the kluge scheme in there now. It had gone back and forth on the newsgroups again and again, and it was always the same arguments one way or the other. He thought it was time to make a decision.

I reminded everyone that we'd decided in a May 2024 special session that if Walter implemented the suggested changes, then we'd be good with passing it on to Átila for approval. So the question then was whether we were still good with it.

Rikki said all of his remaining issues with it could be dealt with by an attribute later, so he was for it as it currently stood.

Adam said he'd discussed it with Walter recently, and Walter had told him the implementation was the de facto standard. Technically, C bitfields were implementation-defined, but everyone used the same implementation. Adam thought we had to go with that. As long as we were following the same implementation as everyone else, then go for it.

Átila said that so much code would break if someone changed that, it wasn't even funny. There might be some weird compiler or weird architecture out there, but there wasn't.

(__UPDATE__: Walter submitted the DIP a couple of weeks before this post. He and I are revising it, and I will soon pass it on to Átila for the verdict.)

### -verrors=context as default

Dennis pointed us at a pull request that a SAOC participant had submitted. It did two things.

First, it made `-verrors=context` the default, meaning that under every error message would be printed the source code line and a little caret pointing at the error. It also updated the test suite with a massive 10,000-line addition because every test there now had an extra line of code.

Since the error interface was such a big part of the compiler, he thought we shouldn't merge this kind of PR willy-nilly. He wasn't sure the test suite should have all those lines, even if we did turn on context error messages by default.

Walter said we could add a line to the test suite that turned them off because they were just noise as far as the tests were concerned. Dennis agreed and said the test runner could set `-verrors=none` or something. Átila agreed, but said that at least one test should check the caret, etc.

Dennis said there were a few tests with `context` set explicitly already. He did notice that the line numbers were correct in the test output, but many of the columns were wrong. For example, it would say something about the "foreach index", but then would point at the start of the `foreach` tokens rather than the index.

Max said he was mentoring the SAOC participant working on this. He said Razvan had approved the PR, but Max didn't want to touch it without some deliberation first because it was such a large change.

He said that at this point, any language with a compiler written in the last decade did this. GCC had had it for a decade, and Clang had had it forever. One of the things that initially drew people to Rust was that it had elaborate error messages.

Razvan said he didn't think the test suite was a problem. We could automatically update what was expected from a test and didn't need to get in there manually. If the caret was off by one or something, we should fix it, but he didn't see anything as a blocker.

Walter said many of the failed compilation errors had 30 or 40 messages in the output. If `context` were turned on by default, that would become a blizzard that wasn't useful.

We didn't need context error messages in the test suite. We needed tests to show that context error messages were working. So he preferred turning it off for the test suite rather than merging that 10,000-line PR.

Jonathan said that regardless of whether we wanted the test suite to have this, we needed to merge the actual code changes without the changes to the test suite. Then we could review it properly and ensure it was working. We could go back later if we wanted to and enable it in the test suite piece by piece rather than all at once. which was just too much to look at.

Walter agreed. There were so many changes, you couldn't tell if anything was broken. Jonathan gave an example of a big change at work. They had decided to go at it piece by piece and uncovered several places where things would have gone wrong had they done it all at once.

Max said he'd initially asked his mentee to make the test suite changes because he wanted to ensure it all worked, not so much that it should be the default for the test suite. Someone else had attempted it before, but had segfaulted a lot. That was now obviously gone away.

Because of all the noise, he didn't think it should be the default for the test suite, but there should be a decent amount of exposure to the error messages for the compiler developers. If they were particularly noisy, that probably meant we should have better error messages. We could think of it as kind of a statement of our values.

He said that when working on DMD, you did tend to iterate by writing test cases. We should have some number of tests that exposed the full error messages so that the compiler developers weren't too detached from them.

Steve noted that Walter had said we should turn them on only to test the error message output. If you were looking for an exact error message when testing something else and expecting it to fail, then you didn't want them turned on for that. We could turn them on only when it was important to do so.

Max's point was that there was value in having it for more than just testing outputs.

Walter said it was just going to get in his way. He only had so many lines on his screen. When everything scrolled off the screen because the test generated 50 error messages, it became a maddening nuisance. It wasn't helpful to him at all when debugging. He knew it was the right default generally, but he dealt with failed compilations all the time. Having it on for all the tests would only be a major irritation.

I said it looked like the consensus was to accept having context error messages as the default, but to turn them off for the test suite. No one objected.

Dennis asked how to turn them off. There was `-verrors=none`, but that looked like there would then be no error messages. Max said they'd be painting the bikeshed soon. It would be something simple and short.

(__UPDATE__: [A subsequent PR](https://github.com/dlang/dmd/pull/20566) added `-verrors=simple` to disable context error messages.)

### Integrating DMD-as-a-library with D-Scanner

The student working on integrating DMD-as-a-library with D-Scanner had finally managed to get rid of D-Scanner's dependency on libdparse. He did it with [a huge pull request](https://github.com/dlang-community/D-Scanner/pull/964). The CI couldn't even run the tests.

Razvan had been reviewing the student's PRs for the past year, but this one was so big no one was going to review it. He wondered if we should go ahead and merge it once the tests passed after CI stopped being problematic and started running them. He brought it up in case anyone had other ideas. A 10,000-line PR was kind of scary to merge.

Steve thought we should be talking to the people who depended on D-Scanner for their projects, like IDE developers, Webfreak with code-d, or maybe the guy who was doing the Tree-sitter stuff. He didn't know who depended on what.

Razvan said he'd spoken with Webfreak about it and they were willing to merge it. Rikki thought we should just do it. We could always revert it if need be.

Steve asked if there were any open PRs on projects like dub that might be invalidated by this. Razvan said there shouldn't be. There might be some minor differences, but the behavior would largely be the same. Some of the checks would become more restrictive because D-Scanner wasn't able to do what the compiler did, but it would be able to once you hooked the compiler into it. Then it would catch more things. He thought that was a good thing.

Martin asked if we had a good idea of what the test coverage was in the test suite. Razvan said that they'd initially just taken the existing D-Scanner tests, but those were pretty slim. They discovered a lot of corner cases while they were implementing it and added quite a lot of tests as a result.

He didn't know if they were breaking any code out there, but in his experience, once they migrated a check from libdparse to dmdlib, the check got better. You would get more warnings, but that was because it was catching things that libdparse didn't.

Martin said he was just wondering because in his first glance at the PR, it didn't look like the test suite would be covering a lot of cases, unlike for DMD, for example, where the test coverage was more complicated. It was quite easy to break the test suite there if the implementation was off. But now that he had an overview, it seemed fine.

Razvan said that in testing Phobos against D-Scanner, they'd had to fix Phobos in some cases, but those were good fixes that made the code better.

He said it seemed the consensus was to merge the PR once the test suite was passing. Walter said yes and no one objected.

### Upstreaming Objective-C support

Rikki said that before Luna's work on Objective-C support was merged into LDC, she'd spoken with him about it and mentioned the need to upstream it into DMD. She'd done a good amount of work on it, but there was still more Objective-C support she'd like to have in the language.

I asked if upstreaming it to DMD would be an enhancement or replacement of the existing stuff. Rikki said it was an enhancement.

Walter asked if the PR was large or small. Rikki said Martin had been the one reviewing it and deferred to him. Martin said it was hard for him to tell. He didn't know what parts of it would be upstreamed. As far as he'd understood, it was just for LDC to catch up with DMD support, implementing all the stuff Jacob Carlborg had implemented years ago for DMD that had never been brought over to LDC.

Adam Ruppe had implemented something for the OpenD version of LDC. Luna had taken that, reworked it, and upstreamed it. It was a preliminary implementation, so surely wasn't 100% perfect. As far as he understood, it would be just a few small pieces that DMD was missing. He had no idea what those would be.

Walter said when Jacob first implemented it, they'd had a discussion. Walter had told him he'd like to see it in its own module with hooks into the rest of DMD to minimize disruption. With this new stuff, he'd like to continue in this vein to minimize the changes to DMD. They should be hooks to functions in `objc.d`. There should be almost no functions implementing Objective-C stuff in the rest of DMD.

Max said that if he'd been following the PRs and chit-chat correctly, he didn't think the DMD aspect would be very big. Walter said whether it was big or not, he wanted it to follow the same path. He thought it had been a successful approach.

Rikki said he would forward that to Luna.

As a coda to this discussion, Steve wanted to emphasize that it was important for DMD and LDC to have the same interface to Objective-C so we weren't forced to handle two different approaches in code.

Martin said this was mostly in the glue layer, so it was mostly about codegen. It wasn't like we could put a piece of code upstream in DMD and LDC could just use it. That was the main problem. The front end changes were quite minimal. It was mostly about codegen, which meant that every D compiler that wanted Objective-C support would have to reimplement all that crappy stuff. Even GDC.

Steve said his concern was with the interface. If there was a UDA in the LDC implementation now that wasn't in DMD, that needed to be upstreamed. We shouldn't let them drift apart.

Max said Steve was getting it. We should have any required UDAs implemented upstream in DRuntime and then shared. And we should be careful to make sure they were exactly the same.

### Safe values of void*

Dennis said he'd seen some issues popping up related to `void*` and safety. For example, people wanting the casting of a class reference to `void*` to be `@safe`. The question was, should it be?

On the one hand, you couldn't really do anything unsafe with a `void*`. On the other hand, people were like, "Void pointers are kind of low-level C stuff, so they should be @system."

He thought it would be nice if we could nail this down in the spec. Did void pointers contain safe values or not? Currently, there were some examples in DRuntime showing there were safe ways to obtain arbitrary void pointers. So by that logic, either every void pointer should be safe, or we should mark functions returning `void*` as `@system` even if they were currently `@trusted`.

Jonathan said passing `void*` around wasn't the issue because you were just copying a value. The problem was conversions. You wanted to be able to grep and capture all the places you'd created a `void*` because you were introducing something that you would then have to track down to have any clue that what you were dealing with was `@safe`.

Átila asked why. Jonathan said if you were doing anything to cast it back, you needed to know where it came from. Átila said that was never safe. Jonathan said he knew that, but you needed to be able to find the cast *to* `void*` so you could know if what you were doing was actually safe. Because you couldn't do anything useful with it anyway, except maybe print out the pointer value, you didn't really gain much by making it `@safe`.

Steve said it shouldn't be `@safe` because it was mutable. If you changed a void pointer, you didn't know what you were changing. Átila said you couldn't. How could you change a void pointer? Walter said you couldn't change it without converting it again. That was unsafe.

Steve said it was hard to think about, but he felt sure there was something wrong here if you found a way to modify the pointer itself. Walter said you couldn't even increment it in `@safe` code. Steve said there were probably ways you could accidentally change the type. Átila said there was nothing you could do to it without casting, and casting was `@system`. Martin said slicing was `@system`, too. He'd just verified it.

Rikki said we should ban `void*` from `@safe` entirely. The only thing you could really do was cast it to `size_t` immediately. Then you could print it out, compare it, whatever you wanted. Martin disagreed, and there were heads shaking in disagreement all around.

Martin said the one place he did use `void*` conversion was with associative arrays. When he wanted to have the key type be dependent on the identity of some object, `void*` did fine. This way, he prevented `opCmp` from being used when he was sure he had a single reference to an object and just needed to check the pointer.

Walter said if anyone could find a case where you could corrupt memory with a `void*`, we'd fix it. He didn't know of a case in the compiler where that was possible without going into `@system` code.

Robert said that after watching everyone discuss this and not agreeing, and because he had no idea what was and wasn't possible, maybe it shouldn't be in `@safe` code because it seemed too complicated to understand in the first place.

He referred to his DConf talk about how `@safe` should be easy. You shouldn't have to think whether it was `@safe` or not. He didn't know the language inside and out enough to say whether something with `void*` was `@safe` or not. That made him feel like we shouldn't put it in `@safe`.

Walter said that what `void*` did, and Windows used it this way, was it gave you a reference to a memory object that you couldn't do anything with. You couldn't break the wall to see what was there. It was a handle of a type you didn't know and you couldn't do anything with, and it was quite useful for that. You could still safely keep track of it. It was fine. It had uses that were perfectly `@safe`.

Robert said most things were `@safe` and, he knew this was stretching, felt safe. Whenever he saw a `void*`, it didn't feel safe. He was inclined to agree with Walter that they were technically `@safe`, but as a user, from a user's perspective, any time he saw a `void*` he'd be wondering, "Did we check that? Is it actually safe?"

Walter said that was a fair argument, but you couldn't do anything with it. If you tried to do anything unsafe with it, the compiler would kick you on it.

Átila noted that the Pony language had six different reference capabilities, which was basically a type of pointer with six different kinds. One of them only allowed you to make comparisons. That was basically `void*`. This was a pattern that was obviously useful.

In his opinion, the way to `void*` was `@safe`. If someone found a way to do something with `void*` that corrupted memory, that was a bug and we should fix it.

Steve said if you could copy one `void*` to another `void*`, you were going to break the invariants of the code surrounding it. A good example of this kind of thing was that we didn't allow people to set the length of an array without going through an interface even though writing an integer was just fine in `@safe` code. But doing that was going to break the invariant of the array. Similarly, we didn't allow setting the context pointer of a delegate in `@safe` code, because you could set it to something else.

Átila said it would then be pointing to garbage. You couldn't do anything with the `void*` itself. Steve said something else would do something with that `void*`. Walter and Átila both said, "No". Walter said it was blocked off. Steve said it was blocked off in `@safe` code. Walter said yes, you could not do anything with a `void*` in `@safe` code, that was the whole point.

Steve said that if the only point of `void*` was to compare pointers, then yes, but he thought a lot of code expected to cast it back to something else. Walter said that would be unsafe.

Steve said to consider a trusted function that assumed a `void*` was really a pointer to `T` and so cast it back to `T`. Átila said that was unsafe. Steve said you had to cast it back because that was the point.

Átila repeated that the way *to* the `void*` was `@safe`. Steve said his point was that every `@trusted` function had to assume that any `void*` could have been mucked with by `@safe` code because you could write to `void*` in `@safe` code.

Átila said the problem was when you cast it back. Steve said the point was that you now had to review all of your `@safe` code to see if it set that `void*`. Walter said you didn't.

Jonathan said you had to know where the value came from, otherwise you couldn't cast it back, even in `@trusted` code. Because if you screwed that up, then you had something invalid. Walter said when you cast it back, that was unsafe code.

Jonathan agreed, but in order to put `@trusted` on it and actually validate it was `@safe`, the user was doing that, not the compiler. You still needed to manually verify it to put `@trusted` on it. You had to be sure where that `void*` came from, otherwise you could be casting the wrong thing. Having `@safe` code able to change the `void*` at all, even if it was just pointing it somewhere else, risked it not being what you thought it was. Then you'd be doing something invalid with it in that `@trusted` code because you couldn't track all the places where it was changed.

Walter said Jonathan was right, but said to think about allocating memory with `malloc`. You could only do that in `@system` code and you could only free it in `@system` code. But in the meantime, you've passed the pointer into your `@safe` code. You assign another value to it and you pass it through a `@trusted` interface to free it. Then you'd crash your program. We couldn't disable that from happening.

If you were passing a `void*` into `@safe` code, you were going to have to ensure that it worked. If you were passing it to unsafe code and the unsafe code started mucking about with it and cast it to something else, that was your problem. That wasn't a language problem. 

He said it was the same thing Jonathan and Steve were talking about. The `@safe` code was correct, but if you misused it inside the `@trusted` code that you called, that was your problem.

Steve said `@safe` code had a set of rules that you had to follow. You could change integers and do certain things. `@trusted` code had to assume that all those things could have been done. Walter said yes, it was up to the user to make their trusted code correct.

Steve said that we had this set of rules to limit what we could do in safe code. The mechanism we had to prevent this kind of a problem was `@system` variables, which allowed you to say something that would normally work in `@safe` code shouldn't, because you were going to change it in a way that made it an unsafe value. Now you could know the `@trusted` code was invalid. The whole idea behind `@safe` code was to reduce the amount of code you had to review.

Walter said you had to review `@trusted` code. Steve agreed, but said you shouldn't have to review the `@safe` code. Walter said you should then have an interface that enforced that you could only pass a void pointer to `T` to the trusted code.

Adam said he had a few observations after watching this back and forth. First, he thought he understood now why the Windows API people aliased `void*` to `HANDLE`: precisely to avoid arguments like this.

He said that Walter had been very clear about what `void*` could and couldn't do. If we were to just ban `void*` in `@safe` code, well now we couldn't use the Windows API in `@safe` code, ever. That was a big loss for D right off the bat.

And if you changed a `void*` and passed it back to Windows, did you know what it would do? It would blow up. It would give you a nice little error code that you could check and move on. That was because the Windows API was doing for you what we were talking about.

At some point, you had to accept that you were going to do something that might have a side effect even in `@safe` code. At some point you were going to have to launch the nuclear missiles. The only way to write perfectly safe code was to write no code at all.

Dennis said he agreed with Robert's assertion that `@safe` code should be simple and easy. That was exactly why he wanted us to nail down that memory safety was about memory corruption.

If someone said that something should be unsafe, then he wanted to ask them to show him how it could corrupt memory. He'd often get arguments back about whether it was actually useful in `@safe` code. That was irrelevant.

It would be really simple if we just said that `@safe` was about whether you could corrupt memory or not. Then we wouldn't have all of these discussions. But we did keep having them because people had something they wanted to do in a part of their code base where `@system` code happened to be nearby. And when you got into those kinds of arguments, it got really muddy and complex. It became a subjective rather than a mathematical matter.

Robert said Dennis was right. He often made the mistake of misunderstanding safety as null safety, as Dart had, for instance. That was not what `@safe` was. He thought the argument Steve made boiled down to that. The only good path forward was to explicitly say that `@safe` code was not about null pointer safety, but about memory corruption. Those were two different things.

Walter said he agreed. Null safety was not a memory safety issue. He also wanted to point out that if we prohibited `void*` in `@safe`, then you could never allocate memory with `malloc` to use in safe code. You couldn't guarantee that the reference you were passing to a `@trusted free` function was allocated with that storage allocator and not something else.

Robert said he wasn't arguing that we shouldn't allow casting to `void*` in `@safe`. He was saying we needed to do better marketing about what void pointers in `@safe` actually meant. Walter agreed.

Ali said we should be protecting from unsafe stuff, but too many constraints were always bad. If you were crazy enough to do crazy stuff, then yes, you would shoot your foot off. But that didn't happen, so we shouldn't go to that level.

There was a digression about defining null safety and Walter's experience with Nicklaus Worth's PASCAL, which he said was so safe you couldn't actually write any programs in it.

Dennis said that null safety was a separate issue. His goal here was to nail down whether `@safe` was just about memory safety, or if it was also about protecting from footguns. Because people kept having discussions saying something should be `@system` wihthout showing how it could actually corrupt memory. He'd seen them come up again and again, and he just wanted something he could use to put an end to it.

Walter said `@safe` was all about memory safety and not about other problems, and Dennis was right that we should make that clear.

Rikki said he had his stance on `void*` because it was an entry point into memory unsafety. `@safe` should not be an entry to memory unsafety, even if it wasn't actually the cause of it.

Dennis said we had `@trusted` and `@safe` interfaces which were well defined. We had all the tools you needed to encapsulate your handles or void pointers in a struct that had `@system` variables or whatever. But in the current specification, you could not assume you could dereference a raw `void*` parameter.

I asked Dennis if he had an answer to his question about casting to `void*`. He said it seemed not everyone agreed, but with the current specification and definition of memory safety, it should be allowed and `@safe`.

Martin noted that implicit casts to `void*` were `@safe` for every pointer type. He wanted to make sure everyone remembered that. What we were talking about now was just explicit casts of class references to `void*`.

Átila said it should be `@safe`, because why wouldn't it be?

### Bugzilla vs GitHub

Rikki thanked Robert for handling the Bugzilla to GitHub transition and said there was something we needed to consider. Bugzilla was open with its privileges. Anyone could modify the fields. That wasn't the case on GitHub.

As an example, he didn't have any rights on the dlang organization. He was sure there were other long timers in the same boat. We should consider what we were going to do about people who should have rights to administrate the Issues, but only the Issues.

I asked if admin rights for Issues were separate from everything else. I didn't think they were. Adam said GitHub actually conflated the PRs and Issues.

Átila asked what concrete problem Rikki was trying to solve. Rikki said he needed the rights to administer Issues because that was what he'd done in the past on Bugzilla.

I said I didn't think there was anything we needed to change to our approach here. We could add people who requested it and who we trusted just like we'd always done. And we sometimes had invited people in. I didn't have any objection to giving Rikki access. No one else did either.

I didn't think we should open it up to anybody and everybody. We should handle it on a case-by-case basis as we always had.

Adam told Rikki that we had to be more careful with GitHub Issues management, because we were seeing a lot more spam. Because GitHub was so popular, there were a lot of bots written to spam it and attack it. We couldn't just hand out access like Skittles. We needed to know the people we were adding.

This reminded me of a discussion I'd had with Razvan and Dennis about cleaning up the GitHub access list. Several people on it weren't around anymore. We should probably prune the list. If they ever came back and wanted back in, we could add them again. I asked if anyone objected to that.

Walter said we should do it. He had taken a couple off the list, but he'd been unsure about the rest, so he'd just let it be.

Adam said there were a couple of long-gone people whose names kept showing up all the time on PR review requests. He could remove them, but he'd rather someone else do it.

I said I could do it.

## Conclusion

Our next meeting was a quarterly on January 3rd, 2025. We held our next monthly meeting on January 10th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.