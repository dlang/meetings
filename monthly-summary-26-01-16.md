The D Language Foundation's January 2026 monthly meeting took place on Friday the 16th and lasted about 50 minutes.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Mathias Lang
* Mike Parker
* Átila Neves
* Razvan Nitu
* Robert Schadek
* Steven Schveighoffer
* Nicholas Wilson

## The Summary

### Safely casting `shared` away

Rikki said that people had been trying for a long time to find a safe way to cast `shared` off of an object. This was kind of important, because if you couldn't do it safely, then you were basically in `@system` territory, which wasn't a good thing.

He thought there was a simple way to solve it using a new storage class, `onstackonly`. Any value marked with it would be stack-only and couldn't go into another variable with an equal or longer lifetime. It couldn't be put into return values, globals, pointers, or anything similar. It could be passed down the stack and put into `scope` things. He asked if the storage class made sense.

Nicholas asked whether this needed to be another storage class, or whether it was already covered by `scope`. Rikki said `scope` was very similar, but it didn't communicate the same thing.

Jonathan said that without DIP 1000, `scope` didn't do much. He wasn't even sure that what it currently did was memory safe, because without DIP 1000 it mostly took the programmer's word for it. With DIP 1000, extra checks were added, but without it, `scope` was kind of iffy. He also suspected that we'd need to see a DIP listing it out in detail before we could really understand what Rikki was getting at.

Walter said he didn't understand what it meant to cast `shared` off safely and asked Rikki to write up a short description. Rikki said he could do that, but he wanted first to determine whether this was something worth sending straight into development with a DIP.

Mathias said his understanding was that Rikki wanted to be able to cast `shared` away in a safe context. Rikki said it was. Mathias didn't see how it would be safe then if you passed it down the stack. Walter said the only way to do that was by putting a mutex on it and then casting away `shared`. Mathias agreed, but said that if `shared` was in `@system` territory, trying to paint `@safe` over it defeated the type system.

Jonathan said that if there were a mutex associated with a variable, and the language understood that association, then in theory there might be limited contexts where the compiler could implicitly cast `shared` away safely. That, to an extent, was what Andrei's proposal about `shared` for synchronized classes had been trying to do. It was very limited in what it could do. There would still be many cases where you'd have to use `@system` casts because the language couldn't understand every strange thing someone might want to do. It might be possible on a limited basis, but only if we could associate a mutex with specific variables.

Steve didn't know why D needed to support casting `shared` away in a safe context without something like `@trusted`. At that point, the programmer was writing `@system` code anyway. Even if the language could understand that a particular mutex was tied to one particular shared thing, the model would be extremely limited and still wouldn't be immune to problems such as deadlocks. This had been a chronic problem with `shared`, which was why the way forward was basically to say you couldn't access shared data directly. You had to first cast it away through a system function and build safe primitives around that using trusted code.

Jonathan said it wasn't like what we were trying to do with DIP 1000 in the sense that we had what you could currently do in the language and determine was memory safe, but in theory we could make more stuff memory safe with some extensions. The question was whether we could make enough additional stuff memory safe. With DIP 1000 there had been a lot of issues where it had been over complicated in order to get that extra benefit.

The basic concept of being able to say "here's a set of things we can do where shared is removed, and therefore, there's more code that can be memory safe without `@trusted`" was great. The question was whether any particular solution would be simple enough to be usuable while still allowing enough to make it the extra complication worthwhile.

Átila asked whether this would really be better than a library solution, since he had written one and thought its API was pretty good. The code that did the actual casting was `@trusted`, but it presented a safe interface. He didn't see why a new language feature was necessary. Rikki said that without a borrow checker, the library couldn't really ensure that the lock was held as long as there was any reference to the protected data. Átila said that was probably good enough.

Robert said that from what he had seen, the two solutions that had really been adopted for sharing data across threads were atomics and message passing. Everything in between was half-baked. It was a nice solution in no man's land and nobody wanted to go there. It didn't give you enough and was too complicated for what it gave you. So either you were very smart and used atomics and then fell on your face, or you used message passing. Then it was good enough.

Jonathan understood why Rikki didn't want to invest the time to figure out or implement a solution if he wasn't sure it would be accepted. But any solution like this would require a detailed DIP explaining exactly how it would work. If it was too complicated, it wouldn't fly. If it didn't allow much more than D could already do, that probably wouldn't fly either. Without an actual proposal, we could only talk in generalities, and that wasn't very useful.

Walter said that one of Rust's features was that it prevented data races. He thought that fell out of the borrow checker semantics. He said he had implemented a borrow checker for D and nobody wanted it. It was only a proof of concept and didn't cover every detail, such as lazy functions, but he had seen no point in continuing because not a single person had wanted the feature in D. If people didn't want a borrow checker, he didn't see a reason to go further down the path of trying to solve this by adding more language machinery. D should use mutexes or the other usual techniques.

