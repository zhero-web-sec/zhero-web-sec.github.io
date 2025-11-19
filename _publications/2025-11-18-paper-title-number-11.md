---
title: "Unlocking Reflected XSS in the Astro framework"
collection: publications
permalink: /research-and-things/unlocking-reflected-xss-in-the-astro-framework
excerpt: 'CVE-2025-64764'
date: 2025-11-18
venue: 'zhero_web_security'
---

<img src="/images/astro-rxss-0.png">

## Introduction
Back to the Astro framework. Following our initial, [rather fruitful research on the framework](https://zhero-web-sec.github.io/research-and-things/astro-framework-and-standards-weaponization), which we encourage you to read if you haven't already, this paper details a small discovery concerning a Reflected XSS vulnerability that led to CVE-2025-64764.

As I mentioned in a previous article, I’ve decided to be more selective about which vulnerabilities deserve a dedicated paper. This helps avoid redundancy, prevent a drop in quality, and, of course, maintain exclusivity on a few exploits that support our side activities as a bounty hunters.

We hesitated to write a full article for this one, but an RXSS affecting a framework downloaded [hundreds of thousands of times per week](https://www.npmjs.com/package/astro) deserved it, even if it's shorter than usual.

## Index
- [Partial SSR with Server Islands](#section-1)
- [Exploit lock in a conditional block](#section-2)
- [Reflected XSS](#section-3)
    - [Anti-WAF Helper](#section-3-1)
- [Security Advisory - CVE-2025-64764](#section-4)
- [Conclusion](#section-5)

<h2 id="section-1">Partial SSR with Server Islands</h2>
Astro includes a feature called [Server islands](https://docs.astro.build/en/guides/server-islands/), which allows specific components to be isolated from the rest of the page and rendered dynamically on the server. This approach keeps the page itself static and fully cacheable, while each server island can generate dynamic content on the fly, enhancing overall performance.

Server islands run in their own isolated context outside of the page request and use the following pattern path to hydrate the page: `/_server-islands/[name]`. These paths are called via `GET` or `POST`, depending on the size of the data transmitted, and use three parameters (`GET` verb):

<img src="/images/astro-rxss-1.png">

[source code](https://github.com/withastro/astro/blob/create-astro%405.0.0-alpha.0/packages/astro/src/core/server-islands/endpoint.ts#L70)

`e`: Contains the name of the component export, which Astro uses to look up the correct export in the island module

`p`: Contains the props of the island component, encrypted server-side (*AES‑256‑GCM*) before being sent to the browser. The encryption key is generated or imported once, stored in `manifest.key`, and reused when the server islands component request arrives so the server can decrypt `p` and rebuild the props. This ensures that props are not sent in plain text between the browser and the server, and prevents an attacker from spoofing their content.

```
[ERROR] OperationError: The operation failed for an operation-specific reason
at AESCipherJob.onDone (node:internal/crypto/util:437:19)
```

<img src="/images/astro-rxss-2.png">

[source code](https://github.com/withastro/astro/blob/create-astro%405.0.0-alpha.0/packages/astro/src/core/server-islands/endpoint.ts#L124)


`s`: Contains the slot values ​​in JSON format, where each key is the name of a slot. A slot is a location in a component where you can inject content, these act as placeholders for external HTML and, by default, allow code injection **if the component template supports it**. Nothing exceptional in principle, just an optional feature.

<h2 id="section-2">Exploit lock in a conditional block</h2>
This is where it becomes interesting/problematic: it is, in fact, possible, **regardless of the component template being used** and whether it initially provides slots or not, to inject a slot containing an XSS payload. This **enables reflected XSS on any application**, as long as a server island is used at least once.

The key lies in the [following conditional block](https://github.com/withastro/astro/blob/190106149908ef6826899459146ef9f0ead602ab/packages/astro/src/runtime/server/render/component.ts#L279):

<img src="/images/astro-rxss-3.png">

This code handles the case where `Component` is not a function (*Astro/React/.. component*). In this case, Astro generates a complete template and returns the built HTML, independently of any template originally provided by the developer.

- the `Component` value, being a string here, is sanitized and then injected as the tag name
- `childSlots`, the value provided to the `s` parameter, is injected as a child of the tag without any sanitization

Access to this block would unlock the exploit, and for that, the value of the `Component` must be a `string`, which is not originally the case.

`Component` is the export of the island module selected via `data.componentExport`, [itself defined by](https://github.com/withastro/astro/blob/create-astro%405.0.0-alpha.0/packages/astro/src/core/server-islands/endpoint.ts#L128) the parameter `e`. The value of `e` is used as a key to access the corresponding property in `componentModule`: 


```
const componentModule = await imp();
let Component = componentModule[data.componentExport];
```

By default, its value is `default`, whose type is a `function` :

<img src="/images/astro-rxss-4.png">

**Context note**: Minimalist vibe coded component named `ServerTime.astro`, without any slots, used on the main page (`index.astro`). On each visit to the root `/` (*or when the client decides to refresh this server island*), a call is made to `/_server-islands/ServerTime`. The value of `e` is `default`, the encrypted data is set as the value of `p`, and `s` is naturally empty since there are no slots.

<h2 id="section-3">Reflected XSS</h2>

As explained earlier, `Component` is defined by the value passed to `e`, which acts as the key for the `componentModule` object. From there, a simple console.log is enough to reveal all existing exports:

<img src="/images/astro-rxss-5.png">

A very pleasant sight appears in the output: in addition to `default`, two other exports are present by default : `url`, whose value is `undefined`, and… `file`, **a string** whose value is the absolute path of the island file.

Since the type is a `string`, we can fulfill the condition of the block above, unlocking the exploit by setting `file` as the value of the parameter `e` and injecting an XSS payload as a slot (`s` parameter):

<img src="/images/astro-rxss-6.png">

There’s no need to set anything in the value of `p`, since the returned template is the one generated by Astro, **completely ignoring the template intended by the developer**. For the same reason, the JSON keys (`s` value) are irrelevant here, since we are not pointing to any existing slots. As you’ve probably guessed, the same outcome occurs even if the original template were entirely empty.

<h3 id="section-3-1">Anti-WAF Helper</h3>

Another interesting fact is that all values (*from the key/value pairs*) provided in the JSON passed to the `s` parameter are [concatenated together](https://github.com/withastro/astro/blob/190106149908ef6826899459146ef9f0ead602ab/packages/astro/src/runtime/server/render/component.ts#L282) before being injected as a child:

```
const childSlots = Object.values(children).join('');

(...)

${markHTMLString(
    childSlots === '' && voidElementNames.test(Tag) ? `/>` : `>${childSlots}</${Tag}>`,
)}`;
```

This allows us to divide our payload into as many pieces as we want, offering an almost guaranteed bypass of all WAFs :

```
?e=file&p=&s={"zhero;":"<img+src=x+one","inzo":"rror=a","dz":"lert(0)>"}
```

<img src="/images/astro-rxss-8.png">

<h2 id="section-3">Security Advisory - CVE-2025-64764</h2>
Affected versions : =< 5.15.6

Patched versions : 5.15.8

CVSS score : `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:H/A:N` (High - 7.1)

[https://github.com/withastro/astro/security/advisories/GHSA-wrwg-2hg8-v723](https://github.com/withastro/astro/security/advisories/GHSA-wrwg-2hg8-v723)

<h2 id="section-4">Conclusion</h2>
As we have seen, arbitrary JavaScript code execution is possible, requiring a user click, and the sole condition for its exploitability is that the targeted application uses at least one server island. The consequences can obviously be severe, especially considering the popularity of the framework (*+800,000 downloads per week*). The impact aligns with the typical range of what is achievable through classic XSS attacks: from the exfiltration of sensitive data to potential full account takeover.

Fortunately, the team was once again very responsive and the fixes were implemented and released within 48 hours of the vulnerability being reported.

Timeline :
- 2025-11-13 : Report sent via the GitHub template
- 2025-11-14 : Report acknowledged and accepted
- 2025-11-14 : PR review requested by the Astro team
- 2025-11-15 : PR review completed
- 2025-11-15 : Implementation of the latest fixes and release of the patched version - `astro@5.15.8`
- 2025-11-19 : Publication of the security advisory

Thank you for reading.

Al hamduliLlah;

Research conducted by [zhero;](https://x.com/zhero___) & [inzo_](https://x.com/inzo____)

*Published in November 2025*