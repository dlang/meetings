The D Language Foundation's June 2025 monthly meeting took place on Friday the 13th and lasted just under an hour and forty minutes.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Timon Gehr
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Robert Schadek
* Steven Schveighoffer
* Adam Wilson
* Nicholas Wilson

## The Summary

### run.dlang.org

Rikki said someone had asked him to bring up that run.dlang.io/.org needed updated libraries and compilers. He said now was a good time to remind everyone that infrastructure was important to the users.

I asked if anyone had spoken to Petar recently, as he was the maintainer. No one had. I reminded everyone that I had spoken with him before about migrating the service to a DLF-owned server. He'd first wanted to rearchitect everything to make it easier to manage and deploy before migrating it, as right now it was a bit of a pain. I didn't know where he was with it.

Steve said someone had reported an issue that the GitHub actions weren't running. He'd thought he'd be able to restart them and it would deploy, but it appeared not to have worked. The ideal thing would be something where it automatically deployed when you ran an action. I agreed, and added we needed to get it onto a DLF server at that point.

(__UPDATE__: The service has since been updated, but it's not yet configured for easy deployment nor on a DLF server.)

### DFA methodology, risk assessment, and goals

Rikki said that before embarking on implementing a Data Flow Analysis (DFA) engine, he had done a risk assessment, as it was a lot of work with a high probability the implementation would fail. His main concern was whether the community would accept such an engine attached to the language by default.

He listed three criteria by which we could evaluate it: performance, false positives, and the extent of the language it modeled. We'd done the false positive approach with DIP 1000. We didn't like it. Pretty much no one was willing to give up on performance. That just left the extent of language modeled. That's what he was sacrificing to keep it fast and minimize false positives.

Walter said there was another option. He'd implemented DFA in both the front end and the optimizer. It was a lot harder to do in the front end. He suggested Rikki investigate adding it to the optimizer.

Rikki said he'd looked into how other languages were doing it. Clang Analyzer, for example, was in the front end. That included Swift, Objective-C, C, and C++, and they were way ahead of where we currently were. It was definitely doable.

Walter knew it was doable, it was just a lot more difficult with lower odds of success. The problem with doing it in the front end was that the ASTs were far more complex there than they were in the intermediate code. Covering all those cases meant that every time you added something new to the language, you had to go in and account for it in the DFA. That made the burden of work much higher. He'd rarely needed to add anything to the DFA in the intermediate code because it was much simpler and lower level.

Rikki said the big problem with that approach was that it would then be compiler specific. Walter said that was true.

Átila said one possibility was to copy what LLVM did and have an intermediate layer that could be shared between compilers. Rikki said you were basically working with a front-end DFA at that point if you weren't organizing the glue layer. That was well beyond what he was doing. He was successfully doing a front-end DFA straight off the AST. That wasn't really the issue.

Átila asked what the issue was, noting that Rikki was trying to figure out whether it would get merged if he implemented it. Rikki said it would have to be merged as part of the implementation. He wanted to find out if we could turn it on by default in the language. Átila said it wasn't in the language. It was a compiler implementation detail. Rikki disagreed, saying that there would be attributes eventually. It would be replacing DIP 1000 and things like that.

Átila noted Rikki had given Clang as an example of what was ahead of us. Were they adding attributes to C++? Rikki said yes, they were adding type qualifiers for nullability. So you could do non-null as a type qualifier, pointers for function parameters, and things like that. It was actually pretty advanced and pretty cool how they were trying to make C and C++ code safe to work with.

Átila asked why this required changing the language. Couldn't we just put some attributes into `core.attributes`? That wasn't part of the language. Rikki said he wasn't here to talk about attributes today. That was a different ballgame. Átila said Rikki's question was about turning it on in the language. Asking why it had to be a language thing rather than a compiler thing was an obvious response to that, and Rikki had brought up attributes in response to that question. Átila didn't think attributes required language changes. 

Rikki said that for now, he was just concerned with whether we could even turn on a DFA that tested the language. Whether attributes were in `core.attributes` or in the language didn't really matter because he wasn't supporting them at the moment. His only concern now was whether we could even turn on a DFA that tested the language.

Átila asked if it would throw errors, and noted Rikki was saying 'in the language' again. He still didn't understand how the language had to change to accommodate this. Rikki said if the engine were reporting errors, then it would make the language fail when it was turned on. The idea was that it would be turned on by default. It wouldn't be opt-in.

Átila repeated that he still didn't understand how the language was being changed. Rikki said it would restrict what you could do based on rules that an average person might not understand, and he had to eliminate that. Átila thought he understood and asked if Rikki meant that he was trying to turn what is valid code today into errors. Rikki said yes, and without any attributes. If it had to have attributes right now, then that wasn't a good sign for it.

Walter said that when he'd implemented `@live`, he deliberately did not turn it on by default because of the tremendous increase in compile time. A lot of people use D for its fast compilation speed. Other languages have very fast compilation, too, and if DFA is on by default, it would be a big problem for us. It needed to be optional, either turned on with an attribute or a compiler flag.

Átila said that if not an attribute or a flag, then it had to somehow be super duper fast. Walter said DFA was slow. It required a fair amount of data to do the tracking. That couldn't be avoided. People used DMD because it was a fast compiler. Taking that away would be very costly. It needed to be optional.

He didn't see a big problem with it being optional. You'd throw a switch that turned on DFA and it would compile more slowly, but it would find more errors. Wasn't finding errors the reason why Rikki was proposing a DFA? Rikki said it was also meant for enabling language features like isolated and such that we couldn't do now.

Martin said that, if he understood correctly, Rikki was mainly after getting first test results, whether any code would break. He asked if there were any BuildKite results for Rikki's PR. How many projects broke? 

Rikki said there was no PR yet because it wasn't finished for nullability and truthiness. Once he got it to the point where he was happy with it, hopefully in another month or two, there would be a PR. It would be behind a preview switch so that nothing was at risk. Then we could answer some questions. Did it do false positives? Did it introduce a slow down compiling real code bases? It was running at the same speed as DIP 1000 on his code base. He thought that was pretty good.

Martin suggested temporarily merging a PR with it turned on by default just to get some results out of BuildKite, then go from there. Rikki wanted to do that. He expected projects in the wild to throw out a lot of messages. Because he'd worked so defensively in his code base, any message it was spitting out was basically a bug in the DFA.

Átila thought there was value to this if it was something optional. We could talk about the language stuff like isolated later. If it was worth doing, why not do it? The worst that could happen was that we made it optional and didn't change the language. He didn't see how that would be an impediment to doing the work.

Rikki said the only bad outcome for him was that he didn't prove if we could have a DFA engine to test the language. It didn't matter if it were true or if it were false, he just had to prove it one way or the other. Átila said Rikki could do that anyway, given that there was value to this no matter what. He could still do an experiment to see what happened to the language afterwards.

Rikki said he didn't care if it were removed later or rewritten or any of that sort of thing. He just wanted to prove whether that fact was true or not. Átila said he wasn't entirely sure what Rikki needed if he'd already worked on it and it would be good to have anyway. Rikki said he just wanted to make sure everyone knew that he'd done his risk assessment. He had a methodology in place that was very low risk to the rest of the compiler, and we could go ahead and merge it to let people have a play.

Martin asked if Rikki had a rough lines-of-code estimate. Rikki said it would be around 15K.

Walter said he was okay with merging it as an experimental thing or as a development thing. He was doing the same with the ARM64 stuff. As long as it wasn't disruptive to the main code base. Once it started rewriting the main code base or extending it everywhere, that would be a difficult pill to swallow.

Rikki said it was at one point in the function declaration class. It added one or two values for time trace, and it had an entry point in Semantic 3. It was all behind a preview switch. Walter said in that case he was okay with it.

__UPDATE__: Rikki's PR for his proof of concept [DFA engine for nullability and truthiness](https://github.com/dlang/dmd/pull/21965) was merged in October.

### Library ecosystem

Adam said that he did a lot of library stuff. He thought we ought to be able to build a static library, a `.a` or `.lib`, with DMD or GDC or LDC and have it work with all of them in the same way that C++ libraries built with Clang could be used with GCC.

He said not being able to was a huge ecosystem impediment. He wouldn't say it was a blocker, as given enough time some enterprising soul could figure out how to make it work. But why couldn't he build something with GDC and then use it with LDC?

Steve said he'd run into this when building the new GC. He was building with SDC, which used the LLVM backend so was kind of equivalent to LDC. There were ABI differences between DMD and LDC. The biggest one he'd seen was that the order of arguments was reversed. You couldn't just call D functions between the two. All the API had to be `extern(C)` to make it work. He thought there were a lot of runtime issues there, too. Even though they used the same code, each compiler had slight differences. 

He wasn't sure what the right answer was, but wondered if C++ compilers were really so interoperable. Adam said he'd done some searching and apparently people did it all the time.

Martin said that type infos in the C++ world were very simple, so it was much easier than in the D world. With ABI compatibility, it wasn't just about calling conventions, but also, e.g., the exception handling personality function, which was completely different on Windows between DMD and LDC. He had no idea about GDC, but it presumably had its own little thing in the MinGW libraries or something. The precision of `real` was another place where they differed.

The biggest bugger, the argument reversal thing, was something he'd changed a while back. Calling convention wise, LDC should be compatible with GDC, though he wasn't 100% sure, but the situation should be much better than before. Also, LDC supported the GDC-style syntax for inline ASM.

The runtime would need the most work. We'd need to have a streamlined implementation of some low-level stuff, like scanning segments of the binary to figure out where all the globals and TLS globals were at, etc. That was a complete mess. GDC had its own implementation of some things, LDC had its own, and they all diverged. They diverged in how the runtime was initialized with regard to how to register binaries with a shared runtime. He said the list would go on quite a bit. Getting there would be very complicated.

Rikki noted that the compilers had differences in the lowerings they did, and he didn't see those being converged.

Adam said this was something where we could tell people to solve it by using source libraries. But what we were really saying to people who didn't want to use source libraries was, 'we can't help you'. Martin disagreed that was the case. 

I said they could use the same toolchain. Martin said they could use dub because it had separate builds for each compiler. So whenever you were using different compilers with it, you should be fine.

Adam said to imagine getting a pre-built library from somewhere else. In library ecosystems, that happened. We couldn't tell the whole world they had to ship their source with everything.

I brought up libSDL as an example. They provided Windows binaries for MinGW and for MSVC because of the incompatibilities there. They weren't 100% compatible. I didn't see the problem with that. If you were distributing binaries and you wanted to support a wide user base, then you shipped binaries for each compiler.

Rikki said the simplest solution was to designate LDC as the LTS. Then people who wanted to ship binaries could build with that. He thought that would be our best bet.

Martin said he'd just thought of something. Our situation was much worse. It wasn't just three compilers with quite a few differences. It was also each version. You'd have to ship binaries for version 2.110, 2.111, and so on, for all three compilers. I suggested you could just settle on a minimum supported version.

Martin said the compiler had this fixed assumption for the DRuntime interface. If you were using a shared DRuntime and that interface changed... that was really messy. We weren't so stable like C++.

Adam said that added more complexity to your build chain and development process. The only answer he could see would be to slow down our release cadence to something like once a year. He wasn't saying we *had* to do anything about this. He was just pointing it out as a libraries guy who was going to be dealing with this problem in the future.

Steve had seen it suggested quite a few times that GDC should be the LTS compiler because of GCC's requirements that things couldn't change too much between versions. He didn't know how much it changed between point releases. Maybe it could be an option to tell people to use GDC for binary releases. He thought LDC was breaking just as much as DMD because they were releasing quite often.

Martin said we could take every 10th LDC release or so.

Steve said in a binary release ecosystem, you would probably specify the compiler version to use with your binary releases. Every so often you would bump it to another release. He thought it pretty unlikely we'd get to the level of binary compatibility that Adam was talking about.

Nicholas asked if this wasn't exactly what we wanted to use editions for. Steve said no. Editions were source compatible. Nicholas asked if there were any reason why we couldn't lock binary compatibility in with the same thing.

Martin shook his head no. He said implementing something like the templated hooks that we had now wouldn't be possible in that case. If we had an edition every ten releases or so, the last compiler release for a specific edition would still need to work with the shared runtime library from the first compiler release for the same edition. That could be very, very tricky. Nicholas said that at least our intention for what binary compatibility should be would be defined in that case.

Adam wondered if we could say that we couldn't do it now, but that we would work toward it in a future edition. It was going to be a major point for people who didn't want to ship their source. A lot of the C++ ecosystem was like that. You never shipped the source. You shipped a header file and a binary blob. Steve noted that C++ header files often included a lot of implementation. Adam agreed and said we'd end up doing something similar, but from an ecosystem standpoint, it had to be something the user decided.

Rikki said GDC would be a less desirable choice for an LTS. Windows and macOS were great examples where LDC trumped GDC. But the whole idea that you would tie codegen changes and lowerings to the edition was absolutely nuts. It meant if someone hit what was for them a crucial bug, we couldn't fix it. And it was going to happen and would be a problem for us. The end result was that we would have to designate a specific version of a specific compiler as the LTS. We'd support that for two years or whatever, and if you were going to ship binaries, that would be what you used.

He noted that Hipreme shipped a compiler toolchain with his game engine. That was effectively an LTS. The problem was solved for him. That was precisely how we needed to handle this.

Adam said this was going to take a lot of infrastructure work. He wondered if there was any benefit to starting the process of normalizing the runtime stuff. He recalled Martin had talked about upstreaming the LDC runtime stuff.

Martin wasn't sure what Adam was referring to. He said it would require a lot of work on DMD for very low benefit. LDC wouldn't be improved in any way. DMD's functionality might improve a little bit, but it wasn't worth the amount of work to get there.

Adam asked if we should look into slowing down the release cadence. He was thinking that if we wanted to get fancy, we could maybe have dub do this. The build manager itself would become a compiler manager. He didn't know what our install process was, so he was shooting in the dark, but why not treat the compiler itself as just another dependency? It could download it and extract it and give you a library of compilers to choose from.

Steve interjected with a situation he'd encountered with raylib-d. It used the DLL version of raylib, so he'd assumed it shouldn't be too hard. He'd found it was almost impossible because he had to have the exact same MSVC version that the binary had been built with. He'd ended up telling people to download MSVC and build raylib themselves.

If you were going to have a binary release of something, you had to put in the work to say, 'Here are the entry points. You have to use them this way.' And everything inside was kind of self-contained. He didn't think there was a way you could just use whatever version of whatever library you wanted as a dependency. With a template-heavy language, we were never going to have that kind of binary compatibility. It just wasn't going to happen.

Robert didn't think it was a technical thing. His first thought about Adam's suggestion to slow down the release cadence was that the next time we were here, we'd discuss increasing the release cadence because it was too slow. At some point, we had to tell people what we were and what we wanted, and we had to say 'no'. Leaving people in limbo about things while we talked about it was just prolonging the inevitable. At some point, we had to say, 'Nope. That's not a thing. You can try this or that, but mostly you're on your own.'

He said we could do no releases and be binary compatible forever, or we could be really quick and annoy some number of people. The D community was not a community in lockstep. We weren't here because we all liked the same things and were interested in the same ideas or had similar tastes. We were here because we were all weird and we were all weird in different ways. So he didn't think it was a technical thing. It was an organizational thing. A people thing.

Razvan pointed out that the release process right now was really cumbersome. We had relied on Iain alone for a long time. Now Dennis had looked into the script and it was really, really complicated. You needed to manually resolve all sorts of conflicts. If we wanted to do more releases, then it should be entirely automated. Right now it was half-automated, half-manual. We shouldn't need a person doing manual stuff. He'd talked to Dennis about it and they thought the script could be slimmed down a bit because it was doing things it probably shouldn't be doing.

Steve remembered Hipreme struggling because he'd wanted to have all of his stuff using DLLs and binary loading. You basically had to cut down to no templates in the API to get all that stuff to work correctly. That was something you had to accept if you wanted binary compatibility.

Martin said if he were shipping a library in binary form, he'd probably specify the compiler version he was using, then anyone who wanted to use the library would have to use the same version. That was the only version guaranteed to work. So you'd probably need to stick with one compiler unless you really wanted to put out different binaries. Then it was a set of supported compiler versions. The main problem here was when you wanted to combine multiple such libraries, or when your code wasn't compatible with a specific D version because something changed or something regressed.

Walter didn't think C gave you binary compatibility across compiler versions. Adam said they released maybe every decade.

Martin said that compiling LLVM with clang and the C++ parts of LDC with GCC, they worked fine. You could link them together. On Windows, the LLVM binaries were compiled with clang and the C++ parts of LDC with the MS compiler. That was also binary compatible and everything worked. Everything was more or less stable in the C++ world. Ignoring the C++ standards for now.

Adam noted that a lot of people who came to D came from the C and C++ world. For him, this was almost a kind of marketing thing. It was very easy to have a big backdoor of people who just quietly left. Most people wouldn't throw big rants on the forum. They'd just walk away. That was part of why he was harping on this.

Also, he'd been reading about ecosystems. He viewed this Phobos 3 project as the core of the library ecosystem because everyone could use it. As the Phobos guy, he had to care about the entire ecosystem on top of it. He'd started researching it and had hit exactly what we were talking about.

Maybe the answer really was to just say which compilers a thing supported. He wasn't expecting to get a solution out of the meeting, though it was a really instructive discussion. It was something he wanted to keep in mind because it was an ecosystem and marketing problem. He added that Robert wasn't wrong. At some point we had to say that we just couldn't do that, but we had to understand who was walking out the backdoor when we said it.

He said that at DConf in Munich, he'd written a build system there in the room. He would have to add things like offering a selection of compilers to download and figure out. We would have to treat the build ecosystem differently than we did right now. That was something to think about.

In the meantime, we needed to be mindful of what we were throwing in the compiler. What impact did it have across other libraries in terms of ABI? Maybe we needed to get this ABI situation sorted out. Maybe we had to pick and choose things over time and we could start lighting things up.

He said it was an option to identify the stuff what was and wasn't binary compatible. I thought that was an actionable item. The first step then would be to go through and identify what was binary compatible and decide where to go. Maybe put up a page describing the incompatibilities or see if there was a way forward to reducing them.

Steve said that you'd need buy-in from the language team that they wouldn't change those things if you wanted that kind of binary compatibility. For instance, when he had done the work to enable static initialization of associative arrays, he'd needed to add things to the underlying AA type that would totally break when used with a different AA layout. 

Recently he'd had a problem where AAs were building fake type infos and allocating with the GC. In certain cases with the new GC, those type infos were being collected before the AA block, resulting in crashes. He didn't understand how the compiler guys did these things, but Rainer had set it up so that the compiler now built the AA type info for the element type, and that worked flawlessly. But that required adding a new piece to the AA type info, and that was another binary incompatibility. 

If we wanted to achieve binary compatibility, we had to commit to only changing things like that on a non-regular basis. Adam speculated about doing it on an edition basis. Steve said maybe. But the AA thing was a huge bug that was crashing things at Symmetry, so they'd needed that one fixed ASAP.

Rikki pointed out that if AAs were templated, all those issues across compiler versions wouldn't have existed. That had to be taken into account. Just because an implementation detail changed didn't mean they'd stop being interoperable between versions.

Steve said that would also cause binary incompatibility. If you changed that template, then it would no longer be binary compatible. Anytime you had a template, it made things really sketchy. Things could change without you even thinking about it.

Rikki said it could, but it wasn't always guaranteed. If you were careful, like with a templated class for instance, the vtable didn't necessarily have to change. And you could always add entries to the vtable without breaking, so older compiler versions would see the older methods and newer compilers would use the newer methods.

Martin brought up a big problem with shipping a precompiled static library that was different for each compiler when it came to templates. In that case, if the semantics of the template changed so that the mangled name remained the same, the linker was going to pick one symbol, probably the first one it saw in the first object file it pulled out of these libraries. Binary compatibility would mean that whenever we had a mangled name, the semantics were never going to change. The implementation might defer some implementation details, but the semantics must be completely the same, with the expected result, the expected exceptions, all of that. This was immensely complicated.

Steve said that was why we ought to focus on things that weren't templates if we wanted to guarantee any kind of binary compatibility. At least in those cases we could see when they would change and they wouldn't change because of some dependency somewhere.

I suggested that Adam might gather his thoughts, then somewhere down the road put a document together on what testing binary compatibility might look like. Then we could discuss it at a future meeting and see about getting some community feedback and participation on it. Adam thought that was a good idea.

### Assign expressions and with statements

Rikki told us that [Dejan had posted in the forums](https://forum.dlang.org/post/jzycjbygpwkvujwqbtrj@forum.dlang.org) requesting we add support for assign expressions in with statements. Why didn't we have it?

I noted that Adam Ruppe had just announced it was in OpenD, so we could use that implementation. Walter said he had no issue with that. Rikki said he had looked at the code base and wasn't going to be the one to do it, but he would reply in the forums that it had been pre-approved.

Nicholas said it wouldn't be hard to do. He volunteered to do it. Steve thought it made a lot of sense given all the other places we had it.

As an aside, Steve didn't recall where he'd seen it, but someone had asked about with statements and UFCS. Right now, given a struct instance `s` with a method `foo`, inside `with(s)` you could call `foo` without doing `s.foo`. That didn't work with UFCS functions. So with a function `foo(S s)`, you still had to use `s.foo`. UFCS made `foo` look like a member, so inside of `with` it seemed like an inconsistency that you couldn't just call `foo`. He wondered if there was a reason we couldn't do that.

I asked it if was intentional or an oversight. Walter said it was might have been an oversight, but it was more likely that it was complicated. The name lookup rules were supposed to be trivial, but over time all these features had been layered on to it. Now we were sometimes left wondering what was actually happening when we used these shortcuts. He didn't know the cause of it not working, or if it would break other things or be ambiguous if we made it work. He hadn't looked at it yet.

I suggested that was something to consider in the future, but it looked like we were good for the assign expressions.

Walter said he looked through the front end source code now and then and was horrified at how complicated it had become. He'd had a couple of PRs merged recently that simplified some things in the front end, but it was like mowing the lawn with scissors. He found it a bit discouraging. Sometimes he wondered if we weren't driving ourselves into a swamp by adding on more and more complexity.

Walter said there were times when he didn't feel like working on whatever his main project was at the moment, but he still wanted to do *something*. So he'd open up a random source file and consider how he could make it better or more understandable. A few days before the meeting he'd opened the `outbuffer` module. It was using a method called `write`, and he wondered why it wasn't called `put`. `OutBuffer` was an output range. Why didn't it have the output range API? So he submitted the PR to replace `write` with `put`. It wasn't anything major, but it was something that reduced the cognitive load for anyone reading the code. Now you could see it was an output range and it was one less thing to learn in the source.

Another example was that he'd noticed that struct parameters were initialized in all kinds of random places. So he'd put all the initializations into one section. Little things like that made the code base nicer and easier to understand.

He urged everyone to do something similar when they got bored with working on their main thing but wanted a break from their usual stuff. Just pick a file, look at it, and see what you could improve.

### Spreading the word

We had gone through our scheduled agenda items at this point. Walter wanted to talk about marketing.

He regularly saw articles on Hacker News written by one or more Zig enthusiasts. They were always touting various features of Zig as being innovative and new when they were actually copied more or less from D.

He found this very upsetting because D had those features 20 years ago, and suddenly Zig announced they had compile-time function execution and everyone was going, 'Wow, wow, wow, greatest feature ever! I didn't know you guys were so innovative.' 

He said if any of us had any ideas for articles, we should write them. They didn't have to be very long. Even just a page of text. It was worth doing it just on anything and everything.

He brought up a few other Zig features he'd seen people touting, including the `restrict` keyword, and how the people touting it hadn't realized it was a C feature that nobody really used because nobody understood how it worked. Martin said it had a few uses for performance code and vectorization. Then we veered off into a bit of a discussion about the pros and cons of `restrict` and whether it could be reliably analyzed to determine if it was used correctly. 

Walter reminded us he'd just wanted to mention that we should be publishing more stuff about D.

Rikki told us he'd talked about what D did in some comments about compiler theory on Reddit. He usually got some likes on comments like that.

I said if anyone wanted to put some articles together for the blog, they could send them to me and I'd publish them. I didn't have time to be chasing people for articles anymore like I used to, but if anyone sent me any D articles, I'd publish them.

We then talked a bit about where to share D articles. I said I had always shared our articles on Reddit and Twitter, and Walter or someone else would push them on HN. Walter thought Reddit had gone into a decline from what it used to be. I said what I liked about it was that we used to get really trash comments on most of the blog posts I shared there, but over time they got better and more positive.

Walter said he used Twitter to post a paragraph about something he was working on now and then and he'd gotten surprisingly good feedback about things like that. It didn't have to be big. Even short posts could be effective, so he was regularly using his Twitter account to promote D.

Adam noted Walter's Twitter posts about the ARM backend had gotten some traction. Walter had noticed his follower count increased after he started posting about that. He thought Twitter was a major marketing tool. Emails just went to spam folders, but people actively signed up for your Twitter messages. Keeping it short was effective. He'd seen other people doing it to good effect: John Cook, Bartosz Milewski. He encouraged us to do that.

I added that YouTube was by far the biggest place people were searching for anything these days. We had a serious absence of content there. Mike Shah had his ongoing D series, but we didn't have much going on with our channel. We had the conference videos, my interviews, and a tutorial series Dennis had submitted. I would love to have videos of people going about their daily D work, showing how they're fixing bugs or implementing compiler features, that sort of thing.

Adam said there was a guy doing short videos about C# features. They were a minute and a half, two minutes long. The guy was killing it and had started selling C# courses. He wasn't saying we had to make a whole thing out of it, but the guy was doing a mix of short-form and long-form content.

I said all it had to be was you just sit down and say, 'Okay, I'm going to fix a bug today.' Turn on your screen recorder and talk about what you're doing as you're doing it. Now we've got a compiler bug fixed and we've got a video showing what it's like to program in D. It was real-world D programming. There were people out there who would watch that. Whether it got a lot of views didn't matter. Ultimately it would help us grow our YouTube channel because it would give me content to publish and build engagement. 

If I could have a video to publish every week from people, not just Walter, but anybody going about their regular D programming, that would be fantastic. It didn't have to be the same people all the time. It could be anything about D. I had a series in mind that I'd been wanting to do for a while but just couldn't make the time for. I wanted talk about fundamental things like object files, static vs. dynamic linking, dynamic loading and things like that.

Adam said Google Ads was still huge. You didn't hear people talking about Facebook ads anymore, or LinkedIn. But you had to be on Twitter and you had to be on YouTube.

I said that it didn't cost us anything to upload videos to YouTube. That was as much free marketing as we wanted. Walter said it didn't cost us anything to post on Twitter either.

I said one of the big benefits about YouTube was that videos on an evergreen topic were evergreen videos. They'd show results over time. A tutorial about D or showing how you fixed a bug, those were evergreen. They would be as valid a year or two years from now as they were today, barring language changes. They weren't going to disappear into the ether. So when people were searching for dlang in the future, those videos would come up in some of the search results.

As an example, I had people coming to my personal YouTube channel through search results keeping some of my older videos going. And new subscribers often went back through my library to view my older content. Those older videos were still working for me.

If somebody saw a video of Walter fixing a D bug and thought it was pretty cool, they might go back and look at other videos of other fixed bugs. It would be a small number, but there were definitely people who would be interested in that sort of thing. And that would cause them to go into our back catalog, and maybe even fix a bug themselves. And at the same time, it was going to be generating money for us through the AdSense program.

Walter said it was a win all around and he understood that. He'd given Google ads a try years ago and had zero results. I said we weren't even going to try that. Andrei and I had done something with it a few years back to promote DConf and it had been a waste of money.

Rikki said people in Walter's position sometimes did seminars and things like that, taking audience questions and recording it all. Those videos got uploaded and people watched them. At some point, Walter should see about doing a seminar for Mike Shah's students. I noted that Steve and Ali had done that. Rikki said the difference was that it would be Walter. Steve said, 'Thanks.' Rikki said people were always interested in meeting someone with a lot of experience, having a chat about some interesting project the students were working on, soliciting advice, maybe having some sort of presentation.

Adam added that Ben Jones at Utah University was someone else we shouldn't forget. Adam had recently passed through Salt Lake City and had the opportunity to meetup with Ben and one of our GSoC students. Ben was teaching his CS and engineering classes in D. I noted that Ben was going to be at DConf.

Steve said that every month we had the online BeerConf. He always posted an open invitation for presentations. In the past, he'd recorded them and posted them on his YouTube channel. We could put them on the DLF channel, too. I said if the speaker was okay with it, then I'd be happy to publish them.

### Quick items

I said I had to announce Symmetry Autumn of Code at the end of the month or in early July. We needed to start thinking about projects. I was going to talk about that with Razvan and Dennis soon in one of our team meetings, but asked everyone to please let me know if they had any ideas for SAOC projects.

Another thing was shop.dlang.org. Someone had pointed out in the Discord a week or so before that it was down. I'd been unable to get it back up. I'd done everything possible within my sphere of knowledge, then reached out to Vladimir for help. He'd been able to determine that it wasn't anything on our end. It was something to do with either CloudFlare or Fourthwall, the provider we were using for the shop. The next step was to reach out to support at both places to see what could be done. In the meantime, I was going to disable the custom domain at Fourthwall and just go with dlang-shop.fourthwall.com.

(__UPDATE__: My support requests got me nowhere, so I ended up going with `store.dlang.org`, which has had no issues so far. If you're looking for D swag, that's where you can get it now. Proceeds go into our general funding pool along with all donations made via PayPal and GitHub.)

### Conclusion

Our next meeting was a quarterly on July 4th. Our next monthly meeting was on July 11th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