Robert said there was an interesting story that had recently come up regarding Rust in the Linux kernel. The kernel developers had needed to mark some code as unsafe to do something they wanted, and then they ran into an issue where they corrupted memory. His point was that the cool things they wanted to do, they weren't able to do in safe Rust, so they basically slapped `@trusted` or `@system` on it and let it rip, and it ripped right into memory and they had a segfault and stuff.

He understood the desire for compiler-validated, memory-safe shared code, but roughly 60 years of computer science research had not yielded the perfect solution. He doubted there was one. The understandable solutions for the average programmer were atomics and message passing.

I said it sounded like the the consensus was against Rikki's proposal. Átila agreed. I suggested Rikki's next step if he wanted to pursue it would be to put a written proposal together. Not necessarily a full DIP, but something in the DIP Ideas forum.

Rikki said it looked more like this would become documentation for how the problem _could_ be solved, rather than the direction we actually wanted to go. Walter said it was still worth taking a shot at the problem. He didn't think Rikki needed a detailed proposal right away, just a general proposal about how it might possibly work, posted to the DIP Ideas forum, so people could react to it.

Átila said we had a library solution for this anyway. Just use it. Could we guarantee at the compiler level? No, but that was fine and would be good enough.

He thought we should make `preview=noSharedAccess` the default in the next edition, which was why he was trying to get the runtime to compile with it. We could move forward with that, then you were on your own with anything that didn't go through a library like the one that associated a mutex with a variable. Or you could do what Robert said and pass messages around or use atomics. He thought that would cover nearly all the bases. Languages like Pony had more ambitious solutions, but they paid for that with an incredibly complicated type system.

Walter then reminded everyone we had synchronized classes. Átila immediately said he wanted to delete those, too. Walter had been about to say that D could remove them in an edition. Átila reiterated that he wanted to nuke them. Jonathan generally agreed, but he expected some people would complain, including Funkwerk.

He also pointed out that `preview=noSharedAccess` still had issues that needed to be fixed before it could be enabled. One problem was that we had no way to distinguish between a reference to a shared object and whether the reference was also shared, especially with classes. That could become a problem with member functions. He wasn't sure how we could tackle that. As a general thing outside of member functions, he thought it required another helper struct to fix it, just like `Rebindable`. We needed to get those details sorted before we enabled it.

Átila said this was another consequence of classes being references by default. If the pointer were explicit, then `shared` could be applied to the pointer or to the type as needed. Jonathan agreed that the missing syntax was the problem. He said Amaury had suggested in the past that classes should have been written as pointers everywhere. They would still be reference types, but the pointer would be explicit. Átila agreed.

Nicholas asked whether the explicit `this` parameter that had come up in discussions about DIP 1000 and attributes could be a solution to this. Jonathan said that could potentially help. Steve said explicit `this` only changed syntax, not semantics. Jonathan agreed that an explicit `this` by itself wouldn't be enough, but coupled with explicit pointer syntax for classes, it could work.

I asked if Rikki was good with the outcome, and then we moved on.

