The D Language Foundation's October 2025 monthly meeting took place on Friday the 10th and lasted about fifty minutes.

## The Attendees

The following people attended:

* Walter Bright
* Ali Çehreli
* Jonathan M. Davis
* Timon Gehr
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Nicholas Wilson

## The Summary

### The Editions DIP

I opened by noting that the only agenda item was the editions DIP. I asked whether anyone had any problems with the current draft. I added that Rikki would not be attending, but had mentioned in an email that the DIP should clarify that the `-E` switch added nothing to the import path and only affected the target edition for the specified path.

Átila said someone had emailed him feedback, but he hadn't read it yet.

Walter said one of his objections to adding a new import path was that import behavior was already undocumented and rather Byzantine, so making it more complicated would be a mistake. Jonathan pointed out that the intent had always been that it should not affect import paths. I said that the DIP just needed to say that more clearly. Walter said he had misunderstood that point initially, and I said that all the `-E` switch would do was apply the specified target edition to the given path.

Walter then mentioned that the `-E` switch was partially implemented. He had implemented the user interface for it, but it still needed to look at the gathered data and apply it to modules as appropriate. He didn't know why that hadn't been implemented yet. He said he'd really like to get the editions feature in.

I said that if there were no other problems with the draft, then the remaining question was whether the DIP should specify the format of edition identifiers. I asked if we should explicitly say editions must be identified by the year, or whether we should leave that open.

Átila said using the year was probably the easiest approach. Walter agreed, saying he liked the year because it prevented the language from accumulating too many editions. Once you started allowing too many editions, things would become unmanageable. He liked the idea of limiting it to one per year, somewhat in the spirit of C++ spacing out its major revisions.

I noted that we could always do something like 2025A, 2025B, or 2025-2 if we ever needed more than one. Jonathan pointed out that as the DIP was currently written, the edition identifier could only be a number. In practice, that made it effectively an incrementing numeric sequence, and while we could technically pick arbitrary numbers, that would be silly. To sum up, I said the DIP should clarify the `-E` behavior and specify that the edition ID should be the release year. Timon and Dennis both agreed.

Walter said this matched the spirit of how versions already worked. Versions could have either an identifier or a number, but almost nobody used the number. He saw editions as being like versions with a number, except with more constrained semantics. Restricting editions to something simple was an advantage, because allowing random identifiers or something more elaborate would inevitably create a mess.

He said editions shouldn't be fine-grained. They should be blunt. The best way forward would be to collect all the features for a particular year and treat that as the edition for that year. We didn't want too many dialects of the language floating around, since every new dialect increased complexity.

Nicholas asked whether the DIP specified what the `__EDITION__` token was supposed to evaluate to. He assumed it should be the integer value of the edition. Átila said that was what made sense to him. As far as Nicholas could tell, the DIP mentioned introducing the special keyword and said it was evaluated at the call site, but didn't say what value it would actually take. I said it would evaluate to the target edition for the current module. Nicholas said that meant it was an integer. I said we would specify that.

Jonathan added that the DIP already said the edition identifier was a decimal integer, but it didn't explicitly tie that to `__EDITION__` or say that the number had to be a year. Nicholas replied that as long as it was specified, that was fine.

Walter asked what the purpose of `__EDITION__` was supposed to be. Was it so you could do conditional compilation on it? Jonathan said it might also be used for printing out the edition with a pragma. Walter objected to that. He said that if pragma support was needed, we could simply have `pragma(edition, ...)`, with `edition` as a context-sensitive keyword there.

His main objection was that once edition became a testable value inside code, it would invite a rat’s nest of conditionals. In his view, the edition should apply to the module as a whole, not to individual pieces of code. If people could test the edition and branch on it, they'd wind up with the equivalent of C preprocessor hell. A module should not effectively contain multiple editions.

Átila didn't think anyone was proposing that a single module should be made up of multiple editions. Walter said that was what it looked like to him.

