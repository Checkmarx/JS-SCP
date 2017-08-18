Communication Security
======================

When approaching communication security, developers should be certain that the
channels used for communication are secure.

Types of communication include server-client, server-database, as well as all
backend communications. These must be encrypted to guarantee data integrity and
confidentiality.
Failure to secure these channels allows known attacks like Man-in-the-Middle
(MitM) attacks, which let's criminals intercept and read the traffic in these
channels.

The scope of this section covers the following communication channels:

* [SSL/TLS][2]

[1]: https://www.owasp.org/index.php/Man-in-the-middle_attack
[2]: ./ssl-tls.md
