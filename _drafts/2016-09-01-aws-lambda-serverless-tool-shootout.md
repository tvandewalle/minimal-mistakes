---
header:
    overlay_image: aws-lambda.jpg
    overlay_filter: 0.6
excerpt:
    This is a series of posts aimed at building a medium-ish authentication
    microservice using API Gateway, Lambda, and DynamoDB. The
    series examines  how three open source projects
    [Kappa](http://kappa.readthedocs.io/en/develop/),
    [Gordon](https://github.com/jorgebastida/gordon), and
    [Serverless](https://github.com/serverless/serverless) can assist in this
    complex task.
title: "AWS Lambda Serverless Tool Shootout"
tags:
    - AWS
    - Serverless
categories:
    - Hacking
---

## Series Background
The service is inspired by some of the work done by my team at
[Findaway](http://findaway.com) in developing an authentication microservice
that will be used by several new products and product extensions. The code here
is simpler, but the general principals and challenges with API Gateway, Lambda,
and DynamoDB are the same. Additionally all of the code is written in Python so
the examination of the tools is slanted that direction.

This is deliberatley not about only using API Gateway with Lambda functions.
While API Gateway and Lambda is a powerful (and popular) combination Lambda
functions have a much broader set of use cases. For example we used Lambda and
S3 to create an mp3 encoding farm that scaled on demand and allowed us to
encode and normalize several million mp3 files very quickly and at a modest cost.
I did not want to limit the tool shootout to those specifically designed for
creating Web Services only.


## AWS Lambda Background
AWS introduced Lambda at reinvent:2014 and the developer community was
immediately excited about the possibilities (and about the potential money
savings). Since then AWS has introduced API Gateway, bumped the time limit on
Lambda functions, introduced additional native runtimes and added more event
sources. With all of this advancement, the tooling provided by AWS has remained
the same and is one of the more significant barriers to getting productive in
this environment quickly.

Several open source projects have developed over the last couple of years to
help address the complex multi-step process that is necessary to develop and
deploy even the simplest Lambda functions. Mitch Garnaat, the author of Kappa
(one of the oldest tools in the serverless category), sums up the steps for
deploying a Lambda function to AWS simply as:

1. Write the function itself.
2. Create the IAM role required by the Lambda function itself (the executing
   role) to allow it access to any resources it needs to do its job. Add
   additional permissions to the Lambda function if it is going to be used in a
   push model (e.g. S3, SNS) rather than a pull model.
3. Zip the function and any dependencies and upload it to AWS Lambda.
4. Test the function with mock data.
5. Retrieve the output of the function from CloudWatch Logs.
6. Add an event source to the function.
7. View the output of the live function.

That's a lot of steps and having tooling built around automating this sure
would be nice. There are many other limitations to Lambda functions that might
not make it the right job for your task, but this series of posts isn't about
tradeoffs when thinking about a serverless architecture or provider. For those
topics take a look at TODO: Insert some links.

## The Challenge
In order to go a little deeper in understanding the pros and cons of each of
these tools I've chosen to implement an authentication microservice using API
Gateway, Lambda, and DynamoDB. The authentication microserivce will perform the
following functions:

1. Support creation of users and secure storage of passwords.
2. Support authentication of users and corresponding JSON web tokens.
3. Support returning HTTP 404 status if authentication fails.
3. Support deletion of users.

While this is not a full featured authentication service, it serves as a good
sized project for understanding the pros and cons of each of the chosen Lambda
tools. The project includes: 

* Developing and testing the Lambda function
* Packaging and deploying the Lambda function.
* Integrating with external services.
* Creating an API.
* Deploying the complete package in multiple environments (production, staging,
  etc.).

## The Criteria
No reviews are completely independent. They all come with colored with the reviewer's experience and point of view on how the world should work. With that in mind, here are some of the criteria and biases that I bring to this:

* I generally prefer Cloudformation when creating resources as it allows nice
  uniform packaging of things (and cleanup when you are done). It is not
  without its flaws however, so I am willing to choose something else.
* Testing matters to me. If testing your project is difficult you won't do it.
* Logging matters to me. If logging is difficult then you won't be able to find
  out what's broken later.
* Production management matters to me. If you can't deploy and roll-back with
  confidence then you will be afraid to deploy and innovation will move
  slowly.
* I write most things in Python so if the tool doesn't support Python well, I'm
  out. This is not to knock other languages, it's just the one I'm most
  comfortable with and that my team uses most often.

## The Challengers
I'll be building this project with 3 different packages in order roughly of
increasing capability and complexity:

1. [Kappa](http://kappa.readthedocs.io/en/develop/) - The original tool on the block. Kappa is geared primarily at building and deploying lambda functions. Everything else happens outside of Kappa.
2. [Gordon](https://github.com/jorgebastida/gordon) - One of the newer kids on the block and with some more extensive features. Gordon aims to integrate more of AWS into the tool.
3. [Serverless](https://github.com/serverless/serverless) - It's been around for sometime (it used to be called [JAWS](https://aws.amazon.com/blogs/compute/getting-started-with-jaws-on-amazon-web-services/)) and is ambitious. It aims to integrate many services and simplify the workflow of managing services in complex environments.

## Next
Part 2 of the series will take on the challenge with Kappa. All of the code and
supporting infrastructure definition will be shared on github for persual.
Enjoy!
