The D Language Foundation's May 2025 monthly meeting took place on Friday the 9th and lasted about an hour and thirty-five minutes.

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

Some background for those who aren't familiar with the current DIP process. Any DIP authored by one of the language maintainers is required to go through a pre-approval process before the other maintainer can render a final verdict. We do this in monthly meetings or, when necessary, special sessions focused exclusively on the DIP. If any objections are raised, they must be addressed before the DIP can move forward. For example, [DIP 1051, the Bitfields DIP](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1051.md), went through multiple such meetings.

### Editions

__NOTE__: This discussion was about an earlier draft of the Editions DIP, not the final draft that was eventually accepted.

I told everyone they should have reviewed Átila's Editions DIP before the meeting and that we needed to decide if it was ready to move forward. I asked if anyone had anything to say or object to.

Walter said the DIP implied, but did not make clear, that each edition will include all previous editions, i.e., it was a linear progression and not a branching tree. He asked Átila if that was a correct assumption. Átila said it was. Walter suggested the DIP should explicitly say so.

Next, the DIP added two switches. He thought merging them into one would be an improvement.

Next, he was confused by a section about `@system` and editionless code. He didn't understand what that had to do with editions. Átila said that was an example of what we could do with the feature.

Timon said that some breaking changes we currently were not making would be required to ensure memory safety. If we allowed future editions to call the default edition, or any prior edition that had memory safety issues, then that would undermine `@safe` in future editions without any way to fix it. We could commit to breaking code that relied on features that were not safe in the default edition, but the reason for editions was that we didn't want to break it.

Steve thought the only solution to that was to have a new `@safe`. In new `@safe`, you'd be unable to call old `@safe` under some conditions because it would break. That shouldn't break new code. It wouldn't compile to begin with. But it would mean losing access to other code that would then need to be wrapped with `@trusted` or something.

Timon thought the pragmatic way to do this would be to add `@trusted` imports. This limitation didn't apply there. Then you'd at least see that you were importing something from an edition where not all the guarantees actually applied.

Walter didn't understand. If you were writing in a modern edition and you called into a module that had no edition statement, or that was using code from an older edition, then you'd get the old code semantics. He knew that broke trust, but if you were using an edition that had safe by default, then you could require that any legacy code you called also be updated to the new edition, or you could just accept it if you wanted safe things. Wasn't that the whole point of editions?

Timon said the whole point of `@safe` was that it should give you guarantees. This didn't hold in practice at the moment. If we just allowed implicit calling of all editions without adding `@trusted` anywhere, we'd be undermining safety in future editions.

Walter said you could use the command line switch, `-edition=2025` or whatever had safe by default, and it would compile all that stuff with safe by default. The whole point of this was not to break legacy code. And that meant legacy libraries. And that meant if the legacy libraries did unsafe things, then that was what you got. But that switch allowed all the unmarked code to be compiled with the right edition. He didn't think it was a problem.

Jonathan said that if he understood correctly, the issue wasn't safe by default. It was that we might find a bug in safe that meant this particular case wasn't actually memory safe. For instance, right now, slicing a static array was treated as safe even though it wasn't. That was a hole in `@safe`. In order to fix that, we had to make it an error. DIP 1000 was an alternative workaround for it, but either way, it was a bug in `@safe` code right now. In a new edition, we would have to make that code no longer `@safe`. You would get an error in the new edition. But you could potentially call old code using the old edition, which was treating that code as `@safe` even though it wasn't.

Walter said your old code was supposed to keep working and not give errors. That was the whole point of editions. The user could add the switch to have all unmarked modules conform to the new edition. He didn't see the problem. Átila said he did see it as a problem, but pragmatically you'd probably just have to bite it.

Steve said it wasn't just about editionless code. It could also be a previous edition. The 2026 edition could have a bug that we fixed in the 2027 edition. Now you were stuck with that. Walter said the `-editions` switch would override the editions and ensure everything was compiling with the latest edition. Rikki pointed out that the DIP did not specify that the `-editions` switch would override any editions specified in module declarations, nor did it do so in his implementation.

