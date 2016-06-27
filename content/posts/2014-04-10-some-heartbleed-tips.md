---
title: How to make a less-leaky Heartbleed bandage
author: yan
layout: post
date: 2014-04-10
url: /some-heartbleed-tips/
categories:
  - encrypt the web

---
Mashable just put out a nice-looking chart showing &#8220;<a href="http://mashable.com/2014/04/09/heartbleed-bug-websites-affected/" target="_blank">Passwords You Need to Change Right Now</a>&#8221; change in light of the recent <a href="http://heartbleed.com" target="_blank">Heartbleed</a> carnage. However, it has some serious caveats that I wanted to mention:

  1. It&#8217;s probably better to be suspicious of companies whose statements are in present-tense (ex: &#8220;We have multiple protections&#8221; or even &#8220;We were not using OpenSSL&#8221;). The vulnerability existed since 2011, so even if a service was protected at the time of its disclosure 3 days ago, it could be have been affected at some point long before then. I am also skeptical that every single company on the list successfully made sure that nothing that they&#8217;ve used or given sensitive user data to had a vulnerable version of OpenSSL in the last 2 years.
  2. The article neglects to mention that password reuse means you might have to change passwords on several services for every one that was leaked. The same goes for the fact that one can trigger password resets on multiple services by authenticating a single email account.
  3. You should also clear all stored cookies just in case the server hasn&#8217;t invalidated them as they should; many sites use persistent CSRF tokens so logging out doesn&#8217;t automatically invalidate them. (Heartbleed trivially exposed user cookies.)
  4. Don&#8217;t forget to also change API keys if a service hasn&#8217;t force-rotated those already.
  5. It remains highly unclear whether any SSL certificates were compromised because of Heartbleed. If so, changing your password isn&#8217;t going to help against a MITM who has the SSL private key unless the website has revoked its SSL certificate and you&#8217;ve somehow gotten the revocation statement (LOL). This is complicated. Probably best not to worry about it right now because there&#8217;s not much you can do, but we all might have to worry about it a whole lot more depending on which way the pendulum swings in the next few days.
  6. Related-to-#5-but-much-easier: clear <a href="https://www.imperialviolet.org/2013/06/27/botchingpfs.html" target="_blank">TLS session resumption</a> data. I think this usually happens automatically when you restart the browser.

Nonetheless, Mashable made a pretty good chart for keeping track of what information companies have made public regarding the Heartbleed fallout.
