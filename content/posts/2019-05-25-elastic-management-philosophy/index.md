---
title: Elastic Management Philosophy
tags:
- management
aliases:
- /2019/05/25/elastic-management-philosophy.html
---
How to build a great engineering organization — advice from Elastic's SVP Engineering.

_**Update 2019-11-29:** Kevin published [Leadership @ Elastic](https://www.elastic.co/blog/leadership-elastic-kevin-kluge-distributed-for-the-better?blade=tw&hulk=social) blogpost. Kevin's post is more of high-level and strategical than the content here._

I've recently come back from Elastic's Global All Hands 2019 (or simply GAH 2019). It is a conference-style internal Elastic event. The goal is to facilitate cross-team communication, better understand the company strategy, and to simply chat with your colleagues. This post contains notes from Kevin Kluge's Management Philosophy session.


## Session on engineering management philosophy

One of the sessions I've really enjoyed was delivered by [Kevin Kluge](https://www.linkedin.com/in/kevin-kluge-715156/) --- our SVP Engineering. Working for Elastic is a great experience and I think Kevin's approach to engineering management and the culture he fosters is a big part of that. It is good to see his thoughts on that topic  distilled into a single session.

Disclaimer: these are my notes, I might have misunderstood Kevin or failed to capture some of his thoughts. Always think about your context and your challenges instead of blindly copying advice from the internet.

Without further ado, my notes from the session. Hope they are useful.


### Think in levels

*The "levels" here refer to job positions levels, e.g. Software Engineer --> Senior Software Engineer, etc. All job titles have assigned numerical levels; you can change your job track, e.g. from engineering to management independently of your level.*

- Start with job tracks.

- Know the levels of the folks in your team and what do different levels mean in terms of expectations. Also, know the requirements and responsibilities of the next levels, so that you can have conversations about their progression with the team.

- Don’t up-level people because of the salary, they may perform below expectations.

    *This is mostly in the context of new hires --- e.g. if someone expects a higher salary than Elastic can offer at their level. It is not good to artificially consider them a higher level, to be able to offer them the money. They may underperform and it's not a comfortable position to anyone. It is better to be honest and offer them a clear career progression. E.g we pay A on level X, but if you perform above this level, we will promote you to level X+1 and offer B in salary.*

- We don’t down level at Elastic.

    *Mostly around motivation and morale --- to avoid folks considering not doing their best work because in the worst case they can be down-leveled.*

- Hiring: don’t be too nice to recruiters --- don’t feel bad for them.

    *Don't accept the candidates you don't believe are a good match for the job opening. Push the recruiters to find better candidates. This is their job, and they are as passionate about challenging projects as engineers are.*


### Diversity

- Job ads and funnel analysis.

    *We try to write job ads using an inclusive language. No more rockstars in the ads ;). We also analyze our funnel to make sure that we don't decrease the diversity of the candidates as they progress through the hiring process.*

- Sourcing is the main focus right now.

    *Various efforts in HR to get a more diverse pipeline. Mostly about reaching out with our message.*


*Elastic is a distributed company and puts a lot of emphasis on inclusion and diversity. See our source code --- [As YOU, Are](https://www.elastic.co/about/our-source-code#as-you-are). In this part of the session, Kevin gave some metrics and a quick overview of the tasks that we as a company do to increase the diversity of the team. I personally feel this subject is really close to the heart of our HR team and especially to [Leah Sutton](https://www.linkedin.com/in/leahsutton/), VP Global HR at Elastic. She is always very passionate about it, and I see various efforts the HR undertakes to improve the diversity of the team.*


### Growth and development

- Give an opportunity to do some ${level}+1 work. Sometimes this may not work out. And that’s OK. Give an opportunity to roll back.

- Personal development template.

    *The members of your team should know what's needed to level-up (or change tracks); discuss it with them, plan for it, document it.*

- On people moving/switching teams.

    *If they are good, they'll be missed in their current team. Nevertheless, if they want to move you should support it. Because if you don't they may move to a different company. If Sally is great in team X, she is likely to be great at team Y. Note, however, that the prospective team should +1 the transfer and put her through a similar interview they put new hires through.*


### Feedback

- The sooner the better; positive is more effective, but still you have to give constructive feedback.

- Some openers for constructive feedback: “Did you consider ...”, “Have you thought about ... when making this decision”.

- Keep a google doc/file on team members you manage. Record interesting points from the 1:1s, good things they did over the year, etc.


### Fired

- Sometimes it just doesn’t work out. 

- Good interview question for prospective people managers: “Tell me about the time you’ve fired somebody” ("... or put on them on a performance plan ...").

- Firing someone is never easy, it hurts but you stand for good performance, the team culture, etc. 

- If you end up firing too many people this is a sign of problems somewhere else ==> you still may end up firing them, but you have to fix the underlying issue (poor recruitment, unclear responsibilities or requirements, etc.).


### Care for each other

- This is a marathon, not a sprint; Elastic wants to offer a good place to work for 10--15 years.

- Bad stuff will happen, even at a personal level. Don’t let folks burn out. 

- This is the most important part of the talk. 

- If someone is going through a tough time, do help them as much as you can. This includes letting them take a few weeks off to recover.

- If someone is offered help when they need it, they are likely to pay it forward. This creates a virtuous circle and allows for a great, inclusive and supportive teams.

*I can assure you this is not just a marketing BS. At least not from my perspective. I've seen that happen in Elastic. My colleagues and the HR and the management in Elastic try their best to care for their fellow Elasticians. I'm really happy to be a part of this.*


### Empowerment

- Make sure the decisions are well understood.

    *"I did this because Kevin said so" --- this is the worst, people need to understand and believe in **why** something needs to be done. Don't accept such justifications. As a side note, I think our CEO, Shay Banon, is an expert on that. Just tell him that we need to do something **because we are a public company now**.*

- Give ownership, empower people. Everything needs an owner. The expectations should be clearly defined. Ask a lot, empower and you get a lot. Sometimes you may ask too much, and you need to have plan B, but it helps people to grow and do amazing things.

- Don't manage for outliers. We are burned by this sometimes, but adding extra rules takes empowerment away. Why do people hate big companies? Often because of silly rules that feel disempowering. 

- Don't say "my engineers". This is disempowering. 

    *There are real people working on the team you manage!*.


### Organization

- Do you organize by geo/location or by product?

- Geo may work, but often the teams work on similar items, but they connect only on a VP level. This VP may manage a few thousands of employees. This makes it hard to resolve conflicts. Product teams are better. 

- Functional teams (like QA, etc.) often do not work --- different priorities between the teams.

- Deep or flat?

    *Kevin is a fan of flat. Maybe not completely flat, but as few levels as possible --- communications through levels is hard. Also, in a flat structure, it’s easier to deal with managers moving away from the org. He does skip level 1:1s from time to time.*

- Team lead, tech lead, product lead.


### Internal communications

*It was mentioned, but wasn't really part of the talk --- we have an internal communications charter in Elastic. The gist of it is that you should document decisions, discussions, etc. You should prefer async, like email or GitHub, and you shouldn't expect people to read the slack backlog or respond to messages right away. It is important as Elastic is distributed and has folks around the globe.*


### Do the right thing

- If you’re stuck with a difficult problem, think what’s the right/fair thing to do.

- If you break the rules because of that you will have a good explanation to persuade the other stakeholders.


## Summary

The key takeaways for me are these:

- Take care of the people you work with, take care of the team (this also means making the hard decisions when something doesn't work).
- Ownership and empowerment are super important.
- Do not manage for outliers, even if this will sometimes bite you.
