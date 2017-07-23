Database Security
=================

This section on OWASP SCP will cover all of the database security issues and
actions developers and DBAs need to take when using databases in their web
applications.

# The best practise
Before implementing your database in JavaScript, you should take care of some
configurations that we'll cover next:
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
database, be sure to take a look into the [Input Validation][4] and
[Output Encoding][5] sections of this guide.