---
title: Java 8 Released
tags:
- java
aliases:
- /2014/03/30/Java-8.html
---
Whats new in Java 8? Where to start learning?

[Last week we saw the new release of Java](http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html). Version
8 brings a lot of new features into the language. I think the most
important are:
- lambda expressions,
- default method implementations in interfaces,
- new datetime api.

There is also a lot of smaller improvements and additions in the
standard library.

![Lamdas FTW!](/archive/2014-03-30-lambda.jpg) _Image from [http://www.jamesiliff.com/speechless-protagonists-spatial-storytelling-and-immersive-worlds-half-life-game-narrative-review-at-gdc-2012](http://www.jamesiliff.com/speechless-protagonists-spatial-storytelling-and-immersive-worlds-half-life-game-narrative-review-at-gdc-2012)_

I'm a huge fan of the functional programming paradigm and of course
I'm very excited about the release. Hope it will be quickly and widely
adopted in production. Also the library improvements will make a
compelling alternative to popular external libraries like
[JodaTime](http://www.joda.org/joda-time/) and
[Guava](https://code.google.com/p/guava-libraries/). This is good,
because I think everybody were including them, even for small
projects.

How to get start with Java 8?

- I can recommend a webinar by Venkat Subramanian -
  [Functional Programming with Java 8](http://blog.jetbrains.com/blog/2014/03/27/functional-programming-with-java-8/) - very good to get started.
- [this blog post](http://www.techempower.com/blog/2013/03/26/everything-about-java-8/)
  offers more detailed overview. It is 1 year old, but gives a very
  good content
- There is also an
  [official Oracle tutorial for lambdas](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html).

If you want to dig into more details of lambdas in Java 8 check these
two articles by Brianz Goetz:
- [http://cr.openjdk.java.net/~briangoetz/lambda/lambda-state-final.html](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-state-final.html)
- [http://cr.openjdk.java.net/~briangoetz/lambda/lambda-libraries-final.html](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-libraries-final.html)

*This is now Java* (example from
 [the latter](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-libraries-final.html))

    Pattern pattern = Pattern.compile(\\s+");
    Map<String, Integer> wordFreq =
        tracks.stream()
            .flatMap(t -> pattern.splitAsStream(t.name)) // Stream<String>
            .collect(groupingBy(s -> s.toUpperCase(),
                counting()));

Happy coding with Java 8!
