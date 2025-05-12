[Back](/)

# Staff Engineer: Leadership Beyond the Management Track
Will Larson (2021)

---

## TL;DR
- Think hard about taking the step—it's a very different job, fedback cycles are long. "If you’re more focused on hitting staff than on setting yourself up to do work that energises you, it’s easy to end up stuck in a role you don’t want." (Michelle Bu)
- Go into meetings with a plan to quickly get alignment, and a reasonable answer; people should leave feeling good
- Align with authority
- Act like a founder
- Partner with executives
- Understand the points made by others quickly
- Become an engineer everyone wants to work with
- De-risk difficult problems

## Notes
### Staff Engineer Archetypes
- Tech lead
  - Works in one team (or more), reports to that team’s manager
  - Defines the technical vision
  - “Roughly one tech lead for every eight engineers…” 14
- Architect
  - Works in a particular, critical area; reports to a senior manager
  - Once a company has ~100 engineers
- Solver
  - Works wherever needed
  - This role develops later than tech lead
  - Often in companies that emphasise individual (not team) ownership 
- Right hand
  - “… extends an executive’s attention…” 13
  - Once a company has ~1,000 engineers

### Impact
> Being a Staff-engineer is not just a role. It’s the intersection of the role, your behaviours, your impact, and the organisation’s recognition of all those things. 14

> You’re far more likely to change your company’s long-term trajectory by growing the engineers around you than through personal heroics. 22

Feedback cycles are long, sometimes months. 

Focus on existential problems, but consider areas other than where everyone is focused: “those that matter to your company but still have enough room to actually do work.” 39

Think about what will be critical in the future, and where teams can really use your help. “In most companies, it’s glue work.” 40

You must be aligned with authority.

> Teaching a company to value something it doesn’t care about is the hardest sort of work you can do, and it often fails, so you should do as little of it as you can, but no less. 40

### Writing an Engineering Strategy
This is a tool for alignment.

Grab five design docs, pull out the similarties. For the vision, forecase the implication of those five docs years into the future.

Get to good, “rather than fixating on your own best as the relevant quality bar.” 46

Don’t wait for missing information. Even if you wait, you’ll have to make changes. Write in specifics, make it opinionated, provide context/reasoning.

It must be grounded in business/user needs.

### Random Stuff
> … remember that a good process is evolved rather than mandated. 55

> Effective data models are not even slightly clever. 57

> Some representative components to consider including in your quality definition:
>
> - What percentage of the code is statically typed?
> - How many files have associated tests?
> - What is test coverage within your codebase?
> - How narrow are the public interfaces across modules?
> - What percentage of files use the preferred HTTP library?
> - Do endpoints respond to requests within 500ms after a cold start?
> - How many functions have dangerous read-after-write behavior? Or perform unnecessary reads against the primary database instance?
> - How many endpoints perform all state mutation within a single transaction?
> - How many functions acquire low-level granularity locks?
> - How many hot fixes exist which are changed in more than half of pull requests? 61

In more junior roles, “you may become comfortable with your manager guiding your development and providing a safety net for your continued success. After reaching a staff role, your safety net will cease to exist…” 71

> Part of growing as a leader is developing your own perspective on how the world should work… Having a clear sense of how things ought to work sharpens your judgment and enables you to act proactively. 73

> The most effective engineers go into each meeting with the goal of agreeing on the problem at hand, understanding the needs and perspectives within the time, and identifying what needs to happen to align on an approach. 79

> In a potentially contentious meeting, ask good questions before you share your perspective, and you’ll see the room shifts around you. 79

Make it so the team doesn’t rely on you.

Good discussions are quick, get a reasonable answer and alignment; people should leave with positive feelings.

### Feedback
“If a piece of feedback won’t meaningfully change a project’s success, then consider not giving it.” 85 This creates “insecurity over impact and prevent[s] others from growing as leaders.” 85

Encourage, help, suggest, guide others to do the work you do. “When critical work comes to you, your first question should become, ‘Who could be both successful with and grown by this work?’” 86

> Counsel, give advice, provide context, but ultimately sponsorship includes letting them take an approach that you wouldn’t. 86

☝️ Even if it goes poorly.

Ritu Vincent,

> I have a decent number of recurring monthly lunches, coffee chats, and dinners with people who’ve worked with me in the past, know me, and I trust. It’s those conversations about care challenges and growth that have gotten me to where I am in my career. 87

