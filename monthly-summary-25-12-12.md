The D Language Foundation's December 2025 monthly meeting took place on Friday the 13th and lasted about an hour and thirty-five minutes.

Unfortunately, I had the wrong configuration selected in my screen recording software. My system audio was muted and I only have the recording for my microphone. As such, I can't put together the kind of detailed summary I like to write up. The best I can do is provide some highlights reconstructed from the chat log, my audio, and memory.

## The Attendees

The following people attended:

* Walter Bright
* Iain Buclaw
* Rikki Cattermole
* Jonathan M. Davis
* Dennis Korpel
* Mathias Lang
* Mike Parker
* Razvan Nitu
* Átila Neves
* Robert Schadek
* Nicholas Wilson

## The Summary


### DConf '26

Before we got started on the agenda items, I gave everyone a quick update on DConf '26. Since our first time at CodeNode in 2022, which was during a period when they were ramping back up post-pandemic, they had offered us the same significant discount for any dates in August every year, and a smaller discount for September. This time, they had extended the August discount rate into the first week of September. I had already confirmed with Symmetry that they were okay with that week, so CodeNode was holding the dates for us. The next step was for Symmetry to sign the contract with Brightspace, our event planners, so they could in turn sign the venue contract.

(__UPDATE__: In case you missed the subsequent announcement, [DConf '26 is happening September 2 - 4](https://dconf.org/2026/index.html) at CodeNode in London.)

### Type system analysis algorithms

The first agenda item was from Rikki regarding his idea for Type System Analysis Algorithms. [He posted in the forums about it](https://forum.dlang.org/post/waerqsesrohtokjctnyj@forum.dlang.org) a couple of weeks before the meeting, so you can read about the the concept there.

This took up over 30 minutes of the meeting. Rikki posted the following links in the chat log:

__Fast and sound static analysis__: https://www.absint.com/astree/index.htm
__Principles of Abstract Interpretation__: https://mitpress.mit.edu/9780262044905/principles-of-abstract-interpretation/

I recall there was a lot of skepticism about this. At one point, we were talking about the refactoring going on aimed at preparing to enable running the compiler as a daemon. I warned that we should keep that in mind when discussing major structural changes to the compiler source. Beyond that, I have no details of the discussion. I can say the proposal was neither accepted nor rejected.

I ultimately asked Rikki if we could put a pin in it and move on to the other agenda items, and he said yes.

### Status of editions

Walter wanted to know about the status of the editions implementation. The conversatoin veered into what the first edition was supposed to encompass. I reminded everyone we had agreed some time ago that the first edition would be about fixing incomplete and broken features.

We decided to have a separate meeting to finalize the details about the first edition, e.g., which incomplete or broken features we wanted to include. Given the holiday season in December an a quarterly meeting in January, we opted to hold the editions meeting in late January. I'll summarize that in a separate writeup.

### `with` expressions

Nicholas had [a PR to implement with expressions](https://github.com/dlang/dmd/pull/22196), e.g., `with(auto x = whatever())`, a feature that had been requested in the forums. The only issue with it was [a BuildKite failure](https://github.com/dlang/dmd/pull/22196#issuecomment-3627316856) with one of Átila's projects where he used `with(immutable S())`.

I recall someone mentioned that `if(immutable S())` failed to parse, as did `for`. The discussion focused on which of three possible options to pursue:

1. merge the PR as is, with the breakage;
2. change the PR so that the breakage doesn't occur;
3. go with Option 2 and also fix the parsing errors in `if` and `for`.

As a first step, we opted for Option 2. You'll find this [in the changelog for the next DMD release](https://dlang.org/changelog/pending.html#dmd.with-assign):

> For backwards compatibility, `with` still also accepts a qualified expression without assignment to a variable as in `with(immutable expression())`.

We also decided someone should look at the parser and see what sort of changes are required to get `if` and `for` to behave the same way. If it's not too gnarly, we can make `if` and `for` conform with `with`. Otherwise, we might modify `with` in a future edition to conform with `if` and `for`.

### The next compiler release

Walter wanted to know when the next DMD release was coming. This led to a discussion about the release process in general. As part of that, we touched on code signing certificates. Thanks to Luna, we should be okay on Mac releases. The problem was Windows.

We haven't had a certificate for our Windows releases for quite a while. Cost was part of the problem, but more importantly was the requirment for a dongle. There was a conflict there with our desire to automate the release process. It was doable, but a bit of a headache.

We had discussed this before and decided we needed to get the new release process in place first. Then we could look at options we could integrate and their costs.

(__UPDATE__: [DMD 2.112.0 was released on January 7, 2026](https://dlang.org/changelog/2.112.0.html).)

### An intern

Before we left, I let everyone know that Fei, one of Ben Jones's students, was interested in working with us as an unpaid intern for a few months. He had participated in GSoC 2025 working on Steve's jsoniopipe. He was interested in whatever work we could throw his way.

(__UPDATE__: There was some red tape to sort out first, but Fei officially started working with us a few weeks later. He has since joined a couple of monthly meetings, and Nicholas Wilson has been keeping him occupied.)

### Conclusion

Rikki had one other agenda item we discussed briefly, a suggestion for a GSoC project to implement cent/ucent integers, but I have no context for, or memory of, that conversation.

The first Friday of January is usually when we have a quarterly meeting, but given that this year the first Friday was on January 2nd, I suggested we push it back to the 9th, and our January monthly in turn to the 15th. Then we could have our editions meeting on the 23rd. Everyone agreed.

If you have anything you'd like to bring to us in a monthly meeting, please let me know.
