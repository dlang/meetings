The D Language Foundation’s quarterly meeting for January 2026 took place on Friday the 9th. It lasted a little under 25 minutes.

Our quarterly meetings are where representatives from businesses big and small can bring us their most pressing D issues, status reports on their use of D, and so on.

## The Attendees

The following people attended the meeting:

* Walter Bright (DLF)
* Luís Ferreira (Weka)
* Dennis Korpel (DLF/SARC)
* Mario Kröplin (Funkwerk)
* Mathias Lang (DLF/Symmetry)
* Mike Parker (DLF)
* Átila Neves (DLF/Symmetry)
* Razvan Nitu (DLF)
* Robert Schadek (DLF/Symmetry)
* Bastiaan Veelo (SARC)
* Nicholas Wilson (DLF)

## The Summary

### Mario

Mario said that Mathis, who wasn't present this time, liked named arguments for function arguments. Funkwerk had started using them and they worked fine. The problem was that VS Code still had an old `dscanner` and an old `dfmt`, so whenever they used named function arguments, VS Code marked them as errors and underlined them in red.

Robert said that, by accident, he had been in the code-d repository the day before and had seen that Jan appeared to be preparing a new release. He suggested Mario keep an eye on it, because the problem might simply go away soon. He added that Jan was usually quite responsive.

### Bastiaan

Bastiaan said he didn’t have anything to report. He only wanted to congratulate Dennis on an upcoming DMD release.

Dennis thanked him.

### Mathias

Mathias had nothing specific to report.

### Dennis

Dennis had nothing to report.

### Átila

Átila had nothing to report either. He said he wasn’t doing D at the moment and was back on Python.

### Robert

Robert said he also had no bad news. He joked that he was sorry about that.

### Walter

Walter had sent me an agenda item thinking this was a monthly meeting rather than a quarterly. Since nobody had much to report, I suggested we go ahead and use the opportunity to get feedback on editions from the industry representatives.

Walter wanted to know what edition years meant. If we said something was in the 2025 edition, did that mean it was added to the language in 2025? If he was adding an edition-based feature now, should he pick a future year? If a feature required an edition, which edition should it use: 2024, 2025, 2026, or something else?

I said that, as I understood it, an edition was a group of preview switches that had been enabled. Whatever compiler release turned those features on by default and no longer treated them as previews was what determined the year of the edition. So if the compiler with those features enabled by default was released in 2026, that edition was the 2026 edition.

Walter said that made the most sense.

I then said that the default pre-editions edition was intended to be Edition Zero. Walter objected. He thought it should be the edition before we started doing editions, rather than zero. I said that was the meaning of edition zero: there was no edition.

Walter said that  going from zero to 2025 was a discontinuity. I argued that assigning a year to the pre-editions state implied that it was itself an edition, when really it was the pre-editions edition. Robert asked how Walter thought it should be specified. Walter said 2025 would be the last edition of the old way.

I pointed out that the DIP explicitly called for a zero edition. Walter said that wasn't how it was implemented. In the implementation, there was a minimum edition. So something like 2020 wouldn't be a valid edition. Zero would require extra checks. To him, it seemed easier to say that 2025 was the edition before editions. Then someone who wanted to mark code as using the old pre-edition set of features could write that explicitly.

I said that I didn’t think people would mark the pre-editions edition in the code. The absence of an edition number in the module statement would be the pre-editions edition, and if someone needed to specify it explicitly, they would do so on the command line. Walter understood that a module statement without an edition defaulted to the most current edition, then someone who wants to use a module statement but stick with the old semantics needs a way to specify that old edition in the source. I agreed that was the case.

Bastiaan asked how this would work for old code, since old code wouldn't have an edition statement. He understood that such code would then use the latest edition, which made it sound as though editions were breaking code after all. The code could be unbroken by specifying an old edition, but the default behavior still seemed surprising to him.

I explained that we had been through this several times in the monthly meetings and other meetings about editions. The idea was that dub would handle this behind the scenes. Old code using dub would still compile because dub would set the default on the command line to the pre-editions edition. The issue mostly came up for people invoking the compiler directly.

Átila clarified that if a package had a `dub.json/sdl` with nothing specifying an edition, dub would know that and set the pre-editions edition.

I said that brought us back to the question: should the default pre-editions edition be zero or 2025?

Walter thought it was currently coded to be 2023. Dennis noted that it hadn't been released yet, so it could still be changed. Walter said that since 2025 was now in the past, perhaps 2025 should be the pre-editions edition.

