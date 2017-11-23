Other Guidelines
================

Authentication is a critical part of any system, therefore you should always
employ correct and safe practices. Below are some guidelines to make your
authentication system more resilient:

* "_Re-authenticate users prior to performing critical operations_"

* "_Use Multi-Factor Authentication for highly sensitive or high value
  transactional accounts_". In Node.js there are packages that allow the easy
  implementation of two factor authentication. One of the most popular is called
  `speakeasy`.

  The following usage example is taken from the package's documentation:
  ```javascript
  const speakeasy = require('speakeasy')

  // ...

  // token generation
  const secret = speakeasy.generateSecret()

  // the token the user entered - for demo purposes
  const userToken = "123123"

  // save the generated secret in base32 encoding.
  const base32secret = secret.base32;

  // use verify() to check the token against the secret
  const verified = speakeasy.totp.verify({
    secret: base32secret,
    encoding: 'base32',
    token: userToken
  });

  if (verified == true) {
    // token matches
  }
  ```

  Additional tokens are available, such as time-based tokens. Other encodings
  are available, including - `ASCII` or `hex`. The complete documentation and
  package information can be found [here][1].

* "_Implement monitoring to identify attacks against multiple user accounts,
  utilizing the same password. This attack pattern is used to bypass standard
  lockouts, when user IDs can be harvested or guessed_". As is characteristic of
  Node.js, there are packages available to allow rate limiting if a brute-force 
  attack pattern is detected. An example of this is the `express-brute` package
  for `express`. It allows request slowdown (after 5 failed logins), as well as
  setting a daily maximum login attempt number (1000).

  A simple example taken from the documentation:

  ```javascript
  const ExpressBrute = require('express-brute');

  // stores state locally, DO NOT use this in production
  const store = new ExpressBrute.MemoryStore();
    
  const bruteforce = new ExpressBrute(store);

  app.post('/auth',
    bruteforce.prevent, // error 429 if we hit this route too often
    (req, res, next) => {
      res.send('Success!');
    }
  );
  ```

  To see all supported features, please see the documentation available
  [here][2]. It's also important to log all of these requests, not only to
  depend on an external package to limit the login request number. For more
  information about logging, please see the [Error Handling and Logging][3]
  section.

* "_Change all vendor-supplied default passwords and user IDs or disable the
  associated accounts_".

* "_Enforce account disabling after an established number of invalid login
  attempts (e.g., five attempts is common).  The account must be disabled for a
  period of time sufficient to discourage brute force guessing of credentials,
  but not so long as to allow for a denial-of-service attack to be performed_"

  This is an additional protective measure against brute-force. If possible, use
  this combined with the `express-brute` (or the package chosen to deal with
  brute-force) to comply with good security practices.

[1]: https://www.npmjs.com/package/speakeasy
[2]: https://www.npmjs.com/package/express-brute