I said that this had come up in the editions DIP ideas thread. Timon had asked how a library could expose different interfaces to importers using different editions, especially if editions changed interface guarantees or something user-visible in DRuntime types. Walter said the answer was simple: use versions for that, not editions. I said that Átila had [proposed using version for this in reply to Timon](https://forum.dlang.org/post/itssadkelgljdwvyixly@forum.dlang.org), and then [Dukc had suggested a special `__EDITION__` token](https://forum.dlang.org/post/v3n4u2$127r$1@digitalmars.com) because version blocks would be controlled by the library’s edition, not the client’s edition.

After Walter read the posts, he said it was anticipating a future need without spelling out a concrete case. We could always add `__EDITION__` later if we truly needed it, but if we added it now and it produced version hell, we wouldn't be able to undo it. Making the language infinitely flexible on this stuff would just lead to a mess. People should decide what edition their module used, and if they truly needed multiple interfaces, they could have multiple source files. There was no shortage of source files you could have.

Nicholas noted that C++ had a preprocessor token for its standard version, though he added that this wasn't necessarily something D should copy merely because C++ had it. Walter said he didn't want code full of edition-based conditionals. Forcing people to organize their code more cleanly was a good thing.

Nicholas said the only use case he could really see for `__EDITION__` was diagnostics, such as printing a clearer error message if someone messed up command-line ordering and applied a version or edition in the wrong place. Jonathan said that in those cases, users would just get a compilation error anyway. Nicholas thought we could improve the error message. Walter reiterated that he really did not want to see source code littered with “if this edition, do this; if that edition, do that.”

Jonathan said the only case where he could imagine the feature being theoretically useful was if code had to change its behavior based on the edition used by an imported library. But ideally that shouldn't happen. If you found yourself in that situation, it would already be a serious rat’s nest, because the whole point of editions was that a library should do its thing and continue to work when it updated the edition it was using. We would need a situation where that wasn't the case and you would have to care about which edition the imported module was using. That would be a bad situation if we got there.

Walter asked, rhetorically, what you would have to do if you wanted to use, for example, a library using an older edition. You would have to have an adapter module. Jonathan said the way you'd deal with this was with your build system. You knew a library had updated what they were doing, so you were going to update what you were doing because of that, and you'd just update what you were doing and update your dub stuff. It shouldn't be necessary to have version-management stuff in your actual code. In general, that was the purview of the build system.

If a library changed from version 1 to version 2, you didn't normally just magically handle that in your code. You changed your build dependency and now you needed version 2. He would expect this stuff to fall into the same camp. It was the only kind of case he could think of where someone might potentially need to be thinking about "if this version, do this, else do that" for editions.

Walter said that was a "let's cross that bridge when we come to it" thing. If we ever found ourselves wedged into a situation where editions didn't work well enough, we could always make them more flexible later. But if we started with maximal flexibility, we could never take it back.

Timon said it ultimately came down to this: if you called a function written against edition A from a module written against edition B, which language rules did you use to type-check the function call? Átila broke the subsequent 30 seconds of silence by saying he didn't know. Jonathan said the prime example was `scope`. Whatever we did with it could completely change what was going on with it. Átila agreed.

Walter said that editions were meant to remove old features and enable new ones, so if there was an incompatibility between your code and the edition used in a library...

I noted that the DIP specified source compatibility only, meaning all modules still had to be compiled with the same compiler version. Jonathan clarified that the real issue was not compiler version, but semantic changes across editions when newer code imported older libraries. He said pure additions or removals weren't a problem. The real problem was when an existing feature changed meaning.

Walter gave the example of removing `lazy`. A library written under an older edition could still publish a lazy interface, and if code written under a newer edition no longer allowed `lazy`, then that code simply could not call the function directly. In that case, users would have to write an adapter, which he didn't consider unreasonable. He said there should still be a common root of D accessible to all editions. As long as there was a way to call a function in any edition, we'd be fine because you could write an adapter function then.

Jonathan said that this just reinforced the need to choose edition changes carefully. If a particular kind of change would require edition algebra or branching logic, then we might decide not to make that change at all. He expected everyone present would agree that the ideal outcome was to avoid ever creating a situation where code had to say "if this edition, do this, else do that," because that would be ugly.

Walter proposed that we simply wait and see, and add `__EDITION__` later if it turned out to be necessary. Timon wasn't fully sold on `__EDITION__` as the right solution anyway. It was simply something he anticipated might be needed. He said he was fine moving ahead without it for now. Jonathan agreed because we didn't have a concrete use case. We might be able to come up with a better way to do it if and when we actually hit a use case, so it was better just to wait.

I summarized the action items so far: clarify `-E`, specify that the edition ID should be the release year, and remove `__EDITION__` from the DIP.

Walter suggested perhaps adding a comment section explaining why `__EDITION__` was not included and noting that it could be reconsidered later, since the question was likely to come up again. Jonathan said he really didn't want to think about the problems we'd face if code had to start asking which edition something used.

I asked if we were going to run into this kind of issue immediately if one of the first changes under editions was making `@safe` the default. Jonathan said no, because in that case you already had `@safe`, `@trusted`, and `@system` anyway. That might change what would compile on your end, but functionally there was no difference. You didn't functionally care whether a function was `@safe` or `@system`. It just affected whether you needed `@trusted` on your end or not.

Nicholas added that if we decided later that we did need `__EDITION__`, that wouldn't be blocked by old editions, since support would effectively be tied to compiler versions anyway. Jonathan noted that we'd already discussed the possibility of adding new non-breaking features to older editions. If we did end up adding `__EDITION__`, we could retroactively add it to the old editions on that basis.

I asked if we needed to answer the compiler-complexity question now or if it was something else that could wait. Did we need to specify whether the compiler would support infinite editions or not? Átila thought we had agreed on three. Jonathan said we'd discussed three, but he didn't think we'd officially decided one way or the other. Walter said that at some point we might stop supporting older editions.

I asked what that would mean for Edition Zero. If in some future year we dropped support for everything before, say, 2027, would Edition Zero then refer to 2026, or would it always refer to the original pre-Editions D?

Nicholas asked what exactly I meant by Edition Zero. I explained it was intended as the way to opt out of the default, which was always the latest edition supported by a given compiler release. You'd opt out with `-Edition=0` to compile pre-Editions code. Jonathan said that if we stopped supporting that old baseline, then effectively we would simply stop supporting Edition Zero. The only place this should really matter in practice would be old code sitting around in the package registry for years without being updated. At some point, if an old edition stopped being supported, that code would break unless someone updated it. Even if we chose to keep Edition Zero forever, we'd still face the same problem later with some specific numbered edition that was eventually dropped after years of neglect.

Walter said he didn't think this would be too big a problem in practice. He noted that even in C, if he dug up code he wrote twenty years ago and tried to compile it, he got all kinds of compiler errors. So even languages with a reputation for backward compatibility weren't really pain-free in practice after enough time had passed.

I asked if we needed to specify any of this in the DIP or if we could leave it alone for now. Nicholas said this was clearly a down-the-road problem and there was no reason to specify a solution before we actually needed one. Átila agreed and said it was basically an implementation detail. Jonathan added that we'd be better informed after actually dealing with editions for a few years and seeing what worked and what didn't.

Walter said it sounded like we'd decided things. I said I would make the changes that weekend and forward the DIP to Walter for his stamp.

Razvan wanted to play devil’s advocate. He said that listening to the discussion, editions sounded to him like a way of sweeping dirt under the rug. D already had a mechanism for deprecating features, eventually turning them into errors, and users who wanted to avoid deprecations could stick with an older compiler. Now we were moving to editions, where users would select an edition and avoid deprecation noise that way. If we eventually removed a feature anyway, someone’s code would still break. Jonathan replied that for some things there was no technical reason we couldn't continue to use deprecations.

Razvan said it still felt as though we were adding compiler complexity mainly so people wouldn't have to see deprecations. People would now have to deal with understanding editions, selecting them, and figuring out what was in which edition. He didn't see a major benefit beyond that.

Átila responded that some things simply couldn't be done cleanly with deprecations and cited `@safe` by default as an example. Razvan said that sounded more like a major release.

Jonathan said that, in principle, dub would be updated so that when you generated a new project, it would automatically put the edition flag in there. Old projects wouldn't get the flag until someone explicitly updated them.

Walter said the issue with deprecations was that there was a giant tangle of them.

Razvan said something like `@safe` by default was a really major change, and we'd still have to support code where it wasn't the default. In his view, it was easier just to have a major compiler release that made `@safe` the default. It was a big shift. If you had a code base that wasn't `@safe` by default, you'd have to architect it so that it was. You might as well use different compiler versions.

Timon felt like that had been tried with the D1/D2 split. The question was, did we really want to repeat that?

I noted that right now, we expected everybody to eventually update to the latest compiler release. If you didn't do that, if you froze at an older compiler, you didn't get the bug fixes. Weka, for example, had at times been frozen on a specific compiler release because the latest broke their code. Editions gave you a path to upgrade to the latest compiler because you'd freeze on the edition rather than the compiler version. You could upgrade the compiler and get the benefit of bug fixes without language changes breaking your code.

Walter said we'd lost a lot of users in the D1 to D2 transition because we tried to force them to change to D2. That simply did not work, especially with large code bases. Jonathan said editions would allow that kind of transition to happen in pieces. If one compiler version had `@system` by default and the next had `@safe` by default, then without editions every piece of code and every dependency would have to be updated in one step. With editions, a project could update piece by piece, and libraries on dub could transition at their own pace, while old and new code continued to coexist.

I added that this even created a new use for the deprecation switch. When we eventually decided to remove an old edition, we could deprecate it first for some period instead of simply ripping it out.

Razvan said he got it.

I said it looked like we had tied it all off and asked if there was anything else anyone wanted to bring up.

__UPDATE__: I updated the DIP as discussed and [it was eventually accepted](https://github.com/dlang/DIPs/blob/master/DIPs/accepted/DIP1052.md).

### Other Stuff

Nicholas said CircleCI had released a change related to how it checked out git repositories. He had already merged updates to DMD’s CI to enable it, and the change had passed there. The plan was to roll it out to the rest of the D repositories and see whether anything broke. He believed the CircleCI change would become the default around November 3, so he wanted to give everyone a heads-up in advance. If anyone had other projects using CircleCI, they might want to check them as well.

Razvan brought up the work being done by a SAOC contributor to separate semantic logic from AST nodes. The work was mostly done, with only a couple of tasks left, and they were approaching the point where they could move on to eliminating duplication in `ASTBase`. One remaining issue was more like a philosophical question than a technical one.

There was still a `Scope` definition in `dscope.d`. The parser itself didn't really work with `Scope`s at all. They were created during semantic analysis. So the question was whether `Scope` belonged purely to semantic analysis, or whether the definition should be thought of more like a building block for an AST node.

The problem was that `DSymbol` had a `scope` field, which meant it had to import `dscope.d`, and that file contained search methods that relied on semantic analysis. So there was a split. If `Scope` was treated as semantic-analysis information, then they'd need to find a way to get rid of the `scope` field on symbols. If it was treated as some sort of building block for the AST nodes, then they'd need to get rid of anything that relied on semantic analysis in `dscope.d`.

Mathias said that if the problem was that methods on the struct were pulling in dependencies on semantic analysis, then the data structure could be separated from the methods and the methods could be implemented externally. Razvan asked if he meant that the struct itself could be seen as an AST node building block, and bundled with the AST nodes, while the functions that depended on semantic analysis were moved out of the definition. Mathias said yes, more or less. He didn't know if it was the most correct way forward, but it was the simplest he could think of.

Razvan said they could also use a hash table for symbols and eliminate the `scope` field on symbols entirely. Walter said that would be rather slow. He suggested instead keeping a `scope` reference there and making it effectively a dummy when it wasn't needed. If a symbol was only being parsed, the field could simply be `null` or otherwise unused. Razvan noted that the parser didn't depend on `Scope` at all. Walter said that didn't mean `scope` couldn't still be there in the definition of the symbol. It could just be unused until semantic analysis needed it.

Razvan said that what he was hearing was that they could move the functions that relied on semantic analysis out of the `Scope` definition. Walter agreed, adding that inheritance could be used to add those functions only when needed. Razvan said that made sense.

Finally, I told everyone that DConf '26 would be three days. We'd decided to drop the hackathon. With costs increasing every year and lower hackathon attendance, I didn't think there was anything we could realistically do to get enough people to stay that would justify the expense. Dropping the hackathon would make it easier to stay within Symmetry's DConf budget and save us money on speaker reimbursements as well, since we'd be paying for one fewer hotel night per person. I'd already spoken to Symmetry about it, so it was a done deal.

## Conclusion

Our next monthly meeting took place on November 14th.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
