---
layout: post
title:  "Howto: VPC Traffic Mirroring"
date:   2019-07-08 07:00:00 +1000
categories: AWS
---

AWS recently released a new feature called "VPC Traffic Mirroring" which allows you to forwrd or "mirror" traffic from an ENI, to a destination(ENI) of your choosing.[1] You might ask "whats the difference between this and vpc flow logs?" and the answer would be, a lot!.

VPC flow logs only provides metadata which is largly concernred with port,source and destination IP addresses, wheter the traffic was allowed or denied. Whereas this feature provides full packets which includes the payload.

So in the following blog post I simply setup a test environment where I have one EC2 instance running Jenkins, and another EC2 instance running the latest AL2, which will act as the destination for packets. The idea is to forward all web traffic from Jenkins, to the destination instance.

Before we start, ensure that you have port 4789(UDP) allowed on your destination instance, before continuing to configure VPC Traffic Mirroring. I'm going to assume you already know about setting up Security Groups and ACLS, I also used an extra ENI attached at the destination, but is not required.

There are 3 things we need to configure:

- Mirror Source(Jenkins in this example) 
- Mirror Target(our AL2 instance)
    NOTE: This can be an ENI or a NLB
- Mirror Filter: Where we can configure what network traffic we allow or reject based on source/dest and protocol.
- Mirror Session: Essentially joins the above together, limited to one target.

First we'll set up the mirror target:

![Creating our mirror target](/assets/images/traffic_mirror_target.png)

Second we create our mirror filter, here we set destination port as the port we want to capture(8080 in this case) and source port as any.

![Traffic Mirror Filter](/assets/images/traffic_mirror_filter.png)
![Traffic Mirror Filter rules](/assets/images/traffic_mirror_filter_rule.png)

Once our filters are set, we just need to create a session which combines our target and filter we just set up.

![Traffic Mirror Session](/assets/images/traffic_mirror_session.png)

With these setup, lets gain shell access to the target via Session Manager and take a look at our mirrored packets via Tcpdump.