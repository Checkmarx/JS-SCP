## Hazardous Characters

Any character or encoded representation of a character that can effect the
intended operation of the application or associated system by being interpreted
to have a special meaning, outside the intended use of the character. These
characters may be used to:

* Altering the structure of existing code or statements
* Inserting new unintended code
* Altering paths
* Causing unexpected outcomes from program functions or routines
* Causing error conditions
* Having any of the above effects on down stream applications or systems

## Regular Expression

A regular expression (regex or regexp for short) is a special text string for
describing a search pattern. ([source][regex])

## ReDoS

Regular expression Denial of Service (ReDoS) is a Denial of Service attack,
that exploits the fact that most Regular Expression implementations may reach
extreme situations that cause them to work very slowly (exponentially related
to input size).

## dot-dot-slash

Path alteration characters like `../` and `..\`

## Sanitization

Sanitization refers to the process of removing or replacing submitted data to
ensure that it is valid and safe.

## Canonicalize

To reduce various encodings and representations of data to a single simple form.

## CDN

Content Delivery Network or Content Distribution Network is a geographically
distributed network of proxy servers and their data centers. The goal is to
distribute service spatially relative to end-users to provide high availability
and high performance. ([source][cdn])

## SRI

Subresource Integrity (SRI) is a security feature that enables browsers to
verify that files they fetch (for example, from a CDN) are delivered without
unexpected manipulation. It works by allowing you to provide a cryptographic
hash that a fetched file must match. ([source][sri])

## CSP 

Content Security Policy is a computer security standard introduced to prevent
cross-site scripting (XSS), _clickjacking_ and other code injection attacks
resulting from execution of malicious content in the trusted web page context.
([source][csp])

## Sandbox

Sandbox is a security mechanism for separating running programs, usually in an
effort to mitigate system failures or software vulnerabilities from spreading.
It is often used to execute untested or untrusted programs or code, possibly
from unverified or untrusted third parties, suppliers, users or websites,
without risking harm to the host machine or operating system.
([source][sandbox])

## XSS 

Cross-Site Scripting is an attack that consists in injecting malicious
JavaScript code in the website.

## DOM

Document Object Model (DOM) is a cross-platform and language-independent
application programming interface that treats an HTML, XHTML, or XML document as
a tree structure wherein each node is an object representing a part of the
document. ([source][dom])

## URI

Unified Resource Identifier

## JSON

JavaScript Object Notation is a lightweight data-interchange format. It is
easy for humans to read and write. It is easy for machines to parse and
generate. It is based on a subset of the JavaScript Programming Language,
Standard ECMA-262 3rd Edition - December 1999. JSON is a text format that is
completely language independent but uses conventions that are familiar to
programmers of the C-family of languages, including C, C++, C#, Java,
JavaScript, Perl, Python, and many others. These properties make JSON an ideal
data-interchange language. ([source][json])

## CSRF

Cross-Site Request Forgery (CSRF) "_is an attack that forces an end user to
execute unwanted actions on a web application in which they're currently
authenticated._" ([source][csrf]).

## MitM

The Man-in-the-Middle attack intercepts a communication between two systems.
Using different techniques, the attacker splits the original TCP connection
into 2 new connections, one between the client and the attacker and the other
between the attacker and the server. Once the TCP connection is intercepted,
the attacker acts as a proxy, being able to read, insert and modify the data in
the intercepted communication. ([source][mitm])

## TLS

Transport Layer Security is a cryptographic protocol that provides
communication security over a computer network.

[SSL](#ssl) is the TLS predecessor. 

## SSL

Secure Sockets Layer is a cryptographic protocol that provides communication
security over a computer network.

[TLS](#tls) is the successor of SSL.

## POODLE

Padding Oracle On Downgraded Legacy Encryption is a Man-in-the-Middle (MitM)
exploit which takes advantage of internet and security software clients'
fallback to SSL 3.0, allowing an attacker to reveal unencrypted messages (on
average after 256 requests, 1 byte of unencrypted messages will be revealed).

## BEAST

Browser Exploit Against SSL/TLS is a proof-of-concept demonstrated by Thai
Duong and Juliano Rizzo back in 2011, which violates browsers'  same origin
policy constraints, for a long-known cipher block chaining (CBC) vulnerability
in TLS 1.0. ([source][beast])

[regex]: http://www.regular-expressions.info/
[csrf]: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_
[mitm]: https://www.owasp.org/index.php/Man-in-the-middle_attack
[beast]: https://en.wikipedia.org/wiki/Transport_Layer_Security#BEAST_attack
[dom]: https://en.wikipedia.org/wiki/Document_Object_Model
[json]: http://www.json.org/
[cdn]: https://en.wikipedia.org/wiki/Content_delivery_network
[sri]: https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity
[csp]: https://en.wikipedia.org/wiki/Content_Security_Policy
[sandbox]: https://en.wikipedia.org/wiki/Sandbox_(computer_security)
