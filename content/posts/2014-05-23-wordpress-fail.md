---
title: don’t forget to secure cookies ppl
author: yan
layout: post
date: 2014-05-23
url: /wordpress-fail/
categories:
  - code
  - encrypt the web

---
Update (5/28/14): Regrettably, most of the stories covering this blog post have been all &#8220;OMG EVERYTHING IS BROKEN&#8221; rather than &#8220;Here&#8217;s how to make things better til WordPress rolls out a fix&#8221; (which I humbly believe will take a while to \*fully\* fix, given that their SSL support is so patchy). So, given that most people reading this are probably coming from one of those articles, I think it&#8217;s important to start with the actionable items that people can do to mitigate cookie-hijacking attacks on WordPress:

  1. If you&#8217;re a developer running your own WordPress install, make sure you set up SSL on all relevant servers and configure WordPress to auth flag cookies as &#8220;secure.&#8221;
  2. If you&#8217;re a WordPress user, don&#8217;t be logged into WordPress on an untrusted network, or use a VPN. If you are and you visit a wordpress.com site (which confusingly may not actually have a wordpress.com domain name), your auth cookies are exposed.
  3. [Experimental, probably not recommended] You can manually set the &#8220;secure&#8221; flag on the WP auth cookies in your browser. There&#8217;s no guarantee that this works consistently, since the server can always send a set-cookie that reverts it into an insecure cookie. It may cause some WP functionality to break.
  4. If you suspect that your WP cookie may have been stolen in the past, you can invalidate it by (1) waiting 3 years for it to expire on the server or (2) resetting your wordpress.com password. Note that logging out of WordPress does \*not\* invalidate the cookie on the server, so someone who stole it can use it even after you&#8217;ve logged out. I verified that resetting your WP password does invalidate the old cookie; there may be other ways, but I haven&#8217;t found any.

Original post below.

&nbsp;

While hunting down a bug report for [Privacy Badger][1], I noticed the &#8220;wordpress\_logged\_in&#8221; cookie being sent over clear HTTP to a WordPress authentication endpoint (http://r-login.wordpress.com/remote-login.php) on someone&#8217;s blog.

<div id="attachment_377" style="width: 904px" class="wp-caption alignnone">
  <a href="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_cookies.png"><img class="size-full wp-image-377" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_cookies.png" alt="uh-oh" width="894" height="864" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_cookies.png 894w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_cookies-300x289.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_cookies-624x603.png 624w" sizes="(max-width: 894px) 100vw, 894px" /></a>
  
  <p class="wp-caption-text">
    uh-oh
  </p>
</div>

Sounds like bad news! As mom always said, you should set the &#8220;secure&#8221; flag on sensitive cookies so that they&#8217;re never sent in plaintext.

To check whether this cookie did anything interesting, I logged out of my wordpress account, copied the wordpress\_logged\_in cookie into a fresh browser profile, and visited http://wordpress.com in the new browser profile. Yep, I was logged in!

This wouldn&#8217;t be so bad if the wordpress\_logged\_in cookie were invalidated when the original user logged out or logged back in, but it definitely still worked. Does it expire? In 3 years. (Not sure when it gets invalidated on the server side, haven&#8217;t waited long enough to know.)

Is this as bad as sending username/password in plaintext? I tried to see if I could reset the original user&#8217;s password.

[<img class="alignnone size-full wp-image-378" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpresspassword1-e1400805127825.png" alt="wordpresspassword1" width="992" height="428" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpresspassword1-e1400805127825.png 992w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpresspassword1-e1400805127825-300x129.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpresspassword1-e1400805127825-624x269.png 624w" sizes="(max-width: 992px) 100vw, 992px" />][2]

That didn&#8217;t work, so I&#8217;m assuming WordPress uses the actually-secure cookie (wordpress_sec) for super important operations like password change. Nice job, but . . .

It turns out I could post to the original user&#8217;s blog (and create new blog sites on their behalf):

[<img class="alignnone size-full wp-image-379" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postblog-e1400805350246.png" alt="wordpress_postblog" width="928" height="581" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postblog-e1400805350246.png 928w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postblog-e1400805350246-300x187.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postblog-e1400805350246-624x390.png 624w" sizes="(max-width: 928px) 100vw, 928px" />][3]

I could see private posts:

