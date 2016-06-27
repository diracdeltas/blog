---
title: certificate transparency for PGP?
author: yan
layout: post
date: 2014-08-14
url: /certificate-transparency-for-pgp/
categories:
  - code
  - encrypt the web

---
Yesterday, Prof. Matthew Green wrote a nice [blog post][1] about why PGP must die. Ignoring the UX design problem for now, his four main points were: (1) the keys themselves are too unwieldy, (2) key management is hard, (3) the protocol lacks forward secrecy, and (4) the crypto is archaic/non-sane by default.

Happily, (1) and (4) can be solved straightforwardly using more modern crypto primitives like Curve25519 and throwing away superfluous PGP key metadata that comes from options that are ignored 99.999999% of the time. Of course, we would then break backwards compatibility with PGP, so we might as well invent a protocol that has forward/future secrecy built-in via something like Trevor Perrin&#8217;s [axolotl ratchet][2]. Yay.

That still leaves (2) &#8211; the problem of how to determine which public key should be associated with an endpoint (email address, IM account, phone number, etc.). Some ways that people have tried to solve this in existing encrypted messaging schemes include:

  1. A central authority tells Alice, &#8220;This is Bob&#8217;s public key&#8221;, and Alice just goes ahead and starts using that key. iMessage does this, with Apple acting as the authority AFAICT. Key continuity may be enforced via pinning.
  2. Alice and Bob verify each others&#8217; key fingerprints via an out-of-band &#8220;secure&#8221; channel &#8211; scanning QR codes when they meet in person, reading fingerprints to each other on the phone, romantically comparing short authentication strings, and so forth. This is used optionally in OTR and ZRTP to establish authenticated conversations.
  3. Alice tries to use a web of trust to obtain a certification chain to Bob&#8217;s key. Either she&#8217;s verified Bob&#8217;s key directly via #2 or there is some other trust path from her key to Bob&#8217;s, perhaps because they&#8217;ve both attended some &#8220;parties&#8221; where people don&#8217;t have fun at all. This is what people are supposed to do with PGP.
  4. Alice finds Bob&#8217;s key fingerprint on some public record that she trusts to be directly controlled by Bob, such as his Twitter profile, DNS entry for a domain that he owns, or a gist on his Github account. This is what Keybase.io does. (I only added this one after **@gertvdijk** pointed it out on Twitter, so thanks Gert.)

IMO, if we&#8217;re trying to improve email security for as many people as possible, the best solution minimizes the extent to which the authenticity of a conversation depends on user actions. Key management should be invisible to the average user, but it should still be auditable by paranoid folks. (Not just Paranoid! folks, haha.)

Out of the 3 options above, the only one in which users have to do zero work in order to have an authenticated conversation is #1. The downside is that Apple could do a targeted MITM attack on Alice&#8217;s conversation with Bob by handing her a key that Apple/NSA/etc. controls, and Alice would never know. (Then again, even if Alice verified Bob&#8217;s key out-of-band, Apple could still accomplish the same thing by pushing a malicious software update to Alice.)

Clearly, if we&#8217;re using a central authority to certify people&#8217;s keys, we need a way for anyone to check that the authority is not misbehaving and issuing fake keys for people. Luckily there is a scheme that is designed to do exactly this but for TLS certificates &#8211; [Certificate Transparency][3].

How does Certificate Transparency work? The end result is that a client that enforces Certificate Transparency (CT) recognizes a certificate as valid** **if (1) the certificate has been signed by a recognized authority (which already happens in TLS) and (2) the certificate has been verifiably published in a public log. The latter can be accomplished through efficient mathematical proofs because the log is structured as a Merkle tree.

How would CT work for email? Say that I run a small mail service, yanmail.com, whose users would like to send encrypted emails to each other. In order to provide an environment for crypto operations that is more sandboxed and auditable than a regular webpage, I provide a YanMail browser extension. This extension includes (1) a PGP or post-PGP-asymmetric-encryption library, (2) a hardcoded signing key that belongs to me, and (3) a library that implements a Certificate Transparency auditor.

