Access Control
==============

When dealing with access controls the first step to take is to use only trusted
system objects for access authorization decisions.

The component used for access authorization should be a single one, used
site-wide. This includes libraries that call external authorization services.

In case of failure, access control should fail securely. 
More details in the [Error Logging section][1].

If the application cannot access its configuration information, all access to
the application should be denied.

Authorization controls should be enforced on every request, including
server-side scripts as well as requests from client-side technologies like AJAX
or Flash.

It is also important to properly separate privileged logic from the rest of the
application code.

Other important operation where access controls must be enforced in order to
prevent unauthorized user from accessing them, are:

* File and other resources.
* Protected URL's
* Protected functions
* Direct object references
* Services
* Application data
* User and data attributes and policy information.

Here is a small example of how to restrict some application routes to
authenticated users, using [Express Node.js web application framework][3]

```javascript
function isAuthenticated (req, res, next) => {
  if (req.session && req.session.auth === true) {
    return next();
  }

  res.redirect(302, '/account/signin');
}

// ...

req.get('/', isAuthenticated, (req, res, next) => {
    res.end();
});

```

In order to help to enforce a role and attribute based access control for
Node.js, it is possible to use the [accesscontrol package][2]. The following
code, from the [accesscontrol package][2] shows how to implement different roles
and permissions:

```javascript
var ac = new AccessControl();

ac.grant('user')           // define new or modify existing role
    .createOwn('video')
    .deleteOwn('video')
    .readAny('video')
  .grant('admin')          // switch to another role without breaking the chain
    .extend('user')        // inherit role capabilities. also takes an array
    .updateAny('video', ['title'])  
    .deleteAny('video');

// check permissions  
router.get('/videos/:title', function (req, res, next) {
    var permission = ac.can(req.user.role).readAny('video');
    if (permission.granted) {
        Video.find(req.params.title, function (err, data) {
            if (err || !data) return res.status(404).end();
            // filter data by permission attributes and send. 
            res.json(permission.filter(data));
        });
    } else {
        // resource is forbidden for this user/role 
        res.status(403).end();
    }
});
```

When implementing these access controls, it's important to verify that the
server-side implementation and the presentation layer representations of access
control rules match.

If _state data_ needs to be stored on the client-side, it's necessary to use
encryption and integrity checking in order to prevent tampering.

Application logic flow must comply with the business rules.

When dealing with transactions, the number of transactions a single user or
device can perform in a given period of time must be above the business
requirements but low enough to prevent a user from performing a DoS type attack.

It is important to note that using only the `referer` HTTP header is
insufficient to validate authorization and should only be used as a supplemental
check.

Regarding long authenticated sessions, the application should periodically
re-evaluate the user's authorization to verify that the user's permissions have
not changed. If permissions have changed, log the user out and force him to
re-authenticate.

Your application should implement account auditing in order to comply with
safety procedures e.g. disable user accounts 30 days after password's expiration
date.

The application must also support the disabling of accounts and termination of
sessions when a user's authorization is revoked. (e.g. Role change, employment
status, etc.).

When supporting external service accounts and accounts that support connections
_from_ or _to_ external systems, these accounts must run on the lowest level of
privilege possible.

[1]: /error-handling-logging/error-handling.md
[2]: https://www.npmjs.com/package/accesscontrol
[3]: https://expressjs.com/
