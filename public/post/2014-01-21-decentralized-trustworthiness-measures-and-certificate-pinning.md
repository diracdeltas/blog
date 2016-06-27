---
title: decentralized trustworthiness measures and certificate pinning
author: yan
layout: post
date: 2014-01-21
url: /decentralized-trustworthiness-measures-and-certificate-pinning/
categories:
  - code
  - encrypt the web
  - hacktivism

---
On the plane ride from Baltimore to SFO, I started thinking about a naming dilemma described by [Zooko][1]. Namely (pun intended): it&#8217;s difficult to architect name assignment systems that are simultaneously secure, decentralized, and human meaningful. [Wikipedia][2] defines these properties as:

  * **Secure**: The quality that there is one, unique and specific entity to which the name maps. For instance, [domain names][3] are unique because there is just one party able to prove that they are the owner of each domain name.
  * **Decentralized**: The lack of a centralized authority for determining the meaning of a name. Instead, measures such as a [Web of trust][4] are used.
  * **Human-meaningful**: The quality of meaningfulness and memorability to the users of the naming system. Domain names and nicknaming are naming systems that are highly memorable.

It&#8217;s pretty easy to make systems that satisfy two of the three. Tor Hidden Service (.onion) addresses are secure and decentralized but not human-meaningful since they look like random crap. Regular domain names like stripe.com are secure and human-meaningful but not decentralized since they rely on centralized DNS records. Human names are human-meaningful and decentralized but not secure, because multiple people can share the same name (that&#8217;s why you can&#8217;t just tell the post office to send $1000 to John Smith and expect it to get to the right person).

It&#8217;s fun to think of how to take a toy system that covers two edges of Zooko&#8217;s triangle and bootstrap it along the third until you get an almost-satisfactory solution to the naming dilemma. Here&#8217;s the one I thought of on the plane:

Imagine we live in a world with a special type of top-level domain called .ssl, which people have decided to make because they&#8217;re sick of the NSA spying on them all the time. .ssl domains have some special requirements:

  1. All .ssl servers communicate only over SSL connections. Browsers refuse to send any data unencrypted to a .ssl domain.
  2. All .ssl domain names are just the hash of the server&#8217;s SSL public key.
  3. The registrars refuse to register a domain name for you unless you show him/her a public key that hashes to that domain name.

This naming system wouldn&#8217;t be human-meaningful, because people can&#8217;t easily remember URLs like https://2xtsq3ekkxjpfm4l.ssl. On the other hand, it&#8217;s secure because the domain names are guaranteed to be unique (except in the overwhemingly-unlikely cases where two keys have the same hash or two servers happen to generate the same keypair). It&#8217;s not truly decentralized, because we still use DNS to map domain names to IP addresses, but I argue that DNS isn&#8217;t a point of compromise: if a MITM en route to the DNS server sends you to the wrong IP address, your browser refuses to talk to the server at that IP address because it won&#8217;t show the right SSL certificate. This is an unavoidable denial-of-service vulnerability, but the benefit is that you detect the MITM attack immediately.

Of course, this assumes we already have a decentralized way to advertise these not-very-memorable domain names. Perhaps they spread by trusted emails, or word-of-mouth, or business cards at hacker cons. But still, the fact that they&#8217;re so long and complicated and non-human-meaningful opens up serious phishing vulnerabilities for .ssl domains!

So, we&#8217;d like to have petnames for .ssl domains to make them more memorable. Say that the owner of &#8220;2xtsq3ekkxjpfm4l.ssl&#8221; would like to have the petname &#8220;forbes.ssl&#8221;; how do we get everyone to agree on and use the petname-to-domain-name mappings? We could store the mappings in a distributed, replicated database and require that every client check several database servers and get consistent answers before resolving a petname to a domain name. But that&#8217;s kinda slow, and maybe we&#8217;re too cheap to set up enough servers to make this system robust against government MITM attacks.

Here&#8217;s a simpler and cheaper solution that doesn&#8217;t require any extra servers at all: require that the distance between the hash of the petname and the hash of [server&#8217;s public SSL key] + [nonce] is less than some number D [1]. The server operator is responsible for finding a nonce that satisfies this inequality; otherwise, clients will refuse to accept the server&#8217;s SSL certificate.

[1] For purposes of this discussion, it doesn&#8217;t really matter how we choose to measure the distance between two hashes, but it should satisfy the following: (1) two hashes that are identical have a distance of 0, and (2) the number of distinct hashes that are at distance N from a hash H0 should grow faster than linearly in N. We can pick Hamming distance, for example.

