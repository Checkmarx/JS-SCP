SSL/TLS
=======

`SSL/TLS` are two cryptographic protocols (TLS is SSL's successor) that allow
encryption over otherwise unsecure communication channels. The most common
usage of `SSL/TLS` is for sure `HTTPS`: Hyper Text Transfer Protocol Secure.

These cryptographic protocols ensure that the following properties apply to the
communication channel:

* Privacy
* Authentication
* Data Integrity

SSL/TLS protocols are available through Node.js' [tls module][4].

In this section, we will focus on the Node.js implementation and its usage.
Although the theoretical part of the protocol design and its cryptographic
practices are beyond the scope of this article, additional information is
available in the [Cryptography Practices][1] section of this document.

As a reminder, TLS protocol should be used to protect sensitive information
manipulated by your application such as credentials, Personally Identifiable
Information (PII) and credit card numbers.

In general, it is strongly recommended to use only HTTPS instead of switching
back and forth between HTTP and HTTPS.

## TLS or HTTPS

In Node.js, you have two distinct modules to create a server using `SSL/TLS`:

* [`tls`][4]
* [`https`][5]

In fact, [https module][5] literally implements "HTTP over TLS" as it depends
on both the `http` and `tls` modules to provide an HTTP server capable of
handling secure connections[^1].

The following is a simple example of an HTTP server using the `tls` module
capable of handling secure connections

```javascript
const tls = require('tls');
const fs = require('fs');

const options = {
  key: fs.readFileSync('/my/certs/safe/location/key.pem'),
  cert: fs.readFileSync('/my/certs/safe/location/cert.pem'),
};

const server = tls.createServer(options, function(res) => {
  console.log('server connected');
  res.write('Hello JS-SCP!\n');
  res.setEncoding('utf8');
  res.pipe(res);
});

server.listen(443, () => {
  console.log('server bound');
});
```

If you're looking to serve your application over HTTPS you're better to go with
the [https module][5]. Bellow, you will find a sample using the [Express -
Node.js web application framework][9].

```javascript
const express = require('express');
const https = require('https');
const fs = require('fs');

const app = express();
const certOptions = {
  key: fs.readFileSync('/my/certs/safe/location/key.pem'),
  cert: fs.readFileSync('/my/certs/safe/location/cert.pem')
};

app.get('/', function (req, res) {
  res.send('Hello World!')
});

https.createServer(certOptions, app)
    .listen(443);
```

## SSL/TLS ciphers

It is important to highlight that Node.js by default accepts only a subset of
strong ciphers.

Quoting the Node.js documentation about the [Default TLS Cipher Suite][2]:

> The default cipher suite included within Node.js has been carefully selected
> to reflect current security best practices and risk mitigation.
> Changing the default cipher suite can have a significant impact on the
> security of an application.

If, for a very strong reason, the default configuration has to be changed, it is
recommended to follow the [best practices given by Mozilla on its Server Side
TLS Guide][3].

## SSL/TLS Protocols

In order to protect against some well known attacks such as POODLE
([CVE-2014-3566][10]) and BEAST ([CVE-2011-3389][11]), it is recommended to
disable some protocols on the server side.

By default, the `tls` and `https` modules negotiate a protocol from the highest
level down to whatever the client supports. Nowadays, current browsers are not
allowing SSLv2, however, it is important to disable SSLv3 to protect against
POODLE and TLSv1.0 to protect against BEAST.

The `secureOptions` parameter on the `createServer` function from `tls` and
`https` modules allows to specify the allowed SSL/TLS protocols as demonstrated
below

```javascript
const https = require('https');

https.createServer({
  //
  // This is the default secureProtocol used by Node.js, but it might be
  // sane to specify this by default as it's required if you want to
  // remove supported protocols from the list. This protocol supports:
  //
  // - SSLv2, SSLv3, TLSv1, TLSv1.1 and TLSv1.2
  //
  secureProtocol: 'SSLv23_method',

  //
  // Supply `SSL_OP_NO_SSLv3` constant as secureOption to disable SSLv3
  // from the list of supported protocols that SSLv23_method supports.
  //
  secureOptions: constants.SSL_OP_NO_SSLv3,

  cert: fs.readFileSync('/my/certs/safe/location/cert.pem')),
  key: fs.readFileSync('/my/certs/safe/location/key.pem')),
}, function (req, res) {
  res.end('works');
}).listen(443);
```

## HTTP Strict Transport Security header

To further improve the communication security, the Strict Transport Security
HTTP header (HSTS) can be added as follows

```javascript
res.SetHeader("Strict-Transport-Security", "max-age=63072000; includeSubDomains")
```

The [helmet package][12] can be also used to add not only the
`Strict-Transport-Security` header but also other security HTTP headers

```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();
app.use(helmet.hsts({maxage: 63072000, includeSubDomains: true}));
```

## TLS certificate: client-side validation

Invalid TLS certificates should always be rejected.
Make sure that the `NODE_TLS_REJECT_UNAUTHORIZED` environment variable **is
not** set to `0` in a production environment.

## TLS compression

The [CRIME][6] attack exploits a flaw with data compression.

By default, Node.js disables all compression.

## UTF-8 encoding

All requests should also be encoded to a pre-determined character encoding such
as UTF-8. This can be set in the header:

```javascript
res.SetHeader("Content-Type", "Desired Content Type; charset=utf-8")
```

## HTTP referer

Another important aspect when handling HTTP connections is to verify that the
HTTP referer does not contain any sensitive information when accessing external
sites. Since the connection could be insecure, the HTTP referer may leak
information.

[^1]: Actualy you can see it on source code: lines 26 and 28 of [https module source code][8].

[1]: ../cryptography-practices/README.md
[2]: https://nodejs.org/api/tls.html#tls_modifying_the_default_tls_cipher_suite
[3]: https://wiki.mozilla.org/Security/Server_Side_TLS
[4]: https://nodejs.org/api/tls.html
[5]: https://nodejs.org/api/https.html
[6]: https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/2012/september/details-on-the-crime-attack/
[7]: https://nodejs.org/dist/latest-v6.x/docs/api/http.html
[8]: https://github.com/nodejs/node/blob/467385a49b659b704973b8195328775b620ae6ab/lib/https.js
[9]: https://expressjs.com/
[10]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
[11]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2011-3389
[12]: https://www.npmjs.com/package/helmet