Robert said that one thing we could do is restrict which past editions a newer edition could call. So if we fixed a memory safety bug in Edition 2028, then in Edition 2030 you couldn't call code in Edition 2027 or lower. Then you'd have to make a judgement call. The old code could still compile, but you couldn't rely on semantics down the line.

Átila said you could always put a `@trusted` wrapper in the calling code, or a `@trusted` import as Timon had suggested for a whole module. At least then you'd have something that said `@trusted` that you could then go look at as a source of bugs.

Walter said we should just use the command line switch to override everything. Jonathan said that Walter was insisting it had to be compiled with the new edition, but the problem was in trying to use old code that didn't necessarily work with the new edition, but you didn't want it to be treated like it was `@safe` if it wasn't actually `@safe`. Walter repeated that the point of editions was that you either compiled with the new edition or accepted the code as it was. Use the switch to override it if you didn't want to accept it.

Steve said there seemed to be a disconnect here. If a module specified an edition, it couldn't be overridden with a switch. The switch was there to override the default, the case where a module had no edition explicitly specified. If a module that declared it used 2027, then it would forever use 2027. That was it.

I asked if we even wanted to be overriding a module's explicit edition. Steve and Átila said no. I agreed and said that if I specified a module used 2026, I didn't want someone else coming along and forcing it to use 2028. If that were possible, then what was the point of specifying 2026 in the first place?

Walter said you needed a way to override it. That was what the edition switch was for. What was being proposed here made the edition switch useless. I said it set the baseline default. When no edition was specified, your edition switch became the default. It wasn't intended to override an edition I specified. Walter said it *should* override it. That was the point of having switches. I disagreed.

Walter said another aspect of this was that he didn't want to add another feature to make this work, like `@trusted` imports. He thought that was getting down to too much complexity for this. He liked the simplicity of it.

Adam said that this was why he had been so focused on making the current edition the default. The default *had* to be the current version. In that case, you didn't need an override switch. The compiler would assume that this code was going to accept whatever behavior the compiler enforced on it.

Walter asked what the purpose of the `-edition` switch was. Átila said it was there so that projects could work as per Steven's suggestion. If someone was trying out the language and writing a D file without specifying an edition, they would, as Adam had just said, get the latest and greatest of the current edition. The way we were dealing with legacy projects, if dub saw a recipe that didn't have an edition key in it, then it would use the `-editions` switch in the compiler invocation to set the default to the pre-editions edition. Otherwise, it would break stuff.

Walter didn't see a point to the switch in that case. He didn't know what dub had to do with it. Átila explained that if we said modules without an explicit edition declaration used the current edition, then existing projects would break unless we had a switch that dub could use. 

After a little more back and forth on this, Walter asked what the point of `@trusted` imports would be if the editionless code was treated as the current edition by default. Steve said it didn't have to be `@trusted` imports. If Edition B had a fix for a bug in `@safe` called Edition A, we could have a rule that all `@safe` functions in Edition A were treated as `@trusted`.

Walter said that would never work. Steve said if you cared about fixing memory safety bugs, then it would have to be that way. Walter said that if you cared at all about memory safety bugs, then you would force `@safe` by default. Steve said you couldn't force it on an old project. It would just fail. Walter agreed. Then you'd be able to decide what to do about it. You could upgrade it or just accept that it was unsafe code.

I said that was why you had the `@trusted` import. Because then it was documented in the code that you needed to verify it. Walter disagreed. If you were calling old editions that were unsafe, then you got the unsafe behavior. If you didn't want the unsafe behavior, you forced it to use the new edition. If you then got compilation errors, you would have to decide what to do about it. Saying something that was `@safe` was now `@trusted` just seemed wrong to him.

Rikki suggested we table the discussion about `@trusted` imports. We didn't need to make a policy on that right now. Regarding the override switch, he could implement it the way Walter wanted it. It wouldn't be hard.

I suggested we table the whole discussion for now and come back to it in a planning session. This was going to take time to resolve and we had other items to cover. No one objected.

