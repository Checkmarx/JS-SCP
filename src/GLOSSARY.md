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

## MitM

The Man-in-the-Middle attack intercepts a communication between two systems.
Using different techniques, the attacker splits the original TCP connection
into 2 new connections, one between the client and the attacker and the other
between the attacker and the server. Once the TCP connection is intercepted,
the attacker acts as a proxy, being able to read, insert and modify the data in
the intercepted communication. ([source][2])

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
in TLS 1.0. ([source][3])

[1]: http://www.regular-expressions.info/
[2]: https://www.owasp.org/index.php/Man-in-the-middle_attack
[3]: https://en.wikipedia.org/wiki/Transport_Layer_Security#BEAST_attack
