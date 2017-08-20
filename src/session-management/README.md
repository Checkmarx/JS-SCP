Session Management
==================

In this section, we will cover the most important aspects of session management
according to [OWASP Secure Coding Practices] [16]. An example is provided along with
an overview of the rationale behind these practices.

Although in-depth knowledge about how Session Management works is provided,
**readers are encouraged to use well known, deeply tested and actively
maintained session management mechanisms like the ones provided by all major
frameworks**.

As an introductory note, let's make clear that Session Management has nothing
to do with HTTP session, fully described on [A typical HTTP session on MDN][1].
Instead, it refers to an applicational layer mechanism to workaround the
**stateless** property of protocols like HTTP: each request is independent and
no relationship exists between any previous or future requests.

In this chapter, we will cover the session flow shown by the picture below,
which illustrates how web applications implement User Sessions to persist
user's identity across multiple HTTP requests.

![Session Flow][2]

From a macro perspective establishing a user session is quite straightforward:

1. The client makes a request;
2. The server creates an identifier and sends it back along with the response;
3. The client reads and persists the identifier sent unchanged;
4. The client makes a request, sending the identifier read and persisted on
   _step 3_;
5. The server reads and validates the identifier. Then the workflow continues
   from _step 2_.

From the workflow described above, it is clear that it is server's
responsibility to create the identifier. Why? Because this identifier is one of
the most critical parts on all this mechanism and so:

* **it should always be created on a trusted system** and
* **the application should only recognize as valid session identifier created
  by/on this trusted system**.

Regarding the identifier itself, **it is important that session management
controls use well vetted algorithms that ensure sufficient randomness** so that
identifiers can not be predicted by other systems.

As soon as the server creates the identifier it should then send it back to the
client along with the response (_step 2_). Of course the client should know
where to read this information as it will have to store it locally until the
next request.

Back in 1994 Netscape introduced [HTTP Cookie][3] support on its navigator,
based on the concept of "_magic cookie_": "_a token or short packet of data
passed between communicating programs, where the data is typically not
meaningful to the recipient program_" ([source][4]). Then it was added to the
HTTP specification as [HTTP State Managament Mechanims][5].

To send the identifier along with the response, the server has to add a
`Set-Cookie` HTTP response header providing one or more directives[^1].

```
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly
```

Unlike other cookies, Session Cookies do not have an expiration date (this is
how web browsers distinguish them from other cookies) and they exist only
_in-memory_ while the user navigates the website. When browser is closed all
session cookies are deleted.

To properly set a `Session Cookie`, the server should define the following cookie
attributes:

1. `Domain`: hosts to which the cookie will be sent;
2. `Path`: indicates a URL path that must exist in the requested resource
   before sending the Cookie header;
3. `Secure`: prevents cookies transmitted over an encrypted connections to be
   transmitted over unencrypted connections);
4. `HttpOnly`: prevents access to the cookie using the JavaScript API
   `document.cookie` (this should be done unless JavaScript access to the
   cookie is strictly required);

**Note 1**: Chrome recently introduced the `SameSite` attribute which "_allows
servers to mitigate the risk of CSRF and information leakage attacks by
asserting that a particular cookie should only be sent with requests initiated
from the same registrable domain._". Although it is not (yet) standard, it has
Firefox's support and developers community interest ([source][6]).

Keep in mind that **session identifiers should not be exposed in URLs or
included in error messages or logs**.
In the past some web frameworks and/or applications used to pass the session
identifier as `GET` parameters (`jsessionid` and `PHPSESSID` were commonly used
parameters names). This not only is a bad practice but also leads to security
issues like [session fixation][7]: sharing the URL may suffice to allow others
to impersonate you.

The [OWASP Secure Coding Practices - Quick Reference Guide][8] has other
recommendations you should use to evaluate any Session Management control.
Before showing a practical example, some general best practices your
application should follow regarding User Sessions:

1. Consistently utilize HTTPS rather than switching between HTTP and HTTPS. If,
   for some reason you have to switch from one to the other, you should
   deactivate and generate a new identifier;
2. Whenever possible you should opt by a per-request, as opposed to per-session
   identifier;
3. Logout functionality should be available from all pages protected by
   authorization and it should fully terminate the associated session or
   connection.

A final note about **server-side session data which should be protected from
unauthorized access by other users of the server, by implementing appropriate
access controls on the server**.

The example below uses [Express - Node.js web application framework][9] and the
[express-session middleware][10] which supports multiple store types. The
example uses the [connect-sqlite3 module][11] to store session data on a
[SQLite][12] database, nevertheless this may not be suitable for a production
ready application.

```javascript
const express = require('express');
const session = require('express-session');
const SQLiteStore = require('connect-sqlite3')(session);
const util = require('util');

// express-session configuration
const sessionMiddleware = session({
  store: new SQLiteStore({
    table: 'sessions',
    db: 'sessions.db',
    dir: __dirname
  }),
  secret: 'The quick brown (Fire)Fox jumps over the lazy IE',
  saveUninitialized: false,
  resave: false,
  rolling: true,
  name: 'ssid',
  domain: 'localhost',
  httpOnly: true,
  secure: true,
  sameSite: 'strict'
});

const app = express();

// tell Express to use our `sessionMiddleware`
app.use(sessionMiddleware);

app.get('/', (req, res) => {
  // the next line will trigger the `Set-Cookie` (otherwise no cookie would be
  // set)
  req.session.counter = (req.session.counter || 0) + 1;

  res.send(util.format('You have visited this page %d times',
    req.session.counter));
});

app.listen(3000, () => {
  console.log('Example app listening on port 3000');
});
```
The image below shows the `Set-Cookie` HTTP response header:

![`Set-Cookie HTTP response header][13]

and the next one, the database entry on `sessions` tables:

![entry on database `sessions` table][14]

Note that even this sample application is served over HTTP, we did set the
'Secure' cookie attribute: this does not prevent the application from running
and when switching to HTTP (may on production), the attribute won't be
forgotten (or course you can set it conditionally base on, for example,
`process.env.NODE_ENV`).

[^1]: For a complete list of Directives visit [Set-Cookie on MDN][15]

[1]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Session
[2]: images/session-flow.png
[3]: https://en.wikipedia.org/wiki/HTTP_cookie
[4]: https://en.wikipedia.org/wiki/Magic_cookie
[5]: https://tools.ietf.org/html/rfc6265
[6]: https://www.chromestatus.com/feature/4672634709082112
[7]: https://www.owasp.org/index.php/Session_fixation
[8]: https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide
[9]: https://expressjs.com/
[10]: https://www.npmjs.com/package/express-session
[11]: https://www.npmjs.com/package/connect-sqlite3
[12]: https://www.sqlite.org/
[13]: images/browser-set-cookie.png
[14]: images/database-session-entry.png
[15]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Directives
[16]: https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_Checklist

