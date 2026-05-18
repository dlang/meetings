The D Language Foundation’s quarterly meeting for October 2025 took place on Friday the 3rd. It lasted about fifty minutes.

Our quarterly meetings are where representatives from businesses big and small can bring us their most pressing D issues, status reports on their use of D, and so on.

## The Attendees

The following people attended the meeting:

* Walter Bright (DLF)
* Luís Ferreira (Weka)
* Martin Kinkelin (LDC/Symmetry)
* Dennis Korpel (DLF/SARC)
* Mathias Lang (DLF/Symmetry)
* Mike Parker (DLF)
* Carsten Rasmussen (Decard)
* Bastiaan Veelo (SARC)

## The Summary

### Bastiaan

Bastiaan said SARC had no major news to report. They were looking forward to a new release, and there had been no major setbacks. I joked that this might be the first time he had shown up with nothing for us. He agreed that maybe it was.

### Luís

Luís said that at DConf, people had suggested Weka try the new GC. They still had not had a chance to upgrade the compiler far enough to test it, but it was something they very much wanted to do.

The main issue for them was on the LDC side. Some of Weka's binaries, especially their unit test binaries, were so large that they exceeded four gigabytes. At that point, the linker started complaining about overflowing relocation address space. As he understood it, the relocation addresses were signed 32-bit values, so once the binary got past that size, the relocations overflowed.

Walter asked how they were managing to get a multi-gigabyte amount of code in the first place. Luís said part of it was that they kept a lot of static data in the binary. Some of it was configuration-related, and some of it was for arrays where they liked to have precomputed values. They also used a great many templates. In the unit tests especially, those symbols took up a lot of space. One workaround they had already used was converting some of the static data into runtime-initialized variables, since performance did not matter much for unit tests.

Walter said that if the arrays were that large, then the program could just load them after startup rather than baking them into the binary, and he suggested they also look into why they were using so many templates. Luís replied that a lot of those templates came from their upgrade and downgrade infrastructure, which they had presented at DConf. He agreed there were things they could improve on their side and stressed that not all of this was the compiler’s fault.

Walter asked whether this was happening in a 32-bit or 64-bit compile. Luís said it was 64-bit, but the relocations were still signed 32-bit. He had tried moving to 64-bit relocations in LDC but had not managed it.

Luís then brought up a second issue. Weka wanted to move away from statically linking everything and start using dynamic linkage instead, but they had run into a lot of problems there as well. He expected that once they got further into that transition, they would start filing bug reports. One example he had already seen was that export did not work properly together with `-fvisibility=hidden`. He thought this was probably LDC-specific, though he wasn't sure whether there was an equivalent DMD-side mechanism for the same thing. He had noticed cases where compiler-generated symbols, such as postblits, weren't being marked for export, which meant they weren't exported from the shared library.

I asked whether he already had a bug report for that second issue. He said yes, there were two bug reports related to export not working properly, but he didn't have them handy. I told him to send them to me and I would forward them on to Dennis and Nick. He added that Ilya might also have some other issues to raise, as he was working on a new build system. Weka were integrating Bazel and had run into some issues there, though Luís was not sure of the details.

### Carsten

Carsten said Decard were building and testing a larger network and trying to push throughput higher. The issues they were dealing with were not really anything for us to solve. They were just having some problems they didn't fully understand yet, and as part of that they were writing a new network in D. They wanted communication across several hundred nodes, and what they currently had was not enough for that.

I asked whether there was anything blocking them or any issue to report. Carsten said no. They were happy with D and had no problems with it at the moment.

### Dennis

When I asked Dennis whether he had anything work-related to report from SARC or the DLF side, he had nothing from work.

On the DLF side, he said that he had finally managed to build the 2.112 beta and upload it, so the files were now present on the pre-releases download page. What still remained was updating the changelog, the website, and the announcement.

He said there had also been a tagging issue that Bastiaan had spotted involving dub. The tag still appeared to come from May, and Dennis wasn't sure whether the tag was wrong, the build was wrong, or both. He needed to look into that.

Bastiaan said that from what he could see, the build appeared to be shipping the latest commit from dub, but when the version was printed it still showed the old tag from May. In practical terms, that probably didn't matter much, because the latest stable version was what should be shipping anyway. Dennis said that on the one hand that was good news, but on the other hand it meant something weird was going on in the build process, because in his mental model the build should have been built from the tag.

Eventually, he planned to move away from the current janky local workflow and transition the build process to GitHub workflows. His goal was to get GitHub Actions building the release artifacts and gradually smooth out the process, since at the moment the most annoying part was simply getting things built at all.

That led into a discussion about code signing. I asked whether, since he was moving things over to GitHub Actions, he would want to get the build process settled first before looking into code signing. Dennis said probably yes. He also still needed to investigate macOS code signing, which Luna had offered to help with, and was not even sure yet whether that could be done on GitHub or if it would still require a local VirtualBox setup.

Walter said it was a terrible look for us when people clicked the download link and got a warning that the file was unsigned and should be used at their own risk. He understood that the release process was complicated, but asked whether we could at least fix the certification problem. He asked what it would cost us to get a certificate.

