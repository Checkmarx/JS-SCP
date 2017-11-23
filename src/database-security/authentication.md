Database Authentication
=======================

## Access the Database With Minimal Privilege

If your JavaScript web application only needs to read data and doesn't need to
write information, create a database user whose permissions are `read-only`.

Always adjust the database user according to your web application's needs.

## Use A Strong Password

When creating your database access, choose a strong password. [OWASP Guidelines
for enforcing secure passwords][2] defines what a strong password is.
You can use the [npm OWASP Password Strength Test package][3] to test your
password according these rules.

Some password managers generate strong passwords in addition to some online web
applications. Use them at your own risk.

## Remove Default Admin Passwords

Most Database Management Systems have default accounts, most of them with no
password set for the highest privilege user accounts (e.g. MariaDB and MongoDB
use `root` with no password) which means an attacker can gain access to
everything.

This should be fixed by setting a password, or better yet by creating a new
account with a non-default username with the same access rights.

Also, don't forget to remove your credentials and/or private key(s) if you're
going to post your code on a publicly accessible repository in GitHub.

[1]: https://strongpasswordgenerator.com/
[2]: https://www.owasp.org/index.php/Authentication_Cheat_Sheet#Implement_Proper_Password_Strength_Controls
[3]: https://github.com/nowsecure/owasp-password-strength-test
