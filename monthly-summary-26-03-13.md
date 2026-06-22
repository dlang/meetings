The D Language Foundation's March 2026 monthly meeting took place on Friday the 13th and lasted about an hour and fifteen minutes.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Jonathan M. Davis
* Dennis Korpel
* Razvan Nitu
* Mike Parker
* Gaofei Qiu
* Robert Schadek
* Steven Schveighoffer
* Nicholas Wilson

## The Summary

### DConf '26 and the D blog

Nobody had sent me any agenda items, so I opened with a couple of updates.

First, the DConf '26 contract had made it through Symmetry's legal department. I was just waiting for someone to sign it. The venue contact had been pushing our event planner because they were still holding the dates for us, but had since gone on vacation, which gave us a little breathing room. I was hoping to wake up the next morning to news that the contract had been signed, which would let me formally announce the dates and put out the call for submissions.

The website was mostly done. I still needed to finalize the logo, and I was thinking of using "D Rox London 2026" as the theme. We hadn't used that one before.

Walter suggested using AI to improve some of his D-Man art. He had fed some of his own drawings into one and asked it to make them look professional. It had made much better drawings from them, and he had been having fun feeding more things into it.

I said that for DConf, I needed vector graphics files I could send out for banners and t-shirts. I was keeping it simple and just using the rocket logo we had been using for DConf for years plus some text. I wasn't planning to do anything along the lines of the Tower Bridge outline or the Elizabeth Tower outline I'd used in the past.

