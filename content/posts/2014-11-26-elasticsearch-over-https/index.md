---
title: Accessing Elasticsearch Cluster over https
tags:
- elastic
aliases:
- /2014/11/26/elasticsearch-over-https.html
---
Access your cluster securely in python or in java.

Elasticsearch offers no security out-of-the box. The connections are
over http or their [native protocol][ntp], both unencrypted, for all
the world to see. This is not a problem if your cluster is in the same
datacenter as the rest of your infrastructure - safely behind a
firewall. Quite often this is not the case. You can put the cluster in
Amazon EC2 or Google Compute Engine or something similar. If you do
this, then need to encrypt the connection. Arguably, the easiest
solution is to use https. There are various ways to configure the
cluster, usually through some third-party proxy like nginx (please see
the [discussion][es664]). Recently, we've decided to spin a new
cluster in Google Compute Engine and to allow only https access. I was
curious if my existing java and python client code will work
out-of-the box.

[ntp]: http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_talking_to_elasticsearch.html
[es664]: https://github.com/elasticsearch/elasticsearch/issues/664

For java we use [jest client][jest]. Under the hood it uses Apache
http client and the change was only to substitute http with https. If
you use self-signed cert then you'll need to make an additional
step. It is possible to
[convince http client accept a self-signed cert](http://stackoverflow.com/questions/19517538/ignoring-ssl-certificate-in-apache-httpclient-4-3),
but you will end up with extending
[`JestClientFactory`](https://github.com/searchbox-io/Jest/blob/master/jest/src/main/java/io/searchbox/client/JestClientFactory.java)
to provide own implementation of the `configureHttpClient` method.

[jest]: https://github.com/searchbox-io/Jest

The [python client][elastic-py] is not that smart. It strips the
protocol part of the host url (see the [_normalize_hosts][nh]
method). So `https://elasticsearch.me.org` becomes
`elasticsearch.me.org` and is accessed via regular, unencrypted http
connection. You can switch it to https by providing an additional
parameter `use_ssl=True`. If you already have `https://` in the host
address then this is a duplication and a place for errors. You can use
the following snippet to avoid it and "guess" the correct schema
automatically.

```
es = Elasticsearch(host, use_ssl=host.startswith("https://"))
```

[elastic-py]: http://elasticsearch-py.readthedocs.org/en/master/
[nh]: https://github.com/elasticsearch/elasticsearch-py/blob/1.1.X/elasticsearch/client/__init__.py#L16

*Update 2014-11-27* By the way, the python client doesn't verify the
 certs out-of-the box. IMHO this is not a very good practice and you
 can force the verification with `verify_certs=True`.

```
es = Elasticsearch(host, use_ssl=host.startswith("https://"), verify_certs=True)
```
