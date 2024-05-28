---
title: "WAF as a weapon and DOS as a bullet"
collection: publications
permalink: /research-and-things/waf-as-a-weapon-and-dos-as-a-bullet
excerpt: ''
date: 2024-05-28
venue: 'zhero_web_security'
---

<img src="/images/waf-as-weapon.png">

After listening to the excellent talk by Shubs on WAF bypass techniques at NahamCon 2024 [1], I had a small idea.

Request size limits bypass
------
One of the techniques mentioned during the talk was about the "**request size limits**", to put it simply:

The default configuration of WAFs does not allow for the analysis of the entire request for obvious performance reasons. The WAF, being situated somewhere between the client and the server, must be able to forward the request quickly to avoid negatively impacting the user experience. Site/platform owners are looking for this balance between UX and security.

The technique - **applicable to requests containing a body** - simply involves **adding a parameter with a value that must be greater than or equal to the size limit set by default by the WAF** in question (*the limits can differ from one WAF to another*) and then adding our parameter containing our payload. The WAF will thus skip its analysis and allow the bypass.

<img src="/images/waf-as-weapon-1.png">
*Source: https://github.com/assetnote/nowafpls*

Parameter value reflected in cookies
------

During my research, I often come across **parameters whose values are reflected in cookies**. Their function is often "marketing" or used to track the user's journey on the target platform for A/B testing or other similar purposes. I was trying out some techniques to see if anything interesting could be derived from them, but very often there isn't much to gain.

Using the bypass mentioned above, it is possible **to "DOS" a user and make a site inaccessible to the target** for a duration depending on the lifespan of the cookie.

How?
------
There are **three** prerequisites:

- The parameter reflected in the cookies must be transmitted **via a request containing a body** (such as `POST`). As explained earlier, the request size limit bypass does not work via a `GET` request. If the parameter is transmitted via a `GET` request "by default", try passing it in a `POST` request and check that the cookie is still set, this will indicate that the parameter in question is also processed independently of the HTTP verb used (*for these two at least*), this is obviously not always the case but it happens
- The request should **not contain CSRF protection**, which is quite often the case for unauthenticated requests or those not transmitting sensitive information
- And of course, the target **must be protected by a WAF**, where the configuration concerning the maximum inspection size of the content body has not been changed (or is "capped")

You then need to create a basic HTML page - similar to a CSRF PoC - to send the crafted `POST` request, placing a payload that would typically be blocked by the WAF such as:

```
' OR SLEEP(10);-- 
```

as the value of the parameter reflected in the cookies. This **should be preceded by a parameter designed to "tank" the WAF with a value equal to or exceeding the WAF's maximum inspection size limit**, allowing the bypass.

<img src="/images/waf-as-weapon-2.png">

Since response inspection is not enabled by default on (or most) WAFs, the malicious value can make the round trip without issues: Server --> Client.

Example with **AWS WAF**:
>If you don't configure any of the response inspection options, response inspection is disabled.
[https://docs.aws.amazon.com/waf/latest/APIReference/API_ResponseInspection.html]


The request is therefore sent to the server, **which will return the value and assign it as a cookie**.

From this point on, -cookies from the request being inspected- each time the victim client attempts to navigate the site, **the WAF will block his requests**, resulting in a `403` error for the duration of the cookie's lifespan (*1 month in my case, arm yourself with patience*).

So:

1. during the initial request containing the malicious payload, the bypass request size limits was used
2. on return, the WAF does not inspect the response containing the malicious payload - in the `set-cookie` response header - coming from the server
3. all subsequent requests from the victim will include the malicious payload in his cookies -without bypass this time-, and therefore will be systematically blocked by the WAF

<img src="/images/waf-as-weapon-3.png">

The informed user can of course get rid of the curse by deleting cookies from his browser.
In terms of pure impact for this specific case, it's not exceptional, it's a kind of targeted DOS requiring user interaction. The victim must go to the attacker's site from where the request will be automatically sent and the cookie set.

You would have understood, it can be used in different ways even via XSS. Always useful when you need to demonstrate an impact and you have nothing on hand after going around.

The extremely high frequency of WAFs being bypassable through this technique makes the presence of this vulnerability quite common. This, because the two other constraints mentioned above are not particularly rare.

-----

[1] [#NahamCon2024: Modern WAF Bypass Techniques on Large Attack Surfaces](https://www.youtube.com/watch?v=0OMmWtU2Y_g&feature=youtu.be)