Now say that alice@yanmail.com wants to email bob@yanmail.com. Bob has already registered his public key with yanmail.com, perhaps by submitting it when he first made his account. Alice types in Bob&#8217;s address, and the YanMail server sends her (1) a public key that supposedly belongs to Bob, signed by the YanMail signing key, and (2) a CT log proof that Bob&#8217;s key is in the public CT log. Alice&#8217;s CT client verifies the log proof; if it passes, then Alice trusts Bob&#8217;s key to be authentic. (Real CT is more complicated than this, but I think I got the essential parts here.)

Now, if YanMail tries to deliver an NSA-controlled encryption key for Bob, Bob can at least theoretically check the CT log and know that he&#8217;s being attacked. Otherwise, if the fake key isn&#8217;t in the log, no other YanMail user would trust it. This is an incremental improvement over the iMessage key management situation: key certification trust is still centralized, but at least it&#8217;s auditable.

What if Alice and Bob want to send encrypted email to non-YanMail users? Perhaps the browser extension also hard-codes the signing keys for these mail providers, which are used to certify their users&#8217; encryption keys. Or perhaps the mail providers&#8217; signing keys are inserted into DNS with DANE+DNSSEC. Or perhaps the client just trusts any valid CA-certified signing key for the mail provider.

For now, with the release of Google [End-to-End][4] and Yahoo&#8217;s [announcement][5] to start supporting PGP as a first-class feature in Yahoo mail, CT for (post)-PGP seems promising as a way for users of these two large webmail services to send authenticated messages without having to deal with the pains of web-of-trust key management. Building better monitoring/auditing systems can be done incrementally once we get people to actually \*use\* end-to-end encryption.

Large caveat: CT doesn&#8217;t provide a solution for key revocation as I understand it &#8211; instead, in the TLS case, it still relies on CRL/OCSP. So if Bob&#8217;s PGP/post-PGP key is stolen by an attacker who colludes with the YanMail server, they can get Alice to send MITM-able messages to Bob encrypted with his stolen key unless there is some reliable revocation mechanism. Ex: Bob communicates out-of-band to Alice that his old key is revoked, and she adds the revoked key to a list of keys that her client never accepts.

_Written on 8/14/14 from a hotel room in Manila, Philippines_

_==============
  
_ 

**Update (8/15/14)**:

Thanks for the responses so far via Twitter and otherwise. Unsurprisingly, I&#8217;m not the first to come up with this idea. Here are some reading materials related to CT for e2e communication:

  * [Discussion thread on the Modern Crypto messaging mailing list][6]
  * [Dename][7] (semi-decentralized scheme for associating usernames with profiles, also uses Merkle trees)
  * [Paper on Enhanced Certificate Transparency for e2e mail (pdf)][8]
  * [Ben Laurie&#8217;s paper on Revocation Transparency (pdf)][9]

**Update (8/29/14):**

Since I posted this, folks from Google presented a similar but more detailed [proposal for E2E][10]. There has been a nice [discussion][11] about it on the Modern Crypto list, in addition to the one in the comments section of the proposal.

** Update (7/18/15):**

I know it&#8217;s pretty silly to be updating this after a year, but it&#8217;d be a travesty to not mention [CONIKS][12] here.

 [1]: http://blog.cryptographyengineering.com/2014/08/whats-matter-with-pgp.html
 [2]: https://github.com/trevp/axolotl/wiki
 [3]: http://www.certificate-transparency.org/
 [4]: https://code.google.com/p/end-to-end/
 [5]: http://arstechnica.com/security/2014/08/yahoo-to-begin-offering-pgp-encryption-support-in-yahoo-mail-service/
 [6]: https://moderncrypto.org/mail-archive/messaging/2014/000226.html
 [7]: https://github.com/andres-erbsen/dename
 [8]: https://eprint.iacr.org/2013/595.pdf
 [9]: http://sump2.links.org/files/RevocationTransparency.pdf
 [10]: https://code.google.com/p/end-to-end/wiki/KeyDistribution
 [11]: https://moderncrypto.org/mail-archive/messaging/2014/000708.html
 [12]: http://coniks.org
