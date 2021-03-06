---
title: Some thoughts on the NSA and elliptic curve cryptography
author: yan
date: 2013-09-13
url: /some-thoughts-on-the-nsa-and-elliptic-curve-cryptography/
categories:
  - code
  - hacktivism
  - Uncategorized
tags:
  - cryptography
  - elliptic curves
  - NSA

---
Below is an amalgamation of some posts that I made recently on a popular microblogging platform:

========

<span class="userContent">I&#8217;ve been reading a lot today about what I believe is a super-likely NSA backdoor into modern cryptosystems.</span>

There are these things called elliptic curves that are getting used more and more for key generation in cryptography, especially <span class="text_exposed_show">in forward-secrecy-enabled SSL (which is the EFF-recommended way to secure web traffic). The problem is that the choice of parameters for the elliptic curves most used in practice are set by NIST, and we know for sure that the NSA has some influence on NIST standards.</span>

In 2006, NIST published an algorithm for elliptic-curve based random number generation that was shown to be easily breakable but ONLY by whoever chose the elliptic curve parameters. Luckily this algorithm was crazy slow so nobody used it, even though it was the (only?) NIST-recommended way of generating random bits with elliptic curves.

But it turns out there are some [relatively-obscure papers][1] that suggest that you can gain a decent cryptographic advantage by picking the elliptic curve parameters! [Edit: It was later [pointed out][2] to me that this particular attack is not close to anything <span class="userContent"><span class="text_exposed_show">the NSA could be doing right now, for various reasons. It is of course unclear whether they have knowledge of other elliptic curve parameter-based attacks that are not in the academic literature.]</span></span>

This is terrifying, because the elliptic curve parameters chosen for relatively-mysterious reasons by NIST (probably via NSA) are used by Google, OpenSSL, and any RFC-compliant implementation of elliptic curves in OpenPGP (gnupg-ecc, for instance).

<span class="userContent"><span class="text_exposed_show">==========</span></span>

<span class="userContent">Some people might be wondering if they can trust companies who issue statements that their closed-source software products don&#8217;t contain NSA backdoors (Microsoft, Google, Apple, etc.). Here is an example of why not.</span>

It has been known since <span class="text_exposed_show">2006 that Dual EC DRBG (a NIST-standardized random bit generation algorithm) has a major vulnerability that makes no sense except as an NSA backdoor. Essentially, the algorithm contains some constant parameters that are left unexplained; some researchers then showed that whoever determined these parameters could easily predict the output of the algorithm if they had access to a special set of secret numbers (analogous to an RSA private key). This is a beautiful design for a crytographic backdoor, because the secret numbers are difficult to find except by the person who set the constant parameters in the algorithm.</span>

Despite being slower than other random bit generators, Dual EC DRBG is implemented in a bunch of software products by major companies like RSA Security, Microsoft, Cisco, BlackBerry, McAfee, Certicom, and Samsung. For a full list, see: <a href="http://csrc.nist.gov/groups/STM/cavp/documents/drbg/drbgval.html" target="_blank" rel="nofollow nofollow">http://csrc.nist.gov/groups/STM/cavp/documents/drbg/drbgval.html</a>

Most of these products are closed-source, so you can&#8217;t check whether your encryption keys are being generated with Dual EC DRBG.

This is a pretty strong argument that secure software is necessarily open source software.

<span class="userContent"><span class="text_exposed_show">========<br /> </span></span>

<span class="userContent"><span class="text_exposed_show">Additional citations:</span></span>

<span data-ft="{&quot;tn&quot;:&quot;K&quot;}" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2]"><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0]"><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[3]">Bruce Schneier on the 2006 backdoor discovery in Dual_EC_DRBG: </span><a href="https://www.schneier.com/blog/archives/2007/11/the_strange_sto.html" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[4]">https://www.schneier.com/&#8230;/2007/11/the_strange_sto.html</a><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[6]" /><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[7]" /><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[8]">Google&#8217;s blog post announcing SSL w/ forward secrecy, specifically mentioning that they use the P-256 curve: </span><a href="https://www.imperialviolet.org/2011/11/22/forwardsecret.html" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[9]">https://www.imperialviolet.org/2011/11/22/forwardsecret.html</a><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[10]" /><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[11]" /><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[12]">Wiki section on NIST-recommended curves: </span><a href="https://en.wikipedia.org/wiki/Elliptic_curve_cryptography#NIST-recommended_elliptic_curves" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[13]">https://en.wikipedia.org/wiki/Elliptic_curve_cryptography&#8230;</a><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[14]" /><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[15]" /><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[16]">RFC section stating that P-256 is REQUIRED for ecc in openpgp: </span><a href="http://tools.ietf.org/html/rfc6637#section-12.1" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[17]">http://tools.ietf.org/html/rfc6637#section-12.1</a><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[18]" /><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[19]" /><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[20]">Description of elliptic curve usage in OpenSSL: </span><a href="http://wiki.openssl.org/index.php/Elliptic_Curve_Cryptography" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[21]">http://wiki.openssl.org/&#8230;/Elliptic_Curve_Cryptography</a><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[22]" /><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[23]" /><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[24]">Presentation on security dangers of NIST curves: </span><a href="http://cr.yp.to/talks/2013.05.31/slides-dan+tanja-20130531-4x3.pdf" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[25]">http://cr.yp.to/&#8230;/slides-dan+tanja-20130531-4&#215;3.pdf</a><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[26]" /><br data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[27]" /><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[28]">NSA page recommending you use elliptic curves lol: </span><a href="http://www.nsa.gov/business/programs/elliptic_curve.shtml" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[29]">http://www.nsa.gov/business/programs/elliptic_curve.shtml</a></span></span>

<span data-ft="{&quot;tn&quot;:&quot;K&quot;}" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2]"><span data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0]"><a href="http://www.nsa.gov/business/programs/elliptic_curve.shtml" target="_blank" rel="nofollow" data-reactid=".r[3ztcq].[1][4][1]{comment10200823098442317_5008296}.[0].{right}.[0].{left}.[0].[0].[0][2].[0].[29]"> </a></span></span><span class="userContent"><span class="text_exposed_show">========</span></span>

<span class="userContent"><span class="text_exposed_show">UPDATE (9/23/13): RSA security has issued an advisory to stop using some of their products where DUAL_EC_DRBG is the default random number generator (http://arstechnica.com/security/2013/09/stop-using-nsa-influence-code-in-our-product-rsa-tells-customers/). Matthew Green also made an excellent blog post about this topic over at http://blog.cryptographyengineering.com/2013/09/the-many-flaws-of-dualecdrbg.html.<br /> </span></span>

 [1]: https://eprint.iacr.org/2003/058.pdf
 [2]: http://www.scottaaronson.com/blog/?p=1517#comment-87087