I said that was why I thought it should be zero. That encapsulated all the years before the first edition. Walter said the issue was that if he was writing or maintaining code, he wanted to be able to say what edition he was using without relying on the default. If he put 2025 in old code, then he could rest assured that was what it was, without remembering to pass a command-line switch.

I said that if the first real edition ended up being 2026, then we could update the DIP to specify 2025 as the pre-editions edition. Walter didn't think the DIP needed to be updated at that point, but the behavior would need to be documented.

Mathias said he thought zero was much easier to remember than the year in which the first edition was released. Robert agreed that the memory argument was strong. Bastiaan said it was just a name, so it didn’t really mean anything for the code. In that sense, zero might be easiest. He jokingly suggested names like "Edition Nil" and "Edition None".

Walter still liked 2025. He thought that if someone saw 2025 in the source, they would understand that it meant the old, original semantics. I said the reason I liked zero was exactly that it set the pre-editions state apart. If you see zero, you know it is not an editions version of D. If you see 2025, it looks like the first edition.

Walter said that, in a sense, it was the first edition before editions. He also thought that `module foo` followed by a zero would look strange to someone unfamiliar with D. They might wonder what that weird zero after the module declaration meant.

I said I couldn’t imagine why someone would intentionally put the zero edition in the module statement. If they were compiling old code, they wouldn't go back and modify every source file to add edition zero. They would pass `-edition=0` on the command line, which to me meant no editions.

Átila agreed that he didn’t see why someone would pick the old edition in code either.

Walter said that if he was continuing to work on old code, he would put the old edition directly in the module statement and then upgrade it to a more modern edition when he got around to it. That way, when he looked at a source file, he would know which edition it used without having to check the build system or remember command-line switches.

Dennis said many people used scripts where they just ran DMD on a single D file. If such a file relied on old behavior and would break with a newer edition, the easiest fix might be to add something directly to the script file. If it was compiled with something like `rdmd`, the user might not want to specify the edition every time they invoked it.

I said Mathias’s point about remembering the year was still a good one. If it was 2030 and someone wanted to compile with the pre-editions behavior, they would have to remember which year represented that behavior. Was it 2025? 2026? 2027? With zero, they wouldn't have to look it up. Dennis agreed that was a point.

Robert suggested allowing both zero and 2026. Dennis suggested that any integer under 2025 could mean the default pre-editions edition. Robert joked that this would also solve his problem of trying minus one as the first edition. 

In summary, I asked if we agreed that 2025 or less would mean the default edition. Walter said that wasn't his favorite, but he could go with it.

Átila noted that D had had breaking changes in the past where it might make sense to add editions retroactively. Walter said editions only applied to the modern compiler. He asked what the default should be for someone who wanted to always use the latest edition. Dennis said the answer was no number, just what we do now, `module name`.

Walter brought up two scenarios. One was old code that the user wanted to keep compiling. The other was code where the user wanted to download a new compiler and automatically use the latest edition to keep the code up to date. The question was how to trigger those two scenarios.

Bastiaan asked whether you could specify an edition in the future. If an edition less than 2025 meant the zero edition, could he say edition 3000? Walter said that if you did that, one day your builds would break. More generally, if the compiler didn't know how to compile edition 3000, it should give an error.

Bastiaan asked whether the decision to select the latest edition when there was no edition marking was in the DIP so that he could read up on it. Átila said it hadn't originally been there, but he had changed the DIP to include it after we discussed the issue previously.

Bastiaan said the idea seemed unnatural to him at the moment, but he would read up on it first. Átila said most of the problem came from experience with C#, where people could get confused about new features or the version of the language if the language didn't default to the latest edition. The compromise was that most code was compiled with dub anyway, so dub could preserve compatibility for existing packages.

I mentioned that we had a monthly meeting coming up the next week, followed by an editions-specific meeting the week after that to discuss what the first edition should look like. I invited anyone from the quarterly meeting who wanted to join that discussion to let me know.

### Luís

Luís had joined late and said he hadn’t had much time to sort through things before the meeting. He mostly wanted to hear what other people had to say.

He did have one update. Weka had upgraded to LDC 1.38, and the next step would be upgrading to the latest. He said he would try his best to keep Weka staying very much up to date.

At the moment, some Weka projects were running with the latest compiler while others were still on older versions, such as 1.30. That made upgrades complicated. What they wanted was an easier compatibility path. Luís hadn't known about editions before the meeting, but thought it might solve some of their problems.

### Conclusion

Our next meeting was a monthly meeting on January 9th. We also had an editions-specific meeting scheduled for January 23rd.

If you are running or working for a business using D, large or small, and would like to join our quarterly meetings periodically or regularly to share your problems or experiences, please let me know.
