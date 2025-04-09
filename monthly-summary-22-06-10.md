[Forum Post](https://forum.dlang.org/post/buythmwbdaswuqbtkddv@forum.dlang.org)

The monthly meeting for June 2022 took place on June 10. The following Foundation staff and D contributors attended:

* Walter Bright
* Iain Buclaw
* Ali Çehreli
* Martin Kinkelin
* Dennis Korpel
* Mathias Lang
* Átila Neves
* Razvan Nitu
* Mike Parker
* Robert Schadek

The purpose of this meeting was to revise the latest draft of the Vision Document. I don't have a whole lot to summarize, as the document itself will reflect what we discussed, but I will provide some background for those who haven't been following along.

## Background
We first started discussing a new vision document last summer. This was a direct response to people bringing up "a lack of vision" in the forums. It's one of those things all of us would like to have, but there's always something that takes higher priority. So I put it on a meeting agenda so that we could get something started.

For a little while, Andrei used to post a kind of vision document to the D Wiki twice a year. It included a list of items showing some of the progress made in the previous six months, and a list of goals to accomplish for the next six months. I thought we should do something different, as that's more of a task list than a vision.

My initial concept was a living document describing what Walter and Átila were each focused on, followed by a list of things they intended to do in the future, and a list of things that would be nice to do in the future, resources permitting. These lists would be accompanied by potential tasks contributors could choose to take on in order to make the most impact.

I asked Walter and Átila to send me a write up of the things they were focused on. I would take that, write up the rest of the document, and then we could revise it and release it.

It's an understatement to say we took our time. The vision document was low on the priority list for each of us, and there was always a higher priority task to get done. That said, it was still on my mind. And I began to reconsider the format.

I realized that the format we had was just an evolution of Andrei's version of the vision document. There was no "high-level" vision, no overarching goals. So at our monthly meeting in February, we scheduled a separate meeting to focus on what I referred to as a "vision statement" (you can read about it [in the summary of the February meeting](https://forum.dlang.org/post/qrtjqubrbuqeiffunili@forum.dlang.org)).

We brainstormed a number of goals, some high-level, some low. My task after the meeting was to take this list of ideas and craft it into a coherent vision statement. I would put this statement (a paragraph or two) at the top of the document, and follow that up with three sections: Current Focus, Future Focus, and Wishlist.

Eventually, I decided that version of the document wasn't going to cut it either. We need more than just one or two paragraphs stating high-level goals. We need a list of high-level goals and details describing what they mean. [So at our May 2022 meeting](https://forum.dlang.org/post/uhgndrcnekedjqtarnwl@forum.dlang.org), I proposed that our next monthly meeting be devoted to finalizing that list of high-level goals.

## The June meeting
Before the meeting, I had to come up with the initial list. I took the brainstormed ideas from the vision meeting, sorted them into categories of high-level goals, and fleshed them out with definitions and examples.

At the meeting, everyone provided feedback on the language, the level of detail, and the existing categories. We added a new category, cut some things, added some information in existing categories, and had good discussions about the language, the standard library, the ecosystem, and the community.

My task after the meeting was to make all the suggested revisions. I expect to finish that this weekend. I'll then submit it to the others for a final round of feedback, and I'll ask them to make it a top priority so that it doesn't fall out of sight. I expect we'll have this portion of the document finalized some time next week.

The next step is to add the sections for the Current Focus, Future Focus, and the Wishlist. I've also decided to add a Contributing section, listing specific tasks contributors can carry out, rather than including them in the other sections.

## Other notes
I want to make it clear here that none of us involved in these meetings or in the core development and maintenance of D are oblivious to the complaints about lack of vision or bad management. All of us want to do what we can to turn things around. People will always find something to complain about, but we all want to see the day where there are no grounds to complain about a lack of vision or management.

I'll be talking a little about this in my DConf talk, but we are actively looking at ways to improve the situation. We want to start by focusing on the community and their contributors. As a few examples of what I mean, we want to:

* improve the response time when services fail (e.g., code.dlang.org or forum outages)
* enhance the new user experience
* ensure that everyone can easily learn who is responsible for what
* make contributing less daunting
* create the conditions for contributors to feel their efforts are worthwhile
* establish a framework that allows contributors who require or prefer guidance to know where to direct their efforts and, critically, provides a designated point of contact for follow up

The PR and Issue Manager positions were an early step in that direction. The ongoing plan for the foundation to take over responsibility for maintaining services in the ecosystem is another step in that direction. Our future plan to establish an ecosystem management team will be the next step in that direction.

Getting to where we want to be will take time, and time is a resource none of us have a lot of. The further along we get with things, the more resources we'll have to take us even further.

Almost all of the real work is happening on the shoulders of volunteers. Vladimir Panteleev and Petar Kirov are working out the action plan for the server migration and will subsequently be involved in admin operations, all on their own time.

The members of the ecosystem management team will be volunteers, people who will donate their free time to implement task lists in line with the vision document, find volunteers to carry out those tasks, and provide the support and follow up needed to see the tasks to completion.

There are very few people getting paid for the work going on here. Razvan and Dennis are getting $25,000 a year each for their positions as PR Managers, funded by Symmetry. I get $1200 per month, and Max $1000 per month, directly from the D Foundation's general fund. Beyond that, we pay bounties for blog posts (and in the future, YouTube content), fund the occasional contract, and have provided scholarships for students who focus on D projects for their academic work. I say this because right now we are at the limit of what can be funded for regular pay. There is room for a few contracts still in the work fund, and we continue to pay bounties, buy contributor rewards, etc, but there's nothing available for adding more regular positions.

Everything that exists in the D ecosystem was built by volunteers. In the absence of more funding, that's how it continues to be. It all requires volunteers to maintain it. And given how much we've grown, it requires volunteers to manage it.

Some people may not be happy with the vision document once it's published. I would love it if everyone is, but I expect some people will find it lacking. That's one of the reasons it's intended to be a living document. As time goes by, we'll revise it as necessary. That includes not just changing the current focus, or future focus, or adding and removing goals, but adapting it to meet the needs of our contributors.

The vision document is not a roadmap with designated stops along the way. It's not a checklist of milestones. Our intention is that it serve as a general guide for self-motivated contributors to decide where to direct their efforts, and for volunteer managers to draw up more detailed task lists for those contributors who need a little more guidance.

I'm sure someone will ask when it will be published. I'm still holding to my promise that we'll get it done before DConf. My original plan was to wait until the entire document was finalized, but I now intend to publish the high-level portion once the final draft is approved, then add the rest to it at some point in July.

In the meantime, I'd like to remind everyone of things you can do right now to contribute:

* notify Razvan or Dennis that you are available to fix bugs and what kind (beginner level, compiler, Phobos, etc)---though Razvan is on vacation right now, so you're probably better off contacting only Dennis until after July 1
* make a forum post about any old Bugzilla issues that are still bothering you, or tweet about them with #DBugfix
* contribute to a D project that you use, or would like to see succeed, by reporting and/or fixing bugs
* contribute a blog post to the D Blog
* write a post on your own blog and let me know about it so that I can share it
* post regular update tweets about your current D project with #dlang, or share it on /r/programming if it's in a usable state

At the very least, you can always contact me for ideas on how to contribute (aldacron@gmail.com). I'll be happy to throw some at you.

The theme of my DConf talk is "it takes a village", because it really does. We're working to upgrade our village to a township, but we can't do it without help. We're always thankful for everyone who contributes in ways large and small, and we hope that comes across more clearly as time goes by.

I ask your patience as we continue to take our baby steps toward the future.