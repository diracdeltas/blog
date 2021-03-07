---
title: backdooring your javascript using minifier bugs
author: yan
date: 2015-08-24
url: /backdooring-js/
categories:
  - code
  - hacktivism

---
In addition to unforgettable life experiences and personal growth, one thing I got out of DEF CON 23 was a copy of [POC||GTFO 0x08][1] from Travis Goodspeed. The coolest article I&#8217;ve read so far in it is &#8220;Deniable Backdoors Using Compiler Bugs,&#8221; in which the authors abused a pre-existing bug in CLANG to create a backdoored version of sudo that allowed any user to gain root access. This is very sneaky, because nobody could prove that their patch to sudo was a backdoor by examining the source code; instead, the privilege escalation backdoor is inserted at compile-time by certain (buggy) versions of CLANG.

That got me thinking about whether you could use the same backdoor technique on javascript. JS runs pretty much everywhere these days (browsers, servers, [arduinos and robots][2], maybe even cars someday) but it&#8217;s an interpreted language, not compiled. However, it&#8217;s quite common to minify and optimize JS to reduce file size and improve performance. Perhaps that gives us enough room to insert a backdoor by abusing a JS _minifier._

### Part I: Finding a good minifier bug

Question: Do popular JS minifiers really have bugs that could lead to security problems?

Answer: After about 10 minutes of searching, I found one in [UglifyJS][3], a popular minifier used by jQuery to build a script that runs on something like [70% of the top websites][4] on the Internet. The [bug itself][5], fixed in the 2.4.24 release, is straightforward but not totally obvious, so let&#8217;s walk through it.

UglifyJS does a bunch of things to try to reduce file size. One of the compression flags that is on-by-default will compress expressions such as:

<pre>!a && !b && !c && !d
</pre>