__UPDATE__:  We had a session focused on the DIP two weeks later where we agreed to changes, then another pre-approval discussion in our October monthly meeting. This ultimately resulted in [the DIP that was approved](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1051.md).

### nothrow destructors

Steve's agenda item was about `nothrow` destructors and the detrimental effect of being unable to define which operations were catchable and which weren't. He said it was a longstanding problem.

There were times when you wanted to split your code into separate pieces, e.g., a web server with tasks. If you wanted to catch an exception and continue on, the way stack unwinding worked with `Error` made it impossible. As soon as you went through a `nothrow` function that did not add stack unwinding, then you couldn't catch and continue. You had to exit the whole program. This was something Symmetry had run into.

There were a few options to fix it. We could say that you could never catch `Error`. Therefore, and right now even, the way `Error` was propagated made it possible to get into a situation where you could deadlock on your way up the stack. If you passed through one frame that didn't call destructors and then passed through another that did, then you could deadlock. But you could just print the stack trace and exit.

The other option was to always put in unwinding handling. He'd done a test with LDC and optimizations, finding that the cost of unwinding the stack vs. not unwinding, or a destructor call, was always very minimal. He'd done 10 million calls and it was something like 20 ms vs. 18 ms. A very low performance difference compared to the rest of the program.

Another possibility was to make it possible to override. For example, someone had made a type that was a wrapper of an associative array. If you accessed an element of the AA that didn't exist, it would throw an `Exception` instead of an indexing error. It was always easier to validate while you were processing the input, and you wanted to catch and recover from that. So specifying a way to handle indexing was a possibility for AAs. He'd found a way to do that, so maybe we could add a feature to allow it.

Walter said an `Error` meant your program had failed. It meant your program had entered an invalid state. It had a bug. It had crashed. The correct, pragmatic thing to do in that case was to do as little as possible. The only purpose of catching an `Error` was to optionally print an error message, shut the program down as gracefully as practical, and engage the backup system. It was not a recoverable error. 

If you wanted to catch array out of bounds errors and continue executing, this was wrong. But if you really needed that, then you needed to write your own array type, and then it became your problem. But the language said that if you accessed an out of bounds array, your program had failed and had a bug in it.

The reason to minimize the amount of code executed there is that you couldn't know why it crashed. It could be a memory bug that had corrupted your program. It could be a malware attack. There was no case where you could know why it had failed when an `Error` like that was thrown. An array overflow could be the result of a malware attack or the result of a previous failure in the program.

`Error` should never be used to deal with input errors. Environmental things that weren't handled properly were problems you could detect and recover from. But when you got an unexpected array overflow or some such, you did not want to go unwind your stack to then execute arbitrary code and just hope everything was good. You did not need to decrement your reference counts. All you needed to do was to print a message, exit the program, and engage the backup.

He said he got into this debate regularly with D. `assert` was not a recoverable error and neither was array overflow. Steve agreed `assert` was not a recoverable error. Walter said it wasn't something you used to check your inputs. It was something which meant you'd gotten into a state that you couldn't recover from or didn't know what happened. If you were looking to check that the input a user typed was a `3` when it should have been a `4`, using `assert` was the wrong hammer to use. Steve agreed.

Walter said he disagreed with the idea that destructors ought to run in that case. Steve thought the way it worked now, having some destructors run and some destructors not run, was worse than either exiting immediately or running all the destructors. Walter said that was an arguably correct point of view. We should either do all or nothing, but currently what it did was just not guarantee that destructors would be run. He thought they didn't run on some platforms.

Steve gave the example of an input request from a web server. It sent back a JSON file that had all sorts of cross references inside it. You were looking for an object with an ID of `5`, but an object with ID `5` didn't exist. So now you had user input that had caused you to fail with an error. Not because there was an error in your program, but because there was an error in the input.

Walter said it was an error in your program if you detected it as an error. Steve argued that was just the way AAs worked. You had this nice feature where you didn't have to think about validating everything while you were working with it. You could just let it fail when you got to that point. But the semantic failure was that your input was wrong, not that the program was wrong.

