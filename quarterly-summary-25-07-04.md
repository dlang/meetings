The D Language Foundation’s quarterly meeting for July 2025 took place on Friday the 4th at 15:00 UTC. It lasted a little under 30 minutes.

Our quarterly meetings are where representatives from businesses big and small can bring us their most pressing D issues, status reports on their use of D, and so on.

## The Attendees

The following people attended the meeting:

* Walter Bright (DLF)
* Dennis Korpel (DLF/SARC)
* Mario Kröplin (Funkwerk)
* Razvan Nitu (DLF)
* Mike Parker (DLF)
* Carsten Rasmussen (Decard)
* Bastiaan Veelo (SARC)

## The Summary

### Carsten

Carsten said things were going well at Decard. It was very exciting on the business side because they were going into logistics and security. The problems they were having to solve had nothing to do with the language, just normal development stuff. They were comfortable with what they had in the system and were happy.

### Mario

Mario said he’d emailed Mathis Beer about the meeting three days ago, but suspected he'd forgotten about it. Mathis was programming in D, but Mario was currently the Java programmer, so he had nothing to bring to us this time.

### Walter

Walter said he’d previously complained about the lack of D articles on Hacker News, but one had shown up recently and turned out to be a significant success for us. He wanted more of that kind of visibility.

He then went into a little rant about people who said "I used D for years, but it had this one flaw, so I dropped it," and then presented that flaw as the reason D couldn't succeed. The example he’d seen lately was, "I can’t use D because the exception handler uses the garbage collector."

Carsten said they shouldn't use exceptions if that was a concern. Walter said using exceptions with the GC wasn’t going to break your program. He got frustrated with people turning a single technical detail into a deal-breaker.

I brought up a comment I’d seen on Dennis’s DConf talk from last year: "The single biggest mistake Bright made when designing D was the GC. That is the sole reason D hasn't replaced C++ by now. It's the biggest crutch to D adoption. Really, who wants to deal with this kind of nonsense?"

Carsten said, "No, no, no, no, no, no, no."

Walter said if it wasn't the GC, it would be something else. Throughout his career, he'd had people telling him, "If only your compiler had this feature, our company would switch." He'd add that feature and they'd say, "That's nice, but what we really need is this other feature." Then you ended up on a merry-go-round. They had no intention of ever using it.

Carsten added that the GC was one of the reasons he had switched from C++ to D. He'd gotten tired of hunting wild pointers and was too old to do that anymore. Walter said that the GC had ended up being a surprising asset for CTFE.

I asked Walter if that was all. He said he was done ranting for the moment, but was sure he'd think of something else to rant about.

### Bastiaan

Bastiaan had two work issues he wanted to bring up. He'd spoken a little bit about them with Dennis.

__alloca and 64-bit DMD__

Bastiaan reminded us he’d started using `alloca` as a solution to one of the thread contention problems he'd had. It worked fine with DMD in 32-bit and fine with LDC in 64-bit, but when compiling for 64-bit with DMD, he got an error saying `alloca` didn't work with exception handling. He was wondering why and whether it was really working on 32-bit and 64-bit LDC.

Walter recalled giving up on trying to get `alloca` working on Windows 64-bit. It was ugly to implement, though he couldn't remember what the actual problem had been. He’d never been able to figure out Win64 exception handling. DMD’s exception handling on Windows differed from MSVC’s. He suggested that Bastiaan's problem was probably related to that.

