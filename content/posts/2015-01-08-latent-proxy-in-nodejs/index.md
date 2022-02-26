---
title: Delaying Proxy in NodeJS
tags: []
aliases:
- /2015/01/08/latent-proxy-in-nodejs.html
---
It is about time to add a new tool to your toolbox.

During the Christmas holiday season I had some time to learn
NodeJS. In this post I'll show how to create a simple (but useful!)
proxy server in less than 30 lines of code. I'll also give some hints
on how to start with JavaScript with no prior experience.

# Delaying Proxy

To give some background. I have a piece of code calling elasticsearch
via its REST api and I wanted to test how it behaves in case of
various network delays. I decide to create a simple proxy to simulate
it. The idea is depicted below.

![Proxy to simulate network delays](/archive/2015-01-delayed-proxy.png)

## The requirements for the delaying are:

* Given a server and a latency in seconds it should bind to a
localhost port
* All requests going to this port should be on hold for the
specified number of seconds and then proxied to the specified server address

### An example usage may look like this.

* Run the proxy in one terminal

<script src="https://gist.github.com/puszczyk/e900cfdeeac5019ecdea.js?file=Running the proxy"></script>

<noscript>
<code>
$ node --harmony proxy.js --server http://localhost:9200 --latency 10
Listening at 'localhost:8008', proxying to 'http://localhost:9200',each request is on hold for 10s
</code>
</noscript>


* Issue a request in other terminal

<script src="https://gist.github.com/puszczyk/e900cfdeeac5019ecdea.js?file=Delayed reqest example"></script>

<noscript>
<code>
$ time curl -XGET localhost:8008
{
  "status" : 200,
  "name" : "X-Ray",
  "version" : {
    "number" : "1.3.5",
    "build_hash" : "4a50e7df768fddd572f48830ae9c35e4ded86ac1",
    "build_timestamp" : "2014-11-05T15:21:28Z",
    "build_snapshot" : false,
    "lucene_version" : "4.9"
  },
  "tagline" : "You Know, for Search"
}
 
real	0m10.048s
user	0m0.004s
sys	0m0.004s
</code>
</noscript>

**Note** This is a simple example, in practices, you'll probably make
  the delay logic more complex - according to your use case.

## Implementation

Its quite easy to create such a proxy in nodejs. You need the
following code.

<script src="https://gist.github.com/puszczyk/e900cfdeeac5019ecdea.js?file=proxy.js"></script>

<noscript>
<code>
'use strict';
// Opts parsing
const argv = require('optimist')
          .demand(['server','latency'])
          .usage('Usage: $0 --server [address] --latency [num]')
          .describe('server', 'Server to proxy to, e.g. http://localhost:9200')
          .describe('latency', 'How much time will each request be on hold before proxing to the server (in seconds)')
          .describe('proxy_port', 'Port that the proxy will listen on').default('proxy_port', 8008)
          .argv,
      proxy_host = 'localhost',
      proxy_port = argv.proxy_port,
      proxy_to = argv.server,
      latency = argv.latency;
 
// Modules
const http = require('http'),
      httpProxy = require('http-proxy'),
      sec = 1000;
 
// Proxy definition
const proxy = httpProxy.createProxyServer({}),
      proxy_func = function(req, res, target) {
          return function() {
              proxy.web(req, res, {target: target});
          };
      };
 
// Listen to requests
http.createServer(function (req, res) {
    // Run the proxy logic on each request after a timeoue
    setTimeout(proxy_func(req, res, proxy_to), latency * sec);
}).listen(proxy_port, function(e) {
    console.log("Listening at 'localhost:" + proxy_port + "', proxying to '" + proxy_to +
                "', each request is on hold for " + latency + "s");
});
</code>
</noscript>

The core of the proxy is this:
    
    setTimeout(proxy_func(req, res, proxy_to), latency * sec);

where `proxy_func`

    proxy_func = function(req, res, target) {
          return function() {
              proxy.web(req, res, {target: target});
          };
    };

It simply proxies the request after waiting the `latency` number of
seconds. `setTimeout` is asynchronous so it is not a busy waiting.

This code relies on two external node modules, which we configure and
pull via [npm][npm]. npm is a package manager for nodejs. It can pull
the dependencies and automate the build process (similarly to maven or
gradle). It has a simple configuration - all you need to add a
`package.json` file to the root directory of your project. An example
file for our delaying proxy is depicted below.

<script src="https://gist.github.com/puszczyk/e900cfdeeac5019ecdea.js?file=package.json"></script>

<noscript>
<code>
{
  "name": "latency-proxy",
  "version": "0.0.1",
  "main": "proxy.js",
  "author": "Igor Kupczyński <igor@kupczynski.info>",
  "license": "Apache 2.0",
  "dependencies": {
    "http-proxy": "^1.8.1",
    "optimist": "^0.6.1"
  }
}
</code>
</noscript>

[npm]: http://npmjs.org/

Easy, right?

# Learning NodeJS

I watched a presentation sent by one of my colleagues
[NodeJS For Java Developers][node-for-java] by Tim Boudreau. This is a
good start to get some feeling of nodejs, but you'll need more.

I felt that my javascript knowledge is close to zero so I went though
the [Good Parts][good-parts]. The book is short and concise. I can
highly recommend it. Then finally, I was ready to get a book on node -
[The Right Way][the-right-way]. Again, short and to the point. The
author walks you through a process of build a few apps, covering a lot
of material in a fast-pace. Very good for a busy professional (we all
are them, right? ;-)).

![The Right Way](/archive/2015-01-right-way.jpg)

[node-for-java]: http://vimeo.com/100914090
[good-parts]: http://shop.oreilly.com/product/9780596517748.do
[the-right-way]: https://pragprog.com/book/jwnode/node-js-the-right-way

To sum up, I really enjoy working with NodeJS. It is a great tool to
write network applications and utility scripts - node startup time is
less than 1 sec plus it has a good library to deal with the file
system, processes, pipes, etc. To learn the basics I did the
following. It also gets a very good traction recently. Even if you are
a full time java geek it's worth to explore at least its basics.
