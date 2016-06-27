---
title: lessons from the ad blocker trenches
author: yan
layout: post
date: 2015-07-17
url: /lessons-from-the-ad-blocker-trenches/
categories:
  - code

---
Greetings from the beautiful museum district of Berlin, where I&#8217;ve been trapped in a small conference room all week for the quarterly meeting of the W3C Technical Architecture group. So far we&#8217;ve produced two documents this week that I think are pretty good:

  * [no encryption backdoors][1]
  * [no non-consensual web tracking][2]

I just realized I have a few more things to say about the latter, based on my experience building and maintaining a semi-popular ad blocker (<a href="https://eff.org/privacybadger" target="_blank">Privacy Badger Firefox</a>).

  1. Beware of ad blockers that don&#8217;t actually block requests to tracking domains. For instance, an ad blocker that simply hides ads using CSS rules is not really useful for preventing tracking. Many users can&#8217;t tell the difference.
  2. Third-party cookies are not the only way to track users anymore, which means that browser features and extensions that only block/delete third-party cookies are not as useful as they once were. This 2012 survey paper [<a href="https://jonathanmayer.org/papers_data/trackingsurvey12.pdf" target="_blank">PDF</a>] by Jonathan Mayer et. al. has a table of non-cookie browser tracking methods, which is probably out of date by now: [<img class="alignnone size-full wp-image-577" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-17-at-4.32.55-PM.png" alt="Screen Shot 2015-07-17 at 4.32.55 PM" width="421" height="684" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-17-at-4.32.55-PM.png 421w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-17-at-4.32.55-PM-185x300.png 185w" sizes="(max-width: 421px) 100vw, 421px" />][3]
  3. Detecting whether a domain is performing third-party tracking is not straightforward. Naively, you could do this by counting the number of first-party domains that a domain reads high-entropy cookies from in a third-party context. However, this doesn&#8217;t encompass reading non-cookie browser state that could be used to uniquely identify users in aggregate (see table above). A more general but probably impractical approach is to try to tag every piece of site-readable browser state with an entropy estimate so that you can score sites by the total entropy that is readable by them in a third-party context. (We assume that while a site is being accessed as a first-party, the user implicitly consents to letting it read information about them. This is a gross simplification, since first parties can read lots of information that users don&#8217;t consent to by invisible fingerprinting. Also, I am recklessly using the term &#8220;entropy&#8221; here in a way that would probably cause undergrad thermodynamics professors to have aneurysms.)
  4. The browser definition of &#8220;third-party&#8221; only roughly approximates the real-life definition. For instance, dropbox.com and dropboxusercontent.com are the same party from a business and privacy perspective but not from the cookie-scoping or DNS or same-origin-policy perspective.
  5. The hardest-to-block tracking domains are the ones who cause collateral damage when blocked. A good example of this is Disqus, commonly embedded as a third-party widget on blogs and forums; if we block requests to Disqus (which include cookies for logged-in users), we severely impede the functionality of many websites. So Disqus is too usability-expensive to block, even though they can track your behavior from site to site.
  6. The hardest-to-block tracking methods are the ones that cause collateral damage when disabled. For instance, [HSTS][4] and [HPKP][5] both store user-specific persistent data that can be abused to probe users&#8217; browsing histories and/or mark users so that you can re-identify them after the first time they visit your site. However, clearing HSTS/HPKP state between browser sessions dilutes their security value, so browsers/extensions are reluctant to do so.
  7. Specifiers and implementers sometimes argue that Feature X, which adds some fingerprinting/tracking surface, is okay because it&#8217;s no worse than cookies. I am skeptical of this argument for the following reasons:<br clear="none" />a. Unless explicitly required, there is no guarantee that browsers will treat Feature X the same as cookies in privacy-paranoid edge cases. For instance, if Safari blocks 3rd party cookies by default, will it block 3rd party media stream captures (which will store a unique deviceid) by default too?<br clear="none" />b. Ad blockers and anti-tracking tools like Disconnect, Privacy Badger, and Torbutton were mostly written to block and detect tracking on the basis of cookies, not arbitrary persistent data. It&#8217;s arguable that they should be blocking these other things as soon as they are shipped in browsers, but that requires developer time.

That&#8217;s all. And here&#8217;s some photos I took while walking around Berlin in a jetlagged haze for hours last night:

[<img class="alignnone  wp-image-586" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1.jpg" alt="berlin1" width="391" height="391" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1.jpg 1080w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1-150x150.jpg 150w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1-300x300.jpg 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1-1024x1024.jpg 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1-624x624.jpg 624w" sizes="(max-width: 391px) 100vw, 391px" />][6]

[<img class="alignnone  wp-image-587" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2.jpg" alt="berlin2" width="392" height="392" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2.jpg 1080w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2-150x150.jpg 150w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2-300x300.jpg 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2-1024x1024.jpg 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2-624x624.jpg 624w" sizes="(max-width: 392px) 100vw, 392px" />][7]

Update (7/18/15): Artur Janc of Google pointed out this <a href="https://www.chromium.org/Home/chromium-security/client-identification-mechanisms" target="_blank">document by folks at Chromium</a> analyzing various client identification methods, including several I hadn&#8217;t thought about.

 [1]: http://www.w3.org/2001/tag/doc/encryption-finding/
 [2]: http://www.w3.org/2001/tag/doc/unsanctioned-tracking/
 [3]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-17-at-4.32.55-PM.png
 [4]: http://www.radicalresearch.co.uk/lab/hstssupercookies
 [5]: https://tools.ietf.org/html/rfc7469#section-5
 [6]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin1.jpg
 [7]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/berlin2.jpg