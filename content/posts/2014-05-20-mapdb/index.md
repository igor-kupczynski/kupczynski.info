---
title: MapDB
tags:
- java
aliases:
- /2014/05/20/mapdb.html
---
Fast, off-heap java database with a collection interface.

The problem statement is this - you have a set of text files. Each row
in the file represents a row from the database. The rows can belong to
one of two tables (they can represent one of two distinct
entities). There is a parent-child, one-to-many relationship between
the entities. The parent always comes before its children, but you do
not know how many children there are left. You need to denormalize the
parent data into child entities and dump the data back to a text
file. See the dumbed-down example below.


Input file:

    {type: parent, id: 1, name: foo}
    {type: parent, id: 2, name: bar}
    {type: child,  id: a, parent_id: 1, data: 123}
    {type: child,  id: b, parent_id: 1, data: 234}
    {type: child,  id: c, parent_id: 2, data: 123}

Should result in a following output:

    {id: a, data: 123, name: foo}
    {id: b, data: 234, name: foo}
    {id: c, data: 123, name: bar}


# Draft solution

I had a piece of java code to handle this. This program was not part
of a production system but rather a side tool to perform some
additional analysis; the main constraint was time - I need to have
the results as soon as possible.

The basic idea was to read files line-by-line. I stored parent
entities in a hashmap, with their ids as keys and for child entities I
retrieved the parent data and packed them together with the child
data.

# OutOfMemory issue

This worked fine until I hit a file with 6 million lines and ~3GB in
size. My first try was to assign more memory to JVM - 10 GB instead of
2 GB. But this didn't solve the issue, I hit OutOfMemory again. This
means that java collections have a huge memory overhead - 3GB file
can't fit into 10GB heap (I tried 16 GB with the same result). There
are some other factor to consider - a hashmap needs to fit into one
generation (old or young, so probably it has to be up to 5GB). You can
read more on
[collections overhead](https://plumbr.eu/blog/fat-collections) and
[OOM on overprovisioned heap](https://plumbr.eu/blog/outofmemoryerror-on-overprovisioned-heap)
on the Plumbr blog.

# Enters MapDB

The problem was that I couldn't fit my data into heap using the
regular collections. The solution was either to use more lightweight
collections or to store the data off-heap. I found
[this post](http://www.kotek.net/blog/3G_map) and decided to try the
MapDB project.

[MapDB](http://www.mapdb.org/02-getting-started.html) is a pure-java
database which stores data off-heap either in memory or on disk. Even
you store the data in memory, its very fast because it does not have
to deal with GC by using direct ByteBuffers (pls see
[this](http://stackoverflow.com/questions/6091615/difference-between-on-heap-and-off-heap)
and
[this](http://docs.oracle.com/javase/6/docs/api/java/nio/ByteBuffer.html)). The
great thing about MapDB is that it uses the Map interface and it is
trivial to use it instead of regular maps. All I needed to do was to
substitute this line:

    private Map<String, FileShortRow> files = new HashMap<String, FileShortRow>();

With this piece of code:

    private final DB db = DBMaker
            .newMemoryDirectDB()
            .transactionDisable()
            .asyncWriteFlushDelay(100)
            .compressionEnable()
            .make();

    private Map<String, FileShortRow> files = db.getHashMap("files");

And change the JVM options in order to put less memory to heap and more to direct buffers:

    -XX:MaxDirectMemorySize=10G -Xmx512M

And that's it. I was done with my analysis after 30 mins of googling
and 15 mins of implementation change. Kudos for
[Jan Kotek](https://github.com/jankotek) for creating the MapDB.
