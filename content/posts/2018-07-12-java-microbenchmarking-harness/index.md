---
title: JMH - Java Microbenchmark Harness
tags:
- java
aliases:
- /2018/07/12/java-microbenchmarking-harness.html
---
Notes I've took while doing a microbenchmark of some code. Not really structured, but maybe useful for future me.


Webpage: <http://openjdk.java.net/projects/code-tools/jmh/>


<a id="org34d64d0"></a>

## Getting started

Use an archetype:

```sh
mvn archetype:generate \
    -DinteractiveMode=false \
    -DarchetypeGroupId=org.openjdk.jmh \
    -DarchetypeArtifactId=jmh-java-benchmark-archetype \
    -DgroupId=info.kupczynski \
    -DartifactId=jmh-playground \
    -Dversion=1.0

// Or for scala
mvn archetype:generate \
    -DinteractiveMode=false \
    -DarchetypeGroupId=org.openjdk.jmh \
    -DarchetypeArtifactId=jmh-scala-benchmark-archetype \
    -DgroupId=info.kupczynski \
    -DartifactId=jmh-scala-playground \
    -Dversion=1.0


mvn clean install

java -jar target/benchmarks.jar
```

Recommended approach:

-   setup a standalone project and depend on your jars

Some hints from the scala community <https://github.com/ktoso/sbt-jmh>

> Which means "3 iterations" "3 warmup iterations" "1 fork" "1 thread". Please
> note that benchmarks should be usually executed at least in 10 iterations
> (as a rule of thumb), but more is better.

> For "real" results we recommend to at least warm up 10 to 20 iterations, and
> then measure 10 to 20 iterations again. Forking the JVM is required to avoid
> falling into specific optimisations (no JVM optimisation is really
> "completely" predictable)

Another hints <http://tutorials.jenkov.com/java-performance/jmh.html>:

-   Avoid loops around the code you're trying to benchmark
-   JVM can eliminate dead code, either return the result from the method you're benchmarking or use a black hole:
    
    ```java
    @Benchmark
    public void testMethod(Blackhole blackhole) {
        int a = 1;
        int b = 2;
        int sum = a + b;
        blackhole.consume(sum);
    }
    ```
-   Use `@State` classes for the initialization, esp. to avoid constant folding:
    
    ```java
    import org.openjdk.jmh.annotations.*;
    
    public class MyBenchmark {
    
        @State(Scope.Thread)
        public static class MyState {
            public int a = 1;
            public int b = 2;
        }
    
    
        @Benchmark
        public int testMethod(MyState state) {
            int sum = state.a + state.b;  // if we do `int a = 1; int b = 2; int sum = a+b;`
                                          // jvm will detect that and fold into `int sum = 3;`
            return sum;
        }
    }
    ```