Dennis said he had briefly looked at SSL.com. A one-year license there appeared to cost about $120, and if you bought more years up front, the annual price dropped. Walter asked if that was per executable, and Dennis said no, he thought it was per license and could be used on as many things as needed.

Walter asked if Dennis could just buy the license and be reimbursed. He told him to get the ten-year option. He said he had been afraid it might cost thousands of dollars, but at only a few hundred dollars, it was a no-brainer and something we should obviously do.

I noted we'd had one that Iain had signed us up for, but then the price had become much too expensive. I was sure he'd looked into other options, but ultimately didn't choose one because none of them were easy to integrate into the current release process.

Bastiaan said they had implemented code signing at work a few years ago and that it had been non-trivial and quite a pain. Part of the trouble was that the company selling them the service had not been allowed to explain how to automate it. The apparent intention was that signing should remain a manual step, almost like a physical signature. He said that in practice it was now integrated into their build infrastructure, but it involved a physical dongle that had to be plugged in somewhere. Because of that, he wasn't sure how well it would fit with GitLab- or GitHub-hosted workflows. He suggested Dennis could ask someone at SARC about what they had ended up using.

Martin said that, as far as integration with the release process was concerned, the short of it was that we'd have to install the GitHub Actions runner on a separate machine with a dongle, create a registration token to freely register the runner with the repository, and then we could have a CI job that would download the generated artifacts, sign those on our runner on the machine with our dongle, and then upload them.

I said this wasn't going to get solved in a weekend. I repeated my suggestion that we let Dennis get the refactored release process in place first instead of adding additional complications to the existing one. Walter said okay, but he was worried that this was a very bad look for us.

