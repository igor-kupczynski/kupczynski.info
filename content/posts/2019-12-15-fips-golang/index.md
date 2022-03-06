---
title: FIPS compliant crypto in golang
tags:
- golang
aliases:
- /2019/12/15/fips-golang.html
---

If you work with US government entities or corporations in regulated markets the subject of FIPS compliance may come up. [FIPS 140-2](https://en.wikipedia.org/wiki/FIPS_140-2) is a set of cryptographic standards that your application may need to adhere to.

## FIPS in go

It takes  a lot to be FIPS verified or FIPS compliant ([learn more](https://blog.ipswitch.com/fips-validated-vs-fips-compliant)), but for us the bottom line is that our app must use FIPS verified crypto libraries. This is a challenge with go, where the native crypto is not FIPS friendly.

There are [no plans to change it either](https://github.com/golang/go/issues/11658#issuecomment-120441723):

> Go's crypto is not FIPS 140 validated and I'm afraid that there is no possibility of that happening in the future either. I think Ian's suggestion of using cgo to call out to an existing, certified library is probably your best bet. However, we would not be interested in patches to add hook points all over the Go library, so you would need to carry that work yourself.

Reimplementing crypto with cgo call outs doesn't seem like a fun activity. Are we out of luck? Rewrite in Java? Luckily there are two options for us:

- BoringSSL based crypto.
- RedHat go toolchain.

Let's discuss them.


## Boring gets interesting

> BoringSSL is a fork of OpenSSL that is designed to meet Google's needs.

Critically, it has a [FIPS 140-2 verified version](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/security-policies/140sp2964.pdf).

BoringSSL is [used internally in google's monorepo](https://www.imperialviolet.org/2015/10/17/boringssl.html). [Cloudflare is another prominent user](https://blog.cloudflare.com/make-ssl-boring-again/).

`dev.boringcrypto` is a go repo branch. It is maintained alongside the mainline go (quote from its [readme](https://github.com/golang/go/blob/dev.boringcrypto/README.boringcrypto.md)):

> We have been working inside Google on a fork of Go that uses BoringCrypto (the core of BoringSSL) for various crypto primitives, in furtherance of some work related to FIPS 140-2. We have heard that some external users of Go would be interested in this code as well, so I intend to create a new branch dev.boringcrypto that will hold patches to make Go use BoringCrypto.
>
> Unlike typical dev branches, we do not intend any eventual merge of this code into the master branch. Instead we intend to maintain in that branch the latest release plus BoringCrypto patches. In this sense it is a bit like dev.typealias holding go1.8+type alias patches.

This sounds good, but in an OSS fashion there are no guarantees:

> To be clear, we are not making any statements or representations about the suitability of this code in relation to the FIPS 140-2 standard. Interested users will have to evaluate for themselves whether the code is useful for their own purposes.

The patches are kept up-to-date against minor version releases and master. [List of branches](https://github.com/golang/go/branches/all?query=dev.boringcrypto).

### Build process

Google maintains [docker images with patched go toolchain](https://github.com/golang/go/blob/dev.boringcrypto.go1.12/misc/boring/README.md), similar to `golang:x.y.z`.

> A Dockerfile that starts with FROM golang:1.8.3 can switch to FROM
> goboring/golang:1.8.3b2 (see goboring/golang on Docker Hub) and should
> need no other modifications.
> (...)
> 
> Caveat
>
> BoringCrypto is used for a given build only in limited circumstances:
>
> - The build must be GOOS=linux, GOARCH=amd64.
> - The build must have cgo enabled.
> - The android build tag must not be specified.
> - The cmd_go_bootstrap build tag must not be specified.
> - The version string reported by runtime.Version does not indicate that
>   BoringCrypto was actually used for the build. For example, linux/386 and
>   non-cgo linux/amd64 binaries will report a version of go1.8.3b2 but not be
>   using BoringCrypto.
>
> To check whether a given binary is using BoringCrypto, run `go tool nm`
> on it and check that it has symbols named `*_Cfunc__goboringcrypto_*`.

### Code changes

The patched toolchain is a true drop-in replacement, no changes to the code are needed (maybe with an exception of filtering out unsupported ciphers, etc. but this is a configuration concern).

### Performance impact

The foreign function interface between golang and c is not free. The library performs worse than the build-in crypto. See [golang/go#21525](https://github.com/elastic/cloud/pull/golang/go#21525).

> In general there is about a 200ns overhead to calling into
> BoringCrypto via cgo for a particular call. So for example
> aes.BenchmarkEncrypt (testing encryption of a single 16-byte block)
> went from 13ns to 209ns, or +1500%. That we can't do much about except
> hope that bulk operations call into cgo once instead of once per 16
> bytes.

Note that the benchmarks were done in 2017, so the things might've improved (but then vanilla tls/crypto improved as well). Also note the detailed benchmark results, not all ciphers performed as poorly as the highlight. Some where on the same level or faster. If you don't care about every nanosecond of performance or your code is not on the critical path then this is good enough. Otherwise benchmark your usecase to see if the performance hit is acceptable.

### Boring Example

I've created a [simple echo server in go](https://github.com/igor-kupczynski/fips-echo-server/tree/boringcrypto) to show what changes are needed to switch to the `dev.boringcrypto`. Check the diff between the "regular" (or "native") crypto and the boring version: <https://github.com/igor-kupczynski/fips-echo-server/compare/master...boringcrypto>:

> The boringssl based toolchain is basically a drop in replacement. All we need to do is:

```diff
--- Dockerfile	(master)
+++ Dockerfile	(boringcrypto)
@@ -1,14 +1,16 @@
 # Start with an official image
-FROM golang:1.13.4
+FROM goboring/golang:1.13.4b4
```

> There are some other changes in the branch, to add new option `-fipsMode`, which resets the ciphersuite to a FIPS compliant one. You may want to do something else, like still allow user configured ciphers as long as they are a subset of FIPS compliant ones, etc. But these changes are UX improvements when running in FIPS mode. No "core" app changes required.

> `testssl` reports this:

```
...
 Testing server preferences

 Has server cipher order?     yes (OK) -- TLS 1.3 and below
 Negotiated protocol          TLSv1.3
 Negotiated cipher            TLS_AES_128_GCM_SHA256, 253 bit ECDH (X25519)
 Cipher order
    TLSv1.2:   ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES256-GCM-SHA384 AES128-GCM-SHA256 AES256-GCM-SHA384
    TLSv1.3:   TLS_AES_128_GCM_SHA256 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_256_GCM_SHA384
...
```

## RedHat go toolchain

[RedHat offers an alternative](https://developers.redhat.com/blog/2019/06/24/go-and-fips-140-2-on-red-hat-enterprise-linux/).

> We are excited to announce that we plan to ship go-toolset with a new feature that allows Go to bypass the standard library cryptographic routines and instead call into a FIPS 140-2 validated cryptographic library.

I think they leverage some of the `dev.boringcrypto` patches internally:

> This new feature builds on top of pre-existing upstream work (which instead calls into BoringSSL) ...


The major differences seem to be:

1. RedHat offers support contracts and FIPS mode for RHEL, so they are committed to supporting their FIPS complaint go toolchain.
2. FIPS mode can be enabled at runtime.
3. Older go version — 1.11 at the time of writing, they promise to upgrade to 1.12 in 2019. `dev.boringcrypto` is being kept up-to-date against master (but there are no guarantees it will be like that in the future).


## Summary

The native go crypto is not FIPS compliant, nor it will be in a foreseeable future. Luckily, both Google and RedHat provide go toolchains backed by FIPS validated SSL libraries. We can leverage them to make our go app FIPS compliant.