[<img class="alignnone size-full wp-image-380" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postsecretblog-e1400805413834.png" alt="wordpress_postsecretblog" width="950" height="523" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postsecretblog-e1400805413834.png 950w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postsecretblog-e1400805413834-300x165.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postsecretblog-e1400805413834-624x343.png 624w" sizes="(max-width: 950px) 100vw, 950px" />][4]

I could post comments on other blogs as them:

[<img class="alignnone size-full wp-image-381" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postcomment-e1400805499342.png" alt="wordpress_postcomment" width="1007" height="534" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postcomment-e1400805499342.png 1007w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postcomment-e1400805499342-300x159.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postcomment-e1400805499342-624x330.png 624w" sizes="(max-width: 1007px) 100vw, 1007px" />][5]

I could see their blog stats:

[<img class="alignnone size-full wp-image-382" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_stats-e1400805571221.png" alt="wordpress_stats" width="1042" height="695" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_stats-e1400805571221.png 1042w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_stats-e1400805571221-300x200.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_stats-e1400805571221-1024x682.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_stats-e1400805571221-624x416.png 624w" sizes="(max-width: 1042px) 100vw, 1042px" />][6]

And so forth. I couldn&#8217;t do some blog administrator tasks that required logging in again with the username/password, but still, not bad for a single cookie.

Moral of the story: don&#8217;t visit a WordPress site while logged into your account on an untrusted local network.

Update: Thanks to Andrew Nacin of WordPress for informing me that auth cookies will be invalidated after a session ends in the next WordPress release and that SSL support on WordPress will be improving!

Update (5/26/14): I subsequently found that the insecure cookie could be used to set someone&#8217;s 2fac auth device if they hadn&#8217;t set it, thereby locking them out of their account. If someone has set up 2fac already, the attacker can still bypass login auth by cookie stealing &#8211; the 2fac auth cookie is also sent over plaintext.

Update (5/26/14): A couple people have asked about whether the disclosure timeline below is reasonable, and my response is [here][7].

Disclosure timeline:

Wed, 21 May 2014 16:12:17 PST: Reported issue to security@automattic.com, per the instructions at http://make.wordpress.org/core/handbook/reporting-security-vulnerabilities/#where-do-i-report-security-issues; at this point, the report was mostly out of courtesy, since I figured it had to be obvious to them and many WP users already that the login cookie wasn&#8217;t secured (it&#8217;s just a simple config setting in WordPress to turn on the secure cookie flag, as I understand it). Received no indication that the email was received.

22 May 2014 16:43: Mentioned the lack of cookie securing publicly. https://twitter.com/bcrypt/status/469624500850802688

22 May 2014 17:39: Received response from Andrew Nacin (not regarding lack of cookie securing but rather that the auth cookie lifetime will soon be that of a regular session cookie). https://twitter.com/nacin/status/469638591614693376

23 May 2014 ~13:00: Discovered two-factor auth issue on accident, reported to both security@automattic.com and security@wordpress.org in reply to original email. I also mentioned it to Dan Goodin since I found the bug while trying to answer a question he had about cookies, but I did not disclose publicly.

25 May 2014 15:20: Received email response from security@automattic.com saying that they were looking into it internally (no mention of timeline). Wrote back to say thanks.

26 May 2014, ~10:00: Ars Technica article about this gets published, which mentioned the 2-fac auth issue. I updated this blog post to reflect that.

26-27 May 2014: Some commenters on the Ars Technica article discover an arguably worse bug than the one that the original article was about: WordPress sends the login form over HTTP. (Even though the form POST is over HTTPS, the local network attacker can modify the target on the HTTP page however he/she wants and then it&#8217;s game over.) This wouldn&#8217;t be so bad if everyone used a password manager and changed passwords semi-regularly, since most people are likely to login to WordPress through their blog&#8217;s admin portal (which is always HTTPS as far as I can tell), except that password reuse is rampant. Robert Graham subsequently published [this blog post][8].

29 May 2014, 5:52: Received reply from WordPress saying they would email me again when fixed.

30 May 2014, 14:51: Andrew Nacin says all issues are supposedly fixed.

 [1]: https://github.com/EFForg/privacybadgerfirefox
 [2]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpresspassword1-e1400805127825.png
 [3]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postblog-e1400805350246.png
 [4]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postsecretblog-e1400805413834.png
 [5]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_postcomment-e1400805499342.png
 [6]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/05/wordpress_stats-e1400805571221.png
 [7]: https://zyan.scripts.mit.edu/blog/wordpress-fail/#comment-70
 [8]: http://blog.erratasec.com/2014/05/wordpress-unsafe-at-any-speed.html