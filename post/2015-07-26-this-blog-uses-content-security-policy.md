---
title: this blog uses Content Security Policy
author: yan
layout: post
date: 2015-07-26
url: /this-blog-uses-content-security-policy/
categories:
  - Uncategorized

---
Having recently given some [talks][1] about [Content Security Policy][2] (CSP), I decided just now to enable it on my own blog to prevent cross-site scripting.

This lil&#8217; blog is hosted by the [MIT Student Information Processing Board][3] and runs on a fairly-uncustomized WordPress 4.x installation. Although I could have enabled CSP by modifying my .htaccess file, I chose to use HTML <meta> tags instead so that these instructions would work for people who don&#8217;t have shell access to their WordPress host. Unfortunately, CSP using <meta> hasn&#8217;t landed in Firefox yet ([tracking bug][4]) so I should probably do the .htaccess thing anyway.

It&#8217;s pretty easy to turn on CSP in WordPress from the dashboard:

  1. Go to <your\_blog\_path>/wp-admin/theme-editor.php. Note that this isn&#8217;t available for WordPress-hosted blogs (*.wordpress.com).
  2. Click on Header (header.php) in the sidebar to edit the header HTML.
  3. At the start of the HTML `<head>` element, add a CSP meta tag with your CSP policy. This blog uses `<meta http-equiv="Content-Security-Policy" content="script-src 'self'">` which disallows all scripts except from its own origin (including inline scripts). As far as I can tell, this blocks a few inline scripts but doesn&#8217;t impede any functionality on a vanilla WordPress instance. You might want a more permissive policy if you use fancy widgets and plugins.
  4. [Bonus points] You can also show a friendly message to users who disable javascript by adding a `<noscript>` element to your header.[<img class="alignnone wp-image-599 size-full" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-2.07.14-PM.png" alt="noscript" width="1128" height="381" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-2.07.14-PM.png 1128w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-2.07.14-PM-300x101.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-2.07.14-PM-1024x346.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-2.07.14-PM-624x211.png 624w" sizes="(max-width: 1128px) 100vw, 1128px" />][5]

A fun fact I discovered during this process is that embedding a SoundCloud iframe will include tracking scripts from Google Analytics and scorecardresearch.com. Unfortunately CSP on the embedding page (my blog) doesn&#8217;t extend to embedded contexts (soundcloud.com iframe), so those scripts will still run unless you&#8217;ve disabled JS.

[<img class="alignnone size-full wp-image-600" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-3.02.59-PM.png" alt="scorecard" width="806" height="594" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-3.02.59-PM.png 806w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-3.02.59-PM-300x221.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-3.02.59-PM-624x460.png 624w" sizes="(max-width: 806px) 100vw, 806px" />][6]

That&#8217;s all. I might do more posts on WordPress hardening later or even write a WP plugin (\*shudders at the thought of writing PHP\*). More tips are welcome too.

UPDATE (8/24/15): CSP is temporarily disabled on this blog because Google Analytics uses an inline script. I&#8217;ll nonce-whitelist it later and turn CSP back on.

 [1]: https://zyan.scripts.mit.edu/presentations/txjs2015.pdf
 [2]: http://www.w3.org/TR/CSP/
 [3]: https://scripts.mit.edu/
 [4]: https://bugzilla.mozilla.org/show_bug.cgi?id=663570
 [5]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-2.07.14-PM.png
 [6]: https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/07/Screen-Shot-2015-07-26-at-3.02.59-PM.png