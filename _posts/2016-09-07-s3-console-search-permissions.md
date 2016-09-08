---
header:
    overlay_image: console_snippet.png
    overlay_filter: 0.6
excerpt:
    Getting the right permissions for Read-Only access to an S3 bucket
    while allowing search to work in the AWS console.
title: "S3 IAM Permissions for Read-Only and Console Search"
tags:
    - AWS
    - Tips
categories:
    - Hacking
---


> **tl;dr** You need to add `s3:GetObjectAcl` permission to the resource
  `arn:aws:s3:::<BUCKET-NAME>/*` if you want the filter/search box to work
  in the S3 console

## Motivation

This past week I spent far too long trying to provide read-only access to an S3
bucket while still allowing the user to use the search box in the AWS console.
This should have been very straightforward, following the examples provided by
the AWS team. Unfortunately I wound up spending time triangulating on the right
permissions by opening them wide up and then successively narrowing them down
until they worked.

We have a bucket that contains approximately 200,000 items that are used by our
production image server.  Members of our team need read-only access for
customer support purposes. They are not regular users of the AWS console so
limiting access helps protect us against inadvertant button-clicks.
Additionally scrolling through a couple hundred thousand objects in the S3
console is not practical, but there's a nice filtering/search box on the
console that makes finding items simple.

Seems like the perfect solution, so I set about creating a policy to match what
we were trying to accomplish. The first attempt at the policy was what I
naively thought should work:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ConsoleListBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListAllMyBuckets"
            ],
            "Resource":
                [ "arn:aws:s3:::*" ]
        },
        {
            "Sid": "ListItemsInBucket",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource":
                [ "arn:aws:s3:::<BUCKET-NAME>" ]
        },
        {
            "Sid": "GetItemsInBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource":
                [ "arn:aws:s3:::<BUCKET-NAME>/*" ]
        }
    ]
}
```

This policy is pretty much straight from an [AWS Blog
Post](https://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket)
on the topic. I applied this policy to the group that would be managing the
customer support process and went to test with a member of the team. He was
able to browse the S3 bucket no problem, but when I was demonstrating the
search feature (by searching for an item I knew existed), I got no response from
the console. It didn't respond with "No, Results", there was no response at
all.

I examined the javascript console in Chrome Dev Tools and found several erorr
messages. After ruling out browser incompatibility I thought maybe there was a
permissions problem, although I could not find a specific mention of why this
would be. I openened up the permissions on the policy (to `s3:Get*`) and the problem went
away, so it became a matter of trial and error to find the correct permissions.
None of the permission names were obviously needed for search from the AWS
console.

It turned out that adding `s3:GetObjectAcl` was the missing piece. It's still
not clear to me why this is needed, but I do know that it worked and solved the
problem.

Here's a final sample policy that works (substitute your bucket names of
course):

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ConsoleListBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListAllMyBuckets"
            ],
            "Resource":
                [ "arn:aws:s3:::*" ]
        },
        {
            "Sid": "ListItemsInBucket",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource":
                [ "arn:aws:s3:::<BUCKET-NAME>" ]
        },
        {
            "Sid": "GetItemsInBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectAcl"
            ],
            "Resource":
                [ "arn:aws:s3:::<BUCKET-NAME>/*" ]
        }
    ]
}
```

Hopefully this saves you a little bit of time if you run across this.
