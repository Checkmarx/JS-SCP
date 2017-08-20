Output Encoding
===============

Although it is a six bullets only section on [OWASP SCP Quick Reference
Guide][1], bad practices on Output Encoding are pretty prevalent on Web
Application development, es of Output Encoding are
pretty prevalent on Web Application development, thus leading to the number 1
vulnerability of the [OWASP Top Ten] [3]: [Injection][2].

As complex and rich as Web Applications become, the more data sources they
have: users, databases, thirty party services, etc. At some point in time
collected data is outputted to some media (e.g. web browser) which has a
specific context.
This is exactly when injections happen if you do not have a strong Output
Encoding policy.

Certainly you have already heard about all the security issues we will approach
in this section, but do you really know how do they happen and/or how to avoid
them?

In this section we will cover

* [Cross-Site Scripting (XSS)](./cross-site-scripting.md)
* [SQL Injection (SQLi)](./sql-injection.md)
* [NoSQL Injection](./nosql-injection.md)

[1]: https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf
[2]: https://www.owasp.org/index.php/Top_10_2013-A1-Injection
[3]: https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project#tab=OWASP_Top_10_for_2013
