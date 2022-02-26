---
title: Server Name Indication with Traefik and Go
tags:
- golang
- network
aliases:
- /2019/05/21/traefik-sni.html
---
How to setup Server Name Indication with Traefik.

In this blog post we discuss briefly what an SNI is, show a simple example of setting up SNI with Traefik --- a popular reverse proxy written in go --- and then show some options for implementing it in "plain" go. This blog has a companion repo [traefik-tls-demo](https://github.com/igor-kupczynski/traefik-tls-demo), be sure to check it if you want to try the example.


## Server Name Indication (SNI)

To establish an https connection the server presents its certificate to clients as part of the TLS handshake. Since the connection is not established yet, the server doesn't see the http request --- neither the request line, not its headers. We're not at the application level yet. Therefore, the server has to make the decision on which certificate to present based on the target IP of the connection. In practice, this limits the server to a single certificate per public IP address.

Is this a problem? Imagine you offer software as a service. The service is multi-tenant, i.e. many customers are collocated together. To access your service customers use a URL `https://<random-id>.awesome-service.io`. This is OK --- you own the `awesome-service.io` domain and you present a wildcard certificate `*.awesome-service.io` to any client connecting to your servers.

Now one of your biggest customers wants to use a vanity URL, e.g. `https://awesome.vanity.com`, to access the service. Of course, they own `vanity.com` and can send you the certificate for `awesome.vanity.com`.

This is a problematic use case in the classical TLS / https. You need to assign a dedicated public IP for this customer, as your  endpoint can't distinguish the connections to `https://awesome.vanity.com` and `https://<random-id>.awesome-service.io` --- remember, to see the headers we need to establish the connection first.

*Server Name Indication (or SNI)* to the rescue. It is an extension to the TLS protocol, which allows clients to pass the domain/hostname they want to connect to unencrypted. A client sends this domain as part of the `ClientHello` message (learn more about [TLS Handshake](https://hpbn.co/transport-layer-security-tls/#tls-handshake)). The server can send a different certificate in the `ServerHello` based on the domain.

![TLS handshake with SNI](/archive/2019-05-tls-handshake-with-sni.png)

Is this secure? The connection stays encrypted, however, the domain is sent in plain text. Someone snooping on the network can discover it. The counter-argument is that most of the DNS queries are already unencrypted, so this adversary likely knows the domain already. There is another extension proposed by Cloudflare, on how to make the SNI secure. Check their [blog post](https://blog.cloudflare.com/encrypted-sni/) --- it is quite neat.


## Traefik example

Let's try to setup SNI with [Traefik](https://traefik.io/). Treafik is a quite popular, *cloud-native* (buzzword bingo) reverse proxy.

See the [companion repo](https://github.com/igor-kupczynski/traefik-tls-demo) for full example. We have two apps, each running in a docker container. We want to set them up behind a proxy. We want to route to one or the other depending on the domain `whoami.traefik.local` or `snowflake.traefik.local`. However, both domains are exposed by a single proxy under the same IP address.

**Note**: Examples use Traefik 2.0, which is alpha as the time of writing. Something similar can be accomplished with v1 as well. If you go back few commits in the companion repo to [traefik-1.x tag](https://github.com/igor-kupczynski/traefik-tls-demo/tree/traefik-1.x) you will find the same example with 1.x syntax.

![Two services behind a proxy](https://raw.githubusercontent.com/igor-kupczynski/traefik-tls-demo/master/overview.jpg)

Traefik makes it super easy to configure SNI. You need to specify the list of certificates in the config:

```toml
[[tls]]
  [tls.certificate]
    certFile = "/etc/traefik/certs/whoami.crt"
    keyFile = "/etc/traefik/certs/whoami.key"

[[tls]]
  [tls.certificate]
    certFile = "/etc/traefik/certs/snowflake.crt"
    keyFile = "/etc/traefik/certs/snowflake.key"
```

Traefik will load all of the certificates to its internal certificate store and then on a connection will select the best matching one based on the domain.

Moreover, it will load/unload the certificates dynamically if you modify the file.

We use docker autodiscovery feature of Traefik to match a domain to a service:

```yml
  whoami:
    # (...)
    labels:
      - traefik.http.routers.whoami-https.rule=Host(`whoami.traefik.local`)
      - traefik.http.routers.whoami-https.tls=true
  snowflake:
    # (...)
    labels:
      - traefik.http.routers.snowflake-https.rule=Host(`snowflake.traefik.local`)
      - traefik.http.routers.snowflake-https.tls=true
```

Refer to [Traefik documentation](https://docs.traefik.io/configuration/entrypoints/#dynamic-certificates) for additional details on their SNI support.


## Implementation sketch

How does Traefik implement this? Go allows to specify a custom function `func getCertificate(hello *tls.ClientHelloInfo) (*tls.Certificate, error)`. This function is often referred to as SNI callback. The function will be called on each `ClientHello` and allows you to implement your own certificate selection mechanism.

The Traefik implementation looks like this [src](https://github.com/containous/traefik/blob/ea750ad/pkg/tls/tlsmanager.go#L83):

```go
var tlsConfig *tls.Config  // Construction details not relevant here.

tlsConfig.GetCertificate = func(clientHello *tls.ClientHelloInfo) (*tls.Certificate, error) {
    domainToCheck := types.CanonicalDomain(clientHello.ServerName)  // (1)

    // (2) (...) special handling if we use let's encrypt --- not relevant here.

    bestCertificate := store.GetBestCertificate(clientHello)  // (2)
    if bestCertificate != nil {
        return bestCertificate, nil
    }

    if m.configs[configName].SniStrict {  // (3)
        return nil, fmt.Errorf("strict SNI enabled - No certificate found for domain: %q, closing connection", domainToCheck)
    }

    log.WithoutContext().Debugf("Serving default certificate for request: %q", domainToCheck)
    return store.DefaultCertificate, nil  // (4)
}

// The tlsConfig is eventually passed to `tls.Server(conn net.Conn, config *Config)`
```

We can see the high-level algorithm:
- Normalize the domain name (1).
- Find the *best* certificate for a given domain (2). The definition of *best* can depend on the application, be we'd usually prefer an exact match over a wildcard cert. `store` abstracts a certificate store. We can cache/reload the certificates there.
- If no certificate matches, we may want to drop the connection right away (3), or continue with the default cert for the server (4).


### Alternative use case: certificate rotation

An interesting alternative use case for `GetCertificate` hook is the *certificate rotation*. Every certificate eventually expires, and we need to rotate it. If we rely on the default certificate passed directly via `tls.Config` then we need to restart our app each time the certificate changes. We either loose requests or need a complex infra to handle no-downtime rotation in such a scenario.

An alternative would be to wrap our cert in a container, and instead of providing it directly, use the `GetCertificate` to grab it from the container. Then we are free to replace it in the background without missing a request.

[*Hitless go certificate rotation*](https://diogomonica.com/2017/01/11/hitless-tls-certificate-rotation-in-go/) offers more details on this use case.


## Summary

We discussed Server Name Indication and showed an example of Traefik acting as reverse proxy making use of SNI. We discussed how to implement similar behavior in go. Finally, we've described how SNI hooks can be useful beyond many-domains, single-server use case --- for no-downtime certificate rotation.

Thanks for reading, feel free to drop a comment if you have any suggestions on how to improve this post :)
