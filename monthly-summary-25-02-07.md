The D Language Foundation's February 2025 monthly meeting took place on Friday the 7th and lasted just under an hour.

## The Attendees

The following people attended:

* Walter Bright
* Rikki Cattermole
* Ali Çehreli
* Jonathan M. Davis
* Timon Gehr
* Mathias Lang
* Átila Neves
* Mike Parker
* Robert Schadek
* Adam Wilson

## The Summary

### Advisory email newsletter

Rikki said a recent discussion had highlighted that there were people in the D community doing things like writing entire D front ends and not reading the newsgroups. He'd started wondering what such people, or even companies, might want to hear about D that wasn't just news.

He figured they'd want to hear about things like editions, security issues, breaking changes, and such. He thought a newsletter would be perfect for people like that who might not want to be contacted regularly, but would be okay with it a few times a year.

Mathias thought that's what the Announce forum was for. Rikki said that people weren't reading it, plus it was quite noisy.

Walter said we'd had something like that when Adam Ruppe was doing his weekly D update posts. Doing a regular newsletter was basically a problem of finding someone motivated enough to write one regularly.

Rikki said this wouldn't be a regular occurrence. It would be in response to things like a new edition being released. That would be every two years or so. Security issues only when they happened. The same went for breaking changes. It would be quite a rare event.

