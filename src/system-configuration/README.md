System Configuration
====================

## Updates

Keeping things updated is key in security. So, with that in mind, developers
should keep the JavaScript frameworks updated to the latest versions as well as
external packages and frameworks used by the web application.

In order to help developers keep their packages updated, some tools such as 
[Node Security Platform][1] (`nsp`) will help identify known vulnerabilities.

## Directory Listings

If a developer forgets to disable directory listings (or as OWASP calls it,
[Directory Indexing][4]), an attacker could check for sensitive files navigating
through directories.

By default, in Node.js or Express framework, there is no directory listing
enabled.

Using Node.js, you need to use the `fs` module to simulate a directory listing.
However, for the Express framework, there is a module called `serve-index`.

It is recommended to check your code in order to avoid a directory listing:

* Node.js - check for the use of `fs` module
* Express - check for the use of the `serve-index` module

## HTTP Headers

On production environments, remove all unneeded functionalities and files.
Any test code and functions are not needed on the final version (the version
which is ready to go to production), instead should stay on the developer layer,
not in a location that everyone can see - _aka_ public.

HTTP Response Headers should also be checked. Remove the headers which
disclose sensitive information such as:

* OS version
* Web server version
* Framework or programming language version

In addition, it is important to set-up several HTTP headers in order to improve
the security of the web application. There is an OWASP project called
[OWASP Secure Headers][2] which provides details about each HTTP header.

The `helmet` module can be used in order to help developers easily set-up
the HTTP secure headers in a web application. Here is an example which allows
adding the following HTTP headers:

* HTTP Strict Transport Security (HSTS) is a web security policy mechanism 
  which helps to protect websites against protocol downgrade attacks 
  and cookie hijacking. It is strongly recommended to use this header when 
  using HTTPS.
* X-Frame-Options response header improve the protection of web applications 
  against _clickjacking_. In short, it forbids the use of iframes in your web
  application.
* X-XSS-Protection header enables the Cross-Site Scripting filter in your 
  browser.
* X-Content-Type-Options header prevent the browser from interpreting files 
  as something else than declared by the content type in the HTTP headers. 
  This header is mostly used by Internet Explorer.
* Content Security Policy (CSP) header allows to prevent XSS attacks. You have
  [more about CSP in the General Coding Practices section][6] or you can [read
  "Content Security Policy - An Introduction by Scott Helme][3].

```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// add the HSTS header
app.use(helmet.hsts({maxage: 63072000, includeSubDomains: true}));

// add the X-Frame-Options and allowing only iframe from the same origin
app.use(helmet.frameguard({action: 'sameorigin'}));

// add the X-XSS-Protection header
app.use(helmet.xssFilter());

// add the X-Content-Type-Options header
app.use(helmet.noSniff());

// add the CSP header
app.use(helmet.csp({
  directives:{
	defaultSrc: ["'self'", 'https://ajax.checkmarx.com'],
	scriptSrc: ["'self'"],
	styleSrc:  ["'self'"],
	childSrc:  ["'none'"],
	objectSrc: ["'none'"],
	formSrc: ["'none'"]
  }
}));
```

## Implement Better Security

Put your mindset-hat on and follow the [least privilege principle][4] on the web
server, processes and service accounts.

Take care of your web application error handling. When exceptions occur, be sure
to fail securely. You can check the [Error Handling and Logging][5] section in
this guide for more information regarding this topic.

Prevent disclosure of the directory structure on your `robots.txt` file.
`robots.txt` is a direction file and __NOT__ a security control.
Adopt a white-list approach:

```
User-agent: *
Allow: /sitemap.xml
Allow: /index
Allow: /contact
Allow: /aboutus
Disallow: /
```

The example above will allow any user-agent or bot to index those specific
pages and disallow the rest. This way, you don't disclose sensitive folders or
pages such as admin paths or other important data.

Isolate the development environment from the production network. Provide the
right access to developers and test groups. Better yet, create additional
security layers to protect them. In most cases, development environments are
easier targets for attacks.

Last but not least, have a software change control system to manage and record
changes in your web application code (development and production environments).
There are numerous Github host-yourself clones that can be used for this
purpose.

[1]: https://github.com/nodesecurity/nsp
[2]: https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#tab=Headers
[3]: https://scotthelme.co.uk/content-security-policy-an-introduction/
[4]: https://www.owasp.org/index.php/Least_privilege
[5]: ../error-handling-logging/README.md
[6]: ../general-coding-practices/content-security-policy.md
