---
title: tls everything
author: yan
date: 2014-12-10
url: /tls-everything/
categories:
  - encrypt the web

---
Yesterday the W3C [Technical Architecture Group][1] published a new finding titled, &#8220;[The Web and Encryption][2].&#8221; In it, they conclude:

> &#8220;. . . the Web platform should be designed to **actively prefer secure origins** — typically, by encouraging use of HTTPS URLs instead of HTTP ones. Furthermore, the **end-to-end nature of TLS encryption must not be compromised** on the Web, in order to preserve this trust.&#8221;

To many [HTTPS Everywhere][3] users like myself, this seemed a decade or so beyond self-evident. So I was surprised to see a [flurry][4] of [objections][5] appear on the public mailing list thread discussing the TAG findings.

It seems bizarre to me that security-minded web developers are spending so much effort hardening the web platform by designing and implementing standards like CSP Level 2, WebCrypto, HTTP Public Key Pinning, and Subresource Integrity, while others are still debating whether requiring the bare minimum security guarantee on the web is a good thing. While [some][6] sites are preventing any javascript from running on their page unless it&#8217;s been whitelisted, [other][7] sites can&#8217;t even promise that any user will ever visit a page that hasn&#8217;t been tampered with.

<div id="attachment_502" style="width: 430px" class="wp-caption alignnone">
  <a href="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/wp-wtf.png"><img class="size-full wp-image-502" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/wp-wtf.png" alt="wtf" width="420" height="604" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/wp-wtf.png 420w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/wp-wtf-208x300.png 208w" sizes="(max-width: 420px) 100vw, 420px" /></a>
  
  <p class="wp-caption-text">
    small consolation: the second one has more downloads
  </p>
</div>

Obviously we shouldn&#8217;t ignore arguments for a plaintext-permissive web; they&#8217;re statistically useful as indicators of misconceptions about HTTPS and sometimes also as indicators of real friction that website operators face. What can we learn?

Here&#8217;s some of my observations and responses to common anti-HTTPS points (as someone who lurks on standards mailing lists and often pokes website operators to deploy HTTPS, both professionally and recreationally):

  1.  _&#8220;HTTPS is expensive and hard to set up.&#8221;_ This is objectively getting better. Cloudflare offers automatic [free SSL][8] to their CDN customers, and [SSLMate][9] lets you get a cert for $10 using the command line. In the near future, the [LetsEncrypt][10] cert authority will offer free certificates, deployed and managed using a nifty new [protocol][11] called ACME that makes the entire process take <30 seconds.
  2. _&#8220;There is no value in using HTTPS for data that is, by nature, public (such as news articles).&#8221;_ This misses the point that aggregated browsing patterns, even for only public sites, can reveal a lot of private information about a person. If it weren&#8217;t, advertisers wouldn&#8217;t use third-party tracking beacons. QED.
  3. _&#8220;TLS is slow.&#8221;_ Chris Palmer thought you would ask this and gave an excellent [presentation][12] explaining why not. tl;dr: TLS is usually not noticeably slower, but if it is, chances are that you can [optimize away the difference][13] (warning: the previous link is highly well-written and may cause you to become convinced that TLS is not slow).
  4. _&#8220;HTTPS breaks feature X.&#8221;_ This is something I&#8217;m intimately familar with, since most bug reports in HTTPS Everywhere (which I used to maintain) were caused by the extension switching a site to HTTPS and suddenly breaking some feature. [Mixed content blocking][14] was the biggest culprit, but there were also cases where CORS stopped working because the header whitelisted the HTTP site but not the HTTPS one. (I also expected some &#8220;features&#8221; to break because HTTPS sites don&#8217;t leak referer to HTTP ones, but surprisingly this never happened.) Luckily if you&#8217;re using HTTPS Everywhere in Chrome, there is a panel in the developer console that helps you detect and fix mixed content on websites (shown below). Setting the CSP report-only header to report non-HTTPS subresources is similarly useful but doesn&#8217;t tell you which resources can be rewritten.[<img class="alignnone size-full wp-image-501" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/https-switch.png" alt="https-switch" width="1293" height="843" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/https-switch.png 1293w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/https-switch-300x195.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/https-switch-1024x667.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/https-switch-624x406.png 624w" sizes="(max-width: 1293px) 100vw, 1293px" />][15]
  5. _&#8220;HTTPS gives users a false sense of security.&#8221;_ This comes up surprisingly often from various angles. Some people frame this as, _&#8220;The CA system isn&#8217;t trustworthy and is breakable by every government,&#8221;_ while others say, _&#8220;Even with HTTPS, you leak DNS lookups and valuable metadata,&#8221;_ and others say, _&#8220;But many site certificates are managed by the CDN, not the site the user thinks they&#8217;re visiting securely.&#8221;_ The baseline counterargument to all of these is that encryption, even encryption that is theoretically breakable by some people, is better than no encryption, which doesn&#8217;t need to be broken by anyone. CA trustworthiness in particular is getting better with the implementation of certificate transparency and key pinning in browsers; let&#8217;s hope that we solve DNSSEC someday too. Also, regardless of whether HTTPS gives people a false sense of security, HTTP almost certainly gives the average person a false sense of security; otherwise, why would anyone submit their Quora password in plaintext?[<img class="wp-image-504 " src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/quora1.png" alt="quora" width="603" height="462" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/quora1.png 1038w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/quora1-300x229.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/quora1-1024x784.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/quora1-624x477.png 624w" sizes="(max-width: 603px) 100vw, 603px" />][16]

In summary, it&#8217;s very encouraging to see the TAG expressing support for a ubiquitous transit encryption on the web (someday), but from the resulting discussion, it&#8217;s clear that developers still need to be convinced that HTTPS is efficient, reliable, affordable, and worthwhile. I think the TAG has a clear path forward here: separate the overgrown anti-HTTPS mythology from the actual measurable obstacles to HTTPS deployment, and encourage standards that fix real problems that developers and implementers have when transitioning to HTTPS. [ACME][17], [HPKP][18], [Certificate Transparency][19], and especially [requiring minimum security standards][20] for powerful new web platform features are good examples of work that motivates website operators to turn on HTTPS by lowering the cost and/or raising the benefits.

 [1]: http://www.w3.org/2001/tag/
 [2]: https://w3ctag.github.io/web-https/
 [3]: https://eff.org/https-everywhere
 [4]: http://lists.w3.org/Archives/Public/www-tag/2014Dec/0018.html
 [5]: http://lists.w3.org/Archives/Public/www-tag/2014Dec/0051.html
 [6]: https://blog.matatall.com/
 [7]: http://www.nytimes.com/
 [8]: https://blog.cloudflare.com/introducing-universal-ssl/
 [9]: https://sslmate.com/
 [10]: https://letsencrypt.org/
 [11]: https://github.com/letsencrypt/acme-spec
 [12]: https://www.youtube.com/watch?v=ayD0LiZkWLQ
 [13]: https://istlsfastyet.com/
 [14]: https://support.mozilla.org/en-US/kb/how-does-content-isnt-secure-affect-my-safety
 [15]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/https-switch.png
 [16]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2014/12/quora1.png
 [17]: https://github.com/letsencrypt/acme-spec/blob/master/draft-barnes-acme.txt
 [18]: https://tools.ietf.org/html/draft-ietf-websec-key-pinning-21
 [19]: https://tools.ietf.org/html/rfc6962
 [20]: https://w3c.github.io/webappsec/specs/powerfulfeatures/
