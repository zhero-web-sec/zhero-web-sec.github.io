---
title: "Nuxt, show me your payload - a basic CP DoS"
collection: publications
permalink: /research-and-things/nuxt-show-me-your-payload
excerpt: 'CVE-2025-27415'
date: 2025-03-03
venue: 'zhero_web_security'
---

<img src="/images/nuxt-dos-1.png">

## Introduction
During a research, it is relatively important not to lock ourselves too much into a paradigm when we particularly appreciate a type of vulnerability, our reading is no longer objective and this naturally reduces the range of our radar.
On the other hand, these biases can sometimes be advantageous, allowing us to perceive potential threats in places that may initially seem harmless for most people.

One such bias helped me identify a small vulnerability during my research on [Nuxt](https://www.npmjs.com/package/nuxt), an open-source JavaScript framework used for building full-stack web applications with Vue.js :

The ability to trick the server into rendering the payload on main routes, which, if a caching system is in place, could force the caching of the response on these routes. This could severely impact the application's availability, as the page content would be entirely altered, rendering the site unusable.


## Index
- [Rendering payload](#section-1)
- [Lax check](#section-2)
- [Exploit: CP-DoS - Query](#section-3)
  - [Scope](#section-3-1)
- [Exploit: CP-DoS - Hash](#section-4)
- [Security advisory - CVE-2025-27415](#section-5)
- [Conclusion](#section-6)


<h2 id="section-1">Rendering payload</h2>

On Nuxt, payloads are used in order to transmit data from the server-side to the client-side. These can be in JSON (*default*) or JS format depending on the configuration.

The data is passed to the frontend without requiring additional API requests after the initial page load and injected into the HTML of the page, between the script tags containing the attribute id `__NUXT_DATA__` :

<img src="/images/nuxt-dos-3.jpg">

Nuxt also externalizes this data on a specific route that matches the following pattern:
`/endpoint/_payload.json` (or `.js`) : 

<img src="/images/nuxt-dos-2.png">

<h2 id="section-2">Lax check</h2>

Checking the source code a little more closely, it turns out that Nuxt uses a regular expression to check if it is a rendering payload route:

<img src="/images/nuxt-dos-4.png">

[[source]](https://github.com/nuxt/nuxt/blob/b0729241bcc219b8ec84b2ba4f0d8eefa88849d9/packages/nuxt/src/core/runtime/nitro/renderer.ts#L242)

Nothing weird at first glance, if the "path" ends with `/_payload.json` or `/_payload.js` with a *possible* beginning of query string then the constant `isRenderingPayload` becomes `true`.
I use the word "**path**" but is it really the case? The variable passed to the `test()` method is named `url` and this does not seem to be an *abuse of language*.

If it is really the full URL that is being tested instead of the path, then it is possible to **trick the check** by calling a normal route by adding a query string like: `?poc=/_payload.json`

This will respect the constraints of the regex, and set the value of `isRenderingPayload` to `true`, consequently retrieving the payload:

<img src="/images/nuxt-dos-5.png">

No suspense, after a little test it was indeed the case, the regex is **tested on the entire URL returning the JSON response considering that it is a rendering payload route**:

<img src="/images/nuxt-dos-6.png">

<h2 id="section-3">Exploit: CP-DoS - Query</h2>

Most cache/CDN systems rely -*among other things*- on the path during [content-negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation). Being able to force the payload to be rendered on a "normal" route by passing the rest of the "path" (`/_payload.json`) as a URL parameter (*key/value*) allows to cache the response containing the payload instead of the normal page, completely impacting its availability, the latter simply being no longer usable. The only condition -*apart from the presence of a cache system*- for this, is that the URL parameters are not part of the cache-key, this will result in the cache not making the difference between:

- A: **https://www.example.com/?poc=/_payload.json**
- B: **https://www.example.com/** 

Consequently serving the response of request `A` -*cached by the attacker*- to users making request `B`. The attacker must therefore wait for the cache-duration to be reset and be the first to send their request during this brief window, ensuring that their poisoned response gets cached as is customary during a classic cache poisoning attack.

<img src="/images/nuxt-dos-7.png">
*example of an e-commerce site under Nuxt - (BBP)*

Note: The status code of the poisoned response is `200`, which is in line with most cache system rules.

<h3 id="section-3-1">Scope</h3>

Interestingly, **all pages** [1] **of a Nuxt application are vulnerable**, even the ones not transmitting data from the server to the client, the object will be empty, but still returned, increasing the attack surface for our CP DoS.

[1] Excluding those from the server like `/api` the latter not being managed by Nuxt but by [Nitro](https://github.com/nitrojs/nitro), its web server.

<img src="/images/nuxt-dos-8.png">
*empty payload returned*

<h3 id="section-3-2">Déjà vu</h3>

Those who read [my previous article on Next.JS](https://zhero-web-sec.github.io/research-and-things/nextjs-cache-and-chains-the-stale-elixir) will surely have noticed:
this is an exploit similar to the "*Internal URL parameter and pageProps*" part, forcing the caching of the response altered by the addition of a URL parameter influencing the internal functioning of the framework. Small difference however, on Next.JS it was an "existing" internal URL parameter while here it is a question of abuse of the regex, no URL parameter - even internal - being initially provided for this.

<h2 id="section-4">Exploit: CP-DoS - Hash</h2>

The requirement that URL parameters not be part of the cache key bothered me, and I pondered how to circumvent this constraint until I came up with the idea of ​​exploiting the hash; this had already been useful to me in the past to obtain an arbitrary JS execution during an Intigriti challenge (*for which my [write-up](https://zhero-web-sec.github.io/xss-intigriti-challenge-0523/) was selected among the three winners huh*).

In a URL, the hash portion (`#`) is exclusively processed by the browser and is not transmitted to the server. When a request is sent, the hash portion is therefore not part of the journey. This concerns browsers; sending a hash through a proxy isn't technically a problem, but the server isn't supposed to interpret it, and this part isn't supposed to influence the choice of the resource to retrieve:

From [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986#section-3.5):
>As such, the fragment identifier is not used in the scheme-specific processing of a URI; instead, the fragment identifier is separated from the rest of the URI prior to a dereference, and thus the identifying information within the fragment itself is dereferenced solely by the user agent, regardless of the URI scheme.

With the entire URL tested, it was almost certain that sending a request to `#/_payload.json` through the proxy would achieve the same result:

<img src="/images/nuxt-dos-9.png">
*another BBP target*

It should be noted, however, that depending on the stack (*reverse proxy, CDN, etc*), the hash may be encoded (*stripped or even generating an error*) along the way, arriving at Nuxt in a form that is interpreted as a path -`%23/_payload.json`-, which does not exist, resulting in a 404 error and consequently aborting the attack.

I was able to validate this behavior locally and on certain BBP targets that did not have a cache system and therefore could not exploit it in this form, the few targets at my disposal either having a stack encoding the character or no cache system to poison.

I haven't had time to do a case-by-case analysis yet to look at how each CDN behaves with the hash, so I'll update this section once that's done. I still thought it would be useful to highlight this possibility.

<h2 id="section-5">Security advisory - CVE-2025-27415</h2>

**Affected versions**
3.0.0 < 3.16.0

**Patched versions**
3.16.0

**Severity**

`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H`(High 7.5)


[https://github.com/nuxt/nuxt/security/advisories/GHSA-jvhm-gjrh-3h93](https://github.com/nuxt/nuxt/security/advisories/GHSA-jvhm-gjrh-3h93)

<h2 id="section-6">Conclusion</h2>

A simple request can make a site unusable, which can have a significant financial impact depending on the nature of the application (e-commerce, web3, etc.).

I was pleasantly surprised by the transparency and responsiveness of the Nuxt team, who implemented a fix in less than two weeks after my report. While this wasn't the vulnerability of the century, it was reassuring, especially for a framework as popular as Nuxt, which is currently downloaded over [3 million times per month](https://www.npmjs.com/package/nuxt).

Thank you for reading.

Al hamduliLlah;

*Published in March 2025.*