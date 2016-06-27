---
title: Summertime, and the HTTP traffic sniffing is easy
author: yan
layout: post
date: 2013-07-05
url: /summertime-and-the-http-traffic-sniffing-is-easy/
categories:
  - code
  - encrypt the web
tags:
  - hacks
  - https
  - SSL

---
So it happens that every time you access a URL that starts with &#8220;http://&#8221;, anyone on your local network can see what you&#8217;re doing with almost no effort worth writing about. This includes the page itself as well as any information that you&#8217;re transferring, like credit card numbers and passwords (which are hopefully encrypted). It&#8217;s worth reiterating that this isn&#8217;t difficult at all, even if your network is WPA2-protected, as most supposedly-secure WiFi networks are nowadays.

This sounds like quite a displeasing predicament for those of us who find ourselves using shared wireless networks every day (aka, all of us), but I get the impression that most Internet-users don&#8217;t understand exactly how simple it is to casually sniff HTTP traffic. So, in hopes of convincing you that it \*is\* in fact simpler than using any OS X package manager successfully, here is the absolute fastest, easiest way I found to reliably eavesdrop on HTTP traffic on most local networks. It should take less than 5 minutes to set up on your Linux thing (maybe also your Mac thing, but I haven&#8217;t tried) and is straightforward to understand.

(This is intended for education purposes, and also for encouraging more people to use HTTPS instead of HTTP whenever possible, maybe even via the [browser extension][1] that I&#8217;m contributing to this summer. More on that in a bit.)

1. Run the following to install some packages that we need. dsniff is a collection of various traffic analysis tools, of which we&#8217;ll only use arpspoof; see [here][2]. tshark is a nice terminal-based packet analyzer; tcpdump also works.

  * apt-get install dsniff
  * apt-get install tshark

2. Turn your machine into ip-forwarding mode as root by flipping a 0 to a 1, or else people on your network won&#8217;t be able to see their websites. Don&#8217;t do this step if you actually want to launch a denial of service attack, I suppose. Also you might want to remember to turn it back to 0 after you&#8217;re done.

  * echo &#8220;1&#8221; > /proc/sys/net/ipv4/ip_forward

3. Run the following commands as root on a WiFi network. To find your IP address, you can run ifconfig and look for the wlan0__ inet addr. To find your router&#8217;s IP address, you can do something like run _traceroute google.com_ and pull out the IP address from the first hop, which is likely to be 192.168.x.x.

  * arpspoof -i \[your IP address\] \[the router IP address\]
  * tshark -p port 80 -i wlan0 -T fields -e http.request.method -e http.request.full\_uri -e http.user\_agent

And voila, the second command spews out a stream of HTTP addresses that people on your network are accessing, along with the request method and the type of client software used. This is just the start; you can read the manpage for tshark to figure out how to pull more detailed info from the packets and then start writing scripts to modify them before forwarding them along. Ex: appending _-e text_ will show HTML data as text from the webpages.

What we&#8217;ve done so far is:

  1. Perform a attack called [ARP spoofing][3] on the router. Essentially, we trick everyone on the network into thinking that your computer is the router so that all the traffic intended for the router goes to your computer instead. This is straightforward since ARP doesn&#8217;t include authentication mechanisms.
  2. Use tshark to extract interesting information from the packets that we&#8217;ve intercepted.
  3. Forward these packets along to their intended destination so nobody detects our spying.

While sniffing packets as they floated by on a warm summer night in the attic of a Berkeley farmhouse in the process of testing this out, I could see which HTTP websites my housemates were accessing, what apps they were using on their phones, and what desktop clients they were running without them ever noticing. With a small amount of additional effort and a large amount of additional malice, I could have manipulated their traffic to interchange the titles of Wikipedia articles or whatever else I wanted, the possibilities bounded only by my imagination / knowledge of regular expressions.

[Note that although WPA2 encrypts data that goes between the router and devices on the network, this clearly doesn&#8217;t prevent eavesdropping via arpspoof by other devices that have access to the network. [Stack Exchange][4] explains that by operating at a [layer][5] above that at which WPA encryption occurs, an arpspoof attack causes the data to instead be encrypted for the attacker&#8217;s device instead of the router.]

Sadface time. On the other hand, none of the above works if you&#8217;re using [HTTPS][6], where the S stands for &#8220;secure,&#8221; because in that case traffic is encrypted between you and the server you&#8217;re trying to reach. On the other other hand, outgoing HTTPS requests from your device can still be subtly converted to insecure HTTP via an attack called [SSLstrip][7] that makes use of arpspoof if you try to reach an HTTPS site by redirecting from HTTP. On the other other other hand, you can download the EFF&#8217;s [HTTPS Everywhere extension][1] for Chrome/Firefox, which rewrites HTTP URLs to HTTPS before you try to connect to the HTTP site. On the (other)*4 hand, there&#8217;s some problems with SSL in practice because of intermediate certificate validation carelessness and so forth. I&#8217;m now out of hands.

 [1]: https://www.eff.org/https-everywhere/
 [2]: http://www.monkey.org/~dugsong/dsniff/
 [3]: https://en.wikipedia.org/wiki/ARP_spoofing
 [4]: http://security.stackexchange.com/questions/2372/is-wpa2-wifi-protected-against-arp-poisoning-and-sniffing
 [5]: https://en.wikipedia.org/wiki/OSI_model
 [6]: https://en.wikipedia.org/wiki/HTTP_Secure
 [7]: http://www.thoughtcrime.org/software/sslstrip/
