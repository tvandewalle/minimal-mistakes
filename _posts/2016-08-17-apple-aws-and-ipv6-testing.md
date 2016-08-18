---
header:
    overlay_image: mobile-laptop.jpg
    overlay_filter: 0.6
title: "Apple, AWS and IPv6 Confusion"
tags:
    - AWS
    - Apple
categories:
    - Hacking
---

AWS recently announced [support for IPv6 in
S3](https://aws.amazon.com/blogs/aws/now-available-ipv6-support-for-amazon-s3/).
Comments on Hacker News linking this to Apple's IPv6-only network support
requirement lead me to believe that there is some misunderstanding around this
requirement.

At [WWDC 2015 Apple announced](https://developer.apple.com/news/?id=05042016a): 
"Starting June 1, 2016 all apps submitted to
the App Store must support IPv6-only networking." If you host your mobile
backend on AWS you might be concerned because of Amazon's slow adoption of
IPv6.

> *tl;dr* If your backend is being served on AWS infrastructure you don't have
  to worry about Apple's new IPv6 testing requirement, unless you have some
  IPv4 only code in your app.

All of the information in this post comes from my experience in getting an App
through approval in July of 2016. My company
[Findaway](http://www.findaway.com) hosts all of our backend services on AWS
and still managed to push an App through the approval process (though not
without some challenges).

## AWS and IPv6 Support

AWS has been *very* slow in adopting IPv6. At the time of this writing AWS has
[IPv6 support for S3](https://aws.amazon.com/blogs/aws/now-available-ipv6-support-for-amazon-s3/)
(under certain circumstances) and [ELBs deployed in EC2 Classic](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-internet-facing-load-balancers.html).
So what does it mean that AWS has "limited support" for IPv6. For
the purposes of this discussion it means that AWS doesn't offer IPv6 addresses
for the vast majority of it's services that developers of mobile backends use.
For example, Cloudfront, ELBs in VPC, and EC2 instances all only return IPv4
addresses.

While this limited support is not helpful when you are trying to test your App
to ensure that it will pass the new IPv6 requirment, it does not mean that your
backend will cause failures in testing. The test that Apple performs during App
testing does not check to see that your backend returns an IPv6 address, it is
testing to see that your App will function when it receives only IPv6
addresses.

If you think about this for a moment it makes sense that Apple is testing your
App for it's reaction to IPv6 addresses, not whether or not your backend
responds with IPv6 addresses.

So, if your backend is being served on AWS you do not have to worry about
failiing the test because of this. I can say this unequivocally as our backend
includes servers fronted by Cloudfront and VPC ELBs, neither of which respond
with IPv6 addresses.

## Apple's IPv6 Testing Environment

While I can't say 100% for sure, conversations with Apple support lead me to
believe that Apple's test network is configured to use DNS64 to communicate
with non-IPv6 addresses. This means that if your backend does not return IPv6
addresses then Apple's network will synthesize an IPv6 addresses and it will be
presented to the App. It also means that if you use IPv4 literals in your App
code then your App will fail the test as the DNS64 service will not be able to
synthesize an IPv6 address.

Our initial failure of this test during our app submission lead us down many
roads, because there doesn't appear to be a definitive answer to the question.
As it turned out the failure was because of an older version of AFNetworking's
reachability code that did not support receiving IPv6 addresses. Once we
upgraded AFNetworking, we successfully passed the test with no changes to our
backend.

Apple's [Supporting IPv6 DNS64/NAT64 Networks](https://developer.apple.com/library/mac/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/UnderstandingandPreparingfortheIPv6Transition/UnderstandingandPreparingfortheIPv6Transition.html)
article contains many good suggestions for tracking down code that will not
support IPv6 only networks. In our particular case with AFNetworking it was in
the section [Using Apppropriately Sized
Containers](https://developer.apple.com/library/mac/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/UnderstandingandPreparingfortheIPv6Transition/UnderstandingandPreparingfortheIPv6Transition.html#//apple_ref/doc/uid/TP40010220-CH213-SW26).

## Testing Challenges

So, the good news is that AWS will work just fine with it's IPv4 backend in
Apple's IPv6-only test. The bad news is that it's difficult to test this on
your own. We followed Apple's suggestions for [Testing for IPv6 DNS64/NAT64
Compatibility
Regularly](https://developer.apple.com/library/mac/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/UnderstandingandPreparingfortheIPv6Transition/UnderstandingandPreparingfortheIPv6Transition.html#//apple_ref/doc/uid/TP40010220-CH213-SW16)
and our App was able to connect to our services without issue. However, we
still failed the App Store test and discussions with support informed us that
the test configuraiton they recommend is not exactly the same as the one in
their test facility.

We were then instructed by Apple support to use a T-Mobile SIM (T-Mobile has
deployed IPv6 networks) and test against that. Fortunately we had T-Mobile
SIMs and verified that they were connected via an IPv6 network and tested
again. Well, once again we passed that test without issue, while still failing
Apple's test.

As I mentioned above at this point we dug into the code and found the offending
code in our old version of AFNetworking and resubmitted after upgrading
([AFNetworking Change](https://github.com/AFNetworking/AFNetworking/pull/3174/files)).
We passed on this round, but unfortunately were never able to completely
reproduce Apple's test environment.

## Conclusions

You can absolutely use AWS Cloudfront and VPC ELBs to host your mobile backend
services and still pass Apple's IPv6-only networking tests. However, you will
encounter testing difficulties at least in part because you cannot use an IPv6
only environment hosted at AWS and it is difficult to recereate Apple's test
network environment.
