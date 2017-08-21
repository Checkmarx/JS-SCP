Communicating authentication data
=================================

In this section, "communication" is used in a broader sense, encompassing
User Experience (UX) and client-server communication.

Not only is it true that "_password entry should be obscured on user's screen_"
but also the "_remember me functionality should be disabled_".

You can accomplish both using an input field with `type="password"`, and
setting the `autocomplete` attribute to `off`[^1]

```html
<input type="password" name="passwd" autocomplete="off" />
```

Authentication credentials should be sent on `HTTP POST` requests only, using an
encrypted connection (HTTPS). An exception to the encrypted connection may be
the temporary passwords associated with email resets.

Although `HTTP GET` requests over TLS/SSL (HTTPS) look as secure as `HTTP POST`
requests, remember that in general HTTP servers (eg. Apache[^2], Nginx[^3]) do
write the requested URL to the access log.

```text
xxx.xxx.xxx.xxx - - [27/Feb/2017:01:55:09 +0000] "GET /?username=user&password=70pS3cure/oassw0rd HTTP/1.1" 200 235 "-" "Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:51.0) Gecko/20100101 Firefox/51.0"
```

A well designed HTML form for authentication would look like:

```html
<form method="post" action="https://somedomain.com/user/signin" autocomplete="off">
    <input type="hidden" name="csrf" value="CSRF-TOKEN" />

    <label>Username <input type="text" name="username" /></label>
    <label>Password <input type="password" name="password" /></label>

    <input type="submit" value="Submit" />
</form>
```

When handling authentication errors, your application should not disclose which
part of the authentication data was incorrect. Instead of "Invalid username" or
"Invalid password", just use "Invalid username and/or password" interchangeably:

```html
<form method="post" action="https://somedomain.com/user/signin" autocomplete="off">
    <input type="hidden" name="csrf" value="CSRF-TOKEN" />

    <div class="error">
        <p>Invalid username and/or password</p>
    </div>

    <label>Username <input type="text" name="username" /></label>
    <label>Password <input type="password" name="password" /></label>

    <input type="submit" value="Submit" />
</form>
```

With a generic message you do not disclose:

* Who is registered: "Invalid password" means that the username exists.
* How your system works: "Invalid password" reveals how your application works

As a rule of thumb, avoid implementing your own authentication controls.
Instead, use services that comply with standards and have been properly tested.

A very used package in Node.js that deals with authentication is `passport`
`passport` supports over 300 authentication _strategies_. These _strategies_
include third party services like `passport-facebook`, `passport-oauth`,
`passport-twitter`, etc. As well as local _strategies_ such
as `passport-localapikey` or `passport-hash`.

In the following example taken from the documentation, we are using
`passport-hash` as the strategy for authentication.

```javascript
const passport = require('passport')

[...]

// Strategies in passport require a `verify` function, which accept
// hash , and invoke a callback with a user object. In the real world,
// this would query a database; however, in this example we are using
// a baked-in set of users.
passport.use(new HashStrategy(
 (hash, done) => {
    // asynchronous verification, for effect...
    process.nextTick(() => {

      // Find the user by hash.  If there is no user with the given
      // hash, or the status is not unconfirmed, set the user to `false` to
      // indicate failure and set a flash message.  Otherwise, return the
      // authenticated `user`.
      findByUserHash(hash, (err, user) => {
        if (err) {
          return done(err);
        }

        if (!user) {
          return done(null, false, { message: 'Unknown user ' + username });
        }

        if (user.status != 'unconfirmed') {
          return done(null, false,
            { message: 'This user already confirmed' });
        }
        return done(null, user);
      })
    });
  }
));

[...]

// Use passport.authenticate() as route middleware to authenticate the
// request.  If authentication fails, the user will be redirected back to the
// login page.  Otherwise, the primary route function function will be called,
// which, in this example, will redirect the user to the home page.
app.get('/confirm/:hash', passport.authenticate('hash', {
  failureRedirect: 'login', failureFlash: true
}), (req, res) => {
  res.redirect('/account');
});
```

You can search available Passport strategies at [passportjs.org website][5].

After a successful login, the user should be informed about the last successful
or unsuccessful access date/time so that he can detect and report suspicious
activity. Further information regarding logging can be found in the
[Error Handling and Logging][4] section of the document.

---

[^1]: [How to Turn Off Form Autocompletion][1], Mozilla Developer Network
[^2]: [Log Files][2], Apache Documentation
[^3]: [log_format][3], Nginx log_module "log_format" directive

[1]: https://developer.mozilla.org/en-US/docs/Web/Security/Securing_your_site/Turning_off_form_autocompletion
[2]: https://httpd.apache.org/docs/1.3/logs.html#accesslog
[3]: http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format
[4]: ../error-handling-logging/logging.md
[5]: http://passportjs.org/
