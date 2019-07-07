---
layout: post
title:  "Howto: VPC Traffic Mirroring"
date:   2019-07-08 07:00:00 +1000
categories: AWS
---

AWS recently released a new feature called "VPC Traffic Mirroring" which is essentially port mirroring[1] and allows AWS customers to do deep packet inspections without the use of forwarders on your EC2 instances.

This also differs from VPC flow logs, in that you get full packets rather than metadata.

In the following blog post I simply setup a test environment where I have one EC2 instance running Jenkins, and another EC2 instance running the latest AL2, which will act as the destination for packets. The idea is to forward all web traffic from Jenkins, to the destination instance.

Ensure that you have port 4789(UDP) allowed on your destination instance, before continuing to configure VPC Traffic Mirroring.

There are 3 things we need to configure:

- Mirror Source(Jenkins in this example) 
- Mirror Target(our AL2 instance)
    NOTE: This can be an ENI or a NLB
- Mirror Filter: Where we can configure what network traffic we allow or reject based on source/dest and protocol.
- Mirror Session: Essentially joins the above together, limited to one target.

First we'll set up the mirror target: