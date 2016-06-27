---
title: 'Software Transparency: Part 1'
author: yan
date: 2014-07-11
url: /software-transparency/
categories:
  - code
  - encrypt the web
  - hacktivism

---
Say that you want to &#8220;securely&#8221; acquire an app called EncryptedYo for &#8220;securely&#8221; communicating with your friends. You go to the developer&#8217;s web site, which is HTTPS-only, and download a binary executable. Done!

Perhaps if you&#8217;re paranoid, you fetch the developer&#8217;s GPG key, make sure that there&#8217;s a valid trust path to it from your own key, verify the detached signature that they&#8217;ve posted for the binary, and check that the checksum in the signature is the same as that of the binary that you&#8217;ve downloaded before installing it.

This is good enough as long as the only things you&#8217;re worried about are MITM attacks on your network connection and compromise of the server hosting the software. It&#8217;s not good enough if you&#8217;re worried about any of the following:

  * The developer getting a secret NSA order to insert a backdoor into the software.
  * The developer intentionally making false claims about the security of the software.
  * The developer&#8217;s build machine getting compromised with malware that injects backdoors during the packaging process (pre-signing) or even a malicious compiler.

All of the above are \*Very Real Worries\* (TM) that users should have when installing software. As a maintainer of a [security-enhancing browser extension][1] used by millions of people, I used to worry about the third one before HTTPS Everywhere had a deterministic build process (more on that below). If my personal laptop was compromised by a malicious version of zip that rewrote the static update-fetching URL in the HTTPS Everywhere source code before compressing and packaging it, literally millions of Firefox installations would be pwned within a few days if I didn&#8217;t somehow detect the attack before signing the package (which is basically impossible to do in general).

You might instinctively think that the scenarios above are at least \*detectable\* if the software is open source and has been well-audited, but that&#8217;s not really true. Ex:

  1. How do I know that some binary executable that I downloaded from https://coolbinaryexecutables.com actually corresponds to the well-audited, peer-reviewed source code posted at https://github.com/coolstuff/EncryptedYo.git?
  2. How do I know that the binary executable that I downloaded is the same as the one that everyone else downloaded? In other words, how can I be sure that it&#8217;s not my copy and \*only\* my copy that has a secret NSA backdoor?

So it looks like there&#8217;s a problem because we usually install software from opaque binaries or compressed archives that have no guarantee of actually corresponding to the published, version-controlled source code. You might try to solve this by cloning the EncryptedYo repo and building it yourself. You can even fetch it over Tor and/or compare your local git HEAD to someone else&#8217;s copy&#8217;s if you want a stronger guarantee against a targeted backdoor.

Unfortunately that&#8217;s too much to ask the average person to do \*every single time\* they need to update the software, especially if EncryptedYo&#8217;s target audience includes non-technical people (ex: Glenn Greenwald).

This is why post-Snowden software developers need to start working on new code packaging and installation mechanisms that preserve &#8220;software transparency,&#8221; a phrase perhaps first used in this context by Seth Schoen. Software transparency, unlike open source by itself, is a guarantee that the packages you&#8217;re installing or updating were created by building the published source code.

(Side note: Software transparency has open source code as a prerequisite, but a similar concept that I&#8217;ve been calling &#8220;binary transparency&#8221; can be applied to closed-source software as well. Binary transparency is a guarantee that the binary you&#8217;re downloading is the same as the one that everyone else is downloading, but not that the binary is non-compromised. One way to get this is to compare the checksum of your downloaded binary gainst an out-of-band append-only cryptographically-verifiable log (phew) of binary checksums, similar to what Ben Laurie proposed in [this blog post][2].)

In the last year, software transparency has finally started to become a front-and-center goal of some projects. Organizations like Mozilla and EFF are [beginning][3] to [work][4] on fully-reproducible build processes so that other people can independently build their software packages from source and make sure that their checksums are the same as the ones posted on mozilla.org or eff.org. Mike Perry of the Tor Project has [written][5] about the [painstaking, years-long process][6] that it took to compile the Tor Browser Bundle deterministically inside a VM, but for many other software projects, the path to a reproducible build is as simple as [normalizing timestamps][7] in zip.

Of course, a reproducible build proccess doesn&#8217;t by itself impact the average user, who is unlikely to try to replicate the build process for Firefox for Android before installing it on their phone. But at least it means that if Mozilla started posting backdoored binaries because their build machine was compromised, some members of their open source development community could in theory detect the attack after-the-fact and raise suspicions. That&#8217;s more than we could do before.

IMO, every reasonably-paranoid software developer should be trying to adopt an independently reproducible build process. [Gitian][8] is a good place to start.

(Part 2 of this series, which I haven&#8217;t written yet, is probably going to be about implementing software transparency in a way that protects end users before they get pwned, which nobody is doing much of yet AFAIK. In particular, it would be nice to start discussing ways to enforce software transparency for resources loaded in a browser, in hopes that this will bring either some clarity or more shouting to the debate about whether in-browser crypto apps are a good idea.)

 [1]: https://www.eff.org/https-everywhere
 [2]: http://www.links.org/?p=1262
 [3]: https://bugzilla.mozilla.org/show_bug.cgi?id=885777
 [4]: https://github.com/EFForg/https-everywhere/commit/e06a13a3283d93c96323970e1e43a897e4bfc944
 [5]: https://blog.torproject.org/blog/deterministic-builds-part-one-cyberwar-and-global-compromise
 [6]: https://blog.torproject.org/blog/deterministic-builds-part-two-technical-details
 [7]: https://github.com/devrandom/gitian-builder/blob/master/bin/canon-zip
 [8]: https://gitian.org
