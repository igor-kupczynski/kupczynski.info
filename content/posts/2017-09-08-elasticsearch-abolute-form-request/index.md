---
title: Elasticsearch and absolute_form in HTTP/1.1 request line
tags:
- elastic
aliases:
- /2017/09/08/elasticsearch-abolute-form-request.html
---
Elasticsearch doesn't accept some of the HTTP/1.1 incompliant clients for a security reason.

Described on [the elasticsearch
forum](https://discuss.elastic.co/t/http-request-and-rfc/84056).
Elasticsearch complains when `absolute_form`, i.e. host + path, is present in
the request line.

For example this request works `(1)`:

    POST /_search HTTP/1.1
    host: localhost:9200
    date: Fri, 28 Apr 2017 19:30:04 GMT
    content-length: 42
    content-type: application/json
    
    {"query":{"match":{"_all":"Hello World"}}}

But this one doesn't `(2)`. Notice the `http://localhost:9200` after `POST`:

    POST http://localhost:9200/_search HTTP/1.1
    host: localhost:9200
    date: Fri, 28 Apr 2017 19:30:04 GMT
    content-length: 42
    content-type: application/json
    
    {"query":{"match":{"_all":"Hello World"}}}

It fails with

    HTTP/1.1 400 Bad Request
    content-type: text/plain; charset=UTF-8
    content-length: 74
    
    No handler found for uri [http://localhost:9200/_search] and method [POST]

Looks like the scheme+host+port part of the request is considered to be part
of path.

A HTTP/1.1 client must send `(1)` only, see [section 5.3.1 of the
RFC7230.](https://tools.ietf.org/html/rfc7230#section-5.3.1).

> When making a request directly to an origin server, other than a
> CONNECT or server-wide OPTIONS request (as detailed below), a client
> MUST send only the absolute path and query components of the target
> URI as the request-target.

Is this not an issue then? Unfortunately, the spec goes later on that a server
MUST accept absolute form for backwards compatibility reasons. Technically,
elasticsearch is not HTTP/1.1 compliant. However, there is a good reason for
it.

As Clint puts it

> supporting the absolute-form could expose us to attack vectors like DOS on a
> slow DNS lookup

It is hard to disagree. Elasticsearch will return `400 Bad Request` to a
subset of spec incompliant clients, but on the other hand it reduces its
attack surface.
