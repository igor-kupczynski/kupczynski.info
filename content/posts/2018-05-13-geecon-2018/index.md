---
title: Notes from Geecon 2018
tags: []
aliases:
- /2018/05/13/geecon-2018.html
---
Conference notes.

[Geecon 2018](https://2018.geecon.com) was an important event in Center- and
Eastern- European JVM conference agenda. It was over a year since I've attended
my last non-elastic related event so I thought it is a great venue to pick up
some ideas and look where the industry is heading nowadays. And what a great
venue it was!

In this pots I'll present mostly raw notes I've took during the talks. If some
of the ideas stick with me, I may explore and refine them later on. Oh, there
where 4 concurrent talks at most of the slots, so I can cover only a subset. So
the notes are uncomplete and **may** be misleading. Proceed at your own risk :)

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [GeeCON #10](#geecon-10)
    - [Booths](#booths)
- [Day 1](#day-1)
    - [Innovate or Die ... or Don't](#innovate-or-die--or-dont)
    - [A War of Words: Self-Awareness for Introverts](#a-war-of-words-self-awareness-for-introverts)
    - [Creating Secure Software: Benefits from Cloud Thinking](#creating-secure-software-benefits-from-cloud-thinking)
    - [What the annotations done to us?](#what-the-annotations-done-to-us)
    - [gRPC vs REST](#grpc-vs-rest)
    - [Starting with Ethereum, a developer approach](#starting-with-ethereum-a-developer-approach)
    - [Fn Project — an open source container native FaaS platform](#fn-project--an-open-source-container-native-faas-platform)
    - [Kotlin coroutines](#kotlin-coroutines)
    - [Conference party](#conference-party)
- [Day 2](#day-2)
    - [Demystifying Java 9 features](#demystifying-java-9-features)
    - [The future of Java](#the-future-of-java)
    - [A crash course in modern hardware](#a-crash-course-in-modern-hardware)
    - [Modern SQL: evolution of a dinosaur](#modern-sql-evolution-of-a-dinosaur)
    - [Engineering architecture](#engineering-architecture)
    - [Using an open source library? Beware of vulnerabilities!](#using-an-open-source-library-beware-of-vulnerabilities)
    - [Nothing but Models: Quest for the Roots of Complexity](#nothing-but-models-quest-for-the-roots-of-complexity)
    - [Manage Yourself!](#manage-yourself)
- [Day 3](#day-3)
    - [Deep dive into the Eclipse OpenJ9 GC technologies](#deep-dive-into-the-eclipse-openj9-gc-technologies)
    - [Class Data Sharing](#class-data-sharing)
    - [Java Memory Model Unlearning Experience](#java-memory-model-unlearning-experience)
    - [Blockchain more than Bitcoin](#blockchain-more-than-bitcoin)
    - [NoSQL Means No Security?](#nosql-means-no-security)
    - [QWERTY or DVORAK? Debunking the Keyboard Layout Myths](#qwerty-or-dvorak-debunking-the-keyboard-layout-myths)
    - [Using Reactive APIs](#using-reactive-apis)
- [Conclusions](#conclusions)

<!-- markdown-toc end -->



## GeeCON #10

It was a tenth, anniversary GeeCON. Hence the cake:

![Cake](/archive/2018-05-geecon-cake.jpg)

I think it was a great event, with lot of interesting talks and good speakers. I
had a very good time and feel it was three days well spend. Props for the
organizing committee for all their hard work on GeeCON over the years.

### Booths

Well, I've attended most of the sessions, so I didn't really have the time to
visit and chat with majority of the folks there, but a honorable mention goes to
[zooplus](http://www.zooplus.com/). They are main pet-product supplier for some
of my friends and I was surprised to learn that then have an office in Krakow
with few dozens of software engineers :).

A lot of big financial institutions were present, it's good to see they base a
sizable chunk of their IT operations in Poland. It is a tendency that we've seen
for quite some time, but looks like with Brexit it is not slowing down.

[Allegro](https://allegro.pl) which is headquarterd in Poznań, my home town, and
has offices in 6 or 6 major Polish cities was quite strong.

Scala-wizards from [Virtuuslab](https://virtuslab.com) where also present, hence
the nice stickers.

![scala animals](/archive/2018-05-scala-animal.jpg)

## Day 1

### Innovate or Die ... or Don't

Main sponsor --- EY --- keynote.

### A War of Words: Self-Awareness for Introverts

Second keynote.

A strange talk, coming from Cliff Click, based on his realization that he
can't process emotions and talk at the same time --- find the right words at the
same time. The talk gave some examples of emotional responses of an introvert
during various stressful situations, like being verbally assaulted or ... salary
negotiations.


- Story about emotions when someone shouts at you
- Can't do verbal and emotional processing at the same time
    - I don't have any words until I've processed the emotions
    - Solution: memorize beforehand
        - I need space. I'm leaving. and leave
        - Find some peace
    - Aftermath Stress doesn't go away so I want to attack someone (dog, friend,
      someone safe to attack)
        - solution: physical activity to let the adrenaline bun out
    - kept what happened: a fool attacked me; move on; what can I learn?
        - talk to his boss
        - talk to your fiends (or whatever makes you sort it our)
- More stories
    - Brainstorm: ack the loud persons ("Tom, I've heard youve suggested XYZ"),
      make space for the quite folks ("Anyone else wants to chime in?")
- Aspects
    - Farmer - enjoys hard work ("today I built a house")
    - Wizard - "smart work" ("got that guy working today")
    - Worrier - enjoys fight and its spoils (salesmen, ceos, layers)
    - Know your and other strengths, compensate for the weakness
    - e.g. programmers / introverts are usually not warriors
    - ppl labels
        - green (safe to be around)
        - red (unsafe; unreasonable demands); not "bad in general", but "bad for
          me"
        - yellow :: folks in the middle
- change comes from within (even if I love someone I can't change them; and vice
  versa)
    - you can't read someone else mind => broken expectations, speak up maybe
      they will change
- salary negations
    - HR job is to get your skills at a good price, no emotions, no hard feeling
    - but for you it is your level of life
    - ack it is a fight / conflict; polite, but still a fight
    - prepare for the fight :: preparation
        - know thy worth (glassdoor, books on negotiation and interviewing)
        - BATNA
    - train the warrior muscle :: practice
            - get the partner to play the HR door (power seat, office)
            - play out the negotiations
            - ack your emotions when you get the low-ball
            - you don't need to ans "how much do you make", you can say "I look
              for XYZ, I contribute ABC, etc.")


### Creating Secure Software: Benefits from Cloud Thinking

Nice talk from Daniel Sawano. He preached the concept of rotating secrets often,
and reiterated some important concepts in cloud apps security.

- Cloud
    - 12th factor app
    - cloud-native (designed to be run on pass)
- Configuration
    - store the configuration in the environment
        - as opposed to in code (no audit trail, no idea who has an access to
          it)
        - not in resource files (same problems as with code; even if it lives
          only on server; encrypting the files create new set of problems —
          where to put the encryption key)
        - the platform creates the env and injects the credentials to the env
            - audit trail managed by the platform
            - no sharing secrets (only sys-admins see the secrets not the devs)
            - encryption is still not solved
- Interlude: Confidentiality Integrity Availability --- *CIA*
- Separate processes
    - Run your app as a separate stateless processes
        - stateless processes
            - availability (easy to add / remove instances)
            - integrity (easy to kill misbehaving instances)
        - separate deployment and running
            - so you can run processes with low privilege user
            - principle of least privilege
        - communicate via backend services
            - because app can be stateless (the state is in the backend service)
            - stateless leads to, again, A and I
        - administrative tasks
            - instead of terminal or sql access create admini processes or apis
- Logging
    - Use logging as a service
        - logging to disk — challenges
            - may contain sensitive information
            - access not controlled, no audit trail, possible illegal access
            - similar problems as with resource files
            - I — how do we know if the log file was not changed or deleted
            - A — if the appp is replaced the logs are lost
        - logging as a service
            - send the logs straight to the service that accepts the log events
            - the logging framework should support it (don't write a new one)
            - Solves CIA
- The three R's of enterprise sec
    - Rotate
        - the secrets every few minutes or hours
    - Repave
        - servers and apps every few hours
        - from a known good state (base image, container, etc.)
        - rolling deployments to eliminate downtime (start new, kill old)
        - burn old instances
        - repave the host as well
    - Repair
        - few hours after a patch is available
        - no incremental updates
        - applies to both os and apps, dependencies
    - Diff from the classic deployment: to minimize the risk you increase the
      change
        - the system that stays static for a long of time is a good target
        - ever changing software is the nemesis of the persistent threats
- Solutions:
    - Valut
- Book
    - bit.ly/secure-by-design

Comments:

- Basics/core but I liked it
- I should watch the talk from previous year
- good pace
- clear structure

### What the annotations done to us?

This talk by Adam Warski from software mill was dear to my heart. How the
modern frameworks abuse annotations and offered some alternatives, especially in
the java 8+ world.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/adamwarski?ref_src=twsrc%5Etfw">@adamwarski</a> reveals at <a href="https://twitter.com/hashtag/geecon?src=hash&amp;ref_src=twsrc%5Etfw">#geecon</a> that annotations are not only ugly. They were also tested on animals! <a href="https://twitter.com/hashtag/petclinic?src=hash&amp;ref_src=twsrc%5Etfw">#petclinic</a> <a href="https://twitter.com/hashtag/spring?src=hash&amp;ref_src=twsrc%5Etfw">#spring</a> <a href="https://t.co/XiY4vctGwd">pic.twitter.com/XiY4vctGwd</a></p>&mdash; Jarek Ratajski (@jarek000000) <a href="https://twitter.com/jarek000000/status/994177020119670784?ref_src=twsrc%5Etfw">May 9, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

- successful java feature
- transformed java programming in a positive way
- but don't we go to far?
- history
    - intro 2004
    - replaced xml
- why?
    - express metadata
        - classes, methods fields description @Entity
        - expressing cross cutting concerns  @Transactional
        - orchestrated thee application @Inject
    - easy to introduce
    - separation (@, coloring)
    - close to the referenced elements
    - inspectable statically and at run time
- *popular is not always best*
- Adam claims that in java <= 7 world @@@ are the local optimum — better than
  xml or dynamic typing; maybe there is something even better in java 8+?
- @@@ - is an embedded mini language, interpreted at runtime
    - can't be mixed with the language (e.g. no for loops, et.c)
    - we end up programming the annotation interpreters with annotations
- Patterns that emerged
    - fear of new ... ()
        - DI, we delegate instance creation to the containers
        - alt: manual dependency injections (new, per-package scope), also see
          [Jakub Nabrdalik talk on hexagonal architecture](https://www.youtube.com/watch?v=ma15iBQpmHU)
    - fear of public static void main()
    - class path scanning
        - the container picks stuff automatically, e.g. mongo integration;
          convenient for rapid bootstrapping at the convenience of the reader
        - trade certainty & control for fast and convent bootstrap
    - explorability is important; @annotations cloud it
        - code should be easy to navigate
        - go to definition
        - understand what services are used, whats the ordering, etc.
    - metadata mapping
        - entities, json, endpoints
        - maps java model to some external thingy
        - alt (picture)
- just use java
    - metadata is first class
    - single language for code and meta-data
    - can be generated programmatically
    - annotationmania.com Thanks to @Annotations, @Progress in @Unstoppable
    - Jarek Ratajski on the same subject
- Are ppl actually doing it?
    - sparkjava.com
    - jooq.org
    - functional web framework in spring 5
    - langs
        - scala
            - macros && implicits
        - ceylon
            - type-safe metamodel
            
### gRPC vs REST

Two software engineers:

- Alex Borysov from Google
- Mykyta Protsenko from Roku

Nicely paced talk, one of the speakers acted as a gRPC fanboy and the other as
someone who is satisfied with REST. They took turns speaking and illustrated the
talk with an elaborate [demo](https://github.com/grpcvsrest). I enjoyed that one.

- REST
    - not json over http
    - state transfer (client has the state, server is stateless)
- gRPC
    - rpc framework
    - https2 is transport protocol
    - cncf.io
    - gRPC is not RMI
- Do we compare apple to oranges? 
- uServices
    - discover
    - load balance
    - fault tolerance
- let's write some code REST
- gRPC
    - is about APIs, you start with protobuf description
    - then :magic: code generation
    - non-blocking api: stream observer for async calls 
    - streaming responses
    - JavaRX ???
    - deadline propagation
- zipkin for distributed tracing
- demo

![REST vs gRPC summary](/archive/2018-05-rest-grpc-summary.jpg)

### Starting with Ethereum, a developer approach

Note for future reader --- it's 2018 and a day without crypto/blockchain is a
day wasted.

Nevertheless, nice talk from Nicolas Fränkel --- presented smart contract basics
and with an extensive live demo.

- Blockchain properties
    - Immutable
    - Transparent
    - ...
- Top 10 by market cap  coinmarketcap.com
    - most interesting stuff you can do with bitcoin: buy and sell
    - eth allows you to code on a blockchain
        - then deploy on the blockchain
        - then call from outside of the brokchain
- EVM
    - account types
        - externally owned account
        - contract
    - eth is like a database, just state, nothing happens of its own
        - you can interact via transactions
        - transactions to contracts are similar to stored procedure
        - etherum byte code
            - solidity to generate the byte code
                - static types
                - inheritance
                - libraries
                - user defined types
                - [remix](https://remix.ethereum.org): super basic cloud ide
                - pure function - no writes on the blockchain
                    - you pay for writes
            - deployment
                - how to test? test network vs real network
                    - rikeby 
            - truffle
                - testing "framework"


### Fn Project — an open source container native FaaS platform

David Delabassee from Oracle presented Fn, a function-as-a-service from Oracle.


### Kotlin coroutines

Marcin Moskała presented this new feature of Kotlin. He actually did a deep
dive, and showed how the corutines are implemented.

### Conference party

![Hala Główna](/archive/2018-05-hala-glowna.jpg)

The talks ended in the evening and a party in *Hala Główna* followed. I had a good
time and some interesting conversations with other conference attendees.
However, I think this was the weakest part of the event 👎. The pub was nice and
hipster, but the music was too loud to sustain a good conversation and the
speakers' dinner was held at the same time in a different place. I feel like the
split was a bit artificial and alienated the speakers. Now, I don't mind an
extra speakers' dinner, but it cloud have been held on the day #2. Don't
get me wrong, I had a good time and really enjoyed the conversations, but I feel
it cloud have been so much better.

## Day 2

### Demystifying Java 9 features

Very thoughtful talk by Ionut Balosin. He listed some of the important java 9
features and presented his benchmark results. Good talk both as a tour of the
features and in giving suggestions on when to use them.

![onSpinWait](/archive/2018-05-on-spin-wait.jpg)

- private interface methods
    - private iface methods ==> invokespecial opcode 
    - default is not a bytecode keyword, only at the java code level
    - private static interface methods ==> invokestatic (same as other static
      calls)
    - /an example of private interface method in deference in lambda, but too
      fast to follow with notes/
- strings in java 9
    - -XX:+CompactStrings is enabled by default
        - which means LATIN1 is used if possible and UTF16 if not (the default
          pre java 9)
        - BTW the string max size for UTF16 is Integer.MAX_VALUE / 2
    - string construction
        - it tries latin1, if it fails switches to utf16, so if you have a lot
          of utf16 there is a perf cost
            - so if majority of the strings are utf16 consider disabling compact
              strings
    - char[]: if byte backed latin1 string is asked for char or char[] there is
      a perf cost due to conversion; the impact is minimal though
    - String.eqauls()
        - if the coders are different no need to compare bytes :)
        - no downsides to that
    - there are some improvements in string concatenations
        - not always string builder is used
        - the strategy is being selected at runtime, not at the byte code level,
          so there maybe even more optimizations in the future
- stack walker api
    - lazy stack frame access
        - limit depth
        - filter frames
    - super useful and performant for deep stack traces 
- collection factory methods
    - List.of()
        - a lot of redundant methods (with 0 to 10 arguments; 11+ varargs)
        - when you call a method with varargs there is an overhead with
          allocation of an array on stack
        - also there are custom List1 and List2 wrappers without an array (array
          in java is an array of references, so this way we skip additional
          dereferencing)
        - offers better performance than unmodifiable list
    - Set.of()
        - similar
    - Map.of()
        - similar (but for pirs)
- Thread.onSpinWait()
    - suggests that the thread busy waits on a condition, that will happen
      shortly
    - hint for a scheduler
    - it compiles to x86 PAUSE instruction
        - delays, but doesn't release the CPU splice
- Contended locks in Java 9
    - improved performance with synchronized methods (when a critical section
      exclusivity is guarded by a monitor).

### The future of Java

Tobi Ajila introduced projects Valhalla and Panama. I'm super excited about
Valhalla :) 

![java-future](/archive/2018-05-java-future.jpg)

- current challenges
    - mem latency is the biggest challenge in cpu performance
    - *compliment latency table*
    - identity
        - objects have identity, if the state changes it is still the same
          object
        - but you not always need that, but you always pay the price
        - problem with references - pointer chasing
    - synchronization
        - every object can be locked, so every object needs a monitor
        - imagine the object array, each object has a header and a monitor
    - hardware prefetch
        - seq access with a primitive array prefetches next items
        - with objects array this is not a case, as the objects are not
          allocated in sequence
    - offload processing to GPU
        - JNI is the only way to intro with native data
        - shortcomings:
            - need to write wrappers
            - wrapper per function per platform
        - JNR and JNI – open source tools to generate the bindings (but a bit
          slower)
    - off-heap data
        - unsafe — need to manage pointers and bounds manually
        - direct buffers
    - SIMD
        - no jvm support for SIMD
    - packed objects
        - @Packed — describe off-heap data (similar to C structs)
        - guaranteed mem layout
        - condensed footprint, e.g.:
            - Line (instead of references)
                - start.x
                - start.y
                - end.x
                - end.y
        - drawbacks
            - quality, array, split hierarchy (e.g. PackedByte incompatible with
              Byte) — no interop with existing java code
        - the future is in valhalla and panama, but packed objects started it
- valhalla
    - varhandles
        - better unsafe and atomics
        - released in java 9
        - **note to self: check**
    - value types
        - current focus
        - codes like a class, works like an int
        - a way to specify immutability, which will allow compilation
          optimizations without escape analysis
        - features
            - immutable
            - flattening
            - interop with existing types
        - MVT — prototype implementation (jvm only, no java support)
            - data types
            - drawbacks
                - hard to program (byte code generation), no type checking
                  (because of the former) ==> but this is OK with experimental
                  feature
                - interop is problematic
        - LWorld — next step
            - work in progress, everything may change, under active development
    - generic specialization
- panama
    - goals
        - improve interop with native
        - better FFI for jvm and c/c++
        - off-heap data access
        - intrinsics for simd

### A crash course in modern hardware

Cliff Click again, this talk was a good continuation of the Java Future. Cliff
described what changed in the modern hardware since the von Naumann model.

- intro
    - von neumann machine
        - good for design algorithms
        - not how the model computer works
    - single thread perf is almost stalled in recent decade (~10% . year), the
      gain in the number of cores
    - walls:
        - power
            - too much hit
        - ILP
            - branch prediction limit
        - Memory
            - it takes time to move signal from the chip to memory and back
            - speed of light (if the signal takes longer for round-trip than one
              cycle)
- Instruction Level Parallelism
    - faster cpu in the same clock rate
        - multiple instructions (pipelining)
        - improves throughput, not latency
    - caches, layers, each 10x slower, 10x larger
            - cpus do not wait on the cache resolution but moce forward
            - branch prediction to not wait and proceed even more
                - 95% right
    - multiple issue (supersscalar execution)
        - multiple instructions if unrelated in one clock cycle
    - register reaming
        - put the results of the branch predictions in some hidden registers
        - if you like it, you rename it back to useful, if not you throw them
          away
    - cpu perf is entirely dominated by cache misses
    - itanium project burned $$$ but states ILP is still not a thing (in
      desktop/server apps)
    - mined out
        - complicating ILP units is a diminishing return
        - GPUs – narrow problem domain, a lot of simple cpus 
- mem subsystem & data reaches
    - L1/2: Static RAM, expensive, drains tons of power, fast, same technology
      as CPU
    - Dynamic RAM
        - loses mem and needs to be refreshed
        - less power, higher density at the expense of speed
    - memory is the new disk
    - we try to improve throughput as latency seems mined out
        - complex programing model because of that
    - L1 is per core, so cache do not represent the whole memory
        - this results in data races and we need volatiles 
        - by default we have a low coherence
        - with volatile we force coherence
- specter & meltdown
- new perf model
    - conversions (e.g. prot buf -> json -> dom) is expensive, don't do it for
      simple operations
    - shared data OK, mutable data OK, shared + mutable NOT OK
    - cache misses are hard to spot in profiles
    - cpu gives the illusion of simplicity but there are a lot of moving parts
      and complex under the hood
    - use a profiler (and a hardware profile to get out of cache)


### Modern SQL: evolution of a dinosaur

This was one of the best talks on the conference. Markus Winand blogs at
[modernsql](https://modern-sql.com) and
[use-the-index-luke](https://use-the-index-luke.com). While sql is a boring
subject in 2018 (c'mon we have nosql for over a decade), Markus was super
enthusiastic. He explanation was clear and examples illustrative. Moreover, he
coherently presented the evolution of the SQL standard since the relation days
of SQL92. I've enjoyed it so much.

- SQL-92 same time as Windows 3.1
- SQL:1999
    - breaks with relation only model
    - WITH
        - statement scoped view
        - mostly about maintaince
            - literate sql / code organization
    - WITH RECURSIVE
        - at execution time just a loop
        - to cope with a hierarchy 
        - recursive allows self-references :)
        - it is a loop and needs an abort condition
        - pattern: row generator
        - no correspondence in the relational algebra
- SQL:2013
    - OVER and PARTITION BY
        - merge two concepts:
            - merge rows with same key properties
            - aggregate functions (in 92 can't be used independently)
    - OVER and ORDER BY
        - framing & ranking
        - CURRENT ROW – sliding frame
        - running total, moving avg
    - top n per group
    - avoiding self-joins
- SQL:2016
    - XMLTABLE
- SQL:2008
    - FETCH FIRST
- SQL:2011
    - OFFSET
        - don't use it (similar to egnyte)
    - OVER
    - system versioning (temporary and by temporal tables)
        - time traveling
        - tables can be system versioned, application versioned or both
- SQL:2016
    - LISTAGG
        - kind of like ",".join(a, b, c)
- SQL has evolved from the evolution model for decades

- Oracle bought Sun, who owned MySQL and this was super good thing. MariaDB was
  forked and start adding features, which in turned put pressure on MySQL and on
  Postgres.

### Engineering architecture

Jakub Kubryński who promises he is not an ivory tower architect, and can code
still :) The talk reveled around the idea that you can't improve what you don't
measure. And that it applies to the software architecture as well.

- Good architecture
    - scalable
    - maintain
    - tailored
    - secure
    - flexible
    - resilient
    - testable
        - e.g. i want to verify the interactions within five minutes, and have
          at most 1 bug per month
    - But how to measure it? How to quantify?
    - Architecture can be great, but needs to be implementable
- Architecture is engineering and should be measured
    - How many lines, and how many lines actually used in production
    - "quantify and write down the scale" (this summarizes fir 15 mins of the
      talk)

### Using an open source library? Beware of vulnerabilities!

Bruno Bossola preached on how important is to check your dependencies, as they
**are** vulnerable. Good talk, entertaining speaker and nice demo. Plus, I've
learned that UK fruits are not real fruits. Italian ones are!

- three cases of exploits
    - CVE-2015-4852
        - SF transport
        - apache commons collections bug in serialization 
    - CVE-2017-5638
        - government agency in canada
        - exploits file upload bug in struts 2
    - CVE-2017-5638
        - Equifax, same vulnerability, again, +3 months after canada
- why do we use oss libs
    - deliver code fast
    - do not rewrite what is available
    - state of the art algorithms (e.g. encryption)
    - transitive dependencies
- what is a vulnerability
    - a weakness in a lib that people can exploit to compromise the underlying
      system
- Sample exploit CVE-2017-7525
- Preventive measures
    - run a check on your dependencies on each build
- Common delusions
- Conclusions
    - put something to check the dependencies in the pipelines


### Nothing but Models: Quest for the Roots of Complexity

Julian Warszawski from LendUp. Deeply philosophical talk.

- Learn these before start programming
    - Domain Driven Design
    - Test Driven Development
    - SOLID Principles
    - Functional Programming
- Models
    - Why do we need this model for? Different level of details for different
      purposes
    - Model dependent realism
        - reality should be interpreted based upon models
        - reality is not useful, only models are
        - all of the non existing abstractions (in software world) are invented
          so that we can communicate better
- Complexity
    - delivery speed of the software always decreases


### Manage Yourself!

Jurgen started with a so-so joke, but it only got better afterwards. It turned to
be nice and entertaining talk. Perfect to close the day #2. I've learned how
important is to present your idea in a way that will convince another people and
some strategies on how to do that. I actually want to add one of his books to my
reading list :)

- How to get the idea in folks minds and do something with them
- Should we celebrate success or failures?
    - celebrating all failures make little sense, because we celebrate failing
      when we are stupid
        - otoh failed experiments are learning experiences
- Finish communication is like finish design ==> no need to fill empty space
  with crap
- Many of us have ideas, but other folks are hard to convince
    - but people do change (smartphones, streaming media, etc.)
    - so the challenge is to convince folks to your idea
- gamefication
    - don't break the streak
    - scarcity
    - unpredictability (like gaming industry)
    - meaning / ownership
- You can be a square and a circle
    - the difference is the tax you pay to work in that organization
    - you gain trust being a good square

Comments:

- Oh, learn the [gameification with
  Yu-kai](http://yukaichou.com/gamification-examples/octalysis-complete-gamification-framework/#.Wvid0S-mPys).
- I really liked the _i want to be a circle, but my org makes me a square_ narrative.


## Day 3

Oh day 3. It started with a super sweet JVM internals series. 

### Deep dive into the Eclipse OpenJ9 GC technologies

Charlie Gracie presented OpenJ9 JVM and gave a tour of its GC algorithms. He
also gave some rule of thumb on when to apply which. With the usual disclaimer,
benchmark before and after tuning.

It seems that OpenJ9 has a smaller memory footprint in comparison to hotspot.
One of their GC algorithms --- metronome --- is quite unique and offers soft
real-time characteristics. Another interesting feature is the class cache, but
hotspot is catching up with CDS (see the next talk).

![gc policies](/archive/2018-05-gc-policy.jpg)

- OpenJ9 project
    - JVM open sourced sept 2017 under EPL v2
    - IBM worked on it for 17 years
    - drop in replacement for hotspot
    - adoptopenjdk 
    - eclipse OMR
        - GC JIT and few other technologies
        - open sourced before openj9 
- GC overview
    - responsibilities
        - allocation of objects
        - reclaim objects that are no longer required
    - positives
        - automatic mem management
        - reduce certain category of bugs
    - negatives
        - additional resources
        - unpredictable pauses
        - runtime costs
        - application has little control
- GC technologies in OpenJ9
    - gc collections policies
        - ...
        - gencon
            - generational copying collector with a concurrent global colector
            - two different collections
                    - nursery- copying collector
                        - scavenge --- allocator --> survier part of gc 
                - CMS for global collection
                    - the concurrent work is done on the application threads
                        - you pay a little cost at the object allocation time
                        - but pauses are smaller
            - default collector
                - best throughput for the average best pause times for most apps
            - nursery 1 : tenure 4
                - nursery => allocator : survivor
            - write barrier
        - metronome
            - interment soft realtime collector
            - no compaction, only mark and sweep
            - goals of pauses and all utilization
                - default ~3ms and 70% utilization
            - low mem footprint
        - balanced
            - region based collector (similar to G1)
            - some tunneled is collected during nursery collections (called
              partial collections)
            - NUMA aware



### Class Data Sharing

[https://simonis.github.io/GeeCON2018/CDS](https://simonis.github.io/GeeCON2018/CDS)
 
Volker Simonis described and demoed class data sharing. He also discussed some
issues with the current approach and how to work around them. I'm quite excited
to try it on some of my services and check if there are any start-up time and
mem footprint gains.

![class data sharing](/archive/2018-05-cds.jpg)

- native vs java opps
    - shared libraries are shared between instances of native apps
        - they are mapped to the memory of the process, but they don't take
          extra mem (e.g. if you run two instances of grep)
    - code cache and meatspace are not shared (in java)
- CDS & AOT
    - part of meatspace is stores in a file CDS.jsa and shared with the
      instances
    - if we AOT we can also share part of the code cache
    - this improves startup perf
        - and reduces mem footprint
- CDS
    - introduced in JDK 1.5
    - in JDK10 open sourced to application cache
    - basics
        - java -Xshare:dump
    - tomcat uses custom class loaded, so it doesn't give a lot (3.5k out of 12k
      classes)
- With JDK10 we can also share Strings and Symbols


- how are classes differentiated? E.g. different versions?
    - they have a qualified name and a checksum, so you need to prepare
      different archives for different versions of you apps


### Java Memory Model Unlearning Experience

Aleksey Shipiiev is quite famous in the java internals world, I was really
excited about this talk and Aleksey indeed delivered. Also, I've shaken his hand
after the talk. Does it get any better?

The key take-aways are these two rules:

- **safe publication**: release / acquire rule when reasoning about volatile
- **safe construction** pattern

They can give you a correct code 99.9% percent of cases and are enough unless you
write a high-perf library.

- Theory
    - spec vs implementation, spec is about contract
    - good spec is a balance, under- over- specification
    - lang spec is a abstract machine (the result should be undistinguishable,
      but can be optimized)
    - JMM os part of the abstract machine
    - JMM is simple! but most of the talks talk about implementation
    - This talk is about mimum requirements
- How to parse the JMM?
    - Executions =~ Actions AND orders AND consistency rules
    - executions are behaviors of the abstract machine, not implemented
    - implement produces results which are the subset of allowed outcomes, they
      are not required to produce all of the results
    - One read from the heap is one read from the heap, can't be inlined
      according to JMM
- Coherence
- Writes to final fields happens before reads (special property, not guaranteed
  for volatile) allows for construction safety and benign data races patterns
- Locks
- Practice: double checked locking

![Safe publication](/archive/2018-05-as-r1.jpg)
![Safe construction](/archive/2018-05-as-r2.jpg)
![Ordering modes](/archive/2018-05-as-locks.jpg)


### Blockchain more than Bitcoin

I blockchain, again! this time by Andrzej Grzesik.

what can you do with Bitcoin?
- mine ($$$ and electricity)
- buy / sell
- pay

- Eth maybe?
    - => diff talk
- bitcoin
    - new form of money
        - scarce
        - divisible 
        - durable
        - transferable
        - fungible  (interchangeable): my 50zł is the same as yours 50zł
    - distributed ledger ==> blockchain
        - list<hash<block<list<txn>>>>
        - distributed datastore
            - constantly growing in size
            - append only
            - event sourcing solves this with snapshots
        - features
            - distributed
            - immutable
            - transparent
        - but WHY are people so excited about blockchain?
            - same source of truth for transitions across the different systems
        - c*rda (corda.net)
            - no blockchain, no mining
            - permission networks
            - peer-to-peer interactions across blockchain
            - alt: hyperledger fabric
            - challenges
                - secrecy
                - multiple blockchains
                - trust
    - expensive (energy to be spend)
- other assets / small contracts
    - not really on bitcoin 
    - is your code bug free?
- ICOs
    - initial coin offering

### NoSQL Means No Security?

This was a talk by my colleague Philipp Kern. Fast paced and entertaining.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/GeeCON?src=hash&amp;ref_src=twsrc%5Etfw">#GeeCON</a> slides on &quot;NoSQL Means No Security?&quot; with MongoDB, Redis, and Elasticsearch<a href="https://t.co/7hZ7yEiivq">https://t.co/7hZ7yEiivq</a><br>Thanks for another great GeeCON! <a href="https://t.co/6rIxRDNRp0">pic.twitter.com/6rIxRDNRp0</a></p>&mdash; Philipp Krenn (@xeraa) <a href="https://twitter.com/xeraa/status/994943163654725633?ref_src=twsrc%5Etfw">May 11, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

### QWERTY or DVORAK? Debunking the Keyboard Layout Myths

Hanno Embregts shared his exp on switching to dvorak. Spoiler alert: it wasn't
for speed, and 1y+ was not enough to beat his querty speed record :)

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Thank you <a href="https://twitter.com/hashtag/geecon?src=hash&amp;ref_src=twsrc%5Etfw">#geecon</a> attendees for listening to my very geeky thoughts on keyboard layouts! The slide deck can be found here: <a href="https://t.co/pUYSxOxWie">https://t.co/pUYSxOxWie</a></p>&mdash; Hanno Embregts (@hannotify) <a href="https://twitter.com/hannotify/status/994946610667884549?ref_src=twsrc%5Etfw">May 11, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 


### Using Reactive APIs

Venkat Subramanaiam is a well known speaker for java conference goers. No
surprises this time, nice talk about new reactive APIs introduced in java 9.

- what's reactive programming?
    - way of thinking (function on data flow)
    - respond to stimuli quickly
- java 8 streams  ::   reactive streams
    - pipeline  ::  pipeline
    - lazy  ::  lazy evaluation
    - data flows (only)  ::  data flows (also)
    - errors:  good luck  ::  3 channels of communication (data  -->, error  -->, complete -->)
    - seq vs parallel  ::  sync vs async
    - data just flows  ::  back-pressure
    - single pipeline  ::  multiple subscribers
- nonblocking back pressure
    - you can't tell subscriber go slower  / faster, you need to go at their own pace 
- reactive stream manifesto
    - elastic 
    - message-driven
        - do not expose db, export the data
    - responsiveness
    - resiliency
        - fail gracefully
        - errors are first class citizens
            - like data, not alien
- ==> reactive stream api
    - publisher
        - emit data at its own rate
    - processor (pub + sub)
        - sits in the middle
        - subscribe to a stream of data upstream
        - emits data downstream 
    - subscriber
        - subscribe to a feed of data
        - process it
    - subscription
        - session between pub and sub
- java9 reactive stream api
    - interfaces
        - Publisher
        - Processor
        - Subscriber
        - Subscription
    - basically they've provided interfaces for stuff that already existed
    - almost nothing except the interfaces is available (so a bit similar to jdbc)
    - implementations
        - akka
        - reactor
        - rxjava
- /example, live demo/

## Conclusions

All of the talks were recorded and the videos should be available shortly on
the [geecon youtube channel](https://www.youtube.com/channel/UCVnJYdr91EZW8YvtMrxB1bg).

Very good conference, a lot of new ideas, a lot of interesting conversations. Tiring,
but rewarding :) I've enjoyed GeeCON 2018 a lot.
