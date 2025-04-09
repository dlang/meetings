[Forum Post](https://forum.dlang.org/post/ghjmmpujgldbajzbrzin@forum.dlang.org)

The D Language Foundation's monthly meeting for November 2024 took place on Friday the 8th and lasted around an hour and a half.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Razvan Nitu
* Mike Parker
* Robert Schadek
* Steven Schveighoffer

## The Summary

### Project coordinator

I started by talking about my DConf '22 talk, which had been motivated by my realization that we weren't meeting expectations. In the talk, I used the history of the D community's evolution to demonstrate the changing expectations of the community over time. In the early years, we were a community of do-it-yourselfers, where if you wanted to use D you had to build things up yourself. Everyone was a contributor, and they expected they'd have to do things themselves. There was no such thing as a "core team" back then. It was just Walter working on the language, the compiler, and Phobos, and the community was working on everything else.

Today, I saw the core team as a small group of people who cared about D, doing what we could when we could with the time and resources we had. The community, on the other hand, tended to see us simply as the organization responsible for D, period. That created a set of expectations that we were consistently failing to meet.

After DConf '22, we discussed how to meet and manage expectations. That led to our decision to accept Ucora's offer to participate in their IVY program. I thought that had helped us primarily with our decision making, so we were on the right track in determining what to do and how to do it. But we were still failing in two critical ways: execution and communication.

A lot of the responsibility for external communication fell to me, so failures there were on my shoulders. But we all were failing at internal communication. The delayed compiler release was a good example of that.

Each of us had our responsibilities and we left each other to them. We just assumed things would get done when they got done. Most team members were working on their own time on top of their regular jobs and family responsibilities, so no one ever pushed anyone about their progress.

At the same time, we weren't often communicating the current state of our work to each other aside from an update now and then in a monthly meeting. Since we had switched to agenda-based meetings, we were doing that even less than before. This wouldn't fly in a business. Team leaders would be expected to report on progress, managers would be checking in. Someone would be responsible for monitoring progress and demanding answers when milestones were missed.

I reminded everyone that I had pushed for all of us to use the project tracker on GitHub so that the community could follow our progress, and so that we could follow each other. Only Razvan and I had ever used it much. Eventually, we both gave up on it because no one else ever took it up.

To improve the situation, I proposed we create a Project Coordinator (PC) position. The PC's primary responsibility would be to keep current on the progress of our ongoing projects by checking in periodically with those responsible for them. The PC would provide updates to me each month so that I could post an update in the forums, allowing the community to get a sense of what we were doing.

The PC would have other responsibilities, like making sure everyone had what they needed for their projects, finding someone to take over a project when the person doing it couldn't continue, and so on.

I said that this required buy-in from each of us. We needed to answer the PC's emails when he checked in. We needed to swallow our pride when we discovered we didn't have the time to continue a project and let the PC know so that someone else could take over. And we shouldn't be accepting projects we didn't really have the time for in the first place, even when we felt a sense of obligation to take it on.

With this approach, I hoped we could improve communication internally and externally, and make it less likely for projects to stall. I had spoken to Walter about it. He agreed we should try it. We both felt Razvan was right for the position. I'd already talked to him about it and he had accepted. It would be up to him to fully define the role.

I asked if anyone had any problems with the idea.

Martin said he had no issues, but he was skeptical given how little time we all had. He thought we should be more transparent about how big the team was. We had ten people in this meeting, and as far as he knew everyone except Walter had a full-time job. That was the main difficulty.

He said the DMD release manager role was a really, really special one. He didn't think we could compare that to any other projects, because it stalled progress for the customers when it was delayed. Symmetry had suffered from it. Although they were quite independent, having an official release would be great.

Maybe we could be more transparent about how many people were involved, how many had full-time jobs, and how much money we had. As far as he knew, our resources were very, very slim, and we needed to make that clear to manage expectations.

I agreed with him and said that I had a plan for that. My idea was to take everything related to the DLF in the 'Community' section of the website and split it off to a subdomain, something like 'dlf.dlang.org'. That would take those pages out of the website build, allow us to drop Ddoc for those pages, and give them a dedicated repository. I wanted to add a page listing the core team, who was a volunteer, and who was paid and how much. I wanted to overhaul the donations page to clarify how we used the funds from each service (e.g., PayPal vs Open Collective), list our current funding numbers, and so on.

Rikki said he'd thought of having a project manager ages ago. He expected it would be great to have a central person keeping track of everything.

This prompted me to note the distinction, as I saw it, between 'manager' and 'coordinator'. I wanted to make it clear that we didn't want someone in a position to demand answers, but simply to keep track of and facilitate progress. Steve said that's what he understood the role of a project manager to be. It wasn't someone above everyone else, commanding them to do things. (Ultimately, I stuck with 'Project Coordinator' as the title.)

Walter asked Razvan if this new role fit with his IVY. Razvan said it fit better than the PR and Issue Manager position.

I closed by noting that this meant Dennis would now be the only PR and Issue Manager. We needed to find the funding to hire someone to replace him. In the meantime, Razvan would be pestering everyone to find out where they were with whatever they were working on.

Steve suggested Razvan should be tracking progress on editions, Phobos v3, the Bugzilla to GitHub migration, and the GC work he and Amaury were doing. He said he'd be happy to report stuff to Razvan about it.

### DConf '25 and funding

I informed everyone that planning for DConf '25 had begun. The budget Symmetry had given me this time was higher than the last time we did DConf in August, (2023), but costs were up across the board. The more expensive dates in the fall, which along with spring was peak conference season, were not possible. We did it in September last year only because those dates coincided with Symmetry's 10th anniversary event.

CodeNode had the second week of December and multiple dates in August available. I had informally gathered some feedback from several regular DConf attendees to see if they had a preference or if they would be less likely to attend in December. Most told me they had no problem with it. Some told me they preferred it. I proposed the dates to Symmetry, but they decided it wasn't a practical option as that's a busy time of year for them. That meant the conference was going to be in August.

I said that I was concerned about travel and lodging reimbursements this time. August is peak travel season and the rates in 2025 were bound to be higher than our last August conference in 2023. I needed to secure more funding for speaker and staff reimbursements. I was also trying to raise more money for a new PR and Issue Manager to replace Razvan.

Steve said that we didn't have to just rely on people using D to provide funding. We were a nonprofit in the STEM field. There were lots of grants all over the place from different governments and institutions we might be able to take advantage of. We could look around for opportunities that fit our mission and apply for them.

I agreed, but said we needed a reliable, regular baseline from D companies as our funding foundation that we could build on. I noted that Andrei had set us up a while back with a service called Benevity. They pulled in some small donations for us, but it was unpredictable.

(__UPDATE__: [WEKA has provided the funding required](https://forum.dlang.org/post/eznvmmggimcjrfwuchqy@forum.dlang.org) to hire a new Pull Request and Issue Manager and to alleviate my DConf funding concerns.)

### Rikki's memory safety survey

Rikki told us he'd gathered some notes from [the memory safety survey](https://forum.dlang.org/post/lunzanrfkklespwttyuy@forum.dlang.org) that he'd posted in the forums in September. The highlights he gave us in the meeting were essentially what he subsequently [reported in the forums](https://forum.dlang.org/post/kijxuuxgtyzciuhjanrq@forum.dlang.org). Please refer to that post for the details.

After he gave us the numbers, he said we should consider posting future surveys directly on dlang.org because the sample size was not as big as he would have liked. One thing he noted (in the meeting and the forum post) was that 75% of respondents to his survey said they were hobbyists, and in a 2024 survey on Stack Overflow he'd looked at, 75% were employed programmers.

I said there was some interesting stuff in the survey results, but we should be careful when comparing it to Stack Overflow surveys. We couldn't be sure that employed D programmers regularly read the forums or would participate in this kind of survey. If he wanted to know what employees of D shops thought, he should send them the survey directly. We should be careful about how we interpreted the results.

Rikki said the survey told us about people who really cared about changes today. They were the ones reading the newsgroups. It didn't tell us if there was a silent majority, but he believed there was. This was one of the reasons he wanted to do it during DConf. We had the numbers there.

I said that another thing to keep in mind was that in any online community associated with a product, whether it was a programming language, a game, or whatever, the active forum user base was always a minority of a minority. Dennis agreed.

Rikki said we were definitely missing people. That was why he'd wanted to compare it to Stack Overflow. SO showed the global trend and could tell us what we might be missing in terms of population. Which in turn meant we had to do a better job of getting it out there in the future. That made him think that maybe the people we were missing didn't care much about D changes, anyway. 

Walter thanked Rikki for doing the survey. He said it was interesting to read. There was some good stuff in there. Dennis agreed.

Rikki said there were some things Walter had been concerned about in the past, e.g., memory safety couldn't be too invasive, but the responses here indicated that wasn't an issue. People wanted type state analysis even if it broke code. He thought this was information useful for prioritizing projects.

### Move constructors and placement new

Walter said he'd been quiet lately because he'd been working behind the scenes with Timon and Manu on move constructors. They had started working on it around a table at DConf and had continued via email.

They now had [a design and implementation](https://github.com/dlang/dmd/pull/17050) that was surprisingly simple and seemed to be working. He was pleased with it. He said it was different from the way C++ did it. He thought it a better, simpler approach.

He said Manu had also suggested placement new. Instead of having `core.lifetime` and all its complicated templates and confusing usage, it should be built into the language. With both move construction and placement new, we should be able to get rid of `core.lifetime` and its complexity. The compiler should be able to do lifetime analysis and memory safety stuff. It couldn't do that with the templates.

Martin was confused about what the `__rvalue` thing in the PR had to do with move constructors, so Walter [pointed him to the DIP](https://github.com/WalterBright/documents/blob/master/rvalue.md). Martin said he would read it later.

On the surface, he said `__rvalue` looked like a new builtin that converted an lvalue to an rvalue without any memory operations or moving at all. In that regard, it looked like the forward builtin he'd [proposed in a forum thread](https://forum.dlang.org/post/xnwhexrctbfgntfklzaf@forum.dlang.org).

He was okay with naming it `__rvalue`, but Walter's PR was by no means efficient. The builtin itself, converting into an rvalue at the AST level, was a tiny thing. The tricky part, the core part, was the suppression of the destruction of the original lvalue, which could only be done for select lvalues in a function scope. He didn't know if the DIP explained that.

Walter said he'd explain it, and that Martin was right. The idea was just that a constructor that took an rvalue became a move constructor. `__rvalue` enabled that. When rvalues were passed to a function, they were destroyed at the end of the function. That had always been a semantic thing. If you used `__rvalue` on an lvalue to pass as a parameter, it would be destroyed as an rvalue argument. 

It turned out that C++ did this, too. If you were using it as a move constructor, moving resources out of an object, you needed to replace it with a benign value. For example, if you malloced something and your destructor did a `free` on it, the destructor needed to check it was not `null` before freeing and set it to `null` after freeing. That way, if you got a destructor call on a moved object, it was completely benign. Nothing happened.

He had gone around and around with Manu and Timon on this. This seemed like the best way forward as it didn't involve much effort on the user's part. He suggested Martin read the DIP and give it a try. He thought it was a nice way to do it. It was a complete add-on to D, so it didn't break anything.

He said they'd realized that when you pass an rvalue to a function, you were already doing a move. This just completed the circle on it.

Martin asked if Walter had seen his forum thread about the `forward` builtin, and Walter said he had not. Martin said Walter had asked him to convince Manu of the perfect forwarding case, so he'd focused on that rather than rvalues and showed that move constructors in that case were just a tiny, pretty much uninteresting implementation detail. So having `__rvalue` just to be able to call the move constructor with a new syntax wasn't the way he wanted to go.

Martin said he'd post a link to the thread for Walter to read and that he would read Walter's DIP.

Rikki noted that in the PR there was mention of preventing access to the previous variable after the move. He wanted to point out that was type state analysis, which according to his survey was something people wanted. We could basically just say that would be solved by type state analysis, assuming we got it.

Walter said he knew that eliminating the superfluous destructor call was desirable, but it wasn't necessary. Doing it properly required some form of data flow analysis. There really weren't many shortcuts for that. You could handle some things trivially, but you would rapidly find that the kluge methods required did not scale. You had to actually do data flow analysis.

The current design did not require data flow analysis. You'd just end up with a benign destructor call here and there.

Rikki said he was suggesting not doing it here at all, but to put it off for type state analysis. What he really wanted here was that the data flow analysis for this would be the same one as for the escape analysis: multiple language features, one analysis. We'd get a high return on investment in terms of time and memory.

He said that was where we could do all of our variable tracking of heap-allocated memory, object tracking, and things like that. You'd just hook into different things when you converged. You would call out to one of the different analyses and let it decide if it needed to error out. It would do a second pass where it compiled down all the information to figure out how the error got created.

That was one of the problems we had with error messages now. How did it happen? What contributed to it? Nobody knew, because the way the compiler tried to figure it out was too complex. We needed to talk about it at some point.

Steve said he didn't understand Manu's points. All he knew was that Manu was keen on getting move constructors working and that other people wanted them. He had believed that C++ move constructors were like our rvalue references. Was that not the case?

Walter said we were not having rvalue references, for which he was very happy.

Steve said that he thought the point of a move constructor was to hook the compiler for when it moved something, but it looked like the move constructor in the DIP was just a constructor taking a copy.

Walter said the move constructor was a constructor whose parameter was not a ref parameter, but an rvalue parameter. However, it was still a regular constructor. Because it owned the argument, it was free to move rather than copy.

Steve said to imagine you had an lvalue that you wanted to pass to a move constructor to move it to a new thing. Didn't you have to make a copy of it to call that constructor?

Walter said that was the whole point of `__rvalue`. It told the compiler to treat that lvalue like an rvalue. Under the hood, rvalues were passed by reference for any non-POD struct, so it was already being passed by pointer. 

In D's existing implementation, if you passed an lvalue to an rvalue, it would create a copy and then pass a pointer to the copy. `__rvalue` would skip making the copy and just pass a pointer to the lvalue instead. And then the semantics were that the receiving function owned it and could do what it wanted with it. That was how move construction happened. So the move was a feature that dropped out of the `__rvalue` semantic.

Steve said that if you moved the guts into your local thing, you still had to clean out the thing that was there.

Walter said that was what the destructor call did. You could do it more efficiently by manually resetting those values to their initial values. It should be benign.

Martin said this happened implicitly. Normally, after a move destructor, aside from any special cases it might have like registering stuff or increasing/decreasing reference counts, all you'd normally do was move every field piecewise. This way, the original thing moved from was reset field by field anyway, so you didn't have to reset anything manually yourself.

Glancing at Walter's DIP, as far as Martin understood, the thing was destroyed twice, once at the end of the move constructor and once when the lvalue went out of scope. And now there was this new requirement that you had to ensure the payload was reset in the destructor so that the second destructor call would be a no-op if you were lucky.

He said if you did have things where you had destructors that needed to increase a global destructor count and check it, that was going to get ugly because we'd have two destructions even though we only created one instance that we moved once. In that case, three destructions instead of two. That was kind of weird.

Another weird thing was his main problem with the by-value signature: taking the address of the rvalue parameter gave you the original address of the thing being moved from, so it wasn't a private copy. There was no private copy, but you needed to know about that. That was kind of ugly.

This was why he would have preferred an explicit reference. Then it was clear that the address was going to return the original address of the moved-from thing.

Walter said C++ did the extra destruction, too. Martin said we could do much better. Basically, `__rvalue` was the same as the C++ `std::move`, just a cast to an rvalue reference. And this was what he had a problem with.

He said the main use case he was thinking of was perfect forwarding, where you could skip the move entirely and just pass by reference. He talked about that, and how `std::move` sometimes called moved constructors so things could be adjusted after, but sometimes just passed by ref. This made for some strange semantic terms. What were move semantics?

Ultimately, after more talk about `std::move`, he said he had many things going on in his head and needed to think on it.

Rikki suggested that, based on what Martin had said, maybe we should whitelist which functions could accept it when it wasn't by ref. Because the ref version would do what people expected, but the non-ref version would potentially do something different from what was already implemented. That could break code.

Walter didn't see how it could break code. It was purely an addition. If you didn't use `__rvalue`, then nothing would change.

Rikki said that was true, but the thing being moved might not be designed for it. Walter said yes, you had to fix its destructor if you were going to do it. Rikki said that was why he thought it would be a good idea to whitelist it just to prevent that from being accidental.

Rikki added that we should probably document the guarantees being offered with rvalues and lvalues on the ABI page. People wouldn't know what they could rely on here if it wasn't documented.

Walter said he hadn't written the spec for it yet, but he'd written the DIP and it was documented there for now. Rikki said he wasn't talking about that, but rather what happened currently.

Walter asked if he was referring to the C ABI, where non-POD structs were passed by reference even when they were rvalues. Rikki said he was, and also defining what rvalues and lvalues were.

Walter said it didn't actually matter. In the C ABI, when you had a large struct, it was passed by reference instead of by value. The semantics worked out the same. So the language didn't need to document it. You'd find it wasn't documented anywhere in the C language spec.

Rikki said we were using those terms so often, they probably should be documented somewhere. Walter said we had [a glossary section in the spec](https://dlang.org/spec/glossary.html) for defining our terms. So yes, we should have proper definitions for terms there so as not to confuse them.

(NOTE: [lvalue](https://dlang.org/spec/glossary.html#lvalue) and [rvalue](https://dlang.org/spec/glossary.html#rvalue) are both in the glossary.)

Steve said it was true that it was all new code if you used `__rvalue`, but if you already had a constructor that took a copy of another struct, then suddenly that would become an rvalue. If you had to do something special inside there, that was code that could break. He wasn't 100% sure because he didn't fully understand it, but he thought that was Rikki's point.

Walter said that if you were, for example, freeing a pointer in your destructor, you'd now get a double free when you started passing an `__rvalue` parameter to it. So the idea was that you now had to reset the pointer.

Steve thought that was the case already today. If you were accepting a copy of an `S`, it would be destroyed at the end, so you had to handle the pointer in the destructor anyway.

Walter said yes, and if you were freeing resources, that wasn't safe code anyway. It was going to have to be in `@trusted` code if you were calling `free` or something like it. Steve agreed.

Jonathan said it sounded like we were saying that at the end of the destructor, what you needed to be doing was setting the thing to its `.init` value. Should we just make it so that happened automatically? The compiler could set the state to `.init` after a destructor call.

Walter said that was a great question, but he had mixed feelings about it. Maybe we could do it in `@safe` code, but skip it in `@system` code for performance reasons. That was an option and a great thought.

Jonathan wasn't sure whether it applied here or not, but he noted he'd been surprised to learn that arrays and associative arrays didn't handle copy constructors properly. He didn't know if we needed to do anything with them for move constructors.

Walter said move constructors were just for structs, not classes. Associative arrays were reference objects, so the move constructors didn't apply there either.

Jonathan said it potentially could when you were passing stuff in and out, like when assigning to a dynamic array. Or, for example, setting a value for a particular key in an AA. He didn't know enough about the details to know if anything needed to be addressed there, but he wanted to make sure we didn't overlook it as we had for copy constructors.

Walter said he had not addressed that, and it was a good idea.

Steve said he would say no to that automatically. It was really, really hard to manage when a destructor should be called or not. There was no way to do it in code. If the compiler was adding in extra things that were already being done, it would be a performance problem and would get annoying. There was no way to suppress destructors.

He couldn't imagine why copy constructors were a problem in arrays. A lot of times when appending, you were copying the array elements to a new block, and it wasn't calling the copy constructors. He didn't think move constructors came into play there either. It was all happening inside the array runtime.

Jonathan said it came up when assigning something to an array. Steve couldn't see how you would do that with a move. Jonathan said he didn't understand it well enough to say for certain it applied to move constructors, but it very much applied to copy constructors. An assignment to the array should trigger it. If it was necessary with move constructors, he just wanted to make sure it wasn't forgotten.

Rikki noted that all of this stuff with deciding whether or not to call destructors was type state analysis. Part of its job was to determine when they weren't needed. Walter agreed. Rikki said it functioned as an optimization, not just a safety feature.

Martin said we didn't need to do a full analysis for it. We could use destructor expressions. We could insert guard variables. For example, if you had a local variable that was a non-POD struct and on one line you applied `__rvalue`. In that case, you would set the guard value just before it to something to prevent further destruction calls by having a destructor expression, which was a Boolean expression. Either the guard value was set, in which case you skipped the destruction, or it wasn't set, in which case you did the destruction.

This was something we already had in the compiler. He thought it was mainly for function calls related to temporaries for argument expressions. Those could get quite complicated. They might throw. An earlier argument might need to be destructed because it might not get to the point where ownership was transferred by move semantics to the callee, and stuff like that.

Getting back to what Jonathan had said about arrays, Martin said one thing to avoid was all the compiler blits. Not only for passing an rvalue as an argument but also for return values. In that case, we needed return value optimization built into the language itself to avoid the moving of return values in call chains.

If the runtime made some assumptions in array code, it would need to be changed to use a move. We just needed to avoid all implicit blits, and explicit blits in user code, where we'd just been assuming we could blit stuff around freely. That was our simple move operation and wouldn't be valid anymore. We needed to be calling the builtin.

Jonathan said those blits were definitely there. Martin said we definitely needed to check for them. Johnathan said he'd been looking at some of the templated lowerings and there was some funky stuff with `memcpy`. Martin said that was the thing to watch out for.

Walter had to sadly admit that DMD was generating some unnecessary blit copies as part of its code generation. He, Timon, and Manu had found three cases of it. He complimented Martin that LDC didn't do those. He'd fixed them in DMD at one point. He hadn't understood the problem at the time, but now he'd gotten them fixed.

Rikki said he'd been thinking that if we were going to have moving as a language builtin, then we would have to do swap as well. Currently, we couldn't express that the old value was not valid. With a builtin swap, we could handle it. He suggested we should think about it being part of the move constructor DIP since a swap was a form of movement.

Walter said he was going to do a demo, but he was going to defer it to later. I asked if he was going to post about the DIP soon in the forum, and [he said he was](https://forum.dlang.org/thread/vgna93$19f$1@digitalmars.com).

### The dmd release

I asked what could be done about getting the next release out in Iain's absence.

Jonathan said we needed to document the process so that anybody could do it whether it was one step or five hundred. I said that was something Iain had wanted to do for a while, but I thought there was more to it than just documenting it.

Mathias said one of the blockers for automation had been signing on Windows. We'd had a signing certificate, but doing it via CI was either impossible or extremely expensive.

Martin said he and Iain had talked a while back about setting up our own CI runners and plugging them into GitHub actions just for signing so that we could use the currently built releases. The CI in the installer repository took care of building some install packages that were very similar to the official ones, they just weren't signed on Windows. We could download the artifacts that were generated there, sign them on our private runners where we stored the keys, and then upload them again.

(__NOTE__: We haven't had a signing certificate since long before this meeting because the affordable option we'd been using was no longer available, though we're expecting to have one again at some point. Iain looked into other options at the time and could find nothing within our budget. We need to revisit it.)

He asked Dennis if Iain had given him the whole script. Dennis said he'd provided a gist. Martin said Iain had shown him the script at DConf. It was huge and had so many steps. It was much more complicated than an LDC release. It had to build the whole dlang.org, all the documentation, make thousands of commits...

He said Iain was hosting or had access to some VirtualBox VMs for all of the supported platforms and needed to store them locally on his box. If we were trying to replicate that, then we'd probably need the images for those VMs. He thought it might be easier if we just tried to automate it by expanding on what we already had in the installer repository's CI, get it set up to sign the releases, then figure out a way to handle dlang.org and all of that.

Another problem was that Iain did everything locally first to make sure it all worked. Only in the case where it all worked as expected for every repository involved in the release did he push it. Then there was the problem of the credentials for uploading to dlang.org, managing secrets and all of that. Again, that would probably best be solved by having our own private CI runners.

He said that would take a while to figure out and set up. It wouldn't help us with the 2.110 release.

Rikki said he thought Dennis should be able to set up his SSH key on GitHub. Mathias said that Dennis could generate his key and sign his commits, but it wouldn't accept just any signature. There was a limited list of people who could sign. If Dennis were to dig a bit, he might find it. Martin Nowak had been on it at some point.

Walter asked what would happen to LDC if Martin disappeared in a puff of smoke. Martin said the LDC release process was documented and automated. All you needed to do was push a git tag. Walter asked Martin to make sure Razvan knew the details so that he could find someone to fill the void in that situation (Razvan had left the meeting at this point to attend another one).

(__UPDATE__: Upon Iain's return, he brought Dennis up to speed on the release process. Dennis has managed the release of both 2.110 and 2.111, so it's no longer bottlenecked on one person. The next step is to work out an automated process that can make it more accessible.)

## Conclusion

Our next monthly meeting took place on December 13th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.