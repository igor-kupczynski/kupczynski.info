---
title: Private Link is the IP filtering of the cloud
tags:
- privatelink
- network
aliases:
- /2022/01/30/privarte-link.html
---
Use cases for Private Link and differences in its implementation across the major Cloud Providers

## Private Link

Private Link allows secure connectivity to services hosted on different Cloud provider accounts.

* Private Link allows you to connect to a service in a different VPC—and that VPC can be owned by a different account.
* The connection is secure[^1]:
	* It doesn't leave the cloud provider's network.
	* Data exchanged over Private Link is encrypted (I wasn't able to find much on the encryption in the documentation—it makes sense to run secure the connection with TLS).
	* You can use native features, such as IAM and security policies, to limit who can access the connection.
* The connection is unidirectional.
	* Resources in the source VPC can access a dedicated service in the target VPC.
	* Other resources in the target VPC are not exposed.
	* Resources in the source VPC are not exposed.

![Private Link](/archive/2022-01-private-link-basic.png)

* The service provider creates a _Private Link Service_ with a globally unique _Alias_.
* The consumer creates an _Endpoint_. It uses the _Alias_ of the _Service_ to identify the target. It has a static IP address and a randomly generated DNS name.

**Note on terminology**
* _Private Link_ is not a standard name, it is a service from AWS. It is called _AWS PrivateLink_. AWS was first to offer such a service and then the name caught on. Azure calls their equivalent _Azure Private Link_ and GCP has _Private Service Connect_ (or PSC).
* Private Link can be also used to access cloud provider services (such as S3), and others. Here we are focused on the SaaS service provider use case.

## Use case

We run a SaaS company. We offer a simple service to send emails:

```sh
curl -XPOST -H 'Authorization: ...' https://customerA.my-saas.io/emails -d '{
	"from": "alice@customerA.com",
	"to": "bob@example.com"
	"subject": "foo",
	"body": "bar"
}'
```

The service is easy to use and developer-friendly. It catches on.

### Security comes in layers

Cecilia from MegaCorp, Inc. wants to use `my-saas.io` in her latest project. This is a big contract for us! However, she adheres to [_defense in depth_](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)).