Dennis posted [a link to the relevant code in DMD's `eh.d`](https://github.com/dlang/dmd/blob/9f573c494acc38855027462bde162fabea9cf33f/compiler/src/dmd/backend/eh.d#L50) along with a comment that he thought Walter had probably written:

```
// BUG: alloca() changes the stack size, which is not reflected
// in the fixed eh tables.
```

Walter said we were kind of stuck with that. He suggested using a RAII malloc/free scheme, which would accomplish the same thing as `alloca`. Bastiaan pointed out that `malloc` required the global lock. One of the reasons they were using `alloca` was to avoid that.

As an alternative, Walter suggested that if they knew the maximum size of the allocation, they could use a struct of that size. That would then be the allocated storage. Bastiaan didn't think they could get rid of `alloca`. It wasn’t a deal-breaker because they could just use LDC, but it was a minor inconvenience that they couldn’t fall back on DMD.

Bastiaan recalled an initiative for nonallocating exceptions and wondered what the story was. Walter said there was a compiler switch that enabled them and it did work, but there were some issues with it he never fully understood. I said Razvan could probably explain it, and that I recalled it had to do with stack trace generation. Walter said that sounded like that was probably the issue.

__Link errors in LDC__

Bastiaan's second issue was an LDC linking problem after upgrading the front end. He didn’t expect a full answer on the call and would need to work on a reduction. He asked if Dennis had any thoughts on it.

Dennis thought it was a mix of compiler flags and preview switches causing a difference in symbol generation. He hadn't yet reproduced it.

Walter suggested it could also be a DRuntime/Phobos mismatch and stressed that DRuntime, Phobos, and user code all needed to be built with the same compiler version. Bastiaan believed they were doing that. He said the error mentioned "conflicting weak external definition". He'd only seen it a few times and had forgotten the context.

__Floating point issue__

Bastiaan said that aside from those issues, things were generally fine. The only other issue he could think of was that 64-bit floating point answers sometimes deviated from 32-bit. One reason was that he'd been relying on undefined behavior for some code using `bitmanip`. He was glad they'd disovered that. It was in order now.

The other was because 32-bit DMD used 80-bit floating point in intermediate results. This was code using floats, not doubles. LDC, even in 64-bit, was using 32-bit intermediates. He figured that was introducing a lot of rounding errors in the intermediate results.

Walter explained that if the target machine supported XMM floating point registers, the compiler would use them, otherwise it would use the 80-bit floating point coprocessor, which did everything in 80 bits. He said 32-bit LDC on the platform Bastiaan was working on wasn't using the XMM registers. He suggested looking at the LDC switches to see if there was one to enable it. That should clear up the problem. Bastiaan said that was a good point and he would look into it.

### Dennis and Razvan

After Bastiaan finished, I asked Dennis if he had anything work-related to add beyond what had come up in Bastiaan’s issues. Dennis did not. Neither he nor Razvan had anything DLF-related.

### Me

I said I would be be announcing the Symmetry Autumn of Code in the next few days. We had funding from Symmetry for three slots. I noted Emmanuel Nyarko had already told me he planned to apply. I asked all present to let me know if they had any open source libraries they were using that needed some work. That sort of thing could make potential project ideas.

I asked Carsten if he could send me an updated headcount of Decard's DConf attendees within the next week. We needed to send the caterers an approximation of our final numbers. Then I asked if anyone had anything else to add. Dennis asked if there was going to be a monthly meeting the following week. There was.

### AI interlude

As we were about to close, Walter told us he’d tried very hard to get an AI to draw a D-man. He'd failed miserably no matter how he crafted the prompt. He kept getting something that looked like Superman with a D on his chest.

Carsten offered to ask someone at his company who was "super good at that" to see if they could come up with a prompt that worked. I mentioned that some of the services let you upload a reference image. Walter said he’d tried that, too. He'd tried everything. All these people were saying AI was going to take over everything, and he hadn't been successful with getting it to do what he wanted.

I said it had been very useful for me with research and translation. Carsten agreed. He thought it was great for extracting information and translation. When it came to writing code you wanted to maintain, he thought it was just crap on crap, but maybe he was too old for that.

I noted that I'd gotten GPT to generate a Lua script as a plugin for Pandoc, because I knew nothing about Pandoc plugins. It had mistakes that I'd had to correct, but it got me further than I would have gotten on my own in a shorter amount of time. Carsten thought it was great for that sort of thing, scripts for specific tasks. But for something that you wanted to develop and maintain, he didn't think we'd be losing our jobs any time soon.

Walter said he’d been invited to give a short presentation about D that highlighted concurrent programming. He didn't know much of anything about concurrency, so he'd asked Grok for a sample program and it worked. Then he thought he should send the prompt to Bruce, who was an expert on concurrent programming. Bruce sent back a marvelously written piece of concurrent code that did the same thing. Walter scrapped the one from the AI and went with Bruce's. There was nothing like talking to an expert to get something we could be proud of. That had been fun.

Mario joked that it was good Mathis wasn’t there, otherwise the meeting would run for another hour. Mathis did nothing without AI. He put the code in a loop with the compiler and the AI was fixing his bugs. He was very deep into AI. Carsten said you could probably use it as a tool if you knew what you were doing. Mario agreed and mentioned a saying that it made good people better and bad people worse.

Walter said when he asked AI to write a piece of code, he knew how to specify what he wanted. It was really helpful in getting the results he wanted on that front. He was just annoyed that he couldn't get it to generate a D-man.

He said we had a shortage of D-man cartoons. He had a directory with a collection of all the ones he'd seen. For him to draw one was a long, complicated process because he had no talent for it. He would have to spend a lot of time with the image editor to make it look presentable. Being able to automate it would be wonderful. Something silly like D-man jumping rope or something. If anyone could figure out how to do it, he asked that they send him the prompt.

### Conclusion

Our July monthly meeting took place the following Friday, July 11th. Our next quarterly happened on Friday, October 3rd, 2025.

If you are running or working for a business using D, large or small, and would like to join our quarterly meetings periodically or regularly to share your problems or experiences, please let me know.
