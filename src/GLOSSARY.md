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

A regular expression (regex or regexp for short) is a special text string for describing a search pattern. ([source][1])

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

Subresource Integrity (SRI) is a security feature that enables browsers to verify that files they fetch (for example, from a CDN) are delivered without unexpected manipulation. It works by allowing you to provide a cryptographic hash that a fetched file must match. ([source][sri])

## CSP 

Content Security Policy is a computer security standard introduced to prevent
cross-site scripting (XSS), clickjacking and other code injection attacks
resulting from execution of malicious content in the trusted web page context.
([source][csp])

## Sandbox

Sandbox is a security mechanism for separating running programs, usually in an
effort to mitigate system failures or software vulnerabilities from spreading.
It is often used to execute untested or untrusted programs or code, possibly
from unverified or untrusted third parties, suppliers, users or websites,
without risking harm to the host machine or operating system.
([source][sandbox])

[1]: http://www.regular-expressions.info/
[cdn]: https://en.wikipedia.org/wiki/Content_delivery_network
[sri]: https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity
[csp]: https://en.wikipedia.org/wiki/Content_Security_Policy
[sandbox]: https://en.wikipedia.org/wiki/Sandbox_(computer_security)