### Presenting to Executives
Executive presentations, when a big decision is on the line, require preparation. Some executives will want data, some will want to pattern match against some previous experience, etc.

Typically these presentations are either about planning, reporting or resolving a misalignment. “… Your goal is always to extract as much perspective from the executive as possible. If you go into the meeting to change their mind, you’ll probably come across as inflexible. Go into the meeting to understand how you can align with their priorities.” 95-6

📘 The Pyramid Principle (Barbara Minto)

Use SCQA to structure the presentation:

- Situation: context
- Complication: why is there a problem?
- Question: the core question to address
- Answer: your best answer

Align beforehand (_nemawashi_ 🇯🇵). Aim for discussion, not getting through all the talking points; gather feedback, don’t worry if you don’t agree. “You will create more credibility by agreeing with their perspective and following up with more data later.” 99

If you don’t come with solutions, “in the back of their mind, they’re wondering if they need to hire a more senior leader to supplement or replace you.” 99

> Don’t fixate on your preferred outcome… almost every decision will be reconsidered multiple times over the next two years. 99

### Becoming a Staff Engineer
> How do you become visible internally without hogging all the oxygen? 102

Write your own staff promotion packet to focus your attention and review it with your manager:

- What are your staff projects? What was your contribution? What was the impact? What made it challenging?
  - These kinds of projects are complex, ambiguous, numerous and divided stakeholders, closely watched by leadership
  - This isn’t always a prerequisite. Depending on the type of staff engineer, running complex projects may not even be their focus
- How have you improved the organisation?
- What is the dollar impact of your projects?
- Who have you mentored?
- What glue work did you do?
- Which teams and leaders advocate for your work?
- What are your gaps and how are you addressing them?

### More Random Stuff
> Effective groups are formed from individuals who are willing to disagree and commit. 125

<!--
I feel like there’s a lot of good advice, but it probably didn’t warrant a whole book. E.g., again here, we’re back to what to do but from the lens of making your contributions visible:

- Work on what matters to leadership (this sounds manipulative; if something important isn’t a focus of leadership you should being this up)
- Publish long-lived documents (architecture,etc)
- Lead architecture reviews, company all hands, learning circles
- Cheerlead your team’s and peer’s work
- Share team progress with stakeholders
- Contribute to the company blog
- Host team office hours
-->

Good interview questions for figuring out company culture—clique-based (“meritocratic”) culture, vs consistency drives fairness:

- Do they have rigid compensation bands they stick to? (Consistency)
- Do they create one-off roles? (Meritocratic)
- Is the interview unstructured? (Meritocratic)

