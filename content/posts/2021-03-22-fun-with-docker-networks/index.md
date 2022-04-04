---
title: Fun with docker networks
tags:
- containers
- network
aliases:
- /2021/03/22/fun-with-docker-networks.html
---
How to find out a container responsible for a given open port on a host.

Recently, I've debugged an issue with a server running multiple docker containers. They interact with each other by exposing TCP ports on the host and communicating over these ports.

In this post:
* I describe a surprising discovery about [_host network mode_ containers][docker-network-host] (spoiler: they don't show under `PORTS` in `docker ps`),
* Show to find a container responsible for a given port on the host,
* Link to a repo where I investigate docker networking in much more detail.

## The story

Let's say we have the container exposing `:32080` on the host:
```sh
$ docker run -d --name my-service-17 -p 32080:80 nginx

$ curl localhost:32080
..
<h1>Welcome to nginx!</h1>
..
```

We found out that `my-service-4` reports this error in the logs:
```
ERROR Unexpected response from https://<container-host>:32080/_important
``` 

How do you tie it back to `my-service-17`? Easy, `docker ps` reports the [published ports][docker-published-ports]. You can do for example:
```sh
$ docker ps -a | grep 32080
abfa35be07f8        nginx                                                              "/docker-entrypoint.…"   7 minutes ago       Up 7 minutes           0.0.0.0:32080->80/tcp                                                                                                                                                                            my-service-17
```

At some point of the debug session, I looked for another port, say `32567`. But `docker ps -a | grep 32567` was empty -- no results. To add insult to the injury there was indeed something on the other side:
```
$ curl localhost:32567
..
I'm here!
```

I was scratching my head. There are of course other tools in software engineers' 🧰, e.g. `netstat` or `ss`. We can find the PID of the process and then trace it back to its container:

```sh
$ sudo netstat -ntlp | grep ':32567'
tcp        0      0 0.0.0.0:32567              0.0.0.0:*               LISTEN      884710/nginx: maste

# 884710 is the process PID


$ for i in $(docker ps -a -q); do docker inspect -f '{{.State.Pid}} {{.Name}}' $i | grep 884710; done
884710 /my-service-15
```


And we have the offender, it's `my-service-15` container.

**edit**

My colleague pointed out another way to get from the container name from the PID is to use cgroups namespace, e.g.
```sh
$ grep -ri 884710 /sys/fs/cgroup 2>/dev/null
/sys/fs/cgroup/blkio/docker/a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b/cgroup.procs:884710
/sys/fs/cgroup/blkio/docker/a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b/tasks:884710
/sys/fs/cgroup/hugetlb/docker/a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b/cgroup.procs:884710
/sys/fs/cgroup/hugetlb/docker/a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b/tasks:884710
/sys/fs/cgroup/perf_event/docker/a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b/cgroup.procs:884710
/sys/fs/cgroup/perf_event/docker/a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b/tasks:884710
..
```

In that case `a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b` is our container. 

```sh
$ docker inspect -f '{{.Name}}' a4e509ae2a0875153dce22362dd08aa1f026e2cd7684ff2bcbd355ba981bbe6b
/my-service-15
```

## Host network mode

What is different about `my-service-15` is that it runs with [`--network host`][docker-network-host], which means it doesn't get its own network namespace. Network-wise it acts as a process running directly on the localhost.

At first, I was surprised that the port doesn't show up in the `docker ps` overview, but it makes sense -- the list of open ports is not static, so it would be quite hard for docker to track it. Plus there is the standard OS tooling.


## Deep dive

![Docker containers using various network modes](/archive/2021-03-docker-networks.png)

_The image shows docker containers running with various network modes. To have some fun and research the details of docker networking I've created [igor-kupczynski/fun-with-docker-networks repo][repo]. I encourage you to check it out if you want to learn more!_

## Summary

In the post, we've shown how to find a container responsible for a port on the host. We can do that regardless of the network mode of the container:
* If it uses the default `bridge` network and simply publishes the port then `docker ps -a | grep <port>` is enough.
* If it runs int the host network mode we have to fall back to standard linux tooling to find its PID. Then we can tie the PID back to the docker container.

I've also created a [igor-kupczynski/fun-with-docker-networks repo][repo] to dive deeper into the subject.


[docker-published-ports]: https://docs.docker.com/engine/reference/commandline/run/#publish-or-expose-port--p---expose
[docker-network-host]: https://docs.docker.com/network/host/
[repo]: https://github.com/igor-kupczynski/fun-with-docker-networks
