---
title: Set up AWS Lambda with terraform
tags:
- cloud
aliases:
- /2019/01/19/aws-lambda-terraform.html
---

In [the previous post](/2019/01/09/invalidate-cloudfront-with-lambda-s3.html),
we've presented an AWS Lambda function to automatically invalidate resources in
CloudFront distribution when underlying objects in an S3 bucket change.

Instead of clicking and copy-pasting in the AWS Console, we can use
[terraform](https://www.terraform.io/) to set this function up. In fact, I've
already made it a part of my [terraform static aws
website](https://github.com/igor-kupczynski/terraform_static_aws_website) ---
terraform module which sets up an S3 bucket to host a static website and
CloudFront as a cache; it also handles a redirect `www.domain.com -->
domain.com` and, provided with an AWS generated https cert, the `https://` bit.

The file
[invalidate_cache.tf](https://github.com/igor-kupczynski/terraform_static_aws_website/blob/master/invalidate_cache.tf)
sets the lambda up. Please consult it for all the details. We'll focus on the
big picture here.

Let's go through the resource we need to define.

![Lambda resource diagram](/archive/2019-01-lambda-terraform-resources.png)

- `resource "aws_lambda_function" "invalidate_cloud_front_lambda"` represents
  the lambda function. It specifies the function name and any extra environment
  variables. In our case the ID of CloudFront distribution, so that the lambda
  knows which of the distributions in our account to invalidate.

- The lambda function resource needs to reference a zip file bundling the code.
  We store the source code directly, and then use `data "archive_file"
  "invalidate_cloud_front_lambda_zip"` to create the zip file for us.

- By default everything is locked, a lambda doesn't have the permissions needed
  to invalidate any CloudFront distributions at all. We need to craft a policy
  that will let our lambda invalidate the distribution we want. Enter `resource
  "aws_iam_policy" "invalidate_cloud_front_policy"`
  
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
        {  // (1)
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {  // (2)
            "Effect": "Allow",
            "Action": [
                "cloudfront:CreateInvalidation"
            ],
            "Resource": [
                "*"   // <-- in fact, we should be more restrictive here
            ]
        }
    ]
  }
  ```
  
  The policy does two things: (1) it allows writing CloudWatch logs and (2) it
  allows to create invalidations in CloudFront distributions. We could split
  the policy into two distinct ones, esp. if we have more lambda and plan to
  reuse parts of the policy, but for our simple case, this is OK.
  
  The policy can't be associated directly with the lambda. We need to create a
  role `resource "aws_iam_role" "invalidate_cloud_front_role" `, which we then
  link to the policy via `resource "aws_iam_role_policy_attachment"
  "invalidate_cloud_front_attachment"`. Finally, the role is being referenced in
  the lambda resource.

- Our lambda can write log items to CloudWatch and create CloudFront
  invalidations, but it also needs to tap to the bucket event stream. This is
  being taken care of by `resource "aws_s3_bucket_notification"
  "bucket_notification"`. It specifies the bucket and the types of events that
  will be delivered to the lambda function. Also, we need to let the S3 bucket
  call the lambda function (because of *security*). We do it with `resource
  "aws_lambda_permission" "allow_s3_event_notifications"`.
  
In the short post, we've described AWS resources which we need to create for a
simple lambda function to work. I hope this is useful to understand what happens
*behind the hood* or if you want to terraform your own lambda.
  
  
  
