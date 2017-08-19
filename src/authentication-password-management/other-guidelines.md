Other guidelines
================

Authentication is a critical part of any system so you should always employ
correct and safe practices. Below are some guidelines to make your
authentication system more resilient:

* "_Re-authenticate users prior to performing critical operations_"

* "_Use Multi-Factor Authentication for highly sensitive or high value
  transactional accounts_". In nodeJS there are packages that allows to easily
  implement two factor authentication. One of the most popular is called
  `speakeasy`.

  The following usage example is taken from the package's documentation:
  ```javascript
  var speakeasy = require('speakeasy')

  [...]

  //Token generation
  var secret = speakeasy.generateSecret()

  //The token the user entered - for demo purposes
  var userToken = "123123"

  //Save the generated secret in base32 encoding.
  var base32secret = secret.base32;

  // Use verify() to check the token against the secret
 var verified = speakeasy.totp.verify({ secret: base32secret,
                                       encoding: 'base32',
                                       token: userToken });

 if (verified == true) {
   //Token matches
 }
  ```
  Additional token are available, such as time-based tokens. Other encodings are available, including - `ASCII` or `hex`. The complete documentation and
  package information can be found [here][1].

* "_Implement monitoring to identify attacks against multiple user accounts,
  utilizing the same password. This attack pattern is used to bypass standard
  lockouts, when user IDs can be harvested or guessed_". As is characteristic of
  nodeJS, there are packages available to allow rate limiting if a
  bruteforce attack pattern is detected. An example of this is the
  `express-brute` package for `express`. It allows request slowdown (after 5
    failed logins), as well as setting a daily maximum login attempt
    number (1000).

    A simple example taken from the documentation:
    ```javascript
    var ExpressBrute = require('express-brute');

    // stores state locally, DO NOT use this in production
    var store = new ExpressBrute.MemoryStore();
    
    var bruteforce = new ExpressBrute(store);

    app.post('/auth',
        bruteforce.prevent, // error 429 if we hit this route too often
        function (req, res, next) {
            res.send('Success!');
        }
    );
    ```
    To see all the features supported please see the documentation available
    [here][2]. It's also important to log all the these requests, not only to
    depend on an external package to limit the login request number. For more
    information about logging see the [Error Handling and Logging][3] section.

* "_Change all vendor-supplied default passwords and user IDs or disable the
  associated accounts_".

* "_Enforce account disabling after an established number of invalid login
  attempts (e.g., five attempts is common).  The account must be disabled for a
  period of time sufficient to discourage brute force guessing of credentials,
  but not so long as to allow for a denial-of-service attack to be performed_"

  This is an additional protection against bruteforce. If possible, use this
  combined with the `express-brute` (or the package chosen to deal with
     bruteforce) to comply with good security practices.

[1]: https://www.npmjs.com/package/speakeasy
[2]: https://www.npmjs.com/package/express-brute
