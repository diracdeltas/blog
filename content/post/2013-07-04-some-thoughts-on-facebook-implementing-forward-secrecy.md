---
title: Some thoughts on Facebook implementing forward secrecy
author: yan
date: 2013-07-04
url: /some-thoughts-on-facebook-implementing-forward-secrecy/
categories:
  - code
  - encrypt the web
tags:
  - cryptography
  - security
  - SSL
  - surveillance

---
Last month, [CNET announced][1] that Facebook is working on implementing a property called **forward secrecy** in its encryption of user data. The article is pretty long, but the gist of it is:

  * [Forward secrecy][2] is good news, at least theoretically. Right now, when you send data to Facebook&#8217;s servers, your data gets encrypted so that someone who intercepts your data can&#8217;t read it unless they have Facebook&#8217;s secret key. However, if an eavesdropper is recording your messages now and somehow gets the secret key in the future, they can go back and decrypt all your encrypted communications. Forward secrecy in an encryption protocol, by definition, means that **if the secret key is compromised, your communications are still safe from decryption**.
  * Google is the only major web service that has forward secrecy in its encryption protocol enabled by default. Most services don&#8217;t do this because it&#8217;s computationally expensive.
  * The [leaked slides][3] about the NSA surveillance program, PRISM, suggest that the NSA is tapping into Internet connections and collecting data in transit between your computer and Facebook&#8217;s servers, or at least that this is possible.
  * Once your data is collected by the NSA, it can probably just sit around forever. That gives plenty of time for Facebook&#8217;s secret keys to be brute-forced or obtained by the NSA through a court order. Without perfect secrecy, all your data can be decrypted once this happens.

So in other words, Facebook is implementing an extra security measure to protect **data in transit** from users to their servers, and this announcement comes at an opportune time in light of the [PRISM][2] revelations in early July.

But data in transit isn&#8217;t the only kind of data that needs to be protected! What about **data at rest**?

To clarify what I mean here, let&#8217;s think about the flow of data from you to your friends when you make a FB post. This is simplified, but it goes something like:

  1. You type a message in a text box in your web browser and hit send.
  2. Your browser encrypts that message before it leaves your computer and goes on its way to a Facebook server.
  3. Your message travels over the Internet in encrypted form.
  4. Your message reaches a Facebook server, which decrypts the message and stores it in a database so that it can be retrieved later to be shown on your profile, in your friends&#8217; news feeds, in searches, and so forth.

In Steps 3 and 4, your data goes from in transit to at rest. Here&#8217;s a handy diagram from [Wikipedia][4] that distinguishes the three states of data, which I&#8217;m simplifying into two by merging &#8220;data at rest&#8221; and &#8220;data in use&#8221;:

<div style="width: 576px" class="wp-caption alignnone">
  <img alt="" src="https://upload.wikimedia.org/wikipedia/commons/2/23/3_states_of_data.jpg" width="566" height="343" />
  
  <p class="wp-caption-text">
    Three states of data.
  </p>
</div>

Nowadays, it&#8217;s expected for major web services to encrypt all data in transit by default using [HTTPS][5], which uses the **SSL/TLS** cryptographic protocols. This is generally done by using a persistent private server key for both **authentication** (verifying the server&#8217;s identity) and **encryption** (encoding the data so that only the server can read it). This does not provide forward secrecy, and all encrypted messages are compromised once the persistent server key is found.

Facebook&#8217;s announcement, which follows one by Google in 2011, reflects a recent shift toward supporting forward secrecy by generating ephemeral Diffie Hellman keys for encryption during each session. A persistent RSA key is still used to authenticate the server. The ephemeral keys used for encryption, however, are **not stored beyond a session**. Thus, even if the persistent key is compromised, data that has been obtained through eavesdropping on an Internet connection is still safe from decryption.

However, recall that in Step 4 above, Facebook decrypts your data and stores it in a database, presumably one that is password-protected or has some other security measure that lets a trusted set of people access it (database admins, for instance). At this point, your data is just sitting around in decrypted form, protected by whatever-FB-does-to-protect-its-servers. It would be impractical to do otherwise because performing encrypted query operations efficiently is <a href="https://en.wikipedia.org/wiki/Homomorphic_encryption" target="_blank">hard maths</a>, and basically the definition of a web app is something that stores data and performs interesting/useful queries.

With Facebook&#8217;s announcement of support for forward secrecy in TLS/SSL, we&#8217;ve been assured of increased attention to security for data in transit, but this of course says nothing about data at rest. Indeed, it seems that the NSA surveillance leaks have sparked relatively little discussion of policies at companies like Facebook for securing data at rest. That&#8217;s surprising to me, because the oft-spoken phrase &#8220;NSA back door&#8221; vividly suggests that the NSA is in the trusted set of people who have access to the decrypted data in servers at FB, Google, and so forth.

To be clear, forward secrecy is effective against a particular adversary model, namely wiretapping. Although the CNET article points out that the NSA is probably doing a bunch of wiretapping these days and has agreements with Internet service providers to facilitate such, I assume it&#8217;s still easy for the NSA to walk up to someone at Facebook and demand access to the database. In fact, regardless of how secure data is while in transit, it basically always needs to get decrypted on the server side in order to be useful for a web service. As long as it&#8217;s easy for the web service to access the decrypted data, it&#8217;s easy for the NSA to do so as well*.

*Barring policy changes that would legally prevent surveillance. Even so . . .

In short, there probably exists no NSA-proof cryptographic protocol for securing data at rest so long as companies agree to comply with gov-authorized surveillance programs. Forward secrecy doesn&#8217;t seem like much of a deterrent to PRISM in that case.