The second update was that [blog.dlang.org](https://blog.dlang.org/) had been migrated to a GitHub site and was now much snappier. All the existing posts had been moved under an archive subdirectory, and new posts would go at the top level. The new setup should make publishing much easier. People could submit Markdown files as pull requests. Once merged, those files would be converted automatically to HTML on the blog.

### Fei

Fei introduced himself as a master's student of Ben Jones who had graduated the previous December. He was actively looking for a job in the US and wanted to get experience by contributing to D. He said he was working with Nicholas on the DCompute library's Metal implementation and was excited about it.

Fei was wondering if there were problems or issues he could handle. I said that if anything came up during the meeting, we could give it to him and let him run with it.

### GSoC and the AST refactoring project

Nicholas wanted to know the status of the AST refactoring project and how close it was to being finished.

Razvan said that on the surface it seemed very close, but there was still the issue of indirect dependencies. Those would need to be fixed before we could determine how to get rid of `ASTBase`. That was essentially the milestone. If we could get rid of it, then we'd know the refactoring was done.

Nicholas assumed the project would have a reduced number of hours compared with other potential GSoC projects. Razvan said he wasn't sure there was a way to tell exactly, so he would probably go with a medium-range estimate and see how it went.

Nicholas asked if Razvan could respond to some of the students who had contacted him. Razvan said he thought he had responded to all of them, but the number of messages was overwhelming. His strategy for GSoC was to ask for as many slots as possible to cover all of our projects, knowing that organizations usually received fewer slots than they requested.

Based on his calculations, he wanted to put the performance regression publisher project as the top priority. He wasn't sure how engaged Átila had been with students, but he would put one of the DCompute backend projects second, Átila's project third, the other DCompute backend fourth, and the refactoring project last. That meant we probably wouldn't get a slot for the refactoring work, but he was still spending time responding to all the students interested in it.

I noted that there seemed to be a lot of interest in the refactoring project. I had seen people discussing it in Discord, and I couldn't remember how many emails I'd forwarded to Razvan. Nicholas said he had come across at least two students wanting to do it. Razvan said he had talked to at least fifteen students about it, possibly more.

Nicholas said he had about six or seven students interested in the Vulkan and Metal backends, several interested in the performance regression publisher, and three or four interested in Átila's Ninja backend for dub project. I noted that a lot of PRs tended to come in around GSoC time.

(__UPDATE__: See [Razvan's announcement about our accepted projects](https://forum.dlang.org/thread/yuhxfrromdwmbqjznpfd@forum.dlang.org).)

### Section attributes and linker lists

Rikki said he had been in the process of upstreaming [the section attribute work to DMD](https://github.com/dlang/dmd/pull/22719/) from LDC, GDC, and OpenD, based on Adam Ruppe's implementation in OpenD. He'd encountered some problems with it and he'd basically had to start over.

However, the section attribute by itself wouldn't give us first-class support for linker lists. Rikki explained that a linker list was essentially a technique where a bunch of variables were placed into a section, and you could just grab an array of them. He said this was a very nice trick, and he wanted to make it first-class in D because D didn't have run-time reflection in the way languages like Java did.

Dennis asked what problem Rikki was trying to solve and what kind of run-time reflection he wanted. Rikki gave web development as an example. You could put a `@route` attribute on a function and it would be registered automatically as a route. Current D web frameworks didn't do this automatically. They relied on UDAs in a top-down way, meaning you had to know about the module, iterate over the members, and so on. That was slow and required you to know the module existed. With linker lists, you could get a more automatic experience like in frameworks such as Play.

Dennis asked whether this was also how module constructors were implemented. Rikki said yes. Dennis said that made sense. He noted that the unit test runner had similar use cases, where static runners needed to list modules and use traits to get unit tests, but the run-time version just reflected through `ModuleInfo`. He asked whether Rikki wanted users to be able to customize that sort of thing. Rikki did.

Dennis asked whether LDC already had this. Rikki clarified that LDC had the section attribute, and he was bringing that to DMD, but what he was talking about with linker lists was something on top of that. The section attribute was a prerequisite.

Nicholas said LDC, through LLVM, had a mechanism where variables could be appended to a global list independently of stack initialization. That was how LDC implemented module information. He thought it would probably be relatively easy to do in LDC. For DMD, he didn't know, but perhaps after the infrastructure was in place, some of the infrastructure used for module information could be reused.

Rikki [linked a test in the chat][1] to show how difficult it was to define a linker list and get access to it manually. He had even had to use `pragma(mangle)` because of prepended underscores which prevented the usual linker trick for generating start and end symbols on macOS. There were many platform- and linker-specific details that had to be exactly right. He thought we couldn't tell users to do this manually. It needed to be first-class to be really useful.

Dennis asked whether it could be part of a library. Rikki said Rust did it in a library, but the way he wanted to do the append was through CTFE append on a global variable, which would currently just error out. He liked that approach because it was simple. Reads would happen at run time, not compile time One of the big problems with linker lists was that all the types and mutability details had to be consistent across everything, or things would go bad and fail to link.

I asked if Rikki had a DIP drafted. He said the section attribute was upstream and going into `core.attribute`. That part was not a DIP. The linker list idea would need a DIP, and that was one of the next things he needed to work on once the section attribute was merged. He had [posted about linker lists in the DIP Ideas forum][2], but he hadn't updated it to say he now wanted to get rid of the struct and stick to a global variable. He had originally thought the struct would be needed to handle target and platform differences, but the section structure turned out to be consistently supported, so he no longer saw a reason for the struct.

Walter said this wasn't helpful. He didn't know what a linker list was, and in the PR Rikki had linked, he saw no explanation, no documentation on what it did or how it worked. It needed exposition.

Rikki clarified that there was no PR for the linker list yet. The PR Walter was looking at was for the section attribute, and the way to test that was by doing the linker list manually. Walter said there was no exposition of what that was.

Rikki said it was in both GDC and LDC. It was just a way to say, "Hey, put this variable into a section." Walter asked Rikki to link to documentation on what it was. He didn't know what it was. Rikki said he'd find something. Walter said if it was released in GDC or OpenD, there ought to be some documentation somewhere written by somebody. We couldn't add features without documentation.

Rikki said it wasn't a language feature, but a compiler feature. Walter said if it was expressible in source code, there needed to be something in the language specification in `@section` saying what it meant and what it did. This was not a user-defined attribute. Rikki said it was. Nicholas clarfied that it was in `core.attribute`. The compiler handled it specially, but it was a regular UDA. Walter said it wasn't a UDA because the compiler was being changed to accomodate it.

Nicholas said it was implemented as a UDA in that you could reflect over it. It only needed the compiler PR because the compiler handled it specially. And it was documented in that there was a Ddoc comment in `core.attribute`, which Rikki had pasted in the chat:

```d
**
* When applied to a global variable, causes it to be emitted to a
* non-standard object file/executable section.
*
* The target platform might impose certain restrictions on the format for
* section names.
*
* Examples:
* ---
* import core.attributes;
*
* @section("mySection") int myGlobal;
* ---
*/
```

He said we already had stuff like that in `core.attribute` for selectors and stuff.

Rikki said that linker lists would be a nice feature, and that getting section attributes into DMD was overdue, even if that was only half the work.

(__UPDATE__: Rikki's section attribute PR was merged a week later.)

[1]: https://github.com/dlang/dmd/pull/22719/changes#diff-c0cd594bdc7f14a8a0320c015e52bebef6aaa90675551fa2deaaeb554875bf58
[2]: https://forum.dlang.org/post/pilnkcllbpntvysjhkui@forum.dlang.org

### AArch64 backend

Walter said the compiler could now compile itself for AArch64, but it couldn't link yet because the runtime library didn't exist for AArch64. He was currently working on getting DRuntime to compile successfully.

Nicholas congratulated him. Walter said he had run into some problems because some D code required inline assembler, and inline assembler wasn't working yet. He was faking it for the moment and would have to go back and fix it later.

### `opUnwrap`

Rikki brought up a new operator overload for structs that he was proposing called `opUnwrapIfTrue`. The idea was that in `if` statements for structs, `opCast(bool)` could be used to make the unwrapped value available in the `true` branch using a temporary for the truthiness check.

[His DIP had been in the development forum](https://forum.dlang.org/post/vcozmhysmztnmngzopnm@forum.dlang.org) for about two weeks. He would probably leave it another week. [There was a PR for the implementation](https://github.com/dlang/dmd/pull/22570) that was basically ready to go aside from needing a rebase. It was as simple as he could make it, with small code changes and nothing to remove.

The point was to remove an entire class of bugs where you could call `get` on a result type, `Optional`, `Nullable`, or reference-counted type without first checking that a value was present. The feature combined the check with the `get`.

It had a lot of research behind it with his code base. It was definitely possible to call `get` without a check when writing normal code. That was not good. `Nullable` had shown this exact problem, and `alias this` had been removed because people were calling `get` without checks.

Jonathan said the real problem was that it happened accidentally in templates, which was a somewhat separate issue.

Steve asked why this couldn't be done in a library type. D already had `opCast(bool)` that could be overloaded, and everything else could forward to the thing that was underneath. Rikki said it did use `opCast(bool)`. Steve asked why we needed a new language feature in that case? Rikki said there was just no way to do that with `if` statements. You could do a "tryGet" method, but that was also error-prone.

Jonathan said what Rikki wanted was to be able to write an `if` and automatically do an assignment without any extra checks. Today, it was already possible to check whether something had a value and then call `get` to retrieve it, but those were two separate operations. He said Rikki wanted it to be a single operation. That made it less obvious what was happening, but it did make it so the check and the get were connected, for better or worse.

Walter said anything that could be done with an operator overload could be done with a regular function. It might be useful to try the regular function approach and see how far it went before deciding whether an operator overload was really needed.

Rikki said it wasn't just a function call. It was dealing with variable declarations and `if` statements.

I posted an example from the DIP:

```d
import core.attribute : mustuse;

@mustuse
struct Result(Type) {
     private {
         Type value;
         bool haveValue;
     }

     this(Type value) {
         this.value = value;
         this.haveValue = true;
     }

     bool opCast(T:bool)() => haveValue;

     Type opUnwrapIfTrue() {
         assert(haveValue);
         return value;
     }
}

Result!int result = Result!int(99);

if (int value = result) {
     // got a value!
     assert(value == 99);
} else {
     // oh noes an error or default init state
}
```

Walter said he could see the DIP, but it would take him time to read and figure it out. Rikki said the DIP included a lowering that showed what the code turned into. Walter said he found it confusing what was going on with it and couldn't give an opinion during the meeting.

Dennis asked if Rikki had [seen his comment in the thread](https://forum.dlang.org/post/ztsgfvtdtpcsmsrwqzzh@forum.dlang.org). He said that `foreach` was an explicitly different thing than `for`. This DIP basically said that if this operator overload was found, then a normal `if` would be silently transformed behind the scenes into something entirely different.

Rikki said he had seen the comment, and added that Dennis wasn't the only one who had mentioned it. He emphasized that this was an opt-in operator overload.

Dennis said that wasn't really the case. If he read somebody's code, it could now do some whacky stuff behind the scenes because he didn't know what the types had or hadn't secretly defined. It obfuscated the code for the reader.

Jonathan said if you didn't know the result was a `Nullable` or whatever, and you thought it was something else, you could completely misunderstand what the `if` condition was doing. In normal circumstances, it would be converting the `int` to `bool` directly, but that wasn't what this was doing. You could get used to it, but on the surface it was at least potentially confusing.

Dennis said that in the example of `if (int value = result)`, today, he would know that `int value` was truthy if it wasn't zero. But with this feature, if `result` had the relevant overload, the `int` inside the branch could be zero even though he just checked it was nonzero. Jonathan said that was because the result wasn't actually an `int` at that point, but that was not immediately obvious when reading the code. Dennis agreed.

Rikki said the idea of doing it as a storage class had come up. He wasn't against it. His only complaint was it was something that could be removed since it wouldn't be strictly required. That's why he hadn't pursued it. Jonathan said the main advantage was that it increased clarity, but it was also more verbose.

Dennis said that if Walter couldn't figure it out in a few minutes, then it wasn't really simple. Walter said the DIP example was a bunch of code with complicated functions, and he couldn't make sense of it by glancing at it. He might be able to if he were already familiar with the pattern, but he was not.

Steve said it was about scoping. You had this variable that was only valid at certain times. Once you discovered it was valid, you had a scope where you could access it. Outside that scope, you couldn't access it without unwrapping it first.

Walter noted that people often said DIP 1000 was utterly confusing even though it was not confusing to him. Jonathan said the point was that just because Rikki or others thought this proposal was obvious, that didn't mean Walter would immediately find it obvious.

Walter said he wasn't convinced it was a bad idea, but he wasn't yet convinced it wouldn't be confusing for users. Maybe there was a better way to do it.

One nice thing about `foreach` was that although it had complexity under the hood, people understood right away that if a range was implemented properly, you could `foreach` over it. He didn't yet know whether `opUnwrap` fell into that category. He said he would read the DIP, but it couldn't be resolved in the meeting.

He asked whether this had anything to do with option types. Rikki said yes, that was exactly what it was about, though it wasn't putting an option type into the language. It was just unwrapping structs.

Robert said he favored making it simple. The only issue he saw with `Nullable` was that you could call `get` without a default parameter and then get an assertion if there was no value. This proposal was making it too complicated. Yes, other languages had it, but his answer was to remove `get` without a default parameter on `Nullable` and move on. This was just too complicated.

Steve said you didn't always want to require a default parameter, because if the value being retrieved was complex, you didn't want to build a default every time you called `get` when you already knew the value was valid. His main concern was the syntax. It didn't look any different from a normal `if` syntax. He had seen proposals that modified the `if` statement so that one clause checked the condition and a second clause extracted the value. He would be more comfortable with something like that.

Dennis agreed. Rust and other languages didn't use the exact same syntax for a normal `if` and an unpacking or unwrapping `if`. In Rust, it was an explicit syntax. Here, it was ambiguous whether it was a regular `int` cast to `bool` or an option type unwrapped and then converted to an `int`.

Nicholas noted that C++ had `if` declarations with intialization, kind of like a `for`, where you could have a statement that declared and initialized a variable and then an expression that tested it. Walter asked if that was a new C++ feature. Nicholas thought that had been added in C++17 or C++20. Walter asked Rikki to add the C++ version to the DIP as prior work. Rikki agreed.

Jonathan noted that the C++ feature was useful but didn't fit quite the same niche. Nicholas agreed it was not the same but said it was related.

### `int[$]`

Nicholas said he'd been reading one of the previous meeting summaries and came across `int[$]` again. He asked whether there had been any decision on deprecating static array initialization with mismatched lengths, because using `$` there would be nice. I said I hadn't been privy to any discussions about it. Nicholas thought there had been no further discussion on it.

I asked Walter whether [he had seen the recent PR](https://github.com/dlang/dmd/pull/22623) for `int[$]`. Walter didn't recall it. I said someone had implemented it and decided to skip the DIP process. Walter said he didn't really blame him, since it wasn't a particularly complex thing to implement.

I noted that [the old DIP for this](https://github.com/dlang/DIPs/blob/master/DIPs/other/DIP1039.md) had never been rejected. There was a myth that it had been rejected by Andrei, but it was just abandoned by the author. Steve thought people had said that since we had `std.array.staticArray`, we didn't need this. I said that kind of feedback against the need for the DIP was why the author had abandoned it.

Jonathan said the only downside it had that he was aware of was that it still didn't give you a way to create a static array separate from a variable with a specific size. But it was still an improvement over the current situation where he had to use the `staticArray` template.

Nicholas said he would clean up the PR, see what the CI said, and then write a spec PR. I asked whether we needed to revive the DIP. Nicholas said technically it was a grammar change, which would normally require a DIP, but it wasn't especially complicated. Jonathan said that if we wanted to follow the rules, then yes, we needed a DIP, but the feature wasn't complex. Walter thought a DIP wasn't needed if the feature looked good and had a documentation PR. Nicholas said there would definitely be a spec PR. Walter said it shouldn't be merged without one, and that in this case the spec PR could replace the need for a DIP.

(__UPDATE__: The original PR was closed in favor of [a refactored version of it](https://github.com/dlang/dmd/pull/22745). That one was merged a couple of weeks after the meeting.)

### The DIP Ideas forum

Given that Rikki had skipped the DIP Ideas forum for his DIP, I asked that he and anyone else who was thinking of submitting a DIP please go to the Ideas forum first before drafting up the DIP. If Walter or Átila didn't leave any feedback, then the author could let me know and I would ping them until they did. Then the author could decide whether to move forward with the DIP.

Rikki said he had wanted to see how the idea played out in code, and since it was a small proposal that took him only about a day, he wasn't too worried about having to change it.

Walter reminded everyone that although he had rejected DIPs, many of his own DIPs had been rejected as well. Nobody should take it personally.

### Deleting `build.d` and more

Walter said Dennis had shown him how to build DMD with a one-line command. He hadn't known DMD could be built that way, and he had moved away from using `build.d` to compiling DMD with a one-liner from the command line. He asked if we could just delete `build.d`.

Dennis said he wanted that badly. Átila agreed. He thought `build.d` was awful. Rikki was shaking his head. 

Nicholas pointed out that `build.d` also built test runners and documentation, split out front-end and back-end pieces, and produced headers. Rikki added that it handled C++ interop header generation and tests and a whole bunch of other things. He wasn't saying that was good, only that it did many things.

Walter said he couldn't make heads or tails of how `build.d` worked. At least for users who downloaded DMD and wanted to build it, the instructions should give them the command-line version rather than `build.d`, because `build.d` required everything else to be working, including an already working DMD. He found the command-line version elegant: type DMD with a few arguments and get an executable. `build.d`, by contrast, did all kinds of things nobody understood and had multiple layers of obfuscation. He was fine with keeping it for other things, but for building DMD itself, he wanted users told to use the one-liner.

Dennis wanted to bring up a related issue. Building the website used a giant Makefile with all kinds of rules, and it downloaded an old DMD compiler. He had wondered why he couldn't simply use his own D compiler, and then discovered that it needed `std.xml` for something, which had been removed from Phobos some time ago. He had been looking into modernizing that.

Another big thing he wanted to do, because it had been such a recurring issue with DMD requests, was to move the language specification into the DMD repository. That way, compiler PRs wouldn't constantly need corresponding spec PRs in a separate repository. Walter gave that two thumbs up.

Jonathan said the whole situation was a mess, so doing that would be difficult regardless. His gut reaction was that if we were doing it from scratch, and maybe at some point in the future, we would want to put all the major repos together. Ideally that might be where we ended up, though the question was how to get there.

Átila asked whether we could put Phobos in the same repository and have a monorepo. Walter said no. He didn't want Phobos dependencies creeping into DMD. He had seen people put Phobos dependencies into DMD, and then he had to remove them. Átila said people already did that, and we could enforce against it without using separate repositories.

Walter said that a separate repository at least gave people a clue that Phobos should not be part of DMD. This mattered because he was trying to get DMD working on a different target, and if DMD depended on Phobos, then he couldn't get the compiler to run until everything in Phobos also worked. That was a major complication.

Dennis agreed with Walter in principle, but said that in practice, having separate repositories didn't currently prevent the problem. What mattered was whether Phobos was in the include path when building compiler modules. Currently it was. That meant Phobos dependencies could already creep in. The power operator lowered to `std.math`, and some compiler tests still imported from `std`, though those needed to be removed. The repo boundary itself didn't stop it.

Nicholas said that sounded like a good candidate for CI enforcement, checking that DMD didn't import Phobos. Dennis said that was independent of the monorepo question. It was a build configuration change, at least if we were still using `build.d`.

Walter said he had considered moving the power operator implementation into DRuntime for that reason. Dennis said Iain had been working on that and had a PR, though it was stalled.

Nicholas said a monorepo would make some things easier, but Martin Kinkelin would need to be involved because the LDC configuration was somewhat different, but that was nothing we couldn't fix.

## Conclusion

Our next meeting was a quarterly meeting on April 3rd. We pushed the April monthly to the third Friday, the 17th, since several folks were attending [the D Programming Language Symposium](https://dlangsymposium.com/) April 10th - 12th.

If you have anything you'd like to bring to us in a monthly meeting, please let me know.