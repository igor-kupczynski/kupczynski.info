---
title: The Phoenix Project
tags:
- dev-ops
- book-notes
- theory-of-constraints
aliases:
- /2021/04/30/the-phoenix-project.html
---
Theory of constraints applied to IT operations.

![The Phoenix Project Logo](/archive/2021-04-phoenix.png)

I've read [*The Phoenix Project: A Novel about IT, DevOps, and Helping Your Business Win*](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592) by Gene Kim, Kevin Behr, George Spafford. This book is amazing. If you follow the lean manufacturing practices, or enjoyed _The Goal_ (see [The Slowest Hiker](/2019/02/23/slowest-hiker.html)), or simply want to improve the way IT operations work with the rest of the org (and improve the org in the process!) you'll enjoy the book.

The book presents a case for [DevOps](https://en.wikipedia.org/wiki/DevOps). It follows the story of Bill, recently promoted IT operations VP. His department is in dire straits -- missed deadlines, constant issues and firefighting, and a failed launch of the "phoenix project". The project is an important company initiative and the failure threatens to bring the entire company down.

Spoiler alert: Bill untangles this mess with a help of a mentor by applying manufacturing best practices, better understanding and cooperation with the business and with the development team.


# Notes from the book

## 10 releases per day

- Feature branches and long-lived projects are work in progress. Beware of them.
- Even if you don't need 10 releases per day, you need the options it gives you. 10 releases/day requires a fully automated pipeline, which is a good thing as it reduces errors. It also allows for taking calculated risks (as you can rollback/rollforward the bug, turn off the feature flag, etc.).

## Work in progress (WIP)
* A silent killer
* Easy to see on the factory floor; harder in the IT
* You accumulate capital in unfinished deliverables; it is tied in the system instead of paying dividends.
* Repeated actions — waste, as the earlier actions are discarded


## The theory of constrains

💡The throughput of the system is equal to the throughput on the constraint. All other improvements make no difference!💡 This is maybe the most important takeaway of this book and _The Goal_.

### Five focusing steps
1. Identify the constraint
2. Exploit the constraint
3. Subordinate everything else to the above decision
4. Elevate the system constraint
5. Repeat. Go back to step 1 to hunt for new constraints

See also [The Slowest Hiker](/2019/02/23/slowest-hiker.html#five-focusing-steps) for more details on the steps.

## Business and IT is a single org

- You need to understand the business. E.g. there may be a team in finance that reconciles accounts with transactions weekly -- maybe you don't need these fancy software security controls.
- Don't outsource critical components, the outsourcer may not be able to react to change fast enough.
- Give the business access to analytics so they can make better-informed decisions.
- Business is what drives the roadmap, mainly for the _business projects_, but does it make sense to spend time on internal initiatives which don't focus on the critical business needs?
- Different IT departments need to play in one team:
    - Security can't be an afterthought
    - Don't throw dev deliverables over the wall to ops, work with them
    - Ops need to make it easy for the dev to use the right environments, build pipelines, etc.

## The three ways:

The goal is to maximize the flow of the value stream. Value is what you deliver to customers.

This is grounded in systems thinking and the theory of constraints. Value is not created unless delivered to customers (remember about this when you start your next long-running branch!).

Example stream: `Business  -->  Dev  -->  Ops  -->  Customer`.

### The first way: Find and implement a way to improve/maximize the flow from left to right.
1. Limit WIP 
2. Visualize the work
2. Remove constraints

### The second way: Increase the feedback loop from right to left.
1. Failure signal
2. Visualize the waiting time (when something waits on resource availability)
    1. Visualize the work which needs to go backward.

### The third way: Continuous experimentation and learning, allowed by efficiency (the first way) and safety (the second way).
1. Learning from successes and failures
2. Taking (calculated) risks
3. Inject failure into the system to learn to deal with it and make it more resilient 
4. Improving the work is more important than doing the work (but then the yak shaving!!!)
5. Kata == repetition — allows building habits 
