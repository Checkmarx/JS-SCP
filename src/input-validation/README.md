Input Validation
================

In web application security, user input and the associated data are a security
risk if left unchecked.
We address these problems by using Input Validation and Input Sanitization
techniques. These validations should be performed in every tier of the
application, as per the server's function. An important note is that **all data
validation procedures must be done on trusted systems** (i.e. on the server).

As mentioned in the [OWASP SCP Quick Reference Guide][1], there are sixteen
bullet points that cover the issues developers should be aware of when dealing
with Input Validation. A lack of consideration for these security risks when
developing an application is one of the main reasons [Injections][2] rank as the
number one vulnerability in the [OWASP Top 10][3] list.

User interaction is a staple of the current development paradigm in web
applications. As web applications become increasingly richer in content and
possibilities, user interaction and submitted user data also increases. It is in
this context that Input Validation plays a significant role.

When applications handle user data, submitted data **must be considered insecure
by default**, and should only be accepted after the appropriate security checks
have been made. Data sources must also be identified as trusted or untrusted. In
the case of an untrusted source, validation checks must be made. In this
section, we provide an overview of each technique along with a JavaScript sample
to illustrate the issue.

[1]: https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf
[2]: https://www.owasp.org/index.php/Top_10_2013-A1-Injection
[3]: https://www.owasp.org/index.php/Top_10_2013-Top_10