🌐 [Salary Negotiation: Make More Money, Be More Valued](https://www.kalzumeus.com/2012/01/23/salary-negotiation/)

### Stories
**Michelle Bu** publishes a doc explaining what she’s working on at all times. She documents the scope and impact of her work.

> … pursue work that you find energising, even if on paper it doesn’t seem like it’d get you to a staff-plus. 165

**Ras Kasa Williams** says “it’s about thinking globally… understand the company-wide business/product strategy and distill that into engineering-wider technical strategy that seeks to enable execution for Product, Marketing, and our peers across other functions.” 168

Consider writing your performance review in the third person, makes it easier to be less critical.

To help think through difficult problems, write it all out, even if you don’t share it.

For going better at speaking, watch 🎬 Kelsey Hightower.

🌐 [Delivering on an Architecture Strategy](https://blog.thepete.net/blog/2019/12/09/delivering-on-an-architecture-strategy/)

🌐 [Stepping Stones not Milestones](https://medium.com/@jamesacowling/stepping-stones-not-milestones-e6be0073563f)

**Bert Fan**: have 1-2-1s with engineers and listen.

Develop implicit trust with your manager. “Be honest and direct about what you want.” 187

Become an engineer everyone wants to work with, supportive, strong technically etc.

**Katie Sylor-Miller**: “I think a lot of what gets someone to staff is noticing problems and acting on solving them proactively, instead of letting them go.” 198

> You have to be ready to kill your darlings, pivot, and try something new. If an approach you’re taking doesn’t work, don’t try to force it. 202

**Ritu Vincent**: “In addition to the title, the other thing that gives me confidence was realising that everyone else is also struggling with imposter syndrome. The latter I leaned in a pivotal conversation with someone who I thought was the most confident engineer I’d ever worked with, and when I talked with him about it he said, “I question every single thing I do. I go home and agonise over what I said earlier that day, and whether it was silly.” 207

Your manager is on your team. They want to help. Be totally honest about problems, aspirations, etc. “They’re looking for a way to make you grow, be productive, be happy and become the best engineer you can be.” 210

**Rick Boone**: “Infrastructure Engineering at Uber is about 700 people across six sub-organisations…” 212 😮

About his manager:

> … I go into a room and think, “What would Matthew do here?” What is the question he would want to ask? What guidance has he given on this problem?” … People need to be confident that I’ll always give the same answer that Matthew would give if he were there. 214-5

Give advice/feedback from the perspective of different roles:

> If I meet with someone I’m mentoring, they might want to get advice about changing teams, or even leaving the organisation or company, and they want to know which perspective I’m giving advice from. 215

**Nelson Elhage**: “… I spent a lot of effort as the company grew trying to stay aware of everything that was going on in engineering: the interactions between teams, the scaling points… That helped me know which problems we’re important to work on… If I knew the organisation had a goal of launching a specific product, I would have the perspective to see the reason why it would be hard…” 227-8

> I find it really valuable to be able to look at a system and have the habit of estimating how many gigabytes-per-second is this thing, or how much storage would this data take? 231

**Diana Pojar**: “Folks that focus on breadth usually work more closely with the leadership team…” 234

**Dan Na** recommends 📘 High Output Management (Andy Grove) and 📘 Resilient Management (Lara Hogan), for “unpacking and addressing the hardest parts about navigating emotions and personalities, fostering psychological safety, and sponsoring coworkers.” 248

**Joy Ebertz**: “I think the biggest thing that differs between now and then when I was more junior is my sense of ownership and responsibility… when I was more junior, I would often just assume that something was someone else’s problem. Now it’s all my problem.” 250

**Damian Schenkelman** says that 📘 Fundamentals of Software Architecture “does a fairly good job at describing that role [senior engineer] while also understanding the nuances and grey areas of it.” 271

**Dimitry Petrashko**: “A staff engineer at Stripe isn’t a role, rather it’s a level that corresponds to expectation of impact, communication, people and project leadership skills.” 272

> I feel particularly impactful when I can help improve a proposal that’s well-intentioned and solves a real need, but the team that drafted it lacks either experience or context to write a good plan to capture the opportunity. In such cases, having a well-structured plan can help substantially reduce the scope while getting to most of the value, and thus demonstrate impact sooner. Or, alternatively, see that the proposal in hand addresses more use cases than the team has originally anticipated and refocusing the project towards a use case that was not known by the team would lead to bigger business impact: in both of these cases, I feel impactful by empowering other engineers. 274

> … I advocate for changes that will have outsized impact… 275

Says he would ask for feedback in private chats immediately after meetings, “in particular after meetings that didn’t go perfectly.” 278

**Stephen Wan**: “Get comfortable talking a lot.” 290

“Trust folks, flag issues, and expect them to work it out.” 291

### Resources
Technical specs
- [A practical guide to writing technical specs](https://stackoverflow.blog/2020/04/06/a-practical-guide-to-writing-technical-specs/)
- [Design Docs at Google](https://www.industrialempathy.com/posts/design-docs-at-google/)
- [Design Docs, Markdown, and Git](https://www.caitiem.com/2020/03/29/design-docs-markdown-and-git/)
- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [How to write a great technical design document](https://www.range.co/blog/better-tech-specs)
- [Technical Decision-Making and Alignment in a Remote Culture](https://vuink.com/post/zhygvguernqrq-d-dfgvgpusvk-d-dpbz/blog/2020/12/07/remote-decision-making)
- [Writing Technical Design Docs](https://medium.com/machine-words/writing-technical-design-docs-71f446e42f2e)

Engineering strategies
- [Write Five, Then Synthesize](https://lethain.com/good-engineering-strategy-is-boring/)
- [A Framework For Responsible Innovation](https://multithreaded.stitchfix.com/blog/2019/08/19/framework-for-responsible-innovation/)
- [How Big Technical Changes Happen at Slack](https://slack.engineering/how-big-technical-changes-happen-at-slack-f1569d25ee7b)
- [On Drafting an Engineering Strategy](https://www.paperplanes.de/2020/1/31/on-drafting-an-engineering-strategy.html)
- [Defining a Tech Strategy](https://sarahtaraporewalla.com/agile/design/architecture/Defining-a-Tech-Strategy)
- [Delivering on an Architecture Strategy](https://blog.thepete.net/blog/2019/12/09/delivering-on-an-architecture-strategy/)
- [Stepping Stones not Milestones](https://medium.com/@jamesacowling/stepping-stones-not-milestones-e6be0073563f)
- [Achieving Alignment and Efficiency Through a Technical Strategy](https://yenkel.dev/posts/achieving-alignment-and-efficiency-through-a-technical-strategy)
- [The Difficult Teenage Years: Setting Tech Strategy After a Launch](https://medium.com/ft-product-technology/the-difficult-teenage-years-setting-tech-strategy-after-a-launch-7f42eb94a424)
- [Learning to Have an Engineering Vision](https://unwiredcouch.com/2018/01/03/engineering-vision.html)
- [Run Less Software (Rich Archibold)](https://medium.com/@rich_archbold/run-less-software-23eaeabbd2b6)
- [Product Strategy](https://svpg.com/product-strategy-overview/)

Papers
- [Dynamo: Amazon’s Highly Available Key-value Store](https://s3.amazonaws.com/systemsandpapers/papers/amazon-dynamo-sosp2007.pdf)
- [On Designing and Deploying Internet-Scale Services](https://s3.amazonaws.com/systemsandpapers/papers/hamilton.pdf)
- [No Silver Bullet - Essence and Accident in Software Engineering](https://s3.amazonaws.com/systemsandpapers/papers/Frederick_Brooks_87-No_Silver_Bullet_Essence_and_Accidents_of_Software_Engineering.pdf)
- [Out of the Tar Pit](https://s3.amazonaws.com/systemsandpapers/papers/outofthetarpit.pdf)
- [The Chubby lock service for loosely-coupled distributed systems](https://s3.amazonaws.com/systemsandpapers/papers/chubby-osdi06.pdf)
- [Bigtable: A Distributed Storage System for Structured Data](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)
- [Raft: In Search of an Understandable Consensus Algorithm](https://s3.amazonaws.com/systemsandpapers/papers/raft.pdf)
- [Paxos Made Simple](https://s3.amazonaws.com/systemsandpapers/papers/paxos-made-simple.pdf)
- [SWIM: Scalable Weakly-consistent Infection-style Process Group Membership Protocol](https://s3.amazonaws.com/systemsandpapers/papers/swim.pdf)
- [Hints for Computer System Design](https://s3.amazonaws.com/systemsandpapers/papers/acrobat-17.pdf)
- [Big Ball of Mud](https://s3.amazonaws.com/systemsandpapers/papers/bigballofmud.pdf)
- [The Google File System](https://s3.amazonaws.com/systemsandpapers/papers/gfs.pdf)
- [CAP Twelve Years Later: How the Rules Have Changed](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed)
- [Harvest, Yield, and Scalable Tolerant Systems](https://s3.amazonaws.com/systemsandpapers/papers/FOX_Brewer_99-Harvest_Yield_and_Scalable_Tolerant_Systems.pdf)
- [MapReduce: Simplified Data Processing on Large Clusters](https://s3.amazonaws.com/systemsandpapers/papers/mapreduce.pdf)
- [Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](https://s3.amazonaws.com/systemsandpapers/papers/dapper.pdf)
- [Kafka: a Distributed Messaging System for Log Processing](https://s3.amazonaws.com/systemsandpapers/papers/Kafka.pdf)
- [Large-scale cluster management at Google with Borg](https://s3.amazonaws.com/systemsandpapers/papers/borg.pdf)
- [Mesos: A Platform for Fine-Grained Resource Sharing in the Data Cente](https://s3.amazonaws.com/systemsandpapers/papers/mesos.pdf)

See also [Papers We Love](https://paperswelove.org/).

### Interviewing Staff Candidates
> - **Self-awareness**. Are they accountable for mistakes? Have they demonstrated growth in areas where they’ve previously been weaker?
> - **Judgment**. Are they able to see around corners to identify problems? Are they able to navigate broad, ambiguous problems? Can they effectively mediate between folks in an argument about trade-offs or design? Can they de-risk the execution of difficult problems?
> - **Collaboration**. Do they partner well with others? What about folks less experienced than them? More experienced than them? Their managers? Cross-functionality? Executives?
> - **Communication**. Are they good listeners who understand the points made by others? Are they able to communicate their ideas clearly? Can they communicate in the formats that your company relies upon (written, verbal, etc)?
> - **Development**. Do they grow others around them? Does the “organisational bench” grow in areas they lead or atrophy? Do broken systems and processes get cleaned up? 314-5