Walter said AAs had a method to test if an index was valid. That was what you should use when you were faced with this problem. With linear arrays, you could test your index to see if it was in bounds yourself. But if you just threw an index at it and it caused an overflow, then that was an error in your program.

Steve agreed with all of that. The problem was that it was so nice not to have to think about it while you were writing your code. You just let the bounds checker do its thing. That was part of the type. Convenience was the whole reason to use it.

Mathias said you had three tools for AAs: you had `in` when you needed the thing to be there; you had a `get` with a default value when you didn't care it if was there or not; and you had indexing when you were sure something was in there.

Rikki said this overlapped with his next agenda item. Basically, you had three classes in the exception hierarchy. `Error` was an expensive way to kill the process that had a lot of failure points to it. `Exception` was not so much about logic, but something to do with input that wasn't quite right. It wasn't meant to be environmental. It was meant to be what you saw.

Where we were running into trouble was with the third class, something he was calling a `FrameworkException`. It would run the cleanup routines, but if you chained it, then it would upgrade to `Error` and kill the process because that meant something in the cleanup routines had failed. It couldn't be cleaned up. It was meant for environmental things like something happening on a computer halfway around the world, not a bad argument kind of thing.

He said this was coming up for quite a few people. Manu had this kind of problem within the past month with fibers. Rikki and Adam had it for Fibers v3, and he'd heard Symmetry had also encountered it. A big part of it was recognizing that we actually just didn't need to be throwing an `Error` in the first place. We could just call a global function pointer to let people configure it however they liked, and the default could be whichever way we committed to.

Átila said you could do that now. You could just catch whatever exception you were calling `EnvironmentalException` and do whatever you wanted. Rikki said you couldn't because if you were throwing an `Error`, then your program was 100% corrupt.

Walter said to stop throwing `Error` for environmental things and make sure to vet whatever was coming into your environment. Throwing `Error` for environmental things was just wrong. If you were interfacing with another program and it was behaving badly, you did not throw `Error`. You checked it. If you could recover, you checked for the error, not throw `Error` and expect to catch it. If we were tp start going down that road, we will have completely failed.

In trying to have any kind of coherent strategy with `Error` and `Exception`, they became the same thing. The whole edifice would fall apart if you started saying you could recover from `Error`. You couldn't. The whole point here was to stop throwing `Error` entirely.

Átila agreed. He said to just throw `Exception` and asked Rikki what the difference was. Rikki said it was `nothrow`. That didn't allow you to throw `Exception`. It removed the unwinding tables. Walter didn't think that marking a function as `nothrow` and then deciding to start throwing `Error` because you really wanted to throw was a philosophy that worked. Átila said that sounded like wanting a chastity belt with an escape latch. He didn't get it.

Martin said he agreed with Walter on this. Being familiar with the Symmetry code base, his impression was that there were misplaced asserts in the code they were wrapping.

The problem was that they had a scripting language and some automated wrappers for it. That meant the scripting language inherited the same problems as the D interface. For example, an operation on a range that checked for an empty range used an assertion rather than throwing an exception.

He thought `in` contracts encouraged that behavior. Specifically the shortcut of just using the parentheses and testing inside them if something was `null`. That lowered to an assertion. Maybe it would be better for the shortcut to act as an `enforce` by default. Then if you wanted the assertion, you'd have to do explicitly.

Átila disagreed. That was a programming error and should be an assertion. Martin said if a function wasn't well-defined for empty ranges, then every user of the function needed to check if the range was empty. Átila said that was correct. Martin said if that was supposed to frequently be the case or could happen quite easily, if you wanted to avoid all those checks at the call site, then you should throw an exception once in a single place instead of an assertion in the callee.

Walter said contracts were for detecting programming bugs, not input errors. Martin said that in *something I couldn't make out due to crosstalk--I think it was related to .NET* ten years ago, at the beginning of the function you'd have a whole lot of `if param is null`, then throw an `ArgumentNullException` where you had to specify the parameter name. This was quite normally the case. If you had five objects or so that your function took and you wanted to make sure that each of them were set, you did this by throwing an exception.

He said it was hard, sure, but it wasn't a pure logic error. It wouldn't be a logic error that he would throw in C++, but he would throw an exception in C++ if he was validating user input.

Átila said that if you were doing that whenever you handled user input, then you should validate it there and return a type that was validated user input. And then you would only take that type everywhere else. That would bottleneck the run-time check where it happened. From then on, you were asserting that the code was correct. That type instance could not have been created unless you validated the user input. Martin didn't see a way to do that in Symmetry's scripting language because of its design. Átila said the point was that once you did the validation and had that specific type, invalid values would be a programming error, so assertions rather than exceptions.

Jonathan circled back to Manu's fiber problem that Rikki had mentioned earlier. The issue there wasn't that he wanted to be able to catch an `Error` and recover, it was that there were cases where he wanted to have something thrown that indicated a fiber had failed so it could be handled further up outside the fiber. Nothing in the fiber would catch it. 

To do that kind of thing, you couldn't use `Error` for it because then it would be screwed up and invalid because you were skipping cleanup. You weren't supposed to catch `Error`. But if you threw an `Exception`, then the code within the fiber could just catch it and it wouldn't bubble up to the top. So Manu wanted something in between where you could skip all the exception catching and could catch it higher up while still doing the stack unwinding.

Jonathan didn't know if this was a larger problem that we needed to handle in the language, but that was the gist of it.

Steve noted that you could overload the `in` operator for a type. One idea would be to do the same thing with indexing. You could have an 'Unvalidated' type that when used as an index would indicate it was unvalidated, and then throw `Exception` instead of `Error`. We would need a way to override the indexing from outside the type.

Timon went back to Walter's point about not using `nothrow`. That wasn't possible, because `nothrow` was inferred. Walter had also said not to try to recover from `Error`. Timon agreed with that, but he still believed we should be allowed to react in a custom way to an `Error`. Even DMD did that. Átila agreed. Unit testing frameworks had to catch assertion errors, too, otherwise how would they work? (At this point, Átila had to leave the meeting).

Walter said that in and out contracts also required catching `Error` to function properly. He'd never figured out a solution to that inconsistency. He kind of thought they were a mistake in D anyway. But the thing about unit tests was that they were not about catching programming bugs. They were actually input errors, and probably using assertions in unit tests was the wrong approach. It should be a soft error, not a hard one, not a programming bug. He agreed with Timon and never should have allowed `assert` inside unit tests. That was a mistake.

Mathias took us back to fibers. Sociomantic had had servers that used them heavily. At some point, they went through their library and replaced a bunch of `assert` with `verify`, because in a lot of cases they could recover by just killing the fiber since the fiber was a request. That worked quite well. They didn't need any language feature for it. So if you really wanted a third type of exception, you could just have your own `verify` methods.

Walter wanted to remind us that `enforce` checked user input and threw `Exception`, not `Error`. Mathias said that wasn't good because it operated on the specific exception type it was given.

Timon said his point about DMD wasn't only about the cases where it caught the errors in unit tests or contracts. It was also about what happened when DMD itself threw an `AssertError`. It had a message asking you to please report the bug. So it had a custom way to make the experience of the error more useful.

Walter added that the original use of `assert` in unit tests was to stop the program. The idea that you could continue after the assertion failures came later.

I asked everyone where we were at with the discussion.

Walter said this came up often. He thought he should write a document that laid things out clearly. Then if there was any confusion about it, he could amend the document. Then we'd at least have a touchstone when it came up again. Steve thought such a document should have a list of examples of how to handle specific situations, because it came up over and over and wasn't easy to figure out. 

He also wondered if Walter would be amenable to a possible language improvement to implement what he'd mentioned earlier about overriding indexing. Walter thought we had `opIndex` overloads already. Steve said we didn't have it for AAs. Walter asked if he couldn't write a wrapper.

Steve said the problem he'd always run into was that there were times when he had an index he knew was valid and wanted it to throw an error if he used it. When he had a different index that wasn't necessarily valid, he'd want it to throw an exception. It wasn't the type that needed to be validated, but the index itself. So having a type that said all indexing that fails was an exception vs. an error was also not correct.

