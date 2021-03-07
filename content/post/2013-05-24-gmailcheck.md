---
title: gmailcheck
author: yan
date: 2013-05-24
url: /gmailcheck/
categories:
  - code
tags:
  - gmail
  - hacks
  - python

---
I&#8217;m just not that big a fan of usability, at least not for myself. To be clear, I do care about things being usable, but only enough to delay the eventual onslaught of social alienation. I drink water out of bowls, but only when the apartment is out of cups; I use xmonad as my window manager, but not at potlucks.

Thus far in my computer-using life, I&#8217;ve never owned a monitor. I don&#8217;t have a smartphone, so everything I do is on a pint-sized Thinkpad X220: writing, coding, ebook reading, garden-variety Internet browsing, photo/audio editing, bike trip navigation, sleeping, etc.

But recently, I was in the lovely town of Cambridge, MA on a warm spring day, staring out the window of a tired blue house at trees heavy with the impending procreation of innumerable songbirds, feathers washed in ecstatic glow of pollen. Mysterious stirrings of latent desire dripped from the stretched-out sky. Something changed inside me. Suddenly I felt like editing config files.

So I shut the window and did that for a while, maybe a couple hours or until I started showing symptoms of Vitamin D deficiency, don&#8217;t really remember. 

At the end of this abortive config-puberty, I [threw everything on Github][1] because typing &#8216;git push&#8217; releases a lot of dopamine in my body and apparently I like that. 

The observant git tree-traveler will notice that there&#8217;s a Python script called gmailcheck in that repo. This is something I&#8217;m glad I wrote, because it solves a real problem without being overly complicated. 

The problem is this: I spend most of my productive work time in the terminal, which is usually the master window in my tiling window setup. I usually also have a browser window open for Google searches. Using gmail, as it turns out, is hella dumb in a window of this size. Furthermore, checking gmail in the browser usually leads to an unending parade of distractions as I start to add events to my calendar, respond to some emails that can wait, click on links that friends have sent, open up [Noisebridge][2]-discuss threads with titles like &#8220;Who put mercury in the laser cutter? Also, kindergarteners visit TUESDAY,&#8221; etc.

On the other hand, not checking email at all is not an option, since my phone is usually useless, and anyone who needs to tell me something important is advised to do so over email. 

Essentially, I needed something that would show me whether I had important emails in the terminal. Maybe like Mutt but with the removal of all features that could possibly lead to distractions. 

Seems like a pretty simple Python script, and it was. [Gmailcheck v1.0][3] will show you basic information about unread emails from the last hour, or whatever time you want to hard-code in for now. I added some extra bells and whistles, like hipster colors and highlighting of emails that gmail auto-labels &#8220;Important.&#8221; If you want to actually respond to an email or do other fancy stuff, you can click on the link and just go to gmail&#8217;s web interface. 

[<img src="https://farm8.staticflickr.com/7294/8744854460_31b2c29b19.jpg" width="500" height="301" alt="gmailcheck" />][4]

Upon reaching this milestone, the next logical thought that occurred to me was, &#8220;How do you properly evoke the spirit of emails on the Noisebridge mailing lists?&#8221; 

The answer is that you print them being said by a huge ASCII-art rendering of [Tom][5], Noisebridge secretary and (before this happened) a friend of mine. 

[<img src="https://farm8.staticflickr.com/7305/8746315257_4602acb8bf.jpg" width="500" height="498" alt="gmailcheck trolling" />][6]

[<img src="https://farm9.staticflickr.com/8552/8746315225_338f20b7be.jpg" width="500" height="492" alt="cowsay -f tom" />][7]

I guess I still don&#8217;t care about usability.

 [1]: https://github.com/diracdeltas/dotfiles
 [2]: https://www.noisebridge.net/
 [3]: https://github.com/diracdeltas/dotfiles/blob/master/bin/gmailcheck
 [4]: http://www.flickr.com/photos/71094651@N06/8744854460/ "gmailcheck by diracdeltas, on Flickr"
 [5]: https://twitter.com/flamsmark
 [6]: http://www.flickr.com/photos/71094651@N06/8746315257/ "gmailcheck trolling by diracdeltas, on Flickr"
 [7]: http://www.flickr.com/photos/71094651@N06/8746315225/ "cowsay -f tom by diracdeltas, on Flickr"