But in terms of protecting our privacy from attackers who **aren&#8217;t** government spying agencies, the security of data at rest matters as much as, if not more than, security for data in transit. Unfortunately, whereas TLS/SSL is an established and widely-used standard for encrypting data in transit (albeit flawed in practice), procedures for securing data at rest seem to vary widely between companies (see Appendix A). These procedures are often described in vague terms if at all. For instance, [one service][6] simply states that it &#8220;provides multiple layers of security around your information, from access protected data centers, through network and application level security . . . Sensitive information, such as your email, messages and passwords, is always encrypted.&#8221; The methods of encryption, whether they are applied in transit or at rest, and other important details related to the security of the service are left out.

It&#8217;s hard to blame the product description writers for omitting these crucial details, because most users simply don&#8217;t care as long as they feel reasonably secure. It&#8217;s easy to feel secure with assurances like the ones quoted above, and news articles about implementations of forward secrecy in the works at FB. And then you get things like [Twitter sending passwords in unencrypted form][7] or [Apple failing to properly authenticate SSL certificates][8] or a whole [Tumblr blog of websites that store and send passwords in plaintext][9], oops. There&#8217;s a difference between feeling secure and actually being secure on the web, clearly, but paranoia is tiring and you should probably go check on the 12 Facebook notifications that you got while reading this blog post.

===========

**Appendix A: A Brief, Non-Representative Survey of Company Policies for Securing Data at Rest**

In the course of researching this blog post, I made a post on a certain social media site asking my friends in tech about how their companies secured non-sensitive data at rest, where non-sensitive data includes user-to-user communications and metadata (but not passwords, SSNs, and financial data). The few answers I got were more sophisticated than, &#8220;In plaintext, on a password-protected database,&#8221; but I suspect there&#8217;s some selection bias here. Anonymized responses below.

\___\____

> I&#8217;ve worked with so many apps, and it&#8217;s almost always a database behind a firewall. I&#8217;ve occasionally built systems where financial data was stored in some more secure way: one, where financial data and transaction processing happened on a separate box with no services and a very minimal API (to limit exposure), and where it was impossible to retrieve the sensitive information via the API.<br data-reactid=".r[n8ym].[0]{comment10200377727028310_4734438}.[2:0].[5:0:right].[4:1].[5:0:left].[2:1].[2:0].[2:0:2].[3:0].[4:0:1]" /><br data-reactid=".r[n8ym].[0]{comment10200377727028310_4734438}.[2:0].[5:0:right].[4:1].[5:0:left].[2:1].[2:0].[2:0:2].[3:0].[4:0:2]" />The other one actually encrypted data using a key that was secured with the user&#8217;s password. It was re-encrypted using the user&#8217;s session on login, so the cleartext was only available when the user was logged in, during a request from that specific client. In this way, sensitive information was protected from our staff, and from an attacker, except that anyone who was actively using the system would be exposed (to a reasonably sophisticated attacker).<br data-reactid=".r[n8ym].[0]{comment10200377727028310_4734438}.[2:0].[5:0:right].[4:1].[5:0:left].[2:1].[2:0].[2:0:2].[3:0].[4:0:4]" /><br data-reactid=".r[n8ym].[0]{comment10200377727028310_4734438}.[2:0].[5:0:right].[4:1].[5:0:left].[2:1].[2:0].[2:0:2].[3:0].[4:0:5]" />None of these ideas provide protection from a government, though. Client-side encryption doesn&#8217;t really, either. Just look at what happened to Hushmail. I really want there to be some trustable encryption API in the browser, so that client-side encryption for web apps could be a real thing.

\___\____

> We actually encrypt (with a separately stored per-user key) all of our user message content. It makes me way way more comfortable debugging problems that come up in production knowing I&#8217;m not going to accidentally read a message that was meant for a friend or co-worker. It&#8217;s pretty low-effort and low-resource-intensity for us to do this so it seems silly not to.
> 
> I guess the tradeoff is that for somebody like Facebook they&#8217;d lose the ability to do queries in aggregate to make advertising or similar decisions based on content.
> 
> . . . the keys [for encrypting messages] are stored with symmetric encryption. It defends us against outside attackers getting use out of dumps of our database but in theory with access to the layer that gets the messages and has the keys the encryption doesn&#8217;t matter. The practical limit when we&#8217;ve looked at doing more is one where as long as our site can display your messages then there&#8217;s some set of our infrastructure an attacker could use to read those.

\___\____

> We use client-side encryption with client-generated keys for user history/bookmarks/passwords, etc. So adding a device to access said data involves JPAKE (key exchange) via our servers (which are treated as an untrusted 3rd party).
> 
> This means syncing is hard, but we associate each record with a record ID and sync them if the hash changes.
> 
> For telemetry, we use anonymized data &#8211; uploads are linked to a UUID that only the client stores locally.

 [1]: http://news.cnet.com/8301-13578_3-57591179-38/data-meet-spies-the-unfinished-state-of-web-crypto/
 [2]: https://en.wikipedia.org/wiki/Perfect_forward_secrecy
 [3]: http://www.guardian.co.uk/world/2013/jun/08/nsa-prism-server-collection-facebook-google
 [4]: https://en.wikipedia.org/wiki/File:3_states_of_data.jpg
 [5]: https://en.wikipedia.org/wiki/HTTP_Secure
 [6]: https://www.cloze.com/
 [7]: http://thenextweb.com/twitter/2012/12/20/twitter-com-login-flaw-causes-passwords-to-be-sent-in-plain-text-in-some-cases/
 [8]: https://www.trustwave.com/spiderlabs/advisories/TWSL2011-007.txt
 [9]: http://plaintextoffenders.com/
