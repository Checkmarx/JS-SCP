Database Security
=================

This section on [OWASP Secure Coding Practices - Quick Reference guide][5]
covers database security issues and actions developers and database
administrators need to take when using databases in their web applications.

## The best practise

Before using a database with your JavaScript application, you should take care
of some configurations that we'll cover next:

* Secure database server installation[^1]
    * Change/set a password for `root` account(s);
    * Remove the `root` accounts that are accessible from outside the localhost;
    * Remove any anonymous-user accounts;
    * Remove any existing test database;
* Remove any unnecessary stored procedures, utility packages,
  unnecessary services, vendor content (e.g. sample schemas).
* Install the minimum set of features and options required for your database to
  work with JavaScript.
* Disable any default accounts that are not required on your web application to
  connect to the database.

Also, because it's __important__ to validate input and encode output on the
database, be sure to take a look into the [Input Validation][1] and
[Output Encoding][2] sections of this guide.

---

[^1]: MySQL/MariaDB have a program for this: `mysql_secure_installation`<sup>[3],[4]</sup>

[1]: /input-validation/README.md
[2]: /output-encoding/README.md
[3]: https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html
[4]: https://mariadb.com/kb/en/mariadb/mysql_secure_installation/
[5]: https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide
