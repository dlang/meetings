[Forum Post](https://forum.dlang.org/post/zfmkiqwkkexeaohjsuru@forum.dlang.org)

The monthly meeting for March 2022 took place on March 4 at 15:00 UTC. The following foundation staff and contributors were present:

* Andrei Alexandrescu
* Walter Bright
* Iain Buclaw
* Ali
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Razvan Nitu
* Mike Parker

This was a three-hour meeting that covered a lot of ground.

I want to note that when someone makes an argument or suggestion in these meetings, they usually use a good bit of dialogue to make their case. And, more often than not, there is some amount of discussion as people reinforce the points raised. For context, and in anticipation of reader questions or inferences, I sometimes note some of the points raised when I think it's useful to do so and the points aren't too complex for a concise summary, but I'll usually leave out things people say that reinforce those points. My goal isn't to summarize everything everyone said, but just the major points and counterpoints raised. Sometimes, when I feel like I can't concisely summarize the larger context of someone's argument, or if the larger context isn't relevant, I'll just condense things down to "so-and-so suggested x". I just want to make it clear that there is always more to that "x" than just "x".

## Me
### D ecosystem services
While we were waiting for everyone to arrive, I gave an update on the status of our plans to bring all of the ecosystem services under our control. At that point, I had received information on the resource requirements of services maintained by Vladimir Panteleev (the D forums, D wiki, and more), Jan Knepper (the main dlang.org site, the newsgroup server, the D blog), and Petar Kirov (run.dlang.io and tour.dlang.io). I also had received affirmative responses from those three to my invitation to join our next server meeting. Since that time, I have also received the same from SÃ¶nke Ludwig (code.dlang.org).

Throughout February, I had received advice from different people about which name registrar we should sign up with. At the meeting, everyone agreed I should just pick one and be done with it. I have since signed up with Namecheap. Now I'm waiting for the owner of dlang.io to get back to me about handing it over to us (he brought it up in an email to me back in January). Walter has always maintained dlang.org, and he will transfer it to our new account in the future.

### Flipcause
I informed everyone that Andrei and I had decided to terminate our Flipcause account at the end of this year. We signed up for it initially with the hope that it would be beneficial to us, but in the end, we decided the value we get from it isn't worth the annual fee (if we had a higher volume of donations, it would really be great for us because most donors there cover the transaction fees, an option PayPal doesn't offer, so it could potentially pay for itself). I'll miss the ability to set up targeted campaigns with a variety of optional features to choose from, but the annual membership fee we pay them would be better spent on server hosting. (That said, if you plan to register for DConf '22, please use [the Flipcause page we've set up for it](https://www.flipcause.com/secure/cause_pdetails/MTQ3NjEy)). For those of you making monthly donations through Flipcause, I'll contact you later this year to notify you when we're ready to shut it down.

### DConf registration rates
In the middle of the meeting, I received an email from Symmetry with their proposed DConf registration rates. They asked me to get feedback from the foundation staff. Given the fortuitous timing, I interrupted the meeting to get everyone's opinion. They all gave the thumbs up. Those are the rates [you now see at dconf.org](https://dconf.org/2022/index.html#register). Walter suggested we also offer a hardship rate as we have done at past conferences. I have added information about that to the site.

## Martin
Martin let us know that LDC was prepared for the next major release. He said there was one thing he was getting ready to do that he had been wanting to do for years: change the `extern(D)` calling convention to not reverse the formal parameters anymore. Until very recently, there was a coupling between DRuntime and the compiler which depended on this reverse order. This is an issue that has persistently popped up in the forums. It especially was a problem when someone was trying to fix Dwarf emission.

One problem with this is that naked asm functions that take multiple parameters, and which assume parameters are in specific registers or stack slots, will break. He had to change some things in Phobos, which makes a significant diff between LDC Phobos and upstream Phobos. It will be easier if DMD can follow suit.

## Iain
### Deprecations
Iain has been ending the deprecation period of things that were deprecated and forgotten about, and adding end dates for deprecated features that did not have one. He iterated a few examples, some of which are now errors. Walter asked about the status of complex types. Iain said they were already deprecated, but now they have a date set.

During this discussion, Iain remembered that he would like to add a `toNative` function to `std.complex`. Although the library implementation is binary compatible with the C type, a struct with two doubles may not be passed as a parameter in the same way as a C complex type. The `toNative` function would ensure compatibility in that case.

### scope struct
In his iteration of deprecated features that are now errors, Iain mentioned `scope interface`. This led to a conversation about `scope struct` and `scope class` types. Before the instance-level `scope foo = new Foo` became the accepted means to force stack allocation, it started life as a feature for type declarations, e.g., `scope class Foo {}`, the meaning of which is that all instances of `Foo` are allocated on the stack. This has a problem that occurs with composition (if a type has a member that is a `scope` type, then that type is also effectively a scope type). Ultimately, everyone agreed to deprecate scope types. (Since then, two PRs have been merged that deprecate scope on [struct/union & enum declarations](https://github.com/dlang/dmd/pull/13767), and [also on class declarations](https://github.com/dlang/dmd/pull/13669)).

### GDC
As for GDC, Iain's still keeping track with master. Martin asked about a flurry of PRs Iain had submitted upstream, to which Iain responded that he's been testing on a variety of targets (Big Endian targets, strict alignment targets, etc.), and that has led him to discover several regressions in DRuntime.

## Razvan
### Changes to DMD's lexer
Razvan has gathered a couple of students who are working on integrating DMD as a library for dscanner. They are trying to replace all occurrences of libdparse, but it turns out they need to make some modifications to the frontend to provide an interface that matches what tools using libdparse expect.

The first change was to create a range interface for the lexer, versioned in the frontend. Walter requested that they move the range interface to a new module rather than making such an intrusive change in the lexer itself. Razvan said that's not a problem, but some changes do need to be made to the lexer. Specifically, DMD drops whitespace, newlines, and comments, but libdparse does not, so some code needs to be added to the lexer so that they aren't dropped when DMD is used as a library. Walter suggested creating a libdparse module to the frontend for the new code if possible to avoid adding versioned blocks to the lexer. Razvan agreed to look into it.

### DIP 1008 and Phobos
In his SAOC 2021 project (replacing DRuntime hooks with templates), Teodor Dutu encountered a situation where replacing the hook for throwing exceptions results in linker errors in some cases (it's related to template emission when mixing objects compiled with different switches). The only way to move forward is to compile DRuntime and Phobos with `-dip1008`. Unfortunately, DIP 1008 requires that exceptions not escape the catch block when they are caught, but Phobos does this in some cases by either saving the exception to a variable or returning it. This prevents it from compiling with `-dip1008`.

Razvan says it's not obvious how to solve this. Cloning the exceptions puts the burden on the user to free the memory, and that's not a good solution. Moreover, DRuntime is still using the GC for all exceptions under the hood when `-dip1008` is enabled to allocate the stack trace ([as Razvan first discovered in the now-closed PR he submitted in 2019](https://github.com/dlang/dmd/pull/10561#issuecomment-553824621) to enable `-dip1008` by default). Martin noted that the GC is used in trace demangling as well. There was a good bit of discussion about this, with Walter finally asking Razvan to make sure there was a Bugzilla issue linking to Razvan's PR for context.

### Reference counting
Going through some old DRuntime pull requests, Razvan found several PRs adding reference-counted things (RCArray, RCPointer, etc) from a period when reference counting was a hot topic in the D community. The problem with reference counting in D has been the transitivity of qualifiers (e.g., you can't reference count an immutable object). Razvan remembered he had drafted [a DIP for a `__metadata` storage class](https://github.com/RazvanN7/DIPs/blob/Mutable_Dip/DIPs/DIP1xxx-rn.md). In a forum discussion on that DIP, Timon Gehr had pointed out two fundamental issues with it (anyone interested can see [the forum discussion thread](https://forum.dlang.org/post/3f1f9263-88d8-fbf7-a8c5-b3a2a5224ce0@erdani.org)). Ultimately Razvan's progress stalled and he never submitted the DIP.

In our meeting, he said he would like to see what the community thinks about this DIP now but would like to know first if this is still important for us. This led to a significant discussion that touched on various aspects of reference counting and immutable: the possibility of a more restrictive form of C++ `mutable` (in the form of something like `__metadata`: not part of the type, not part of the object status, possibly stored before the pointer of the object, etc.), why allowing modification of immutable variables from a `pragma(crt_constructor)` is a bad idea, the pros and cons of replacing `pragma(crt_constructor)` with `extern(C) shared static this` (which in turn would allow initializing immutables when there's no DRuntime). I expect Razvan will be revisiting this in the forums in the future.

### Making malloc and calloc @trusted
Razvan next brought up [a two-year-old DRuntime PR](https://github.com/dlang/druntime/pull/2901) for making `malloc` and `calloc` `@trusted`. He had intended to merge it, but Iain removed the automerge tag. Razvan asked why it can't be merged. Iain's primary motivation was that there were 20 months between the last comment and automerge being applied. The PR has a large discussion thread, so Iain wanted to give time for it to be digested in the present and start the discussion again if necessary to prevent the case where it's merged, but then we realize two weeks later it has problems.

So Iain prompted a new discussion in the meeting: is `malloc` really trusted? There was a lot of input on this: what do we gain by it; both require an unsafe cast of untyped memory; `calloc` has a stronger case because it initializes memory; `malloc` might be safe in specific situations, like when emplacing; or maybe not in that case since it takes no type information; and so on. In the end, Andrei asserted that `malloc` and `calloc` are unsafe because they use untyped memory and there's just no way around it. Walter suggested that Andrei's answer be added to the PR discussion and the PR closed. Andrei proposed stating the argument in the PR thread, but not closing it yet to allow for the chance that someone can come along with a compelling code example that shows how making `malloc` trusted would be beneficial.

Razvan did so, and that prompted more discussion on the issue:

https://github.com/dlang/druntime/pull/2901#issuecomment-1060702512

At the end of which, Walter has since weighed in with a final decision:

https://github.com/dlang/druntime/pull/2901#issuecomment-1086999519

## Dennis Korpel
Dennis asked for clarification on a couple of contradictory spec PRs that aimed to clarify what happens with overlarge shifts (e.g., shifting a 32-bit value by 33 bits). [The first was from 2016](https://github.com/dlang/dlang.org/pull/2578), in which Walter changed the spec from saying it's "illegal" to saying it's "implementation-defined". After Andrei requested changes, Walter never got back to it. In response to a Bugzilla issue, [he opened a second PR in 2019](https://github.com/dlang/dlang.org/pull/1420) in which he changed it to "undefined behavior". In that thread, "unspecified behavior" was raised as an alternative. Dennis wanted to know which PR is correct.

Walter explained that it's not actually undefined behavior (which could result in "launching nuclear missiles" or something), but is softer than that. It's either going to result in 0 or in wrapping around, so it should be "implementation-defined".

Dennis asked Martin what LDC's (LLVM's) optimizer does in that case (being based on C). Martin said the LLVM docs specify it to be a poison value, which [is defined in those same docs](https://llvm.org/docs/LangRef.html#poisonvalues) as possibly leading to undefined behavior in certain circumstances. Walter noted that compilers can choose to delete code that is specified as undefined behavior, which is the wrong answer for the case of over-shifting.

There was general agreement that "implementation-defined" is the way to go.

## Mathias Lang
### @dlang.org
Now that we have migrated dlang.org emails from our mail server to Google Workspace, Mathias hopes we can set up a mailing list there to discuss language issues outside of the monthly meetings so that we can save time during the meetings and decrease the chance of overlooking things.

### arrays on the stack with scope
On many occasions, Mathias has wanted to be able to use `scope` to allocate dynamic arrays on the stack in the same way we use it to allocate classes on the stack via, e.g., `scope c = new MyClass`. He's tried wrapping allocations in a typesafe variadic function, but you can't concatenate them. He wanted to know if there was any opposition to this. Walter said it sounds perfectly reasonable. Dennis said he had [already opened a Bugzilla issue for it](https://issues.dlang.org/show_bug.cgi?id=22306).

Mathias said he can look at implementing it if he gets some time. If anyone wants to beat him to it, please let him know. This does not require a DIP, just someone to implement it.

### -preview=in
In our quarterly meeting last October, we agreed that the behavior enabled by `-preview=in` should be the default, with the exception that it should always imply `ref` instead of letting the compiler decide (see the section 'The fate of -preview=in' [in my summary of that meeting](https://forum.dlang.org/post/orhmmphjmtdmpbsiwrjq@forum.dlang.org)).

One of the problems he's faced is in making it work with `extern(C++)`. There's some disparity between the std C++ bindings in DRuntime and those in the DMD test suite, so he's trying to put everything together in DRuntime. He noted that a mono repository containing DRuntime and the D frontend (something we've discussed in the past) would be beneficial here.

Another problem is that when `in` is used as a delegate parameter in `opApply`, then `in` must also be usable with `foreach` variables. Walter said that makes sense.

### Inference of storage classes on delegate parameters
Something that would make writing code easier is the inference of storage classes on delegate parameters. He pointed to [an old issue in Bugzilla](https://issues.dlang.org/show_bug.cgi?id=9423) and [its linked discussion thread](https://forum.dlang.org/thread/mixmakdqfmaznmmnizux@forum.dlang.org) where there were mixed opinions on whether `ref` should be inferred or required on delegate parameters. Those who wanted to require it were thinking of `ref` at the call site in C#, but in D we're not using it on the arguments in the call, but on the parameters in the delegate. Mathias doesn't think that makes much sense. It prevents us from doing things like passing delegates to `std.algorithm` that default to `ref` to avoid copying.

Mathias has wanted this for a while, but has put it off because the behavior in the presence of overloads is "a bit tricky", but he believes this is something we should have. Walter had no opinion on this yet and would like to look into the details first. Mathias suggested this is something we could discuss on the dlang.org mailing list, and Walter agreed.

### -checkaction=context should be the default
Having `-checkaction=context` as the default will be beneficial to the language, but it's currently broken. Unfortunately, a recent change broke it differently. We should find a solution for that. The feature is a no-brainer, and once you start using it you never go back.

Dennis noted that one reason not to use it is that it increases compile times, and Mathias said he hadn't considered that.

### Attribute inference on private functions
Mathias thinks that inference on private functions is something everyone would want. It's something he heard Walter mention in the past. The current blocker on that is that attribute inference doesn't work for recursive functions. He thinks the solution is simple: you should assume that the function has all the attributes; if you recurse on yourself, the rest of the code will help you infer the attributes. Walter said that makes sense; the more attribute inference we can do the better.

Andrei noted that we should be careful in the future if we add an expansive attribute, as that would prevent this from working. He also mentioned that there is a, possibly tenuous, argument to be made about the safety of recursive functions due to the possibility of stack overflows.

Martin suggested this requires evaluation of its compile-time costs. Walter said that's a reasonable point, but thinks we'll just have to endure it. We've got so many attributes that the more inference we can do the better the user experience. Martin said that as long as compile times don't double, that's fine.

Andrei noted that there have been studies, e.g., with Java, showing that people generally don't write attributes. More inference is the way to go.

### alias
Mathias says we resolve aliases in the compiler too early, causing the name assigned as the alias to be lost. For example, in error messages, you see the aliased thing, not the alias itself. It also creates problems with visibility. Sometimes you want to have a public alias to something private, but you can't do that because the compiler sees right through the alias.

Walter said the visibility issue is a feature, not a bug. It was a deliberate decision. Allowing a public alias to a private symbol violates encapsulation. Mathias provided a use case from his company in which they essentially wanted to hide a private getter function in a struct behind an `alias this`, i.e., allow the subtyping but not allow direct access to the getter. Walter argued that the alias allows direct access to the getter anyway and creates a hole in the type system. Mathias argued that it has to be done in the same module anyway, and since `private` is at module scope, he doesn't think it breaks encapsulation. Walter said that if Mathias can make a compelling argument for this, then he'll consider it, but for the moment he can't see any good reason to allow it.

Razvan suggested that if anyone wants to hide a private symbol behind a public alias, they should just wrap the private thing in a public function. Walter agreed, but will still consider a compelling use case if someone can show him one.

## Ali
Ali said he managed to convert one of his programs to SSH copy itself to a server and work like rsync. He said he's a happy D user as always.

One of Ali's coworkers is a Rust enthusiast. He and Ali agreed on a sort of challenge to each code a certain program, Ali's in D, his coworker's in Rust, and maybe write a blog post about their experience. He asked that anyone with suggestions on what that program should be to please let him know. I don't know if any of the others subsequently emailed him with ideas, or if he's still looking, but anyone reading this, please post ideas in this thread. Maybe Ali can chime in on what has happened since.

## Walter
### ImportC
Walter reported that ImportC is coming along nicely, and he has fixed a bunch of ImportC bugs. Unfortunately, some bugs are intractable and we may just have to live with them. Transitive const is an example: C doesn't have it, but ImportC does. Someone found a case in a C header (a const pointer to a mutable object) that resulted in a compile-time error. Walter's not sure if that's fixable.

ImportC has advanced to the point where we need to be working on getting the preprocessor up in some form. Max had volunteered to look into it in, but Walter didn't know if he had done so yet. If not, we need to get seriously working on it. Not having compiler support for running the preprocessor is a crippling user experience problem. Iain has concerns that if the D compiler is invoking the C preprocessor, it might not be calling the correct one. This led to a discussion about invoking a preprocessor vs. having our own.

Martin raised a point about a potential problem with ImportC in LDC (and probably other compilers) related to how ImportC handles C header files. The short of it: importing the same header in multiple D modules will result in the repetition of C declarations. He says the solution is for ImportC to treat each C header as a D module. Walter isn't sure that will be an actual problem in practice, but he thinks Martin's solution is a good one.

Iain raised [an ImportC issue](https://issues.dlang.org/show_bug.cgi?id=22842) that he thought would be difficult to fix. Walter has since fixed it.

### DIP 1000 issues in Bugzilla
Walter had begun working on reducing the number of DIP 1000 bugs in Bugzilla. Many of them turned out to be duplicates, but a number of them are related to foreach loops and delegates which need to be cleared up.

One of the problems with going to DIP 1000 by default is what to do about `ref return scope` ambiguity. Walter had finally come up with a fix that was half-implemented. Some of the PRs had been merged, but others were stalled. Those stalled PRs are blocking progress on DIP 1000, and he wants to get those moving.

## Andrei
Andrei was unable to attend the last meeting where we had an inconclusive discussion about bitfields in D, but we've since had an ongoing email discussion. Andrei suggested we continue that discussion here.

He started by arguing that reflection on built-in bitfields is "just warped". You're going to have things less than 8-bit and to which you cannot take a pointer. Any proposal for bitfields must account for that. Walter's solution of having `__traits(getMembers)` return an aggregate is only half of the story. What about the getters and setters? A library solution has those, but for the built-ins, they're compiler magic. Martin agreed (and this is his main sticking point; he brought it up in the last meeting).

To handle the getters/setters, Walter thinks it may work to have a new trait that can query if a given member is a bitfield or something like that, but he hasn't worked it out yet. An alternative is that the compiler can "rig up" the getters/setters.

Andrei argued that the cost of special-casing reflection just for bitfields must be weighed vs. the convenience of "we already have them in ImportC, let's just enable them in D". He says reflection on bitfields is a serious liability.

Iain asked if bitfields are really the issue? Why not just have n-bit types? To which Andrei responded that D used to have a bit type, but it was removed precisely because of the liabilities that would accompany built-in bitfields.

Ali argued that we should make the library bitfield syntax like the C bitfield syntax. Martin doesn't think that brings us anything. `getMembers` still returns the template instance and not the fields.

Martin noted bitfields have an advantage over the library implementation and Iain's n-bits type in that you can add comments to each individual bitfield member.

Walter noted that using the library bitfields is problematic for static initializers. Andrei said that's a valid point, but it could be addressed in the library code.

Mathias suggested the current implementation takes the wrong approach in defining fields to match how one would use bitfields in C. He thinks instead it should just define accessors and align on byte. His company took that approach for its own solution. Andrei said that's a valid alternative and would simplify the code.

There was a lot more discussion on this and some tangentially related topics (the possibility of making the library solution compatible with the layout of C compilers, simplifying the library syntax, n-bits on FPGA, packed bools, a standardized D bitfield layout vs. compatibility with C implementations, and more). Much too much to summarize here.

I don't believe we reached a conclusion on this, but Andrei did suggest Walter should experiment with the library implementation before moving forward with a built-in implementation.

## The next meeting
Our next meeting is a quarterly meeting, so it will involve the usual industry representatives. It's scheduled for Friday, April 8 at 14:00 UTC.

As usual, please let me know if you have an issue you'd like to bring up for discussion at a foundation meeting. Our quarterly meetings tend to run longer (three hours is highly unusual for a monthly meeting), so I'm unlikely to bring you into this next meeting unless it's something that can benefit from input from the industry reps, but we currently have room at the next monthly meeting in May.
