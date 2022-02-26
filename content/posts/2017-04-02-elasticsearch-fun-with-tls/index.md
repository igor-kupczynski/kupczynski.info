---
title: Encrypt the traffic between nodes in your elasticsearch cluster
tags:
- elastic
aliases:
- /2017/04/02/elasticsearch-fun-with-tls.html
---
Encrypting the traffic between elasticsearch nodes using X-Pack security.

In my opinion, it is quite important to encrypt the traffic between the backend services. Especially, if you don't
control your infrastructure (or don't trust your infra provider). Let's see how to encrypt the elasticsearch cluster
transport traffic with X-Pack.

[X-Pack](https://www.elastic.co/guide/en/x-pack/current/xpack-security.html) uses TLS to encrypt the traffic between
nodes in the cluster and between clients and the cluster. Additionally, a hostname verification can be performed as
well. The prerequisite for internal TLS is to assign a [X.509 certificate](https://en.wikipedia.org/wiki/X.509) to each
node. The certificates should be sign by a trusted certificate authority, or CA.

In this blogpost we will use official docker images of elasticsearch and docker compose to demonstrate the setup. You
can find the code to reproduce the setup on my github
at [elasticsearch-fun-with-tls](https://github.com/igor-kupczynski/elasticsearch-fun-with-tls).

## Goal

In this exercise we want to set up three elasticsearch nodes. Two of them will have certificates signed by a trusted CA
and the third one is an alien --- its certificate is not trusted. We will show that it can't join the cluster formed by
the two.

Note that we use docker images with default config here (including passwords) which is not the best production
config. In this post we are only interested in demonstrating the TLS setup.

## Certificate generation

First lets produce the certificates. Elasticsearch has a tool which helps with
that
---
[`certgen`](https://www.elastic.co/guide/en/x-pack/current/ssl-tls.html#generating-signed-certificates). It
can generate a CA cert + key and a set of certificates + keys for all the
nodes. In a production scenario you want to generate
keys + [CSRs](https://en.wikipedia.org/wiki/Certificate_signing_request) and
then sign the CSRs with you CA to get the node certificates. See the `certgen`
documentation for that.

Let's start with a following `instances.yaml`

```
instances:
  - name: "node-0001"
    dns:
      - "elasticsearch-1"
      - "localhost"  # so that it is easy to curl node-0001 from
                     # host machine
    ip:
      - "172.18.10.2"
  - name: "node-0002"
    dns:
      - "elasticsearch-2"
    ip:
      - "172.18.10.3"
```

It defines two nodes together with their dns names and ip addresses (to allow hostname verification).

And use a temporary docker container to run `certgen`:

```
$ mkdir -p certificates
$ docker run -it --rm \
    -v '$(cwd)/instances.yaml:$(es_dir)/config/x-pack/instances.yaml' \
    -v '$(cwd)/certificates:$(es_dir)/config/x-pack/certificates' \
    -w $(es_dir) \
    'docker.elastic.co/elasticsearch/elasticsearch:$(es_version)' \
    bin/x-pack/certgen -in instances.yaml \
        -out $(es_dir)/config/x-pack/certificates/bundle.zip
$ unzip -o certificates/bundle.zip -d certificates
$ rm certificates/bundle.zip
```

We will end up with the following certs:

```
$ tree certificates
certificates
├── ca                  # CA certificate and private key
│   ├── ca.crt
│   └── ca.key
├── node-0001
│   ├── node-0001.crt   # Node certificate
│   └── node-0001.key
└── node-0002           # Second node certificate
    ├── node-0002.crt
    └── node-0002.key

3 directories, 6 files
```

Let's also create another set --- signed by a different CA --- for the alien node-0003.


```
# file: alien.yaml
instances:
  - name: "node-0003"
    dns:
      - "elasticsearch-3"
    ip:
      - "172.18.10.4"
```

Which after using `certgen` and unzipping produces the following:

```
$ tree alien-certificates
alien-certificates
├── ca
│   ├── ca.crt
│   └── ca.key
└── node-0003
    ├── node-0003.crt
    └── node-0003.key

2 directories, 4 files
```

## Cluster definition

We are going to use docker compose the spin up the cluster. We define three services `elasticsearch-1`,
`elasticsearch-2` and `elasticsearch-3` plus a network bridge between them. To simplify the setup we are going to use
static ipv4 addressing.

First service:

```
elasticsearch-1:
  # We use official elasticsearch docker images which already contain X-Pack
  image: docker.elastic.co/elasticsearch/elasticsearch:5.3.0
  container_name: elasticsearch-1
  mem_limit: 2g
  volumes:
    - "./data-1:/usr/share/elasticsearch/data"  # The default storage driver do not play well with io-instensive databases or search engines
    - "./elasticsearch-1.yml:/usr/share/elasticsearch/config/elasticsearch.yml"        # We are going to mount our own config
    - "./certificates/ca/ca.crt:/usr/share/elasticsearch/config/x-pack/tls/ca/ca.crt"  # CA cert to trust
    - "./certificates/node-0001:/usr/share/elasticsearch/config/x-pack/tls/node-0001"  # Node cert + private key to use
  networks:
    internal-tls:
      ipv4_address: 172.18.10.2  # Static address, the same as provided during the cert generation
  ports:
    - 9200:9200                  # We want to expose the default elasticsearch port to the host
  environment:
    - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
```

and its config:

```
cluster.name: "docker-cluster"
node.name: "node-0001"       # node name, the same as provided during cert generation
network.host: 0.0.0.0
http.host: 0.0.0.0
transport.host: 172.18.10.2  # bind the inter-node communication to docker bridge network

discovery.zen.minimum_master_nodes: 1  # in should be set to quorum n/2 + 1 in a production deployment


# Private key the node-0001 uses to initiate TLS connection
xpack.ssl.key: /usr/share/elasticsearch/config/x-pack/tls/node-0001/node-0001.key

# Public certificate signed by a CA that other nodes trust
xpack.ssl.certificate: /usr/share/elasticsearch/config/x-pack/tls/node-0001/node-0001.crt

# List of CAs to trust, note that a java key store can be used as well; check the x-pack docs for details
xpack.ssl.certificate_authorities: ["/usr/share/elasticsearch/config/x-pack/tls/ca/ca.crt" ]

# Verify both the certificate and the hostname, alternative is `certificate` which won't verify the hostname
xpack.ssl.verification_mode: full

# Use TLS for inter-cluster communication
xpack.security.transport.ssl.enabled: true

# Use TLS (https) for outbound communication
xpack.security.http.ssl.enabled: true
```

The other nodes have similar configs with slight differences.

- `node-0002` and `node-0003` do not expose 9200 to the host, so they can only communicate via docker network bridge
  among themselves

- `node-0002` and `node-0003` want to connect to cluster to formed by `node-0001`

  ```
  discovery.zen.ping.unicast.hosts: 172.18.10.2
  ```

- `node-0002` and `node-0003` have next IPs in the range --- `172.18.10.3` and `172.18.10.4`

- `node-0002` and `node-0003` use their own certs and `node-0003` trust different CA and has its cert signed by a
  different CA

You can find the full configuration in the [repo](https://github.com/igor-kupczynski/elasticsearch-fun-with-tls).

## Connecting it all together

Let's start the first node

```
$ docker-compose up elasticsearch-1

(...)
elasticsearch-1    | [2017-04-01T11:27:51,670][INFO ][o.e.c.s.ClusterService   ] [node-0001] new_master {node-0001}{bSihiuKnS4S-CnEbpdc3VQ}{qB1elbDhRuW31hho1AY_pQ}{172.18.10.2}{172.18.10.2:9300}, reason: zen-disco-elected-as-master ([0] nodes joined)
```

And check how our cluster looks now:

```
$ curl --cacert certificates/ca/ca.crt -u 'elastic:changeme' https://localhost:9200/_cat/nodes
172.18.10.2 13 20 4 0.08 0.06 0.02 mdi * node-0001
```

Things to notice:

- The connection is encrypted with the node certificate, hence we need `--cacert certificates/ca/ca.crt` to establish
  trust. You can additionally verify by removing the `--cacert` part or by changing `https` to `http`. Both won't work.

- We use the default credentials `elastic:changeme`, which should be changed in any
  setup. See
  [https://www.elastic.co/guide/en/x-pack/current/security-getting-started.html](https://www.elastic.co/guide/en/x-pack/current/security-getting-started.html).

- There is one master node --- `node-0001` which is expected as this is the only node we've started.

Now the second node:

```
$ docker-compose up elasticsearch-2

(...)
elasticsearch-2    | [2017-04-01T12:32:56,103][INFO ][o.e.c.s.ClusterService   ] [node-0002] detected_master {node-0001}{WSZmQkfAREa81n9XaYni-w}{NtCHIMQLRbCAtS_HqCxEhw}{172.18.10.2}{172.18.10.2:9300}, added \{\{node-0001\}\{WSZmQkfAREa81n9XaYni-w\}\{NtCHIMQLRbCAtS_HqCxEhw\}\{172.18.10.2\}\{172.18.10.2:9300\},\}, reason: zen-disco-receive(from master [master {node-0001}{WSZmQkfAREa81n9XaYni-w}{NtCHIMQLRbCAtS_HqCxEhw}{172.18.10.2}{172.18.10.2:9300} committed version [9]])
```

```
$ curl --cacert certificates/ca/ca.crt -u 'elastic:changeme' https://localhost:9200/_cat/nodes
172.18.10.3 15 43 4 0.52 0.33 0.25 mdi - node-0002
172.18.10.2 23 43 4 0.52 0.33 0.25 mdi * node-0001
```

As we can see it joined the cluster.

Finally, lets try the third one --- alien signed off by a different CA.

```
$ docker-compose up elasticsearch-3
```

It can't join the cluster:

```
elasticsearch-3    | [2017-04-01T12:35:00,803][WARN ][o.e.x.s.t.n.SecurityNetty4Transport] [node-0003] exception caught on transport layer [[id: 0x1dbe4afa, L:0.0.0.0/0.0.0.0:41884 ! R:/172.18.10.2:9300]], closing connection
elasticsearch-3    | io.netty.handler.codec.DecoderException: javax.net.ssl.SSLHandshakeException: General SSLEngine problem
(...)
```

And it is not present in the node list:

```
curl  --cacert certificates/ca/ca.crt -u 'elastic:changeme' https://localhost:9200/_cat/nodes
172.18.10.3 14 59 3 0.51 0.39 0.28 mdi - node-0002
172.18.10.2 21 59 9 0.51 0.39 0.28 mdi * node-0001
```

## Summary and next steps

There we go, we've configured the cluster in such a way that all the inter-node communication is encrypted.  We see that
we need a set of per-node certificates signed off by a trusted CA.

_Seeing is believing_ so one should dump the docker network bridge traffic to verify it is indeed tls encrypted. This is
left as an exercise for the reader.

Next step is the certificate rotation. In most of the production grade setups you want the ability to rotate the
certificates without cluster downtime. I plan to cover it in some future blog posts.

If you want to play with this setup, be sure to check
the [accompanying github repo](https://github.com/igor-kupczynski/elasticsearch-fun-with-tls).
