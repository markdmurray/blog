---
layout: post
title:  "Howto: VPC Traffic Mirroring"
date:   2019-07-08 07:00:00 +1000
categories: AWS
---

AWS recently released a new feature called "VPC Traffic Mirroring" which allows you to forwrd or "mirror" traffic from an ENI, to a destination(ENI) of your choosing.[1] You might ask "whats the difference between this and vpc flow logs?" and the answer would be, a lot!.

VPC flow logs only provides metadata which is largly concernred with port,source and destination IP addresses, wheter the traffic was allowed or denied. Whereas this feature provides full packets which includes the payload.

Packets are delivered to the destination instance or NLB using VXLAN encapsulation. 

So in the following blog post I simply setup a test environment where I have one EC2 instance running Jenkins, and another EC2 instance running the latest AL2, which will act as the destination for packets. The idea is to forward all web traffic from Jenkins, to the destination instance.

Before we start, ensure that you have port 4789 UDP(VXLAN) allowed on your destination instance, before continuing to configure VPC Traffic Mirroring. I'm going to assume you already know about setting up Security Groups and ACLS, I also used an extra ENI attached at the destination, but this configuration is not required.

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

First I'll do a simple curl to our EC2 instance, which will allow us(if we configured it right) to see our mirrored VPC traffic on the destination instance.

![Curl with terminal](/assets/images/curl.jpg)

Everything going well, we should see our mirrored traffic via tcpdump, arriving encapsualted in a VXLAN encapsulated packet.

![TCPdump on destination](/assets/images/tcpdump.png)

We can take a closer look with wireshark, to see the actual contents of the packets. As you can see there full packet captures and you can no doubt the how powerful this feature is.

In another post, I'll go over some of the things we can do this information.

![wireshark output](/assets/images/wireshark.png)

