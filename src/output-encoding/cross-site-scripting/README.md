Cross-Site Scripting
====================

Cross-Site Scripting (XSS) vulnerabilities are one of the most prevalent 
attacks involving web applications and JavaScript, ranking as the number 1 
in the [2017's OWASP Top 10][1].
The attack consists of executing malicious JavaScript code in the browser's 
context of a user.

For years, XSS attacks were grouped in three differente categories `Reflected`, 
`Persistent` and `DOM` based - which led people to think of them as three
different types. In fact, we can have both `Stored` and `Reflected DOM` based
XSS. To solve this misunderstanding, there are two new terms which are generally
accepted:

* **Server XSS** - when untrusted data is included in an HTML response generated
  by the server
* **Client XSS** - when untrusted user supplied data is used to update the DOM
  with an unsafe JavaScript call

In this article, we will explore the various types of XSS, as well as provide
vulnerable and secure code for each case.

The purpose of this section is to provide some understanding of XSS attacks and
to demonstrate how bad practices can lead to security issues.

If you're only looking for [how to prevent XSS, you can jump directly to the
appropriate section][2].

[1]: https://www.owasp.org/index.php/Top_10_2017-Top_10
[2]: ./how-to-prevent.md