Jonathan said he could use a wrapper function that did the indexing. Symmetry did that in their scripting language for something for safe indexing. So you'd have a function that you could pass the array to using UFCS instead of indexing, it would do the check and throw an exception. And it could even use `.ptr` to avoid the secondary bounds check. Then there was no need for a wrapper type.

Steve said it depended on what you were willing to deal with in terms of verbosity and how you wanted to do your stuff. Sometimes indexing was nice.

We talked a little bit more about Walter's documentation idea, then I suggested we shelve the discussion for now. We could tear apart his document once it was written.

### AI usage in the contribution guides

Rikki said we'd had some PRs which we'd had to close because people had used LLMs, and they weren't positive contributions. He was thinking we should add something to the CONTRIBUTING.md in each core D repository to politely let people know what we were expecting. Something like, 'LLMs are great tools, but we're interested in *your* work, not theirs.'

I noted that in at least a couple of cases, the submitted code was just nonsense. Who knew that Vladimir would turn out to be a good AI detector? He had caught a bunch of stuff during the GSoC application process and he'd been catching PRs. I thought we probably should have a policy. Rikki said it should just be something polite to let people know we'd close the PRs if they didn't adjust.

Walter said a lot of major companies were using a lot of AI-generated code. He didn't think there was anything particularly wrong with it unless it was garbage. I said there was nothing wrong with it unless there was something wrong with it. It wasn't that AI was used to generate it that was the problem, it was the quality of the code it was producing. 

Walter said it didn't matter if it was AI-generated or idiot-generated, if it was bad code, it would be rejected. I agreed, but this was coming up because it was being submitted by people who had little to no experience with D or probably even with compiler development. They were basically GSoC candidates.

Rikki said it was fine to use AI as a tool, but they were using it beyond that to do the whole thing. Even comments in the PR. It was like they weren't understanding the code they were submitting.

Walter said he'd found AI useful to help him understand how to use certain APIs correctly. He generated an AI version of it, then went through line by line to make a PR out of it and it was fine. He wasn't categorically against using LLMs as a tool. Rikki said that was a good usage of it. The kind of PRs we were talking about were not a good usage of it.

Razvan didn't see the point of singling out AI in the contributor guide. The point was if someone made a good contribution, then it was going to get merged. If it was bad, it wasn't going to get merged. What was the point of specifically mentioning LLMs?

I said it was a warning to make sure people were aware they needed to carefully review the code they were submitting if it were generated by an AI. I didn't see the harm in having that.

Razvan said he would rather have something in there saying to make sure the submitted code was at the highest standard, and we'd accept nothing less. I said that was true, but some people were going to believe that any code generated by AI was going to automatically be at a high standard.

Jonathan suggested putting a comment in there saying that some people had submitted AI-generated code that was garbage, so please do not just assume that AI-generated code was going to be high quality.

Timon said the main risk with LLMs was that you'd get flooded with bullshit. This was happening to more popular projects where they just got flooded with more PRs than they could review. Now they somehow had to determine which of those were LLM slop and which were actually useful. That was the basic risk.

Adam said there was a huge email thread on the GSoC mailing list about how to deal with this. They had some specific problems because they were obviously looking for students who were qualified. One of the things that came out of that conversation was that if a student was using an LLM to do it, they were kind of admitting that they weren't qualified to the do the work.

He understood what Razvan was saying and didn't have a problem with LLMs per se. What Walter had described doing was a good use case, going in and changing what you didn't like. What ended up happening there was it didn't look like AI slop anymore, but more like something a human actually wrote. 

But the GSoC guys, particularly the ones with larger projects, were having a huge problem with the kind of thing Timon had talked about. They were getting flooded with crap PRs they had to wade through to figure out if the person was being real or not. GSoC was a competitive program with a financial incentive involved, so this was probably being pushed a little harder there than we'd ever see in our day-to-day. The problem wasn't the LLMs, but the way people used them to as a crutch to try to get around actually producing quality code.

