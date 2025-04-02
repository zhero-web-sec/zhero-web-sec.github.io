---
title: "React Router and the Remix'ed path"
collection: publications
permalink: /research-and-things/remix
excerpt: 'CVE-2025-31137'
date: 2025-04-01
venue: 'zhero_web_security'
---

<img src="/images/remix-url-0.png" width="70%" style="display: block; margin: 0 auto">

## Introduction
Continuing the momentum, inzo_ and I collaborated again to take a look at Remix, a popular full-stack web framework. During our research, we discovered some interesting behaviors, including a vulnerability in [React Router](https://reactrouter.com/), a library used to manage multi-strategy routing in React applications. React Router is downloaded over [13.2](https://www.npmjs.com/package/react-router) million times per week and is developed and maintained by Remix Run. At the end of 2022, Remix and its team [joined Shopify](https://shopify.engineering/remix-joins-shopify).

The vulnerability allows URL manipulation through the `Host`/`X-Forwarded-Host` header and affects all users of Remix 2, as well as, more generally, React Router 7, who use the Express adapter. This could potentially lead to several exploits, as we will demonstrate in this brief paper.

## Index
- [Remix and data request, again](#section-1)
- [_data parameter](#section-2)
  - [Exploitation - CPDoS](#section-2-2)
- [React Router, Express adapter and a talkative port](#section-3)
  - [Exploitation - chain for a less picky CPDoS](#section-3-2)
  - [Exploitation - WAF bypass and escalations](#section-3-3)
- [Security Advisory - CVE-2025-31137](#section-4)
- [Conclusion](#section-5)


<h2 id="section-1">Remix and data request, again</h2>
As mentioned earlier, although our research initially focused on Remix, it led to the discovery of an issue in React Router. During our investigation, we identified several interesting behaviors in Remix, including a URL parameter that, if present, allows the request to be treated as a data request. This request returns a JSON object containing the data (*or not*) transmitted from the server-side.

To transmit data to a route, Remix uses what they call a "loader": a server-side function that allows —during the initial server rendering— to feed the HTML document.

>Each route can define a loader function that provides data to the route when rendering. ([Remix documentation](https://remix.run/docs/en/main/route/loader))

<img src="/images/remix-url-1.png">

<h2 id="section-2">_data parameter</h2>

While analyzing the source code, we quickly came across this interesting [piece of code](https://github.com/remix-run/remix/blob/6ad1f21ae63964221e57b609a57b36f6e8a81e7d/packages/remix-server-runtime/server.ts#L162):

<img src="/images/remix-url-2.png">

If the URL parameter `_data` is present, its value is retrieved and passed to the variable `routeId`. Then, the corresponding data request is retrieved and assigned to the variable `response`, the latter then being returned.

The server naturally behaves differently depending on where and how the URL parameter is used; If it's used on a page that doesn't use a loader the server returns a `400` response JSON object containing an error message:

<img src="/images/remix-url-3.png">

If this is a page using a loader but no valid value is specified, we receive a `403` response, which is a JSON object containing an error message:

<img src="/images/remix-url-4.png">

We saw in the code snippet above that the value of the `_data` parameter was assigned to the `routeId` variable, which is simply the name of the page prefixed by `routes/`. So the correct value for the `/ssr` page (*using a loader*) is `routes/ssr` allowing us to get a `200` response:

<img src="/images/remix-url-5.png">

<h3 id="section-2-2">Exploitation - CPDoS</h3>
Now that we know we can tamper with any response from a Remix application and obtain different types of status codes without any cache-control constraints, we have what we need for a potential DoS attack via cache poisoning.

Most cache/CDN systems rely -among other things- on the path during [content-negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Content_negotiation). Being able to force the JSON object to be rendered on any route by adding the URL parameter allows to cache the response containing the data object instead of the normal page, completely impacting its availability, the latter simply being no longer usable. The only condition -apart from the presence of a cache system- for this, is that the URL parameters are not part of the cache-key, this will result in the cache not making the difference between:

- A: **https://www.example.com/?_data=routes/targetedPage**
- B: **https://www.example.com/**

Consequently serving the response of request A -*cached by the attacker*- to users making request B. The attacker must therefore wait for the cache-duration to be reset and be the first to send their request during this brief window, ensuring that their poisoned response gets cached as is customary during a classic cache poisoning attack.

This behavior is similar to what I previously found on [Nuxt](https://zhero-web-sec.github.io/research-and-things/nuxt-show-me-your-payload) and [Next.js](https://zhero-web-sec.github.io/research-and-things/nextjs-cache-and-chains-the-stale-elixir) ("*Internal URL parameter and pageProps*" part).

Whether it's an internal parameter that's supposed to be stripped, as is the case with some frameworks, or a legitimate feature, it doesn't change the fact that it doesn't mix well with cache systems if no measures are put in place.

Example of a real case where the cache was poisoned (*bug bounty program*), and the initial content of the page was completely altered/replaced by the JSON object:

<img src="/images/remix-url-6.png">
*a very well known public program, rewarding the find with a bounty of $4,500*

Having the ability to alter content with multiple HTTP codes (`400`, `403`, and `200`) allows you to comply with most cache policies, and to be sure that you can poison the cache **if the URL parameters are not included in the cache-key**.

It's cool but not exceptional, the condition related to the cache-key is quite **restrictive** even if we found several vulnerable targets in the wild. We will come back to this a bit later with a method to remove this condition.


<h2 id="section-3">React Router, Express adapter and a talkative port</h2>

While React Router is at the heart of Remix, it can be used in various environments, and there are several adapters to ensure it works correctly and consistently across different contexts. The one affected by the vulnerability in our case is the Express adapter. Long story short:

It is possible to call any path directly from the `host` or `x-forwarded-host` header due to the lack of port sanitization [**1**]. Although it is possible to take advantage of both headers, we will focus on `x-forwarded-host` because manipulating the `Host` value will result in an error with most reverse proxies/CDNs even if it's perfectly valid/possible locally. Furthermore, when both values are present, the `X-Forwarded-Host` value takes precedence as we will see shortly.

Without further ado, here is [the vulnerable code](https://github.com/remix-run/remix/blob/0e9772c8b4456c239ea148c4003932ce63a7198e/packages/remix-express/server.ts#L92):

<img src="/images/remix-url-7.png">

The code is clear and speaks for itself:
- The two values are retrieved then split in order to retrieve the part after the colon character (`:`), so traditionally, the "port"
- The value of `hostnamePort` (`x-forwarded-host`) has priority over `hostPort` (since the values ​​are evaluated from left to right, the first truthy value is chosen) and is assigned to the `port` variable
- The `port` value is directly concatenated - without sanitization/verification - to `req.hostname` (*the value of* `Host`*, without the port*) then assigned to the variable `resolvedHost`; 

[**1**] *Here, we can see that, unlike* `x-forwarded-host`*, it is possible to inject the path directly at the* `host` *level (without needing to go through the supposed port). This is because the value of* `req.hostname` *is used in its entirety, without sanitization/verification, and is then used to create the URL object. However, as mentioned earlier, the use of* `Host` *for URL manipulation is very rarely possible in real-world scenarios, unlike* `x-forwarded-host`.
- The `resolvedHost` variable is concatenated - again, without sanitization/verification - with the protocol and the current path (`req.originalUrl`) to create the URL object

So this allows us to call a path this way:

<img src="/images/remix-url-8.png">

Since the part before the column character is not used, we omit it and place the desired path directly. This will create the following URL :

`http:` (*req.protocol*) + `localhost/ssr` (*req.Hostname* + *port*) + `/` (*req.originalUrl*) 

This results in http://localhost/ssr/ (without modifying the path), allowing the bypass of certain path-based mechanisms.

<h3 id="section-3-2">Exploitation - chain for a less picky CPDoS</h3>

If the target - *in addition to using React Router and its Express adapter* - uses Remix, we can chain this with the previously mentioned behavior to bypass the "*URL parameter not in the cache-key*" condition making DoS via cache-poisoning **much easier to exploit**:

<img src="/images/remix-url-9.png">
*huh*

We specify a path using a loader directly after the column character `:` (to get a `200` response), `/ssr` in our case, then we add the URL parameter `?_data` as well as the corresponding `routeId` as explained previously.
Note that we end the payload with the character (`&`) so that the slash of the current path (`/`) doesn't distort the value, the latter being concatenated after the `Host` + *supposed* `Port`.

The example here illustrates a chain involving Remix-specific behavior, but it is obviously possible to achieve a CPDoS on any target using React Router (with its Express adapter) by forcing the caching of a `404` page simply by specifying a non-existent path, or a `200` page different from the expected one by forcing the caching of a different valid endpoint. All of this, of course, assumes the target has a caching system (*without additional conditions this time*).

We were able to find several large programs vulnerable to this. With the cache-key condition removed, the exploit became **much more accessible**.

<h3 id="section-3-3">Exploitation - WAF bypass and escalations</h3>
It is also possible to bypass any firewall, whether for a reflected XSS [**2**], an SQLi (*whose payload would be in the path or URL parameter*), or simply to access paths normally prohibited. 

In the following example, we assume that a firewall checks both the URL and the value of the headers. We bypass it by splitting the payload — in this case, for an SQLi — into two parts, taking advantage of the fact that the part in the URL is the last to be concatenated (as explained earlier) to form the final URL, and therefore has no impact on routing.

<img src="/images/remix-url-10.png">

*Since the tests are being carried out locally, no firewall is present. However, this specific case was tested in a real-world scenario, where it successfully bypassed the WAF in place.*

Note: When crafting, take the path part into account in order to assemble it harmoniously with the rest of the payload and avoid errors. In the case of our example, we commented out the slash `/` in the path so that it does not break the payload (*via* `/*` `*/` *before/after the slash*).

It is, of course, possible to apply the same technique to access a path managed by the router that is protected by a WAF. While it is not the norm to rely solely on a firewall as a security layer, we all know the gap between theory and practice in the real world so it may be useful to keep this possibility in mind.

[**2**] Additionally, a Reflected XSS can be escalated to a Stored XSS if a caching system is present. The difference is that the path (`req.originalUrl`) must remain unchanged and cannot be used for payload splitting, as the latter is considered during the content-negotiation phase.

As you may have gathered, the techniques and exploits mentioned here are not exhaustive.

<h2 id="section-4">Security Advisory - CVE-2025-31137</h2>

@react-router/express (npm) -> Affected versions: 7.0.0-7.4.0 -> Patched versions: 7.4.1
@remix-run/express (npm) -> Affected versions: >=2.11.1 -> Patched versions: 2.16.3

**Patches**

This issue has been patched and released in Remix 2.16.3 / React Router 7.4.1.

**CVSS Score**

`CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H` (High/7.5)


[https://github.com/remix-run/react-router/security/advisories/GHSA-4q56-crqp-v477](https://github.com/remix-run/react-router/security/advisories/GHSA-4q56-crqp-v477)

<h2 id="section-5">Conclusion</h2>

As explained in this brief paper, this vulnerability can be exploited in several ways, either directly or indirectly if chained with other exploits. As is often the case with widely used software, the impact on the ecosystem can be significant. React Router is no exception; downloaded over 13 million times per week, we found several impacted sites as part of bug bounty programs. 

All of these sites were very popular and widely used globally, and the CPDoS aspect alone could render them completely unusable. This is particularly problematic, especially when the company’s business model relies on its application.


That said, the Remix maintainers were very responsive. The only downside is that there is no security contact listed on the Remix repo/site, nor is there an option to submit a report directly via the GitHub template. As a result, we had to leave a message on the Discord server to get guidance on how to proceed with reporting vulnerabilities. Here is the timeline once the first contact was made:

- **2025/03/26**: Report sent by email
- **2025/03/26**: [Fix](https://github.com/remix-run/remix/pull/10553/files) implemented
- **2025/03/28**: Release of a new version (*v2.16.3*) containing the fix
- **2025/04/01**: Security advisory/CVE-2025-31137

Thank you for reading.

Al hamduliLlah;

[zhero;](https://x.com/zhero___) & [inzo_](https://x.com/inzo____)

*Published in April 2025*