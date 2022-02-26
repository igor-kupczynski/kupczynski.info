---
title: The Slowest Hiker
tags:
- book-notes
- theory-of-constraints
aliases:
- /2019/02/23/slowest-hiker.html
---
How to improve any process.

One of the two [most often requested features of
Go](https://blog.golang.org/survey2017-results#TOC_3.) is the support for
generics. Arguably, generics are super important and they are taken for granted
in most of the modern programming languages. In fact, Go developers get a lot of
flak from their Haskell-minded colleagues being forced to use a language with
such a poor feature set. And yet, Go is one of the [fastest growing languages on
Github](https://octoverse.github.com/projects#languages). It seems like it
became the *lingua franca* of the cloud, with more and more distributed systems
being written (or rewritten) in Go.

How did that happen? Let's take a look at the intro to [go2 generics
proposal](https://go.googlesource.com/proposal/+/master/design/go2draft-generics-overview.md):

> The Go team (..) has been investigating and discussing possible designs for
> “generics” (...) since before Go’s first open source release. We understood
> (...) the topic was rich and complex and would take a long time to understand
> well enough to design a good solution.
>
> Instead of attempting that at the start, we spent our time on features more
> directly applicable to Go’s initial target of networked system software (now
> “cloud software”), such as concurrency, scalable builds, and low-latency
> garbage collection.

That's an application of the [*Theory of
constraints*](https://en.wikipedia.org/wiki/Theory_of_constraints).


# How to optimize any process?

1. Find out what the goal is.
2. Find out your constraints (aka blockers, or bottlenecks, or slowest hikers).
3. Improve them, one blocker at a time.

Improving constraints will improve the whole process.

How do you know if you've identified the constraint? If you speed it up, the
whole process will speed up; if you slow it down, the process will slow down.
Easy.

It is super important to eliminate the obstacles first, before attempting to
aggregate all the marginal gains. Sure, marginal gains aggregation works for
Team Sky, but then competitive cycling is already a heavily optimized process;
marginal gains maybe all that is left there.


## The slowest hiker

Theory of constraints was introduced in [*The
Goal*](https://en.wikipedia.org/wiki/The_Goal_(novel)). The book offers a story
of a plant manager, who was asked to supervise a boy scout team hike. The whole
team is supposed to be back at the campsite together before the sun sets.
However not all of the members hike with the same speed; as the result the group
splits, some folks need to wait and it looks like they are not going to make it.

The solution is to put the slowest hiker, *Herbie*, at the front of the group.
This illustrates that the group can go only as fast as their slowest hiker. Then
we can focus on *optimizing* Herbie. He has a heavy backpack, we can distribute
his load among the group. This speeds him up and the group makes it on time. The
book is from 1984 in the modern, cut-throat world of Jeff Bezoses, Herbie would
likely be fired ;)

### Five focusing steps

The book actually identifies these _five focusing steps_:

1. Identify the constraint
2. Exploit the constraint.
    * This is where you try quick improvements and workaround to improve the throughput of the constraint.
    * _Distribute Herbie's backpack content among the group._
3. Subordinate everything else to the above decision.
    * _Put Herbie at the front of the group, no one can overtake him._
4. Elevate the system constraint
    * Understand how much buffer we have in upstream and downstream systems and how much buffer is needed there so that the constraint can work at capacity all the time.
    * Consider elevating the constraint to a non-constraint (break the constraint). Watch out for an unbalanced system where the constraints keep moving.
5. Repeat. Go back to step 1 to hunt for new constraints


## What's the most important thing to do right now?

Try to think and question the assumptions to understand what the constraint is,
and how it can be eliminated. Ask *why?* until you find the root cause.

Focus on the most important thing right now --- take a few minutes every now and
then to think about the big picture and what is the biggest blocker to the
realization of your goals. Fix it, and move to the next one. Say "no" to things
which are not important right now (don't aggregate the marginal gains before the
obstacles are eliminated), in order to say "yes" to the most important problem
at a given time.


# Go's success

Theory of constraints may in part explain Go success. The team focused on
addressing [common software engineering challenges
first](https://research.swtch.com/vgo-eng). The result was language perfect for
codebases maintained by large teams over time. Its success in the opensource
projects speaks for that. When the slowest hikers are addressed the team has
enough of a breathing room to address next issues.

They've kept an open mind and said "no" to commonly requested features in order to
focus on constraints preventing them from reaching their goal.

Theory of constraints provides a useful mental model for tackling various
optimizations or delivery problems. It originated in the manufacturing space,
but this model is applicable in many other spaces. Especially in the iterative
world of software engineering --- figure out the goal, ask yourself whats the biggest
constraint right now is, improve and repeat.