I asked Rikki if any of this helped him with how to explain it. Rikki said it did. There a very subtle difference between "we're interested in the person's work and the LLM did all of it" versus only some of it that got filtered.

Steve wondered if it could be a note in the PR template. Like a box the submitter could check to say that they used AI but weren't submitting unfiltered AI slop so that the reviewers could know what they were looking at. It was frustrating if someone submitted a PR that was 50 lines of code, and you were looking at it and it didn't make any sense. You tried to work through how they thought this was a good idea only to find out later they didn't do any work at all. They just spit it out of ChatGPT. Now you were the one paying the price.

He thought we should have some way for the submitter to promise they weren't submitting AI slop, then if they were found to be dishonest, that was it. They were out. They didn't get to submit PRs anymore.

I said that was an interesting idea and noted that YouTube now had a box to tick when you uploaded a video to indicate if you used AI to generate anything intended to be realistic. The didn't care if you used it to generate animations or thumbnails or whatever. But if you were doing anything to make viewers think a scene was real when it wasn't, they wanted you to check that box. They weren't doing anything globally with it yet, but down the road they were going to be adding labels to videos that ticked that box. Then if you uploaded some realistic AI stuff and didn't check the box, they'd ban you if they found out.

Walter thought the GSoC thing was about students getting experience coding. Using AI in that case defeated the purpose. He'd be fine with saying we didn't accept AI-generated code for GSoC contributions.

Adam said that was part of it, but it wasn't just students anymore. That was kind of where the problem cropped up. There were just a lot of young people treating it as a paid internship. It was a huge discussion thread. He'd been following it because he'd been able to identify the applicants who'd submitted AI slop for the project he was mentoring.

He thought Steven was right. It ended up being a steep price the PR reviewers had to pay. He'd seen the AI checkbox on YouTube, too, and thought we should do something like that. Walter agreed. If they lied about it, we could just ban them for life. Razvan said they could just create another GitHub account. He didn't see how banning was going to work.

Rikki thought there were two simple things we could do. One, add a line to CONTRIBUTING.md that we wanted contributions from humans, not automated stuff. Two, our bot could just tell people to tell us if they used an LLM on the code. It didn't get in the way for regular contributors. If those we didn't know weren't going to tell us, then they were lying and we could ban them if we found out. It was a nice middle point.

Timon thought that creating multiple GitHub accounts was against the GitHub TOS. So if they had a longstanding account and created another to commit to our repos, they could probably get both accounts banned.

Razvan said we could look into software that identified LLM-generated content. Maybe it would have to be in the CI, then PRs could be checked against it and it could output a message that a PR had been generated with, e.g., 70% or 80% certainty. And tell the submitter it would get less priority for review. Then we could just attach a label indicating it was LLM-generated and be done with it.

Jonathan said that made sense if we started getting a lot of slop. Until then, we shouldn't worry about it. 

Martin agreed. As for CONTRIBUTING.md, he thought it shouldn't be formulated as against AI, but more along the lines of telling people that if they used AI to support a PR, they should show that they reviewed the changes first themselves and understood them. As a reviewer, he didn't want to spend time arguing over or discussing a point in a PR where the submitter didn't even look at it and had no idea about it. Before AI, when someone proposed a change, you could at least assume the person had looked at the surrounding code, understood the problem, tried to tackle it, and didn't just ask AI to improve a certain aspect of the code base. He didn't care if the final change was completely AI-generated, it just needed to be understood by the person submitting it to save the reviewer's time.

He added we should also put something in there about the motivation, that it was about decreasing pressure on our reviewers. Walter said that was a good point.

I suggested that Rikki just go ahead and submit a PR to CONTRIBUTING.md, then everyone could review it, give their feedback, then hit the merge button or not. He said there was one in every repository. He'd start with DMD.

### Hidden feature discovered

Steve discovered that since the inception of DRuntime, there'd been a feature in `ModuleInfo` that if you created a function named `getMembers`, a pointer to that function would be stored in the `ModuleInfo`. He found that very interesting. He had happened across it and wondered what it was for. He traced it all the way back to the original by Sean Kelly. 