(__UPDATE__: Rikki subsequently posted [a writeup in the DIP Ideas forum](https://forum.dlang.org/post/pwhgwshfxfunwtflhmfz@forum.dlang.org).)

### Diagnostic reporting message style

Rikki shared [a link to an example of some error message output](https://gist.github.com/rikkimax/f39721fe9f9377efe6896b30ad39c860?permalink_comment_id=5916804#gistcomment-5916804).

He said that “diagnostic reporting” was his name for a style of compiler error message for compilers that he thought originated with Rust. It was nice in that it went line by line using ASCII graphics to show exactly where error messages applied in the source code. The output was quite dense, but it was nice to see and often used for static analysis.

He said he would need this for the fast DFA engine once he added tracing support. He'd written some toy code that converted existing error messages and found that the they were pretty bad in this style. He proposed we add a new message style for this kind of output. @iskabon10, who had worked on error messages as part of SAOC, had offered to do it.

Robert believed the next GCC release, and therefore the next GDC release, would have an error reporting style very close to this. He gave the proposal two thumbs up. He'd been pedantic in the past for people to have a look at Elm's error messages, which he found even better. If some of that influence could be sprinkled in, he welcomed it. Good error messages mattered because error messages were basically the user interface for developers. We should make them awesome.

Steve seconded Robert. He'd spent about a year working on the new GC using SDC, and the error messages were atrocious, basically dumps of LLVM internal representation. Any time we could make error messages better, we should.

Jonathan expected there would be some bikeshedding over exactly how the output looked, but he didn't think anyone would argue against making error messages clearer. The tracing information Rikki was talking about would be helpful. For example, with `scope` in DIP 1000, one of the big problems had been presenting error messages without that kind of tracing information. If DMD could provide explicit tracing info for attribute inference or whatnot, he didn't think anyone would complain about getting additional information. In the worst case, it would be too verbose. We'd need an extra flag to make it spit out more stuff, but he thought it was a clear win.

Nicholas was all for it. Adding a new error style was the right way to do it, because that would avoid mucking up the whole test suite. Eventually, if the new style was well liked, we could merge it into the default style kind of like we did with context output and keep the test suite using the simpler style.

I asked whether Rikki had gotten what he needed. He said there was one more thing. Currently, the error message code had about four modules sitting in the DMD directory. He thought it was time to refactor that into a package, maybe something like `dmd.messages`, especially if we were going to add modules. That needed Walter's approval.

Walter said he had split `ErrorSink` into a separate module in order to stop the intricate interdependencies among the various pieces of the error message stuff. He didn't think putting things into a package would improve anything. What would improve things, and what he had already done in places, was to stop calling `error` directly and instead call through `ErrorSink`. Getting the whole compiler transitioned to that would make it much more practical to build an LSP server, because it would eliminate the global-variable problem. To him, that was more useful than putting modules into packages.

He asked which modules Rikki wanted to put into a package. Rikki said he didn't remember all of them offhand. Nicholas thought one of them might be related to a previous student project involving a generic interchange format similar to LSP. I asked if that was SARIF. Nicholas said he didn't know, only that it was an acronym.

Rikki said there were four related modules currently sitting in the DMD directory, and at least one or two more would be added for the new diagnostic style. Some things that were currently private would also need to be split into another module for utility code related to line coloring and stuff. In his view, it really needed to be in a package, and it would be better if that were done first. Walter didn't think it needed to be done first, since packaging was simple and could always be added later. He wasn't sure what the gain was.

Rikki asked Nicholas whether he could handle that aspect. Nicholas said he'd be happy to.

### Google Summer of Code

Razvan reminded everyone that GSoC's application period for mentoring organizations would open at the end of January. As usual, we needed mentors. Based on past experience, the people in the meeting were the most likely mentors because people from the wider community usually didn't step up. He urged us to think about whether we had work that could benefit from student help and could reasonably be done in eight to twelve weeks.

He would take care of the application, but we needed project ideas with mentors attached. In 2025, we'd submitted four projects and received only two slots. Typical organizations of our size had more like six to fifteen projects. The more projects we submitted and the more engaged mentors we could show, the better our chances of being accepted. If we applied again with only two, three, or four projects, he feared our chances would be slim.

He also reminded everyone that a person couldn't be the principal mentor for more than one project. Someone could be a secondary mentor on two or three projects while serving as principal mentor on one. But if we wanted to increase our chances, people needed to step up. He had made a forum post, but the pattern was that people often had plenty of project ideas and were much less willing to mentor them. The real need was for mentors.

(__UPDATE__: We very recently recieved word on our application status. I expect an announcement from Razvan is forthcoming. I don't want to steal his thunder.)

### Bugzilla issue links

Steve said some people had complained recently that when you went to the old Bugzilla page, it redirected to GitHub, but not to the specific issue that had been opened. Instead, it redirected to the GitHub DMD issue list. He thought this was really bad and asked whether there was a way to restore Bugzilla. A lot of bugs had already been closed by the time of the migration and weren't moved to GitHub. Without the old instance, all that history was lost.

I said he should probably talk to Vladimir Panteleev, since he was likely the person who had done the redirect. Steve said Bugzilla had been chronically going down. I said that then that would instead be Brad Roberts. I wasn't sure whether Vladimir had ever migrated everything from Brad. There'd been some talk about doing that. Vladimir was definitely the person to talk to about the status.

Walter said that, worst case, all Bugzilla postings had also been sent to him by email, so he had an email archive of every Bugzilla message. Steve noted that they were also on the newsgroup. I thought we had an archive of the database anyway.

Jonathan said the related issue was link preservation. The old links were floating around online in various posts, so ideally those links should still work. If the old site couldn't be restored, then it couldn't be restored, but ideally it would come back in a way that allowed those links to resolve.

Walter said that email wasn't the greatest way to access the data, but we did have it, and a program could always reconstruct a website from it. He saw that Rikki was shaking his head and asked why. Rikki said he had tried to use the newsgroup interface to look at issues for a specific case and couldn't find all the messages. He thought some might be missing because of age. Walter said he had all the messages unless his mail system had failed and caused him to miss a few. He'd been collecting them since Day One, though he hadn't done anything with them.

Steve would be surprised if the Bugzilla instance data didn't still exist somewhere. He assumed Brad still had it and just hadn't wanted to keep the instance running. It would be crazy if all that data had been thrown away. Walter agreed completely and said it needed to be online somewhere.

I was sure the data was still there and that somebody had simply redirected the site. The decision had been to put the Bugzilla instance into read-only mode and leave the links active.

I looked at the DNS. `issues.dlang.org` was a CNAME that pointed to `issues-d.puremagic.com`, which was Brad's domain, but was redirecting to GitHub. Going to the `issues-d.puremagic.com` directly ended up at the D auto-tester on `autotester.puremagic.com`. Mathias said that this was likely a different virtual host or server configuration issue. The redirect was probably coming from the server, not from DNS, because DNS couldn't redirect to a subpath of an HTTP URL. I said that meant Brad was the person we needed to talk to.

Walter said that, at the very least, perhaps he could download the Bugzilla database and we could set it up on another server. I thought Vladimir might already have it. I would email him first and ask. If he didn't know what was going on, then Walter or I could reach out to Brad. I told Steve the issue was off his hands now.

Walter said the best outcome would be to get the database, put it on one of D's own servers with the other infrastructure, and just leave it there.

(__UPDATE__: After some email back and forth, we got everything sorted. https://issues.dlang.org/ now points to a Bugzilla archive on one of our servers, courtesy of Vladimir Panteleev.)

### DMD's AArch64 backend and LLM-generated code

Walter said a recent newsgroup discussion about the release cadence had made him realize how critical it was to get an AArch64 version of DMD. He had taken a short break from working on the code generator to work on other parts of D, but he was now back to trying to get the AArch64 code generator working.

At the moment, he was taking every test case in the test suite and running it through the compiler to see whether the compiler crashed while trying to compile it. It was difficult to run the resulting programs because there wasn't yet a working AArch64 DRuntime, but he could at least try compiling them. Once every test case compiled without crashing, the next step would be to try building the compiler itself as an ARM executable. He was making progress, though he didn't know how long it would take.

He also noted that he had put an 80-bit floating-point emulator on the Summer of Code project list. He'd found that if he gave ChatGPT a prompt, it would generate an 80-bit floating-point emulator. That might make the project relatively simple, but it raised a question: if D incorporated ChatGPT-generated code into the codebase, what would the copyright status be?

Jonathan said one concern he had, though he expected many people would disagree, was that if GPL code was being fed into LLMs, then arguably anything produced from those models was derived from GPL code and should also be GPL. He didn't know what the legal outcome would be because it hadn't really been tested. In that sense, he thought it was fairly dangerous to use LLM-generated code unless there was a compelling reason.

Walter said he knew how to implement floating-point code and had done it before. It was just convenient to have ChatGPT do it for this kind of constrained problem, where the requirements were completely specified.

Robert disagreed with Jonathan. The genie was out of the bottle. From his point of view, whether outputs from LLMs inherited copyright constraints from their inputs was the trillion-dollar question, and if a court eventually ruled that they did, the economy would tank.

Walter said he would at least put a note on any generated code saying it was ChatGPT-generated and we wouldn't try to claim copyright on it ourselves.

Nicholas said D already had LLM-denoted sections from Vladimir, who had used Claude rather than ChatGPT in some recent pull requests. Rikki said he had used Gemini to generate code in DMD, specifically some math for the fast DFA engine's point analysis. It was simple stuff he couldn't be bothered to figure out manually, and he had personally verified it. But he drew a distinction between small utility code and something like an 80-bit floating-point emulator. Small verified code was one thing. A whole module was another.

Walter said the emulator didn't need to be complete. It only needed to implement add, subtract, multiply, divide, compare, and conversions. It didn't need to emulate every x87 function.

Steve said that ethically, with LLM-generated code, the more obscure the problem, the more likely the model had seen only a few examples of that problem being solved, and therefore the more likely it was to produce something close to a copy-paste solution. Asking it to make a checklist program was probably fine. Asking it to make an 80-bit floating-point emulator might be different, because there might be very few examples to draw from.

Robert told Walter that talk was cheap. He suggested Walter put $20 into Claude tokens, give it the last x87 floating-point test suite, tell it to implement the emulator and not change the tests, then make coffee and see what happened. The result might be surprisingly good, half bad, or at least a useful starting point. It would be a fun thing to try. Walter said he did have an excellent test suite for 32-bit and 64-bit floating-point code that had served him well when he implemented floating-point code by hand in the 8088 days. Robert said that made the experiment even better. For fun, Walter could spend another $20 having one model generate a test suite and another implement the emulator.

Rikki said he'd been using Google's Antigravity editor with D. It had successfully fixed bugs in the fast DFA engine. For about $15 a month, the paid plan gave higher limits, extra code features, and Gemini thinking. He hadn't yet run into any limits.

## Conclusion

Our next meeting was a January 23rd planning session focused on editions. We held our next monthly meeting on February 13th.

If you have anything you'd like to bring to us in a monthly meeting, please let me know.
