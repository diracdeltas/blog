---
title: HTTPS Everywhere 3.3 is out!
author: yan
date: 2013-07-27
url: /https-everywhere-3-3-is-out/
categories:
  - code

---
Quick update to mention that a new version of the [browser extension][1] I&#8217;ve been helping with this summer has just been released!

This release was spurred by the impending arrival of Firefox 23, which notably has [Mixed Active Content Blocking enabled by default][2]. In summary, this means that scripts loaded via HTTP on an otherwise-HTTPS site will be blocked automatically for security reasons. Although this is good news in general, it would have suddenly caused HTTPS Everywhere to become [way less usable][3].

[Micah][4] at the EFF did tons of work on this one to make sure that mixed content rules in HTTPS Everywhere wouldn&#8217;t break a massive number of websites for FF23 users, while I implemented the UI to alert users to the fact that they would have to re-input any custom preferences once they updated to 3.3 (unfortunately). I also ended up fixing a previously-undiscovered bug along the way that prevented the HTTPS Everywhere tool-tip from showing up when users install the extension.

It&#8217;s kind of sad that the tool-tip didn&#8217;t show up for most people before. Did you know that you can disable HTTPS Everywhere rules by clicking on the toolbar icon? This is very handy when certain rules cause a site to break for some reason.

Anyway, happy upgrading!

 [1]: https://www.eff.org/https-everywhere
 [2]: https://blog.mozilla.org/tanvi/2013/04/10/mixed-content-blocking-enabled-in-firefox-23/
 [3]: https://trac.torproject.org/projects/tor/ticket/9196
 [4]: https://www.eff.org/about/staff/micah-lee
