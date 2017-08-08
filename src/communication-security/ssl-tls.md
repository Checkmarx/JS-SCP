SSL/TLS
=========

`SSL/TLS` is a cryptographic protocol that allows encryption over otherwise
unsecure communication channels. The most common usage of it is to provide
secure `HTTP` communication, also known as `HTTPS`. The protocol ensures that
the following properties apply to the communication channel:

* Privacy
* Authentication
* Data integrity

In JavaScript, you can use the `tls` package in order to use SSL/TLS protocol.
In this section we will focus on the Node.js implementation and its usage.
Although the theoretical part of the protocol design and it's cryptographic
practices are beyond the scope of this article, additional information is
available on the [Cryptography Practices][1] section of this document.

As a reminder, TLS protocol should be used to protect sensitive information manipulated by your application such as credentials, Personally Indetifiable Information (PII) and credit card numbers.
In a general, it is strongly recommended to use only HTTPS and not mix HTTP and HTTPS content.

## TLS or HTTPS

In Node.js, you have 2 distinct packages to create a server using `SSL/TLS`: 
* [`tls` ] [4]
* [`https`] [5]

In fact, these 2 packages are very similar. The only difference is when using `https`, you are specifically using HTTP over TLS instead of manipulating only the TLS layer.
This means in fact when you are using the `https` module, you are also using module the `tls` module.

The following is a simple example of an HTTP server using the `tls` module (but it will work the same for `https` mdoule):

```javascript

const tls = require('tls');
const fs = require('fs');

const options = {
  // yourKey.pem -  path to your server private key in PEM format
  key: fs.readFileSync('yourKey.pem'),
  // yourCert.pem - path to your server certificate in PEM format
  cert: fs.readFileSync('yourCert.pem'),
};

const server = tls.createServer(options, function(res) => {
  console.log('server connected');
  res.write('Hello JS-SCP!\n');
  res.setEncoding('utf8');
  res.pipe(res);
});

server.listen(8000, () => {
  console.log('server bound');
});
```

This is a simple out-of-the-box implementation of SSL/TLS in a webserver using Node.js.

## SSL/TLS ciphers

It is important to underline that Node.js disable, by default, all weak and medium ciphers. 
Quoting the Node.js documentation about the [Default TLS Cipher Suite] [2]:
> The default cipher suite included within Node.js has been carefully selected to reflect current security best practices and risk mitigation. 
> Changing the default cipher suite can have a significant impact on the security of an application. 

If it is really needed to modify the default configuration, it is recommended to follow the best practices given by Mozilla on the [Server Side TLS Guide] [3].

## SSL/TLS Protocols
In order to protect against some well known attacks such as POODLE (CVE-2014-3566) and BEAST (CVE-2011-3389), it is recommended to disable some protocols on the server side.
By default, the `tls` and `https` modules negotiate a protocol from the highest level down to whatever the client supports. Nowadays, current browsers are not allowing SSLv2.
However, it is important to disable SSLv3 to protect against POODLE and TLSv1.0 to protect againt BEAST.

It is possible to use the `secureOptions` parameter on the `createServer` function from `tls` and `https` to specify the SSL/TLS protocols allowed.

Here is an example allowing to disable SSLv3:
```javascript

const tls = require('tls');

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

  cert: fs.readFileSync('yourCert.pem')),
  key: fs.readFileSync('yourKey.pem')),
}, function (req, res) {
  res.end('works');
}).listen(443);

server.listen(8000, () => {
  console.log('server bound');
});
```

## HTTP Strict Transport Security header (HSTS)

To further improve the communication security, the following flag could be added
to the header, in order to enforce HSTS (HTTP Strict Transport Security):
```javascript
res.SetHeader("Strict-Transport-Security", "max-age=63072000; includeSubDomains")
```

## TLS certificate: client-side validation

Invalid TLS certificates should always be rejected.
Make sure that the `NODE_TLS_REJECT_UNAUTHORIZED` environment variable is not set
to `0` in a production environment.  


## UTF-8 encoding

All requests should also be encoded to a pre-determined character encoding such as UTF-8. This can be set in the header:
```javascript
res.SetHeader("Content-Type", "Desired Content Type; charset=utf-8")
```

## HTTP referer

Another important aspect when handling HTTP connections is to verify that the HTTP referer does not contain any sensitive information when accessing external sites. Since the connection could be insecure, the HTTP referer may leak information.


[1]: ../cryptography-practices/README.md
[2]: https://nodejs.org/api/tls.html#tls_modifying_the_default_tls_cipher_suite
[3]: https://wiki.mozilla.org/Security/Server_Side_TLS
[4]: https://nodejs.org/api/tls.html
[5]: https://nodejs.org/api/https.html