(__UPDATE__: We've since had discussions about signing. After the most recent email thread, Adam Wilson is looking into getting us set up to sign via Azure.)

### Mathias

Mathias said that if we could make `-checkaction=context` work, and ideally make it the default at least in unit test mode, that would make everyone’s life easier.

When Carsten said he didn't understand what that was, Mathias explained that it was the thing that told you what expression had actually failed in an assert. Instead of only getting an assertion failure, it could tell you that this value had been compared against that value and the comparison had failed. The last time he'd checked, there had still been linker errors. They were hard to solve because they appeared at a different stage, but he still thought making it the default, at least in unit test mode, would be a very good move for D because it would provide a much better user experience when running unit tests.

Carsten said it wasn't a good idea to run threaded tests in unit tests. It didn't seem to isolate correctly, which was why they did not run threaded tests that way at Decard.

Dennis thought there had been a move to make `-checkaction=context` the default, but that had been blocked because there were still bugs in the lowering. In some cases it caused errors that didn't exist without it. He had run into one related to scope slices some time ago. He wasn't sure what the current state of that was, but said it still needed some work.

Martin came into the meeting then. Mathias said Martin would agree with him about making `-checkaction=context` the default in unit test mode.

Martin said he did agree, but that came with the existing problem around template instantiations, so he'd have to think about it. He mentioned the workaround currently used for BetterC to enforce template emission and said maybe something like that could help, but he was worried because it still didn't work in quite a few cases. If it only failed in one case out of a million, that might be acceptable, but as things stood he was cautious.

### Martin

Regarding work, Martin said he unfortunately was not using much D for Symmetry at the moment because priorities for him had shifted toward Python. Still, he did have one important thing to report, and that was the status of the new GC.

He explained that the GC, originally Amaury's work for SDC and then taken on by Steven to port to the symgc dub package, had been integrated directly into Symmetry’s LDC fork. In their case, using the symgc package as a separate dependency wasn't attractive because they had many executables, dozens of test runners, shared libraries, and so on. Pulling the package into each one explicitly would have been ugly, so they had baked it directly into the runtime and made it register automatically.

With the latest symgc 0.9.6 release, it now even defaulted to the new collector rather than requiring an explicit opt-in. Their main Symmetry product had already been switched over to use it by default after several months of testing with people opting in first. They hadn't seen any real issues during those months, and the results had been very strong. Peak Resident Set Size (RSS) was down by at least half, and in some cases by as much as two-thirds. He said that in their environment it was now production-ready, and they were shipping it.

At the moment they supported Linux and Windows on x86_64 only. He thought AArch64 support had also gone into the latest incarnation, perhaps Linux-only for now, but macOS was still missing and that was a bigger piece of work. The long-term plan was to get it working more broadly on AArch64 across platforms, hopefully including Windows.

Beyond that, Martin said he hadn't had much time lately to work deeply on LDC-related things, but there was one improvement he wanted to get out in the not-too-distant future. He wasn't yet sure what the exact plan was for 2.112 or whether the stable branch would remain as it was or master would be merged back into stable again, but he said one recent development was that LDC now had a new contributor who had been helping with a major refactoring of the configuration system.

He couldn't recall the person’s name off the top of his head, but said he was Romanian and had been helping primarily with configuration work, especially making separate builds of the runtime and compiler easier, including cross-compilation of the compiler itself. The old single ldc2.conf file in the etc directory was being replaced in spirit by a configuration directory model, similar to the old SysV style on Linux with numbered files. The old single-file setup would still be supported, but the new design would make it easier to inject new configuration fragments or override specific settings without touching the original file.

The main goal was better support for multiple targets. LDC had long had implicit cross-compilation capability, and they wanted to make that more accessible to users. Rather than editing the full config file by hand, future tooling should let someone do something like call a command for a target such as Windows x64 and have the appropriate prebuilt DRuntime and Phobos downloaded, installed, and configured automatically. Then cross-compilation could be done directly with the compiler or with dub using the appropriate target option. He said that was the main recent area of work on LDC.

Luís asked what kind of RSS reduction they had actually seen with the new GC. Martin said that as a general rule of thumb, it was around 50%, and in their tests they had seen fairly consistent reductions in the 50-60% range. In some cases, projects that had once peaked at more than 130 GB of RSS had dropped by about 60%. In a few workloads they had also seen runtime improvements, but he noted that performance varied more than memory usage did.

Luís said that although Weka mostly used fibers rather than many threads, the memory reduction alone would be extremely valuable to them. Some customers ran thousands of nodes, so lowering RSS per node was really important. They had management nodes that used gRPC and JSONValue, where some calls created a lot of GC allocations. The memory was probably very fragmented. He thought they could probably benefit quite a lot from the new collector.

He brought up another issue he'd forgotten about earlier, connected to Weka’s new build system. They generated header-only output from D files as a way of tracking dependencies, but `-deps` was crashing, so they could only use `-makedeps`. They needed a way to generate headers without also forcing code generation. Right now, even for header-only output, the compiler still had to go through code generation, and he thought there should be a simple flag to say "only generate headers". He believed Ilya had a proposal to implement that.

Martin said that at Symmetry they used Reggae for their builds, wrapping a very large collection of dub packages and projects. They relied on `-makedeps` automatically during normal compilation and fed that into Ninja, which then updated its own dependency database. He thought that was the preferable general direction because it gave them the dependency graph for free as part of normal code generation.

From his point of view, `-makedeps` should already be complete in terms of imports. Whatever file was imported as part of compilation should show up there, including DRuntime, Phobos, and imported C files. So he didn't really see a need to move from `-makedeps` to `-deps` in order to get a more complete graph. If `-deps` gave some additional information, that was fine, but `-makedeps` should at least already be complete.

Luís clarified that they did use `-makedeps` now, but had wanted `-deps` because it provided more information. What they were really after was the extra semantic information. The problem was that when they tried to use `-deps`, the compiler got into a loop and crashed.

He said they had also run into issues with generated headers mixing up public and private visibility, so things that were supposed to be public sometimes showed up as private in the generated header. He wasn't sure whether the right direction was to improve header generation or just avoid it and rely on `-makedeps`. He needed to understand the system better.

Carsten said that when they had used `-deps` or `-makedeps` at work, it had missed some imports. In particular, if there was a local import inside a function, it hadn't always picked that up. Martin said that if the compiler actually used that import during code generation, then it should show up. If it didn't, then the compiler had apparently decided it wasn't needed. Carsten said that sometimes they simply stopped using it and ended up making manual dependencies instead.

Luís said that one of the things Weka were doing wrong was how they compiled unit tests. Their unit test binaries were very large in part because when they compiled unit tests, they compiled the unit tests of all dependencies as well. That meant they were dragging in many unit tests that were never actually used. Part of the reason for that was that they used `version(unittest)` in ordinary code, which changed the ABI between unit test and non-unit test builds. He called that a really bad practice, but said getting people to stop doing it was a challenge.

### Walter

Walter said that signing the releases had been his main concern. His other big topic was AArch64, which was a long sequence of taking a few steps forward only to trip over the next thing. He said he had hoped he could get away without building a `memcpy` into the compiler, but it turned out he really did need one. So he had looked up the most efficient instructions on Godbolt and was in the middle of building that into the code generator. That was what he had been working on the night before until he got too tired and went to bed. His immediate plan was to finish that and then move on to whatever the next problem turned out to be.

Still, nothing insurmountable had appeared so far. The worst problem had been around fixups, because every memory model and every platform seemed to invent its own magical, undocumented scheme for fixups and thread-local storage. He was baffled by how completely different all of them were. Since there was no real documentation, what he ended up doing was writing code, compiling it, disassembling it, seeing which fixups were being used for which storage classes, and wiring support in by hand.

He said some TLS references went through three indirections, some through two, some through one, and there was no rhyme or reason to any of it. He grumbled in particular about macOS and AArch64 inventing their own completely different and undocumented schemes instead of following existing standards. He said varargs were also implemented differently on every platform, CPU, and memory model, and complained that macOS had even decided not to follow the standard for varargs, while also adding a large warning saying you could not just write standard C code and use their scheme.

In the end, he said it was a grinding process. There was really nothing anyone could do to help, and he just had to keep his head down and push through all the undocumented details one by one.

### Conclusion

Our next meeting was the monthly meeting the following week. The next quarterly meeting took place on January 9th, 2026.

If you are running or working for a business using D, large or small, and would like to join our quarterly meetings periodically or regularly to share your problems or experiences, please let me know.
