---
title: Make Me a Cluster
tags:
- elastic
aliases:
- /2015/03/23/make-me-a-cluster.html
---
Easily provision an Elasticsearch cluster for fun, experimentation and profit.

If you followed my blog you may known than I'm responsible for
changing my employer's in-house search appliance to a new one backed
by Elasticsearch. We had a huge data volume, even though we are not
fully migrated to Elasticsearch yet. During our day to day operations
we see a lot of interesting cases. I think we reached that point where
reading documentation, news groups and blogs is simply not enough to
explain everything we see in the trenches. I often feel a need to
experiment with an Elasticsearch cluster.

I've created an ansible playbook to provision one. With the playbook,
ut is a dead-easy task to provision the cluster in matter of
minutes. I use vanilla Ubuntu 14.04 machines on [digital ocean][do],
but any other cloud provide will work.

[do]: https://www.digitalocean.com/?refcode=649b18bb5e59

Additionally, on *26.03.2015* I'll conduct a workshop on Elasticsearch
at [Poznań University of Technology][put], i.e. my *alma matter*. In
order to make it simple for students I've created a cluster which we
can use on the workshop.

[put]: http://www.put.edu.pl/

Details are posted on github. Fork me there :-)
[The Elasticsearch Ansible Playbook](https://github.com/puszczyk/elasticsearch-workshop-cluster).
README file describes how it works, what is the cluster architecture,
what is missing to use it in production and finally, why ansible.

