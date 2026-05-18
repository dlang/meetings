The D Language Foundation's September 2025 monthly meeting took place on Friday the 12th and lasted about an hour and thirty-five minutes.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Martin Kinkelin
* Dennis Korpel
* Átila Neves
* Razvan Nitu
* Mike Parker
* Robert Schadek
* Steven Schveighoffer
* Nicholas Wilson

## The Summary

Before we got started, I let everyone know I'd tied everything off about DConf '25. I'd paid out all the speaker and staff reimbursements, and I'd had the final after-action review with Symmetry and Brightspace, our event planners. I was looking forward to starting discussions about DConf '26 in October or November.

We were also on the verge of kicking off Symmetry Autumn of Code 2025 the following week. We had received 11 project proposals, which was a record. The most we'd ever had previously was six. We only had the budget for three slots.

__UPDATE__: DConf '26 planning is ongoing. At the time of publishing this post, I'm waiting for confirmation that the contract between Symmetry and Brightspace is approved and that Brightspace has locked in our preferred dates. I'll announce as soon as I'm able.

### An assert in the ARM back end

Rikki wanted to know why [a specific assert, `assert(elfree)`](https://github.com/dlang/dmd/blob/4c54accfad57077120025e6768f30544ae5759f3/compiler/src/dmd/backend/arm/cod1.d#L958), was in the `cod1.d` file in the DMD ARM back end.

Walter said that function was copied over from the x86 code generator and then modified. The way effective addresses are calculated in ARM is quite different from how they are for x86, so the logic had gotten a bit scrambled.

The assert was there because the code diverges early on in the function, `elfree` is set early on, and then the code merges back together and diverges again. It was the old "if you have X, do some code, else do some other code, and then you have some more code, and then if you have X, do some code, and then do some other code." The assert was there to ensure that it fell down the correct branch.

He also said that since the code had started out as x86-64 logic and then been heavily modified, it was kind of a mess. It didn't really fit what AArch64 did, and that was a fairly common pattern in the AArch64 generator. His goal wasn't to write the best code, optimized code, or clever code. His goal was to get something working. He was putting refinement off until later, because until it was working, he couldn't know which paths were needed and which weren't.

He said he sometimes felt like Dr. Moreau, creating a monster by cutting and pasting and holding things together to make the code generator work.

Rikki said that you could scan the function in its current state and prove at that point that the assert was false, depending on the path taken. Walter said that if Rikki could find a way to trip it, that was fine, but he shouldn't be able to. Rikki insisted there was a path where it could be false, and Walter asked him to send a test case.

Rikki said you could see it just by scanning the code. Walter said that could possibly be true, but the `if` statements causing it were on paths that could only be taken by the x86-64 logic, which never happened there. He wasn't sure it could actually be tripped. If Rikki could send him code that did it, he'd fix it.

Rikki said he'd had some false positives in the x86 files and fixed them. This one ARM file was the only one left, and he believed it was a true positive. Walter again asked him to send some code that triggered it. Rikki said he didn't have concrete code that did so. It was just a static analysis of the function showing a pathway for it to be false.

Átila asked how Rikki knew the static analysis was correct. Rikki said he could scan it himself. Átila said he was with Walter and would want to see code that actually triggered it.

Rikki repeated that he had no code to trigger it. He just needed to confirm whether it was a false positive, which he didn't think it was, or whether it needed to be fixed. Walter said he couldn't confirm that. How it got to that point depended on the states of a bunch of other variables, and that particular state should never happen.

Steve asked if Rikki's static analysis could specify the conditions that caused it to be false. Rikki said it couldn't because he didn't have tracking for the error paths, and that wasn't in scope. Just looking at the code, most of the `if` statements there were `if(0)`.

Walter said to keep in mind that there was a lot of logic in there for x86-64 address generation. That code was never going to be triggered in the ARM generator. He just hadn't gotten around to pruning it out. He wasn't focused on optimizing and removing things yet. His goal was just to get a functioning ARM generator. There were a number of places in the ARM generator where he'd put in an assert for logic that was valid for x86-64 and not for ARM, but he hadn't removed it.

What he was basically saying was that it was premature to worry about this kind of thing. It wasn't a finished project. It was a work in progress.

Rikki said he was just hoping Walter could do something about it that wasn't just putting a Band-Aid over the problem. Walter said there were a lot of Band-Aids in the ARM generator at the moment. Rikki clarified that he meant commenting it out. He could do that in a PR just to get the CI to keep progressing.

Walter said he understood Rikki wanted to get his Data Flow Analysis to work, and that he could comment it out if he wanted to. Rikki said that was a Band-Aid and he didn't want it to be.

Walter said the code was a soup. For example, he'd written an object file generator for the Raspberry Pi, which turned out to be ELF object code. For that, there was very little need to modify the object code generator.

Then he'd looked at the Mac stuff, where they had decided to completely reinvent the wheel on how to generate object files for the M-series processors. That broke a lot of the logic in the object file generator. He was trying to reverse engineer what they'd done because it was completely undocumented. As far as he could tell, the only documentation was in the Clang back end, which he couldn't look at because he never looked at other people's compilers. So the code was a giant mess.

He didn't really know if it was generating correct object files. He just kept trying different cases and looking at the output. If it seemed to match what GCC was doing, he'd keep going. That was the life of trying to get a new code generator working. You often had to work with undocumented things and didn't know how things would turn out. He didn't worry about it much. That was why we had a test suite. He could clean things up later.

He wasn't worried about branches left over from x86_64. They wouldn't be taken. His overriding goal was getting something that functioned.

Rikki said it was good at least that Walter was aware that it had come up. Walter said he wasn't surprised by it, but he didn't think the code generator could actually trigger it. It was logic that didn't exist in the ARM generator. The static analyzer wasn't going to be able to determine that, which was probably why it was tripping on it.

Rikki wasn't convinced. He'd had it running all the way through Phobos and DRuntime without any reports. Walter said Phobos wasn't full of logic that never happened. The ARM generator was full of it because it was a bastardized version of the x86_64 generator. He was basically taking a dog's leg and grafting it onto a giraffe. It was going to be an unholy thing for a while.

Rikki said he'd just comment it out in his PR to keep the CI going.

### The future of DIP 1000

Razvan wanted to know the status of DIP 1000 and what the plans were for its future. We'd said at one point that this was going to be a priority for this year, and we were now a few months from the end of the year.

Átila said this was on him. He thought he knew what we should do about it, but he needed to do the work to ensure it actually worked. He needed to try it out on the D projects at code.dlang.org to be confident. If it did, then he'd put a proposal forward. If it didn't, then it would be back to the drawing board.

Razvan said he would nag Átila about it once in a while, and Átila was okay with that.

I asked if we were planning to finalize this in the first edition. Átila hoped so. Walter had said it might work when we'd discussed it. He just needed to try it out in practice. The idea was to make `scope` explicit and never inferred.

Rikki said Átila had talked about the technical aspects of this, but how were we going to solve the sociological and psychological issues of DIP 1000? People had static analysis fatigue, and Rikki had no interest in using it ever again.

Átila understood that, which was why he wanted to make it explicit so that it would be opt-in. Otherwise, they'd be left alone to use regular GC D.

Rikki brought up a potential issue with that approach. Anyone wanting to take advantage of the stack-allocation optimization for closures would be forced to use `scope` on the closure, even if they didn't want to bother with DIP 1000. Átila said he'd have to think about that one.

Martin thought that was pretty important. It was one of the few occasions where he really wrote `scope` manually, when he was sure a delegate would never escape. Another was on some class instantiations when he knew the instance would only be used locally.

A third case would be variadic arguments where you could pass in an array. If you made that `scope` and an allocation happened, it was going to be on the stack. That had been added just a few years ago, but he thought Dennis had recently changed it so that it only happened with DIP 1000, because otherwise it would be unsafe.

Dennis said it only required DIP 1000 with inferred `scope`, as there were no checks that it was actually safe without it. Martin said that was exactly the kind of case he was worried about. If `scope` became fully explicit, then they would need to think carefully about the practical uses that still depended on inference.

That led into a much longer discussion about DFA, full-program analysis, inference, false positives, cycles in call graphs, and whether DIP 1000 could be improved without taking on much more compiler complexity.

Dennis said he was looking at it pragmatically, both as someone writing `scope`, `@safe`, and `@nogc` code and as someone triaging the DIP 1000 bug list. Pragmatically, the lack of DFA was far down at the bottom of the list of DIP 1000's problems. There were simpler things to solve higher up the list that didn't require adding thousands of lines of code for a new semantic pass. He didn't see the value. The trade-off between the added complexity and how much DIP 1000 would actually solve felt lopsided.

Walter said Dennis had done a great job fixing DIP 1000 bugs, and he appreciated it. He thought DIP 1000 had a lot of potential. He also understood that it was confusing people, and he wanted to address that. He understood Robert's point that users shouldn't be required to annotate everything. That was a proper and correct sentiment.

He would like to make DIP 1000 better on that front, but he didn't see a way to do it without DFA. So he thought DIP 1000 inference would have to be optional.

Dennis said that when he had a complex recursive-function case, he just annotated `scope` manually. He didn't need the compiler to do a complex analysis if he could solve it himself.

Walter said Dennis was an expert at this. When it failed for someone else, like when they had a function A that called function B that called function C that called A again, where were they supposed to put the annotations? That wasn't going to be easy for someone less familiar with the language.

Dennis suggested the error message should tell you the first place it failed, and even subsequent places. He'd just tested what it said in the recursive case, and it was going back and forth. But it did try to follow it and tell you where the scope inference failed. He posted this in the chat:

```d
onlineapp.d(19): Error: assigning scope variable `x` to non-scope parameter `x`
calling `f` is not allowed in a `@safe` function
     f(x);
       ^
onlineapp.d(10):        which is assigned to non-scope parameter `x`
void g()(string x)
                 ^
onlineapp.d(4):        which is assigned to non-scope parameter `x`
void f()(string x)
                 ^
onlineapp.d(10):        which is assigned to non-scope parameter `x`
void g()(string x)
                 ^
onlineapp.d(4):        which is assigned to non-scope parameter `x`
void f()(string x)
                 ^
onlineapp.d(10):        which is assigned to non-scope parameter `x`
void g()(string x)
                 ^
onlineapp.d(4):        which is assigned to non-scope parameter `x`
void f()(string x)
                 ^
onlineapp.d(10):        which is assigned to non-scope parameter `x`
void g()(string x)
                 ^
onlineapp.d(4):        which is assigned to non-scope parameter `x`
void f()(string x)
                 ^
onlineapp.d(10):        which is assigned to non-scope parameter `x`
void g()(string x)
```

Walter agreed it would be a big improvement if the error message wrote the correct declaration for you. Dennis said that sometimes it even suggested adding `scope` or `return scope`, so it was already making some suggestions. Walter said he'd noticed some nice improvements in other compilers suggesting things. It was like when he'd added the spellchecker to DMD. He was amazed by how often it got things right. It was a nice improvement he wished he'd implemented years earlier.

I said this sounded like a good spot to bring the discussion to a close.

### The Editions DIP

I said that Átila and I had been playing email ping-pong for the past few weeks revising the Editions DIP. We were almost there. I was going to make my final revisions and send it back to him to ensure I didn't botch anything. He hadn't gotten all the updates we'd discussed in the meeting we'd all had about it, and that was what we were doing now.

Because we were making some structural updates, I was going to email everyone when we were finished to make sure we were all still on the same page. Then it would just need the stamp of approval from the group and we were done with it. We were finally almost at the end.

__UPDATE__: Átila and I finished the final draft, there were no objections in the email thread, and [Walter approved it in October](https://forum.dlang.org/post/shncynvarhtqfsvzbxae@forum.dlang.org).

### Static array length inference (and a few random things)

I asked if anyone had anything else before we left, and Nicholas had a few things to bring up.

First, he wanted to make sure Martin was aware of [an issue he'd posted for LDC](https://github.com/ldc-developers/ldc/issues/4982). He believed we should be looking into the new LLVM raw byte type for compatibility with Clang. He thought it might be needed for C and C++ interop.

He also talked about an issue he'd had trying to compile LLVM 21, and another issue involving certificates for the release tools and signing. That was something we needed to look into and ensure our releases were signed properly. He was also happy that someone was working on ImportC issues for SAOC. There were all sorts of issues you ran into with intrinsics and whatnot, and hopefully some of those would be fixed.

Finally, he brought up static arrays with declared sizes and mismatched initializer lengths. He believed Andrei had rejected the original proposal to introduce length inference with `int[$]`, because it was already possible with a library solution. Nicholas didn't think that was a great answer. He said Dennis had been talking about using indices as a possible solution, and he asked if Walter or Dennis had any other comments on static arrays.

Steve said a recent issue was that you could initialize a static array with fewer elements than its declared size and the compiler wouldn't complain. With `int[4] foo = [1, 2, 3]`, for example, the compiler would just default-initialize the fourth element. Sometimes that was intentional, but a lot of times it was programmer error, like forgetting to add the 20th item because there were too many to count. The request was for some way to notify the programmer of the potential error, or at least a way to declare it explicitly when that behavior was intended. Obviously, the `staticArray` template in the library was one possibility, but that was the most recent issue that had come up.

As Nicholas understood it, the upshot of the earlier discussion was that we'd like to deprecate initializing a static array with a mismatched number of elements except in the case where you explicitly use indices in the initializer. That was trivial to do.

He also wouldn't mind having the `int[$]` solution in addition to that. He was never satisfied with the `staticArray` template. It would be great if we could come to an agreement on what to do, though it didn't need to happen straight away.

Martin said he'd just looked at the template. One of the not-so-nice things about it was that it took the value by `auto ref`, but returned a copy. For trivial types, that didn't matter much and would easily be optimized away. For non-POD types, it might not be so funny. If you disabled the `postBlit`, for example, you couldn't use that template as far as he could tell, unless CTFE did some magic with it.

He'd had the need a few times for something like `int[$]`. He thought a mismatched initializer length should definitely be an error except in the case where explicit indices are used. That made perfect sense.

I noted that, for the record, the community review for that `int[$]` DIP was in 2021 after Andrei had stepped down. There was no official word from Walter in that thread. Átila left a couple of comments. The DIP was never rejected. The author withdrew it:

> After the first round of Community Review, the DIP author felt that the general opinion of the DIP was negative, and that he needed to revise the DIP with a strong argument in favor of the proposed feature over std.array.staticArray. He was unable to formulate such an argument, so he asked that the DIP be withdrawn.

Walter proposed two possible syntaxes in the chat:

```d
int[6] a = [4,2,8,...];
int[6] a = [4,2,8,3...];
```

The first one initializes the array with `4, 2, 8`, then the remaining elements are default-initialized. In the second one, the remaining elements are initialized with `3`. If someone wanted to write a DIP for that, he saw no reason why it shouldn't be done. Then we could deprecate mismatched initializer length without losing that syntax.

Rikki thought the expression should go after the `...`, otherwise it would be parsed as a float or something nasty. Walter thought it was processed eagerly, so it should be fine.

Nicholas understood why Walter wanted two syntaxes for that, but he suggested using indices for one and not the other. The problem was that `3,...` and `3...` looked very similar. That was going to be typo-prone and difficult to read. I said that without my glasses, I was struggling to see the difference. I hadn't noticed the comma after the `8`.

Walter said that if someone wanted to write a DIP for it, we could debate syntax options then. He didn't think we'd solve it there.

Martin said he'd never had a need for anything like that. He asked whether Walter had ever needed to explicitly initialize the first few elements of an array, then all the rest with the default or the last element. Walter said he had seen it. People sometimes initialized arrays with a couple thousand elements and only populated the beginning. It might not be great practice, but there was a reason C was done that way.

Dennis had sent him an error case where somebody converted a `char` array from C, and C default-initialized to `0`, while D default-initialized to `FF`, producing a different result. That example made it clear to him that we needed a way to specify what the remaining default initializer was.

There was some more discussion about current behavior versus desired behavior, deprecating the current behavior, and the potential new syntax. In the end, Walter repeated that if anyone wanted to write a DIP, they should go ahead and we could hash out the syntax. He thought that would be very productive.

## Conclusion

Our next meeting was a quarterly that took place on October 3rd. Our next monthly meeting was on the 10th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
