---
title: rate-limiting anonymous accounts
author: yan
layout: post
date: 2015-03-05
url: /rate-limiting-anonymous-accounts/
categories:
  - Uncategorized

---
Yesterday TechCrunch [reported][1] that Twitter now seems to be requiring SMS validation from new accounts registered over Tor. Though this might be effective for rate-limiting registration of abusive/spammy accounts, sometimes [actual people][2] use Twitter over Tor because anonymity is a prerequisite to free speech and circumventing information barriers imposed by oppressive governments. These users might not want to link their telco-sanctioned identity with their Twitter account, hence why they&#8217;re using Tor in the first place.

What are services like Twitter to do, then? I thought of one simple solution that borrows a popular idea from anonymous e-cash systems.

In a 1983 paper, cryptographer David Chaum introduced the concept of blind signatures [1]. A blind signature is a cryptographic signature in which the signer can&#8217;t see the content of the message that she&#8217;s signing. So if Bob wants Alice to sign the message &#8220;Bob is great&#8221; without her knowing, he first &#8220;blinds&#8221; the message using a random factor that is unknown to her and gives Alice the blinded message to sign. When he unblinds her signed message by removing the blinding factor, the original message &#8220;Bob is great&#8221; _also_ has a valid signature from Alice!

This may seem weird and magical, but blinded signatures are actually possible using the familiar RSA signature scheme. The proof is straightforward and on Wikipedia so I&#8217;ll skip it here [2]. Basically, since RSA signatures are just modulo&#8217;d exponentiation of some message _M_ to a secret exponent _d_, when you create a signature over a blinded message _M&#8217; = M*r^e_ (where _r_ is the blinding factor and _e_ is the public exponent), you also create a valid signature over _M_ thanks to the distributive property of exponentiation over multiplication.

Given the existence of blind signature schemes, Twitter can do something like the following to rate-limit Tor accounts without deanonymizing them:

  1. Say that @bob is an existing Twitter user who would like to make an anonymous account over Tor, which we&#8217;ll call @notbob. He computes _T = H(notbob) * r^e mod N_, where _H_ is a hash function, _r_ is a random number that Bob chooses, and _{e,N}_ is the public part of an Identity Provider&#8217;s RSA keypair (defined in step 2).
  2. Bob sends _T_ to an identity provider. This could be Twitter itself, or any service like Google, Identica, Facebook, LinkedIn, Keybase, etc. as long as it can check that Bob is probably a real person via SMS verification or a reputation-based algorithm. If Bob seems real enough, the Identity Provider sends him S_ig(T) = T^d mod N = H(notbob)^d * r mod N_, where _d_ is the private part of the Identity Provider&#8217;s RSA keypair.
  3. Bob divides _Sig(T)_ by _r_ to get _Sig(H(notbob))_, AKA his Identity Provider&#8217;s signature over the hash of his desired anonymous username.
  4. Bob opens up Tor browser and goes to register @notbob. In the registration form, he sends _Sig(H(notbob))_. Twitter can then verify the Identity Provider&#8217;s signature over &#8216;notbob&#8217; and **only** accept @notbob&#8217;s account registration if verification is successful!

It seems to me that this achieves some nice properties.

  * Every anonymous account is transitively validated via SMS or reputation.
  * Ignoring traffic analysis (admittedly a big thing to ignore), anonymous accounts and the actual identities or phone numbers used to validate them are unlinkable.

Thoughts? I&#8217;d bet that someone has thought of this use case before but I couldn&#8217;t find any references on the Internet.

[1] http://www.hit.bme.hu/~buttyan/courses/BMEVIHIM219/2009/Chaum.BlindSigForPayment.1982.PDF

[2] https://en.wikipedia.org/wiki/Blind\_signature#Blind\_RSA_signatures.5B2.5D:235

 [1]: http://techcrunch.com/2015/03/02/twitter-tor-phone-verification/
 [2]: http://www.washingtonpost.com/blogs/the-switch/wp/2014/03/24/tor-usage-in-turkey-surges-during-twitter-ban/