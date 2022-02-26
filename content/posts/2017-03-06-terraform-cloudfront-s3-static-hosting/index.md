---
title: How to put AWS Cloudfront in front of S3 hosting a static website
tags: []
aliases:
- /2017/03/06/terraform-cloudfront-s3-static-hosting.html
---
Some AWS API quirks make it hard to use folder level index documents via terraform.

If you use a static blogging software
like [jekyll](https://jekyllrb.com) or [hugo](https://gohugo.io) you
may be tempted to host them on S3. It is simple, cheap and has a great
availability (lets
ignore
[the recent us-east-1 incident](https://news.ycombinator.com/item?id=13755673) for
now).

The static blogging software relies on the fact that you can use
`index.html` as a folder root resource. For example if you want to
check the articles tagged as `elasticsearch` on this blog under the
`/tag/elasticsearch/` url you are in fact viewing
`/tag/elasticsearch/index.html` generated by jekyll. Lucikly for us it
is easy to set up S3 in a *website* mode which does just that.

The terraform config may look like this:

```
resource "aws_s3_bucket" "storage_bucket" {
  bucket = "${var.domain}"

  website {
    index_document = "index.html"
  }
  
  # (...) ommited for brevity
}
```

As you can see we specify the `index_document` which acts as a folder
index document. This results in a following bucket, which responds
with `/folder/index.html` when you ask for `/folder`.

Note that this is not an http redirect.

![S3 Bucket](/archive/2017-03-bucket.png)

Please also note that we have to access this bucket by through its
website endpoint `${bucket-name}.s3-website-us-east-1.amazonaws.com`
otherwise the `index_document` feature is not going to work.

As a next step we are tempted to use a CDN, like AWS Cloudfront.

Again, the config seems to be quite simple.

```
resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = "${aws_s3_bucket.storage_bucket.bucket_domain_name}"
    origin_id   = "origin-${var.domain}"
  }

  default_root_object = "index.html"
  
  # (...) ommited for brevity
}
```

If you use AWS console instead of terraform you will end up with
something similar. Unfortunatelly, this doesn't work well with
static websites relying on the fact that for folder resources their
`index.html`s are displayed.

The issue with this config is the different way of handling *root
objects* (or *index documents* if you will) between S3 in website mode
and Cloudfront - the former applies it to all folders but the latter
only to the root resource and not for the
folders. [As per the docs](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DefaultRootObject.html)

> However, if you define a default root object, an end-user request
> for a subdirectory of your distribution does not return the default
> root object. (...) CloudFront will not return the default root
> object even if a copy of index.html appears in the install
> directory.

To solve this problem we need to point Cloudfront directly to the
website endpoint of the S3 bucket. We can change the

    domain_name = "${aws_s3_bucket.storage_bucket.website_endpoint}"`.
    
Simple, right? Unfortunately, this results in an error:

    The parameter Origin DomainName does not refer to a valid S3 bucket.

Seems this is an AWS API idiosyncrasy. By default it insists on
pointing Cloudfront to the S3 domain endpoint, not the website
endpoint.

Adding some secret source and our CDN is ready

```
resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = "${aws_s3_bucket.storage_bucket.website_endpoint}"
    origin_id   = "origin-${var.domain}"

    custom_origin_config {
      origin_protocol_policy = "http-only"
      http_port = "80"
      https_port = "443"
      origin_ssl_protocols = ["TLSv1"]
    }
  }

  default_root_object = "index.html"

  # (...) ommited for brevity
}
```

Specifying `custom_origin_config` forces AWS API to consider the S3
bucket a proper website and correctly forwards the `/folder` requests
to S3 website endpoint which responds with the content of
`/folder/index.html`.

[This stackoverflow answer](http://stackoverflow.com/a/40096056)
helped to solve the mystery.

By the way, you can find my terraform module that I use to host this
blog (and some other websites) on my [github](https://github.com/igor-kupczynski/terraform_static_aws_website)