The base URL of her account `https://megacorp.my-saas.io` is easy to guess. Even if we switch the _vanity_ domain `megacorp.my-saas.io` for something more random, like `db1534b8-753f-4531-b7dc-fefb3d6f53f1.my-saas.io` it can leak. The host is present in the DNS queries, [SNI headers](https://www.cloudflare.com/learning/ssl/what-is-sni/), request logs, etc.

Cecilia would like us to drop any connections to `https://megacorp.my-saas.io` unless they come from their VPC.

## Solution sketch

### PROXY protocol v2

How can we attribute connections to specific customers? All the connections come from the same IP address (that one of the _Private Link Service_).

PROXY protocol v2 (or PP2 for short) to the rescue. Cloud providers allow us to enable PP2 on the service. Each connection will be annotated with extra attributes:
* **Source IP address** of the connection.
	* Note that it is in the source VPC private address space, this is not useful for traffic filtering.
* **VPC Endpoint ID**. This is a unique identifier of the customer's *VPC Endpoint*. Different cloud providers use a different name for it, we will call it a *link ID* in the post.

What's a PP2? It's a protocol popularized by the HAProxy. Quoting their [docs][pp2]:

> The PROXY protocol provides a convenient way to safely transport connection information such as a client's address across multiple layers of NAT or TCP proxies. It is designed to require little changes to existing components and to limit the performance impact caused by the processing of the transported information.

In practice, PP2 adds a binary (but unencrypted) blob at the start of the byte stream. Since it goes at the start of the connection, it will go even before the TLS handshake. You want to consume it, and then hand the rest of the byte stream over to the http library/application logic.

[pp2]: http://www.haproxy.org/download/2.4/doc/proxy-protocol.txt

Note that PP2 is an option. By default it is disabled and you need to explicitly enable it.

### Back to our solution

1. Customers fill in their *link IDs*.
2. We add a proxy (can work at L7 or L4):
 	1. It reads the PP2 header to understand the source of the connection.
	 	1. Public internet connections have no *link IDs* and can be processed the same way.
	2. It reads the SNI header to understand the target resource.
	3. If the target resource set up a _link ID_, then it compares if the source _link ID_ matches:
		1. **Match** ==> Accept the connection and route it to the target.
		1. **No match** ==> Drop the connection.

All connections to customer resources are dropped, except if they originate from their configured VPC.

![Private Link](/archive/2022-01-private-link.png)


### Similarities with the *classic* IP filtering

![IP filtering](/archive/2022-01-ip-filtering.png)

Conceptually, IP filtering is a similar solution. It is simpler, as source IPs are part of the TCP/UDP connections, no need to involve PP2. The connections are direct, with no extra *Services* or *Endpoints*.

In the cloud this is not feasible, there are no static IPs on either side:

* We have managed load balancers in front of your infra and you don't control their IPs. We (or our cloud provider) autoscale them adding/removing new IP addresses.
* Customers often don't have static IPs for their egress traffic.


### Alternative solution: VPC peering

VPC peering allows two VPCs to share network address space—similarly to how you may use a switch to join two network segments.

![VPC peering](/archive/2022-01-vpc-peering.png)

_Diagram from [https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)_

This sounds like a good idea, but has a couple of drawbacks for our SaaS use case:

1. The private IPs between the VPCs can't overlap, which means you need to coordinate the address space with all your customers.
2. The access is bidirectional as if the resources were in the same network.
    - You can and should set up access-control rules, but an error in them may result in your customers being able to access your infrastructure and/or you being able to access your customers' infrastructure.

## Security considerations

Can you trust the PP2 header? Can it be spoofed?

The scenario we need to watch out for is someone adding their own PP2 header, and therefore spoofing the connection source.

- The Private Link traffic has the PP2 headers always added (if we enable it). Even if the user adds their PP2 header in front of the connection, the cloud provider will add another one in front of that. We can parse only the first one, and be sure that it comes from a trusted third party.

- The public internet traffic is trickier. As far as I know, only AWS allows adding PP2 headers to public internet traffic[^2].

	* **AWS**: The same NLB can be set up to receive public internet traffic and Private Link Service traffic. All the connections made through this NLB will have a PP2 header added by a trusted third party.
	* **Azure**: We need a different LB for public internet traffic[^3]. The traffic from the public internet doesn't carry PP2 headers. You should configure your proxy in such a way that it trusts the PP2 headers from the Private Link LB and ignores any headers from the public internet LB.
	* **GCP**: Similarly to Azure, the traffic from the public internet doesn't get GCP-added PP2 headers.


## Differences between cloud providers

|  | AWS | Azure | GCP |
|:--|:--|:--|:--|
| Name | AWS PrivateLink | Azure Private Link | GCP Private Service Connect |
| Allows cross-region connections? | No | Yes | No |
| PP2 header format | [Link](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#custom-tlv) | [Link](https://docs.microsoft.com/en-us/azure/private-link/private-link-service-overview#getting-connection-information-using-tcp-proxy-v2) | [Link](https://cloud.google.com/vpc/docs/configure-private-service-connect-producer#proxy-protocol) |
| Learn more | [PrivateLink product page](https://aws.amazon.com/privatelink/) | [Private Link overview and docs](https://docs.microsoft.com/en-us/azure/private-link/private-link-overview) | [Private Service Connect guide](https://cloud.google.com/vpc/docs/private-service-connect) |

If you want to parse the PP2 headers in go—or compare notes on implementation—check [`tlvparse` package in pires/go-proxyproto](https://github.com/pires/go-proxyproto/tree/main/tlvparse). A go library which supports all of them.

### Quirks

#### At least one overlapping AZs is required in AWS

Both the source and the target VPCs need at least one overlapping Availability Zone to form the Private Link connection.

Check the [limitations](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html#vpce-interface-limitations) section of the docs, esp.

> When the service provider and the consumer have different accounts and use multiple Availability Zones, and the consumer views the VPC endpoint service information, the response only includes the common Availability Zones. For example, when the service provider account uses `us-east-1a` and `us-east-1c` and the consumer uses `us-east-1a` and `us-east-1b`, the response includes the VPC endpoint services in the common Availability Zone, us-east-1a.

Note that the letters describing the AZs (e.g., **a** in us-east-1**a**) are randomized between the accounts. AWS internally uses numbers to represent the AZs and randomize the number => letter mapping per account[^4].

You can check your AZ mapping with the CLI:

```sh
$ aws ec2 describe-availability-zones --region us-east-1 | jq -c '.AvailabilityZones[] | { id: .ZoneId, name: .ZoneName } ' | sort
{"id":"use1-az1","name":"us-east-1c"}
{"id":"use1-az2","name":"us-east-1e"}
{"id":"use1-az3","name":"us-east-1d"}
{"id":"use1-az4","name":"us-east-1a"}
{"id":"use1-az5","name":"us-east-1f"}
{"id":"use1-az6","name":"us-east-1b"}
```


### Azure requires different LBs

You need a separate LB to handle Private Link traffic and public internet traffic. As soon as you attach Private Link Service to your LB it stops serving public internet traffic. At least this is what I've found in my tests, as far as I know, this limitation is not documented.

Create two different LBs and test the configuration in a pre-prod account before changing the production configuration.

### DNS support

How do customers access our service over Private Link? The endpoints get a static IP (from the source IP subnet) and a DNS alias.

```sh
curl -XPOST -H 'Authorization: ...' https://10.0.10.63/emails -d '{  # How do we pass the domain?
  …
}'
```

The problem is that `10.0.10.63` (or a random DNS alias) won't match our TLS certs for `*.my-saas.io`.

This means that our Private Link customers need to manually create DNS records pointing from `10.0.10.63` => `*.my-saas.io`"

```sh
curl -XPOST -H 'Authorization: ...' https://megacorp.my-saas.io/emails -d '{
  …
}'
```

How do they know that everything is configured correctly? If they forgot the DNS record the `megacorp.my-saas.io` will connect to their resources over the public internet. This will likely result in a connection drop, which should hint at the problem.

Another suggestion would be to use a different domain—unresolvable from the public internet—for the Private Link traffic.

#### Private DNS integration

The cloud providers offer automatic DNS integration. You can configure your Private Link Service, that any endpoint connecting to it will automatically add `my-saas.io` (or other domain) to the private DNS of its VPC[^5].

None of the providers support wildcard domains. In our example, we use the domain to differentiate between customers, but if we can use a different mechanism, e.g., http header, or part of the request, then we can simplify the Private Link setup for our customers using the *Private DNS integration*.

```sh
curl -XPOST -H 'Authorization: ...' https://my-saas.io/emails -d '{
	"account": "megacorp",
  …
}'
```

## Summary

We've learned about Private Link and how it can act as an *IP filtering* for the Cloud. We've presented a fictional SaaS company, and solved their use case with Private Link. We've discussed security considerations, differences in implementation, and quirks of different cloud providers.

### Resources

- See the *Learn more* section in the table above.

- [Elastic Cloud announces their support for Private Link](https://www.elastic.co/blog/improve-network-security-elastic-cloud-traffic-filters) (disclaimer: I work for Elastic when I write this; I've implemented a large part of the backend support for Elastic Cloud Private Link). This is an example of how Private Link is useful for SaaS providers. More examples:
	- [MongoDB Altas Private Link support](https://docs.atlas.mongodb.com/security-private-endpoint/).
	- [Confluent Cloud Private Link support](https://docs.confluent.io/cloud/current/networking/private-links/azure-privatelink.html).
	

[^1]: See [Q: How secure is an AWS PrivateLink connection?](https://aws.amazon.com/privatelink/faqs/)
[^2]: See [Proxy protocol](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#proxy-protocol).
[^3]: I haven't seen this documented anywhere, but as soon as we associate a private link service with an Azure Load Balancer, the public internet traffic stops flowing. This is worth a separate investigation and a blog post.
[^4]: Learn more about the ID to name mapping in the [docs](https://docs.aws.amazon.com/ram/latest/userguide/working-with-az-ids.html).
[^5]: Learn more [VPC Endpoint Private DNS](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html#vpce-private-dns).