In other words, the procedure for getting a .ssl domain now looks like this:

  1. Alice wants forbes.ssl. She generates a SSL keypair and mines for a nonce that makes the hash of the public key plus nonce close enough to the hash of &#8220;forbes&#8221;.
  2. Once Alice does enough work to find a satisfactory nonce, she adds it as an extra field in her SSL certificate. The registrar checks her work and gives her forbes.ssl if the name isn&#8217;t already taken and her nonce is valid.
  3. Alice sets up her site. She continues to mine for better nonces, in case she has adversaries who are secretly also mining for nonces in order to do MITM attacks on forbes.ssl in the future (more on this later).

Bob comes along and wants to visit Alice&#8217;s site.

  1. Bob goes to https://forbes.ssl in his browser.
  2. His browser sees Alice&#8217;s SSL certificate, which has a nonce. Before finishing the SSL handshake, it checks that the distance D1_forbes between the hash of &#8220;forbes&#8221; and the hash of [SSL public key]+[nonce] is less than Bob&#8217;s maximum allowed distance, D1. Otherwise it abandons the handshake and shows Bob a scary warning screen.
  3. If the handshake succeeds, Bob&#8217;s browser caches Alice&#8217;s SSL certificate and trusts it for some period of time T; if Bob sees a different certificate for Alice within time T, his browser will refuse to accept it, unless Alice has issued a revocation for her cert during that time.
  4. After time T, Bob goes to Alice&#8217;s site again. His maximum allowed distance has gone down from D1 to D2 during that time. Luckily, Alice has been mining for better nonces, so D1\_forbes is down to D2\_forbes. Bob&#8217;s browser repeats Step 2 with the new distances and decides whether or not to trust Alice for the next time interval T.

In reality, you probably wouldn&#8217;t want to use this system with SSL certs themselves; rather, it&#8217;d be better to use the nonces to strengthen trust-on-first-use in a key pinning system like TACK. That is, Alice would mine for a nonce that reduces the distance between the hash of &#8220;forbes&#8221; and the hash of [TACK Signing Key]+[nonce].

For those unfamiliar with TACK, it&#8217;s a system that allows SSL certificates to be pinned to a long-term TACK Signing Key provided by the site operator, which is trusted-on-first-sight and cached for a period of up to 30 days. Trust-on-first-use gets rid of the need to pin to a certificate authority, but it doesn&#8217;t prevent a powerful adversary from MITM&#8217;ing you every time you visit a site if they can MITM you the first time with a fake TACK Signing Key.

The main usefulness of nonces for TACK Signing Keys is this: it makes broad MITM attacks much more costly. Not only does the MITM have to show you a fake key, but they have to show you one with a valid nonce. If they wanted to do this for every site you visit, keeping in mind that your acceptable distances go down over time, they&#8217;d have to continuously mine for hundreds or thousands of domains.

Not impossible, of course, but it&#8217;s incrementally harder than just showing you a fake certificate.

Another nice thing about this scheme is that Bob can decide to set different distance thresholds for different types of sites, depending on how &#8220;secure&#8221; they should be. He can pick a very low distance D\_bank for his banking website, because he knows that his bank has a lot of computational resources to mine for a very good nonce. On the other hand, he picks a relatively high distance D\_friend for his friend&#8217;s homepage, because he knows that his friend&#8217;s one-page site doesn&#8217;t take any sensitive information.

My intuition says that sites with high security needs (banks, e-commerce, etc.) also tend to have more computational resources for mining, but obviously this isn&#8217;t true for sites like Wikileaks or some nonprofits that handle sensitive information liked Planned Parenthood. That&#8217;s okay, because volunteers and site users can also mine for nonces! Ex: if Bob finds a better nonce for Alice, he can send it to her so that she has a stronger certificate.

Essentially, this causes proof of trustworthiness to become decentralized: if I start a whistleblower site, I can run a crowd-mining campaign to ask thousands of volunteers around the world to help me get a strong certificate. I win as long as their combined computing power is greater than that of my adversaries.

Of course, that last part isn&#8217;t guaranteed. But it&#8217;s interesting to think about what would happen either way.

&nbsp;

 [1]: https://en.wikipedia.org/wiki/Zooko_Wilcox-O%27Hearn
 [2]: https://en.wikipedia.org/wiki/Zooko%27s_triangle
 [3]: https://en.wikipedia.org/wiki/Domain_name "Domain name"
 [4]: https://en.wikipedia.org/wiki/Web_of_trust "Web of trust"