That expression is 20 characters. Luckily, if we apply [De Morgan&#8217;s Law][6], we can rewrite it as:

<pre>!(a || b || c || d)
</pre>

which is only 19 characters. Sweet! Except that De Morgan&#8217;s Law doesn&#8217;t necessarily work if any of the subexpressions has a non-Boolean return value. For instance,

<pre>!false && 1
</pre>

will return the number 1. On the other hand,

<pre>!(false || !1)
</pre>

simply returns true.

So if we can trick the minifier into erroneously applying De Morgan&#8217;s law, we can make the program behave differently before and after minification! Turns out it&#8217;s not too hard to trick UglifyJS 2.4.23 into doing this, since it will always use the rewritten expression if it is shorter than the original. (UglifyJS 2.4.24 patches this by making sure that subexpressions are boolean before attempting to rewrite.)

### Part II: Building a backdoor in some hypothetical auth code

Cool, we&#8217;ve found the minifier bug of our dreams. Now let&#8217;s try to abuse it!

Let&#8217;s say that you are working for some company, and you want to deliberately create vulnerabilities in their Node.js website. You are tasked with writing some server-side javascript that validates whether user auth tokens are expired. First you make sure that the Node package uses uglify-js@2.4.23, which has the bug that we care about.

Next you write the token validation function, inserting a bunch of plausible-looking config and user validation checks to force the minifier to compress the long (not-)boolean expression:

<pre>function isTokenValid(user) {
    var timeLeft =
        !!config && // config object exists
        !!user.token && // user object has a token
        !user.token.invalidated && // token is not explicitly invalidated
        !config.uninitialized && // config is initialized
        !config.ignoreTimestamps && // don't ignore timestamps
        getTimeLeft(user.token.expiry); // &gt; 0 if expiration is in the future

    // The token must not be expired
    return timeLeft &gt; 0;
}

function getTimeLeft(expiry) {
  return expiry - getSystemTime();
}
</pre>

Running `uglifyjs -c` on the snippet above produces the following:

<pre>function isTokenValid(user){var timeLeft=!(!config||!user.token||user.token.invalidated||config.uninitialized||config.ignoreTimestamps||!getTimeLeft(user.token.expiry));return timeLeft&gt;0}function getTimeLeft(expiry){return expiry-getSystemTime()}</pre>

In the original form, if the config and user checks pass, `timeLeft` is a negative integer if the token is expired. In the minified form, `timeLeft` must be a boolean (since &#8220;!&#8221; in JS does type-coercion to booleans). In fact, if the config and user checks pass, the value of `timeLeft` is always `true` unless `getTimeLeft` coincidentally happens to be 0.

Voila! Since `true > 0` in javascript (yay for type coercion!), auth tokens that are past their expiration time will still be valid forever.

### Part III: Backdooring jQuery

Next let&#8217;s abuse our favorite minifier bug to write some patches to jQuery itself that could lead to backdoors. We&#8217;ll work with [jQuery 1.11.3][7], which is the current jQuery 1 stable release as of this writing.

jQuery 1.11.3 uses [grunt-contrib-uglify 0.3.2][8] for minification, which in turn depends on [uglify-js ~2.4.0][9]. So uglify-js@2.4.23 satisfies the dependency, and we can manually edit package.json in grunt-contrib-uglify to force it to use this version.

There are only a handful of places in jQuery where the DeMorgan&#8217;s Law rewrite optimization is triggered. None of these cause bugs, so we&#8217;ll have to add some ourselves.

**Backdoor Patch #1:**

First let&#8217;s add a potential backdoor in jQuery&#8217;s .html() method. The [patch][10] looks weird and superfluous, but we can convince anyone that it shouldn&#8217;t actually change what the method does. Indeed, pre-minification, the unit tests pass.

After minification with uglify-js@2.4.23, jQuery&#8217;s .html() method will set the inner HTML to &#8220;true&#8221; instead of the provided value, so a bunch of tests fail.

[<img class="alignnone size-full wp-image-670" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.35.48-PM.png" alt="Screen Shot 2015-08-23 at 1.35.48 PM" width="844" height="785" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.35.48-PM.png 844w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.35.48-PM-300x279.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.35.48-PM-624x580.png 624w" sizes="(max-width: 844px) 100vw, 844px" />][11]

However, the jQuery maintainers are probably using the patched version of uglifyjs. Indeed, tests pass with uglify-js@2.4.24, so this patch might not seem too suspicious.

[<img class="alignnone size-full wp-image-671" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.39.47-PM.png" alt="Screen Shot 2015-08-23 at 1.39.47 PM" width="1057" height="281" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.39.47-PM.png 1057w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.39.47-PM-300x80.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.39.47-PM-1024x272.png 1024w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.39.47-PM-624x166.png 624w" sizes="(max-width: 1057px) 100vw, 1057px" />][12]

Cool. Now let&#8217;s run grunt to build jQuery with this patch and write some silly code that triggers the backdoor:

<pre>&lt;html&gt;
    &lt;script src="../dist/jquery.min.js"&gt;&lt;/script&gt;
    &lt;button&gt;click me to see if this site is safe&lt;/button&gt;
    &lt;script&gt;
        $('button').click(function(e) {
            $('#result').html('&lt;b&gt;false!!&lt;/b&gt;');
        });
    &lt;/script&gt;
    &lt;div id='result'&gt;&lt;/div&gt;
&lt;/html&gt;
</pre>

Here&#8217;s the result of clicking that button when we run the pre-minified jQuery build:

[<img class="alignnone size-full wp-image-672" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.44.45-PM.png" alt="Screen Shot 2015-08-23 at 4.44.45 PM" width="511" height="449" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.44.45-PM.png 511w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.44.45-PM-300x264.png 300w" sizes="(max-width: 511px) 100vw, 511px" />][13]

As expected, the user is warned that the site is not safe. Which is ironic, because it doesn&#8217;t use our minifier-triggered backdoor.

Here&#8217;s what happens when we instead use the minified jQuery build:

[<img class="alignnone size-full wp-image-673" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.45.10-PM.png" alt="Screen Shot 2015-08-23 at 4.45.10 PM" width="505" height="465" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.45.10-PM.png 505w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.45.10-PM-300x276.png 300w" sizes="(max-width: 505px) 100vw, 505px" />][14]

Now users will totally think that this site is safe even when the site authors are trying to warn them otherwise.

**Backdoor Patch #2:**

The first backdoor might be too easy to detect, since anyone using it will probably notice that a bunch of HTML is being set to the string &#8220;true&#8221; instead of the HTML that they want to set. So our [second backdoor patch][15] is one that only gets triggered in unusual cases.

[<img class="alignnone size-full wp-image-675" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-7.48.14-PM.png" alt="Screen Shot 2015-08-23 at 7.48.14 PM" width="1020" height="313" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-7.48.14-PM.png 1020w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-7.48.14-PM-300x92.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-7.48.14-PM-624x191.png 624w" sizes="(max-width: 1020px) 100vw, 1020px" />][16]

Basically, we&#8217;ve modified jQuery.event.remove (used in the .off() method) so that the code path that calls [special event removal hooks][17] never gets reached after minification. (Since `spliced` is always boolean, its length is always undefined, which is not > 0.) This doesn&#8217;t necessarily change the behavior of a site unless the developer has defined such a hook.

Say that the site we want to backdoor has the following HTML:

<pre>&lt;html&gt;
    &lt;script src="../dist/jquery.min.js"&gt;&lt;/script&gt;
    &lt;button&gt;click me to see if special event handlers are called!&lt;/button&gt;
    &lt;div&gt;FAIL&lt;/div&gt;
    &lt;script&gt;
        // Add a special event hook for onclick removal
        jQuery.event.special.click.remove = function(handleObj) {
            $('div').text('SUCCESS');
        };
        $('button').click(function myHandler(e) {
            // Trigger the special event hook
            $('button').off('click');
        });
    &lt;/script&gt;
&lt;/html&gt;
</pre>

If we run it with unminified jQuery, the removal hook gets called as expected:

[<img class="alignnone size-full wp-image-677" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.10-PM.png" alt="Screen Shot 2015-08-23 at 4.43.10 PM" width="705" height="545" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.10-PM.png 705w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.10-PM-300x232.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.10-PM-624x482.png 624w" sizes="(max-width: 705px) 100vw, 705px" />][18]

But the removal hook never gets called if we use the minified build:

[<img class="alignnone size-full wp-image-678" src="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.42-PM.png" alt="Screen Shot 2015-08-23 at 4.43.42 PM" width="714" height="544" srcset="https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.42-PM.png 714w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.42-PM-300x229.png 300w, https://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.42-PM-624x475.png 624w" sizes="(max-width: 714px) 100vw, 714px" />][19]

Obviously this is bad news if the event removal hook does some security-critical function, like checking if an origin is whitelisted before passing a user&#8217;s auth token to it.

### Conclusion

The backdoor examples that I&#8217;ve illustrated are pretty contrived, but the fact that they can exist at all should probably worry JS developers. Although JS minifiers are not nearly as complex or important as C++ compilers, they have power over a lot of the code that ends up running on the web.

It&#8217;s good that UglifyJS has added test cases for [known bugs][20], but I would still advise anyone who uses a non-formally verified minifier to be wary. Don&#8217;t minify/compress server-side code unless you have to, and make sure you run browser tests/scans against code post-minification. [Addendum: Don&#8217;t forget that even if you aren&#8217;t using a minifier, your CDN might minify files in production for you. For instance, Cloudflare&#8217;s collapsify [uses uglifyjs][21].]

Now, back to reading the rest of POC||GTFO.

**PS**: If you have thoughts or ideas for future PoC, please leave a comment or find me on Twitter ([@bcrypt][22]). The code from this blog post is up on [github][23].

[Update 1: Thanks @joshssharp for posting this to Hacker News. I&#8217;m flattered to have been on the front page allllll night long (cue 70&#8217;s soul music). Bonus points &#8211; the thread taught me [something surprising][24] about why it would make sense to minify server-side.]

[Update 2: There is now a [long thread][25] about minifiers on debian-devel which spawned [this wiki page][26] and [another HN thread][27]. It&#8217;s cool that JS developers are paying attention to this class of potential security vulnerabilities, but I hope that people complaining about minification also consider transpilers and other JS pseudo-compilers. I&#8217;ll talk more about that in a future blog post.]

 [1]: https://www.alchemistowl.org/pocorgtfo/
 [2]: http://postscapes.com/javascript-and-the-internet-of-things
 [3]: https://github.com/mishoo/UglifyJS2
 [4]: http://blog.jquery.com/2014/01/13/the-state-of-jquery-2014/
 [5]: https://github.com/mishoo/UglifyJS2/issues/751
 [6]: https://en.wikipedia.org/wiki/De_Morgan%27s_laws
 [7]: https://github.com/jquery/jquery/tree/1.11.3
 [8]: https://github.com/jquery/jquery/blob/1.11.3/package.json#L39
 [9]: https://github.com/gruntjs/grunt-contrib-uglify/blob/v0.3.2/package.json#L30
 [10]: https://github.com/diracdeltas/jquery/commit/e50c8ce26736027386aa7a698baeca7740a54a0b
 [11]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.35.48-PM.png
 [12]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-1.39.47-PM.png
 [13]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.44.45-PM.png
 [14]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.45.10-PM.png
 [15]: https://github.com/diracdeltas/jquery/commit/a2092d8a85474c90e2e4d306a21a14af55365b58
 [16]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-7.48.14-PM.png
 [17]: https://learn.jquery.com/events/event-extensions/
 [18]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.10-PM.png
 [19]: http://zyan.scripts.mit.edu/blog/wp-content/uploads/2015/08/Screen-Shot-2015-08-23-at-4.43.42-PM.png
 [20]: https://github.com/mishoo/UglifyJS2/blob/master/test/compress/issue-751.js
 [21]: https://github.com/cloudflare/collapsify/commit/e59253193282f2047eea1c770be57cb10c3c4a3a
 [22]: https://twitter.com/bcrypt
 [23]: https://github.com/diracdeltas/jquery
 [24]: https://news.ycombinator.com/item?id=10108672
 [25]: https://lists.debian.org/debian-devel/2015/08/msg00427.html
 [26]: https://wiki.debian.org/onlyjob/no-minification
 [27]: https://news.ycombinator.com/item?id=10146157
