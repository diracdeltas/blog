---
title: Debunking Google’s HSTS claims
author: yan
layout: post
date: 2013-11-22
url: /debunking-googles-hsts-claims/
categories:
  - code
  - encrypt the web

---
**\*\*Disclaimer\*\***: This post was published before I started working at EFF, hence some stylistic mistakes (calling it &#8220;the EFF&#8221; rather than just &#8220;EFF&#8221;) are excusable and left uncorrected. 🙂

Two days ago, the EFF published a report tiled, &#8220;[Encrypt the Web Report: Who&#8217;s Doing What][1].&#8221; The report included a chart that rated several large web companies on how well they were protecting user privacy via recommended encryption practices for data in transit. The five ranking categories were basic HTTPS support for web services, encryption of data between data centers, STARTTLS for opportunistic email encryption, support for SSL with perfect forward secrecy, and support for HTTP Strict Transport Security (HSTS). It looks like this:

<img class="alignnone" alt="" src="https://www.eff.org/files/2013/11/19/crypto-survey-graphic.png" width="666" height="1158" />

By most measures, this is an amazing chart: it&#8217;s easy to understand, seems technically correct, and is tailored to address the public&#8217;s concerns about what companies are doing to protect people from the NSA. On the other hand, I don&#8217;t like it much. Here&#8217;s why:

The first problem with the report is that it inadequately explains the basis for each score. For instance, what does a green check in the &#8220;HTTPS&#8221; category mean? Does it mean that the company is encrypting _all_ web traffic, or just web traffic for logins and sensitive information? Sonic.net certainly got a green check in that category, yet you can check that going to http://sonic.net doesn&#8217;t even redirect you to HTTPS. Amazon got a red square that says &#8220;limited&#8221;, but they seem to encrypt login and payment credentials just fine.

The second problem is that the report lacks transparency on how its data was acquired. It states, &#8220;The information in this chart comes from several sources; the companies who responded to our survey questions; information we have determined by independently examining the listed websites and services and [published][2] [reports][3].&#8221; Does that mean the EFF sent a survey to a bunch of companies that asked them to check which boxes they thought that they fulfilled? Could we at least see the survey? Also, was each claim independently verified, or did the EFF just trust the companies that responded to the survey?

I looked at the chart for a while, re-read the text a couple times, and remained unconvinced that I should go ahead and share it with all my friends. After all, there is no greater crime than encouraging ordinary people to believe whatever large companies claim about their security practices. That just leads to less autonomy for the average user and more headaches for the average security engineer. So I decided to take a look at the HSTS category to see whether I could verify the chart myself.

For those who are unfamiliar, when a website says that they support HSTS, they generally mean that they send a special &#8220;**Strict-Transport Security**&#8221; header with all HTTPS responses. This header tells your browser to only contact the website over HTTPS (a secure, encrypted protocol) for a certain length of time, preferably on the order of weeks or months. This is better than simply redirecting a user to HTTPS when they try to contact the site over HTTP [1], because that initial HTTP request can get intercepted by a malicious attacker. By refusing to send the HTTP request at all and only sending the HTTPS version of it, your browser protects you from someone sending you forged HTTPS data after they&#8217;ve intercepted the HTTP request.

[1] HTTP traffic can be trivially read by anyone who intercepts those packets, so you should watch out for passwords, cookies, and other sensitive data sent over HTTP. I wrote a post a while back showing how easy it is [to sniff HTTP traffic with your laptop][4].

(HSTS is a good idea, and all servers that support HTTPS should implement it.
  
If you decide to stop supporting HTTPS, you can just send an HSTS header with &#8220;max-age=0.&#8221;)

But HSTS still has a problem, which is that the first time a user ever contacts a website, they&#8217;ll most likely do it over HTTP since they haven&#8217;t received the HSTS header from the site yet! The Chromium browser tried to solve this problem by coming with an **HSTS Preload List**, which is an ever-growing [preloaded list of sites][5] that want users to contact them over HTTPS the first time. Firefox, Chrome, and Chromium all come shipped with this list. (Fun note: [HTTPS Everywhere][6], the browser extension by the EFF that I worked on, is basically a gigantic HSTS preload list of 7000+ domains. The difference is that the HTTPS Everywhere list doesn&#8217;t come with every browser, since it&#8217;s much less stable, so you have to install it as an extension.)

Anyway, let&#8217;s see if Google&#8217;s main search supports HSTS. To check, open up a browser and type in &#8220;http://google.com.&#8221; If it supports HSTS, the HTTP request never returns a status code. If it doesn&#8217;t support HSTS, the HTTP request returns a 302-redirect to HTTPS.

Results, examined in Firefox 25 with Firebug:

[<img class="alignnone size-full wp-image-232" alt="Screenshot from 2013-11-20 01:42:30" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-014230.png" width="980" height="629" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-014230.png 980w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-014230-300x192.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-014230-624x400.png 624w" sizes="(max-width: 980px) 100vw, 980px" />][7]

Nope! As you can see, the HTTP request completes. As it does that, it leaks our search query (&#8220;how to use tor&#8221;) and some of our preference cookies to the world.

The request then 302-redirects to HTTPS, as expected, but that HTTPS request doesn&#8217;t contain an HSTS header at all. So there&#8217;s no way that Google main search supports HSTS, at least in Firefox.

I was puzzled. Why would Google refuse to send the HSTS header, even though they support HTTPS pretty much everywhere, definitely on their main site? I did a bit more searching and concluded that it was because they _deliberately send ad clicks over plain HTTP_.

To prove this to yourself, do a Google search that returns some ads: for instance, &#8220;where to buy ukuleles.&#8221; If you open up Firebug&#8217;s page inspector and look at the link for the ad, which supposedly goes to a ukulele retail site, you&#8217;ll see a secret hidden link that you hit when you click on the ad! That link goes to &#8220;http://google.com/aclk?some_parameters=etc,&#8221; and you can conclude that Google wants you to click on that HTTP link because they put it in the DOM exactly where you&#8217;d want to click on it anyway.

[<img class="alignnone size-full wp-image-233" alt="Screenshot from 2013-11-20 02:18:44" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-021844.png" width="738" height="616" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-021844.png 738w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-021844-300x250.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-021844-624x520.png 624w" sizes="(max-width: 738px) 100vw, 738px" />][8]

Let&#8217;s click on that ukulele link. Yep, we end up redirected to http://googleadservices.com (plain HTTP again), which leaks our referer. That means the site that posted the ad as well as the NSA and anyone sniffing traffic at your local coffeeshop can see what ads you&#8217;re looking at and what you were searching for when you clicked on them.[<img class="alignnone size-full wp-image-235" alt="Screenshot from 2013-11-20 02:22:12" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-022212.png" width="1297" height="502" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-022212.png 1297w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-022212-300x116.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-022212-1024x396.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-022212-624x241.png 624w" sizes="(max-width: 1297px) 100vw, 1297px" />][9]

This is presumably the reason that Google.com doesn&#8217;t send the HSTS header and isn&#8217;t on the HSTS preload list. But wait, there&#8217;s plenty of Google domains that are in fact on the preload list, like mail.google.com, encrypted.google.com, accounts.google.com, security.google.com, and wallet.google.com. Don&#8217;t they send the HSTS header?

I checked in Firefox, and none of them did except for accounts.google.com. The rest all 302-redirect to HTTPS, just like any HTTPS site that doesn&#8217;t support HSTS.

Then I did a bit more reading and found out that HSTS preloads were [implemented][10] such that Firefox ignored any site on the preload list that didn&#8217;t send a valid HSTS header with an expiration time greater than 18 weeks. This seems like a valid design choice. Why would a site want to be on the preload list but not support HSTS at all for people with non-Firefox/Chrome/Chromium browsers? And if they don&#8217;t send the header in the first place, how do we know when the site stops supporting HSTS?

Given that Google doesn&#8217;t really provide the benefits of HSTS to any browsers except Chrom{e, ium}, it&#8217;s hard to argue that it deserves the green check mark in the HSTS category. The moral of the story is that the EFF is awesome, but having a healthy mistrust of what companies claim is even more awesome.

=====

_[IMPORTANT EDIT (11/23/13): The following originally appeared in this post, but I&#8217;ve removed it because it turns out I was accidentally using a version of Chrome that didn&#8217;t have the HSTS preloads that I was testing for anyway. Thanks to Chris Palmer for pointing out that Chrome 23 is a year old at this point, and apologies to everyone for my error.]_

_<del datetime="2013-11-23T06:02:07+00:00">Alright, so I had to also check whether Chrome respected the preload list even for sites that didn&#8217;t send the header. To be extra careful, I did this by packet-sniffing my laptop&#8217;s traffic on port 80 (HTTP) with tshark rather than examining requests with Chrome developer tools. The relevant command on a wifi network, for anyone who&#8217;s curious, is:</del>_

 `tshark -p port 80 -i wlan0 -T fields -e http.request.method -e http.request.full_uri -e http.user_agent -e http.cookie -e http.referer -e http.set-cookie -e http.location -E separator=, -E quote=d`

_<del datetime="2013-11-23T06:20:04+00:00">Let&#8217;s try http://wallet.google.com. Yep, we leak HTTP traffic. (It then redirects to https://accounts.google.com because I haven&#8217;t logged in to Wallet yet.)</del>_

[<img class="alignnone size-full wp-image-237" alt="Screenshot from 2013-11-22 01:38:21" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-013821.png" width="1366" height="713" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-013821.png 1366w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-013821-300x156.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-013821-1024x534.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-013821-624x325.png 624w" sizes="(max-width: 1366px) 100vw, 1366px" />][11]

_<del datetime="2013-11-23T06:02:07+00:00">How about http://security.google.com? Yep, we leak HTTP traffic there too.<br /> </del>_
  
[<img class="alignnone size-full wp-image-238" alt="Screenshot from 2013-11-22 01:40:11" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-014011.png" width="1366" height="768" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-014011.png 1366w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-014011-300x168.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-014011-1024x575.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-014011-624x350.png 624w" sizes="(max-width: 1366px) 100vw, 1366px" />][12]

 [1]: https://www.eff.org/deeplinks/2013/11/encrypt-web-report-whos-doing-what
 [2]: https://www.facebook.com/notes/facebook-engineering/secure-browsing-by-defa%20ult/10151590414803920
 [3]: http://arstechnica.com/security/2013/11/we-still-dont-encrypt-server-to-server-data-admits-microsoft/
 [4]: https://zyan.scripts.mit.edu/blog/summertime-and-the-http-traffic-sniffing-is-easy/
 [5]: http://src.chromium.org/viewvc/chrome/trunk/src/net/http/transport_security_state_static.json
 [6]: https://eff.org/https-everywhere
 [7]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-014230.png
 [8]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-021844.png
 [9]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-20-022212.png
 [10]: https://wiki.mozilla.org/Privacy/Features/HSTS_Preload_List
 [11]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-013821.png
 [12]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2013/11/Screenshot-from-2013-11-22-014011.png