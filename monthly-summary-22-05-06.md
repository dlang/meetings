[Forum Post](https://forum.dlang.org/post/uhgndrcnekedjqtarnwl@forum.dlang.org)

The monthly meeting for May 2022, took place on May 6th at 14:00 UTC. The meeting lasted a little under an hour. The following people attended:

* Walter Bright
* Iain Buclaw
* Ali Çehreli
* Max Haughton
* Martin Kinkelin
* Mathias Lang
* Razvan Nitu
* Mike Parker

## Iain
Iain gave us an update on the release of GDC 12.1. He talked about some of the work he put into getting it ready. Now that he is also involved in releasing DMD, he was extremely busy prepping for both. In the future, he's going to streamline his process so that it's less stressful.

## Max
In the April meeting, Atila noted that he wanted to enable `-preview=nosharedaccess` by default, but [was blocked because it didn't work with syncronized](https://issues.dlang.org/show_bug.cgi?id=22626). Max reported that he had looked into it. He said he tried to have the compiler cast away `shared` in `synchronized` blocks, but this was defeated by the frontend optimizer stripping the cast away.

Razvan [has subsequently submitted a PR](https://github.com/dlang/dmd/pull/14143) to fix the issue.

Max also wanted to make sure Walter was aware of a SIMD issue in DMD. [Walter had already submitted a PR for it](https://github.com/dlang/dmd/pull/14081) the day before the meeting. This prompted a discussion about what the problem was, Iain's attempt to find it, and a tangential discussion about FreeBSD tests.

## Mathias
Mathias let us know he was about to officially start his new job with Symmetry. He also talked a bit about this [Argument Dependent Attributes DIP](https://github.com/dlang/DIPs/pull/198) in the PR queue (a.k.a. Draft Review). He wants to have a fully working implementation that he can test out before moving forward to the first Community Review round. He currently has a partial implementation and will let me know when it's ready.

## Martin
Martin noted that cooperation on D 2.100.0 was great, and acknowledged that Iain had the most stressful time of it. He said the release was in good shape, as it had been tested with Symmetry's codebase and all was well. The beta of LDC 1.30 was released a little over a week after the meeting.

He reported he had been trying out the PGO build feature on DMD. He found a huge effect on binary size. He was planning also to do some performance comparisons.

## Me
I noted that I plan to enter the foundation's YouTube channel into the YouTube Partner Program so we can use it to raise money. Before that, I want to increase the variety of content we put out and the frequency at which we publish. I told everyone that to that end I'd like to start putting out interviews, and they agreed to participate.

I have since recorded and published an interview with [Razvan and Dennis in our 'D Community Q & A' playlist](https://youtu.be/nvo7wzjVDQc). I'll publish more in that format, and also in a longer form 'D Community Discussions' format, with Walter, Atila, Iain, Martin, and others in the coming months.

## Walter
Walter said his next issue was the C preprocessor in ImportC. He had some PRs that hadn't been merged and it was slowing him down. Iain said that he had not merged those yet because he thought it would be better for them to go into the next release. Razvan noted that he was monitoring Walter's PRs, but most of the time would like a second or even third opinion on Walter's code (and he did merge one of the preprocessor PRs while this discussion was in progress).

In the ensuing discussion (which took up a significant chunk of the meeting), Walter said he has no special attachment to the code he's writing for preprocessor support. Iain, Martin, and Max want something more sophisticated (e.g., something that detects preprocessor support rather than calling out to specific preprocessors), and he's all for it. But right now, he wants something that just works so he can make forward progress. If something better comes along, he's happy for it to replace his implementation.

This segued into a discussion about importing header files. Martin brought up the current approach of importing C symbols into local modules that are generated per compiler invocation. This can be a problem with a large codebase like Symmetry's that has a large number of foreign language dependencies, and the same headers can be imported multiple times all over the place: the result would be huge object files, each having its own definition of the same structs, their `TypeInfo`s, their `.init` symbols, and so on.

Iain agreed, noting that if you, for example, import `tgmath.h` and `math.h`, which `#include` each other, then calling e.g., `sin` will result in a conflict. He suggested the way to handle this is that ImportC symbols, rather than going into a module space, should go into a global ImportC module. Each imported C file or header file is then appended to that module. This would also prevent the potential problem of non-identifier characters (illegal in D module names; Iain noted that dashes are a big one) appearing in C header file names.

Martin said that one problem with that approach is that a D module would have access to symbols it didn't import. He suggested the ideal solution would be for each header to have its own module that doesn't change from one invocation of the compiler to the next.

Walter closed by letting us know he was heading to Poland for a three-city speaking tour as part of the Code Europe Conference.

## Ali
Ali said he was scheduled to speak to a class at Beykoz University, and that he was planning to use the opportunity to promote D.

At work, the project he was using D for has slowed down and will not be coming back. He said they might use D for other projects in the future.


## The next meeting
Our next monthly meeting is scheduled for Friday, June 10, at 14:00 UTC. The vision document is the main item on the agenda, and I expect it to take up most of the oxygen. We'll review the current draft and decide what to cut, what to keep, what to add, etc. My expectation after that meeting is that it will be on me to make the revisions, after which I'll email the members for final feedback, then make any final revisions, then publish. It should be ready before DConf, the end of June at the earliest.
