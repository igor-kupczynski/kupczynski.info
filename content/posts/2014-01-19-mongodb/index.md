---
title: Second Week of Mongo M101J Course
tags: []
aliases:
- /2014/01/19/mongodb.html
---
First impressions on MongoDB and M101J course.

I finished the second week of the MOOC [MongoDB for Java Developers][m101j]
which I advertised in my latest post. I really like the course and I think its
worth to spend some time learning Mongo.

[m101j]: https://education.mongodb.com/courses/10gen/M101J/2014_January/about

MongoDB itself is an example of a NoSQL database and you can learn more during
the course. In this post I want to highlight some features presented in first
two weeks which I feel are inconsistent and surprised me a bit.


# Inconsistent handling of exact matches in arrays of values

Mongo stores data as [documents][docs]. The documents can have fields of
various type, including strings, arrays and embedded documents. Two documents
in the same collection can have same fields with different datatypes. Example of two
documents are given below:

[docs]: http://docs.mongodb.org/manual/core/introduction/#document-database

    {
        "_id" : ObjectId("52dc1a428ce25a336d8c849c"),
        "a-field" : 1,
        "b-field" : [
            "array",
            "of",
            "values"
        ]
    }
    {
        "_id" : ObjectId("52dc1a538ce25a336d8c849d"),
        "a-field" : "1",
        "b-field" : "of"
    }


Lets assume that the above documents are part of a collection `documents`. If
you want to find `documents` with `a-field == 1` only the first document will
match, that is comparison is strongly typed. The same with `a-field =="1"`.
Also if you want to match an embedded object the match has to be exact,
not partial. Some examples:

    > db.documents.find({"a-field": 1}).pretty()
    {
        "_id" : ObjectId("52dc1a428ce25a336d8c849c"),
        "a-field" : 1,
        "b-field" : [
            "array",
            "of",
            "values"
        ]
    }
    > db.documents.find({"a-field": "1"}).pretty()
    {
        "_id" : ObjectId("52dc1a538ce25a336d8c849d"),
        "a-field" : "1",
        "b-field" : "of"
    }


There is one exception to this rule. If you look for a value and this field is
an array, then mongo will go through array elements and try to match them one
by one:

    > db.documents.find({"b-field": "of"}).pretty()
    {
        "_id" : ObjectId("52dc1a428ce25a336d8c849c"),
        "a-field" : 1,
        "b-field" : [
            "array",
            "of",
            "values"
        ]
    }
    {
        "_id" : ObjectId("52dc1a538ce25a336d8c849d"),
        "a-field" : "1",
        "b-field" : "of"
    }
    > db.documents.find({"b-field": "array"}).pretty()
    {
        "_id" : ObjectId("52dc1a428ce25a336d8c849c"),
        "a-field" : 1,
        "b-field" : [
            "array",
            "of",
            "values"
        ]
    }


Clearly there is an exception made for arrays. Probably it is a design
decision because this may be a frequent use case. Nevertheless, I think there
should be a dedicated operator enabling you to match also within arrays, since
matching for other types is exact.

# By default update updates only a first matching row, but remove removed all

Update syntax is this `<collection>.update(<select criteria>, <new
values>)`. One would expect, similarly to sql, that this will update all the
matching rows. But this is not the case, update will only affect first
matching row. To do a multiupdate, you need to pass it as additional parameter:

    db.grades.update({"assignment": "home-work-1", "points": {$gt: 90}}, {$set
    {"grade": "A"}, {$multi: true})

Fine, mongo does not need to follow SQL here. But remove in contrast removes
all the matching documents. There shouldn't be such a discrepancy between two
commonly used operations.

# getLastError returns really the status of the last operation

For example, we try to insert to documents with the same primary key:

    > db.documents.insert({"_id": "foo", "foo": "bar"})
    > db.runCommand({ getLastError: 1 })
    { "n" : 0, "connectionId" : 16, "err" : null, "ok" : 1 }
    > db.documents.insert({"_id": "foo", "foo": "bar"})
    E11000 duplicate key error index: test.documents.$_id_  dup key: { : "foo" }
    > db.runCommand({ getLastError: 1 })
    {
        "err" : "E11000 duplicate key error index: test.documents.$_id_  dup key: { : \"foo\" }",
        "code" : 11000,
        "n" : 0,
        "connectionId" : 16,
        "ok" : 1
    }

First operation was successful, we can see see its status by using the
`getLastError` command. For the second operation it returns the error. This
command is used to check status, but its name implies it is only for error
reporting.


Mongo seems to be a mature product with its market niche. It is well
documented, with lot of tutorials and the on-line course. Nevertheless, it has
its share of surprises.