He'd had an idea about using it with unit tests. If you weren't using the default unit test framework, you had to import all the modules you wanted to test and use `__traits(getUnitTests)`, then register everything in your own registry. `getMembers` could be a way to register unit tests for every module at run time. Then there was no need for a compile-time search. He wondered if we could explore this.

Additionally, the unit test system should have a way to say that you only wanted to run the test from module `x` right now. Imagine a unit test in one module that took a long time to run, but you were trying to test a different module. You had to wait for that first one to run or comment it out.

Rikki said he'd explored `getMembers` over ten years ago. There was a major problem with it. You could only have one strategy that was used for the entire process. You couldn't do one per framework or per library. It was per process. Otherwise, it would get confused and then bad things happened.

He wanted a registration system based on UDAs and to put things in a compiler-backed list. The UDA was put on a symbol, the symbol got transformed, and that transformed result went into the list. He wanted it for web frameworks and such. The reason he'd not yet done anything in depth with it yet was that he needed Martin to tell him how the list worked and it was very low priority.

Martin said that if he'd ever seen `getMembers`, it had been banished from his mind. The unit tests were currently implemented as a function pointer in the `ModuleInfo` that pointed to a function that called all the actual unit tests in that module. So it wasn't like all alternative unit test frameworks needed compile-time reflection. It was needed in `unit-threaded` only because it had to iterate over a bunch of UDAs, and maybe to give the tests nice names. Other than that, you could iterate over all the `ModuleInfos` and filter on the module name. He'd like to see some built-in functionality for that.

If the main usage were something like giving the tests better names, then we could come up with a built-in UDA for that. Instead of the function pointer in the `ModuleInfo`, we could have an associative array mapping the names to the actual unit test functions. Or maybe even exempt them with other UDAs that you could convert at compile time to something you could use at run time.

As for `getMembers`, if you were using that to get the UDAs, then you'd need to translate them to a constant variable so that you were then going to get the `ModuleInfo`. That would make it a bit more complicated. He'd prefer to see old things that no one was using like that to just go away. Just like the object factory thing. He hated that every `ModuleInfo` pulled i a whole of symbols, the virtual tables, and then all of the functions of a class. He'd prefer to get the `ModuleInfo` as clean as possible and deal with the unit tests in a different, more structured, future-proof way.

Steve said the idea behind it was you could do all your compile-time reflection stuff in a mixin that would then hide it behind a function. Right now, if you were using unit-threaded or whatever, you had to import all the things you wanted to generate. You needed to implement a central registry, whereas with `getMembers`, the central registry already existed. That was the only thing it helped with, so you didn't have to think about which modules had these things in them.

He said we were due to have some kind of DRT switch to run the unit tests only from specific modules. That was a no-brainer, easy thing we could do today. We didn't have to make any changes to the code or the compiler.

He really liked the idea of an AA to run all the unit tests. We had the ability to make AAs at compile time, so it made sense and would work if we put a name on each unit test.

Jonathan said we should be able to do that without any compiler support because you could just do what unit-threaded was doing. If the test had the UDA there with the string in it, it just grabbed it and then you had a name. Otherwise it was whatever the default was.  Steve said you couldn't do that because the compiler was building the `ModuleInfo`. We had to do whatever it accepted. It didn't have a way to look at those strings. Jonathan said we'd have to update whatever was in there for the `ModuleInfo`.

Steve agreed. He had just thought this was an interesting idea and didn't know if we could exploit it. The code was already in there to do it. He didn't think anyone was using `getMembers` for anything. It was basically just a wasted null pointer for every single `ModuleInfo`.

The other option was to remove it from the compiler completely. He didn't know if that would break anyone's code.

### Conclusion.

As we wound down, I gave some updates on DConf planning. That prompted Walter to remind us about the UK's new ETA (Electronic Travel Authorisation). Dennis wanted some clarification about the behavior of the `-nothrow` switch for a PR he was working on. And then we were done.

We held our next monthly meeting on June 13th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.