I said that anyone who wanted this stuff could [subscribe to the Announce mailing list](https://lists.puremagic.com/cgi-bin/mailman/listinfo/digitalmars-d-announce). We'd been using this for as long as I'd been in the D community, which was over 20 years now. They didn't have to get every message individually. They could receive them batched.

Rikki suggested we could have something on the download page explaining how to do that. I said that would be easy. A newsletter sounded like the kind of thing that would fall in my lap. I already had enough to do and didn't have time to keep up with a newsletter.

Walter said we should go ahead and put something on the download page. Rikki said okay.

### Bitfields

Walter said that the boring, annoying bitfields were still there waiting. He'd implemented Timon's suggestions. The PR was there and ready to go.

Timon asked which suggestions Walter had implemented. Was it gated by `extern(C)`? Walter thought that was just not an attractive way to do it, and that the fear of non-portability was overblown and unnecessary.

He reiterated something he'd said numerous times before: the layout of struct fields was also implementation-defined and it varied between platforms. No one had a problem with it. He couldn't understand the angst about bitfields.

He said that Adam had suggested padding things out with explicit bitfields to fill out the field size and Átila had liked that idea. But Walter had realized that it wouldn't work because, as Timon had pointed out before, alignment wasn't only affected by the fields, but also by whether or not a field started on an aligned boundary. Filling out the fields wouldn't solve that particular issue.

Bitfields were fine and portable if you started with them on an aligned boundary and didn't do anything wacky with them like having them straddle a field edge, e.g., a bitfield with three bits poking out of the end of one field and five bits starting the next one. Nonsense like that wasn't portable, so you were fine if you didn't do it.

Timon said his concern was that his code would take on an external dependency that proved to have an issue, and then his users would be affected by it. Walter believed he had addressed all those issues.

Rikki said if Walter would disable the problematic behavior in `extern(D)` code and require `extern(C)` to enable it, that would solve it. Walter said it was too ugly to require `extern(C)` there.

Jonathan noted that Rikki hadn't suggested putting `extern(C)` on there all the time, only when the user wrote bitfields with problematic behavior. So you'd get no error when writing sane bitfields, but when you did something the dumb way, the compiler would tell you about it. Then people who didn't know what they were doing wouldn't have to just rely on the documentation because the compiler would give them an error.

`extern(C)` would allow you to do the stupid stuff because C did, and we needed to be able to do the stupid stuff sometimes when interfacing with C even if we didn't want to do that sort of thing normally.

So when writing normal D code, you wouldn't have to worry about it. When using ImportC, it was going to be `extern(C)` anyway. It was only when writing explicit bindings that it might come up. If the C side was doing something dumb with bitfields, you'd get an error from the compiler when writing the D binding, and `extern(C)` would let you get it done. Plus, you'd know you were interacting with something dumb at that point. It probably wouldn't even come up outside of that except when people didn't know what they were doing.

Walter reiterated that you could say the same thing about struct field alignment. It varied from compiler to compiler and no one had a problem with it.

Timon said it was a statistical thing. A vastly larger number of people understood C field alignment than understood C bitfield alignment. That was the difference he saw here. He didn't like the differences in field alignment either but thought it was just a more well-known issue.

I noted that when we'd discussed this in our [May 2024 monthly meeting](https://forum.dlang.org/post/avoxqcdipzvlbybqncmk@forum.dlang.org), the two biggest objections had been that we should:

* specify a layout for bitfields
* require `extern(C)` on any bitfield that was dependent on the layout of the associated C compiler's bitfields

That was pretty much what we were discussing here. We'd had a separate meeting about just the bitfields. Steve and Timon had been the most vocal in objecting on these grounds. We'd come to a compromise in which Walter was going to do some testing with it. If everything lined up, they would ease up on their objections and the DIP could move forward. I thought we'd already [decided in December to move forward](https://forum.dlang.org/post/uvqbluehjkljthuhzsjm@forum.dlang.org).

Walter said we'd had so many discussions about it that he didn't recall what we'd said when.

Jonathan said that if it was easy to do the right thing and there were only a few stupid things that you wouldn't normally do, he didn't think it would be a big deal to give an error in those cases. The problem was that we couldn't do that when binding with C. So if we wanted to do it, we had to complicate it a little by requiring `extern(C)` to do the dumb thing.

He didn't know if this was a big enough issue that we needed to do that, but he didn't think it was a big deal from a usability standpoint. You'd only need it when interacting with C code that was being dumb.

Walter said it was even rarer than that. It was rather unusual to find a C compiler that didn't lay things out the way other C compilers did on a given platform. All the C compilers on Linux laid things out the same way. All the compilers on Mac did, too. It was only if you were using Windows that you might run into trouble, as the Microsoft compiler did things differently.

I noted that a lot of people used the Microsoft compiler. Walter said that was true.

Ali said that this feature was so specialized and niche, very few people would use it. If it worked for 90% of those few people, then that should be okay. The ones who did get into trouble with it would find a way to get out of it themselves. He didn't think this was such an important topic to go to these lengths. We used imperfect tools all the time and found our way through them.

Walter agreed. He noted that the only time this would ever be an issue was if you were trying to conform to an externally imposed layout. If you were just compiling with the C compiler on your target machine, it would do the same thing the C compiler did.

There were two cases where it could become an issue: one was a hardware register, and the other was a file format with a dependency on a strange bitfield layout, which, though not impossible, seemed pretty unlikely.

Átila said he'd seen people write networking code that was dependent on what they thought was going to be the layout by using bitfields in C. Walter said that was an externally imposed layout. Átila agreed but emphasized that people did that kind of thing. Walter asked how many bitfields were in the network protocol. Átila said too many. And this was at Cisco, so he didn't know how the internet even worked.

Walter said he was working on an ARM backend. Every instruction in that was a bitfield, and each of them was an externally imposed layout. Was he using bitfields to generate them? No. Átila understood that, but said that other people would use bitfields in that case.

Timon said that there would be two groups of people using this: those who were qualified and those who weren't. Walter said it reminded him of unsigned vs. signed implicit conversions. He'd been dealing with it for 40 years and never had a problem. Átila said that he had. Jonathan said he'd been tripped up by it last month, but it was quite rare for him.

I asked if we could put the bitfields behind a preview switch. Walter said they already were. I suggested we put out a call to the community to test it in their own code with C bindings. If you didn't ask people to do something with preview switches, they generally wouldn't. Could we do that for three or four months and see how it went?

Walter said that was feasible, but it had already been there behind the switch for some time. We could ask people to try it. I said we could ask them to try and break it. Walter reiterated that it would only break on externally imposed layouts.

Timon said he wasn't excited about bitfields. Walter asked why, and Timon reiterated his concern that someone would use them in one of his dependencies and then he would be on the hook for the problems they caused.

Walter said that every time he worked on the custom bitfields in the compiler source code, he hated the fact that we didn't have actual bitfields. If you looked at the code, it stunk. It worked fine, but it was just ugly. Why couldn't we have bitfields in the language?

I said he couldn't use them in the compiler anyway if he implemented them now because of the bootstrap. Walter said that was correct. But he hoped to work with Iain to get the bootstrap moved to a more modern version at some point. Specifically one with bitfield support. That would clean up a lot of code. Besides, he was sure he wasn't the only one doing ugly, manual bitfields.

Ali said he had done them. He'd needed to pack some things together when working within some tight memory constraints, and he'd managed it all in a struct. If D gave him the ability to do it natively, he would.

Before that, 25 years ago, he'd tried to make bitfields templatized in C++ and it hadn't worked. But he'd managed his own exactly for some weird 63-bit hardware registers. He reiterated that experts would always find a way.

Átila said he'd used the Phobos implementation.

Walter said he was just tired of this whole thing. Átila said that was understandable. Walter said that meanwhile there was a blizzard of suggestions for minor language improvements. He didn't understand why he was the only one who wanted bitfields.

I asked if there was any room for him to move on the `extern(C)` thing for whacked configurations. Rikki said it would have to be a lint in D-Scanner if it wasn't in the compiler. That would have to exist. Átila said not everyone used D-Scanner. He referred back to Timon's point about being on the hook when someone did something stupid with bitfields in a dependency.

Rikki said it needed to be in the compiler. The compromise to only allow the stupid stuff in `extern(C)` made a lot of the problem go away. Átila thought it was a good compromise.

Walter said he would be surprised if anyone ever used `extern(C)` around bitfields. Átila said the rest of us would be, too. Timon suggested in that case we should just ban bad behavior. Átila said that was also a thing.

Walter said we couldn't ban it outright because then you couldn't use bitfields that were compatible with C. Timon noted that Walter had said he'd be surprised if anyone used `extern(C)` around bitfields, so if no one was going to use it, we should just ban the bad behavior.

Walter said it made your code ugly having `extern(C)` in the middle of your field declarations. Rikki agreed, adding that it would let you know where the bad code was.

Walter said one of the main goals of D was to look good. We hadn't always succeeded in that. The attributes were ugly, for example. The DIP 1000 attributes were ugly. He regarded that as a personal failure. But having the code look nice was a big deal. He cited some examples of ugly code he'd seen resulting from features in other languages.

I said anyone interfacing with C from D was going to be writing ugly code anyway compared to normal D code. I didn't think requiring `extern(C)` on bitfields was going to make much of a difference.

Walter said it wasn't there in the C code. We were adding something to D that C didn't have. I noted that we had that when interfacing with C already. I'd maintained C bindings for years and they were littered with `@nogc nothrow` and `__gshared`. Jonathan added that if you were using ImportC, it would do it for you and you wouldn't have to write anything.

Adam seconded all of that. He'd been doing a lot of work with ImportC and `.di` files, and that generated a lot of crap. If you didn't want to see ugly code, then you shouldn't look at any `.di` files generated by ImportC. It was always going to be like that and he didn't see any way around it, because it was C and not D. It wasn't written to our standards of "not ugly baby syndrome".

He agreed that attributes were ugly, but we didn't control that when it came to C code. He didn't think it was worth getting bent out of shape over the fact that the C interface looked ugly.

Átila said that if you were using ImportC and you weren't looking at the `.di` files, you wouldn't see the `extern(C)` anywhere anyway. And it would all just work. I added that your D code would still get the pretty D bitfields without the `extern(C)`. Átila seconded that and said that if we got this compromise, he would merge it himself.

Walter said that if we merged it the way it was now, then if someone in the future came up with a case that needed `extern(C)`, we could always add it. But he didn't believe it was ever going to come up. Átila said that wouldn't work because we wanted things to break if they weren't done the right way.

Walter said they would know if it broke. Then they'd come and complain about it and say, "If I only had extern(C) for bitfields, this wouldn't be an issue." Átila said that still meant making it a compiler error if people did stupid things, and he was okay with that.

I joked that Walter was saying he wanted to wait until Timon was on the hook for one of his dependencies. That prompted Timon to give some examples of dependency issues he'd already had related to floating-point rounding and data corruption. That led us to a brief detour about floating-point issues in general.

Getting back on topic, Átila said the compiler should give an error on the stupid things with hand-written bitfields, but exempt the error for ImportC. He'd be okay with that.

Jonathan said that way people who knew what they were doing could then do the dumb thing in a C file if they needed to, and the people who didn't know what they were doing would be fine.

Timon said he was knowledgeable enough that he knew the precedence of comparison operators and bitwise operators, but he was still in favor of the parse restriction we were discussing here.

Átila asked if it sounded good to everyone to make it always break in D code no matter what. Rikki said that approach would break `.di` file generation in ImportC. We needed an escape hatch. Walter and Átila said that was a good point.

Átila said in that case, we were back to `extern(C)` as the escape hatch. Adam said that worked for him. Jonathan said it worked for everyone but Walter.

Walter said he knew he was all alone on this. He would bet all of us a beer that no one would ever want to use `extern(C)` around their bitfields.

Jonathan said that might be the case, but it would only come up when someone was manually writing C bindings and encountered a bitfield that did something stupid. If the C code wasn't doing anything stupid, it would never come up. So you'd only get the ugliness on something that was already inherently ugly and arguably should be treated in an ugly manner because it was doing something dumb.

Timon said that in the implementation you'd have the C layout and the portable D layout, and whenever they didn't match, you'd check for `extern(C)`.

Walter reiterated that we didn't do that with regular fields. No one had to put `extern(C)` around regular struct fields and there were no complaints.

Átila said that was a good point, but that the difference was what Timon had brought up earlier. No one was going to try to change the fields around to get the layout right, but people would do that with bitfields and expect it to work just because it was explicit. There was a number right after the field. With normal struct fields, you didn't have to think about the sizes, you just wrote the types. With bitfields, the number was right there.

Adam said he knew that Walter loved airplane metaphors, and when it came to airplanes, beautiful planes flew better. But under the skin, an airplane had a whole lot of ugly. In engineering, there was only so much we could do to make it all clean and pretty.

This compromise was saying that you had to use `extern(C)` if you wanted to do the dumb thing. That would make bitfields on the D side nice and pretty, and then over here in the C binding we could do the dumb ugly thing because we were already doing ugly things all around it.

Átila reminded us that `__gshared` was ugly on purpose. Walter agreed. Átila said that was what we were asking for here: make it ugly on purpose.

Jonathan reiterated that you were already using `extern(C)` in C bindings anyway, and probably not on every line that needed it. You were probably putting it at the top of a file with a colon. Átila thought it would only really show up in `.di` files.

Adam said if you really had to do the dumb thing in your D code, then you'd be able to explicitly tell the compiler to remove the dumb thing by using the ugly `extern(C)`, and ugly surrounded by beautiful was a red flag indicating you needed to pay attention to what was going on there.

Átila repeated that he thought it was a good compromise. Jonathan repeated that it was ugliness on something that was already ugly, so it was a non-issue. Timon said it was like string mixins. He couldn't count the number of posts in the newsgroups asking for a prettier way to do them, but the answer was always "of course not". Rikki said that good things should be pretty and unannotated by default. If you wanted to do something different that wasn't as safe, you had to opt into it.

Walter said he would think about it some more.

(__UPDATE__: Walter attempted to implement the `extern(C)` proposal, [but encountered a problem with `extern(C):` and how it interacts with structs](https://github.com/dlang/dmd/pull/20840#issuecomment-2647233181). This came up in the March meeting. The DIP [has since been submitted and approved](https://forum.dlang.org/post/wstbhxovohvwokldbdeo@forum.dlang.org).)

### Conclusion

We chatted a bit longer about SAOC and some details about the JSON project proposal for Google Summer of Code. I announced that we were close to finalizing the DConf dates and asked everyone to start thinking about their submissions. I said that I was looking for someone to join me for the next community conversation ([it ended up being Adam](https://youtu.be/HxgnfUZBqHE)).

I scheduled our next meeting for March 14th, 2025.

If you have something you'd like to discuss with us in one of our monthly meetings, feel free to reach out and let me know.
