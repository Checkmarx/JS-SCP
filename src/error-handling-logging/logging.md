Logging
=======

Logging should always be handled by the application and should not rely on
server configuration.

All logging should be implemented by a master routine on a trusted system and
developers should ensure that no sensitive data is included in the logs (e.g.
passwords, session information, system details, etc.) nor is there any
debugging or stack trace information.

Additionally, logging should cover both successful and unsuccessful security
events with an emphasis on important log event data.

Important event data most commonly refers to:

* All input validation failures
* All authentication attempts, especially failures
* All access control failures
* All apparent tampering events, including unexpected changes to state data
* All attempts to connect with invalid or expired session tokens
* All system exceptions
* All administrative functions, including changes to security configuration
  settings
* All backend TLS connection failures and cryptographic module failures

Below is a brief source code sample using the popular [winston package][4].

```javascript
const winston = require('winston');
winston.add(winston.transports.File, {filename: 'myLog.log'});

// Check user level
switch (RoleLevel) {
  case 1:
    // Login successfull - Log
    winston.log('info', 'User logged in!');

    // Using winston.info()
    winston.info('User logged in!')
    break;
  case 2:
    // Login unsuccessful - Log
    winston.warn("Unsuccessful login.")
    break;
  default:
    // Unspecified error
    winston.error("Login error.")
  }
```

Another important aspect of logging is log rotation. This can be added to the
above sample, using the [winston-daily-rotate-file package][5].

As alternative packages for logging you may want to have a look at

* [log4js][1]
* [bunyan][2]

In case of HTTP request logging, [morgan][3] is the most popular package.
Its usage is very simple, as demonstrated below using Express Node.js framework

```javascript
const express = require('express');
const morgan = require('morgan');

const app = express();

app.use(morgan('combined'));

app.get('/', (req, res) => {
  res.send('hello, world!');
});
```

To access morgan's documentation, please [visit its page on npm][3].

From the log access perspective, only authorized individuals should have access
to the logs.

Developers should also make sure that a mechanism which allows log
analysis is set in place, also to guarantee that no untrusted data will
be executed as code in the intended log viewing software or interface.

One of the most popular open source tools for this is the ELK stack:
Elasticsearch, Logstash and Kibana. [More information regarding this can be
found here][6].

As a final step to guarantee log validity and integrity, a cryptographic hash
function should be used as an additional step to ensure no log tampering has
taken place.

The following example shows a working example of log file signing and
validating, using `SHA256` and the `crypto` package.

There are three parts to this algorithm:

* Computing the digest
* Signing the digest
* Verifying the signature

In the first part of our program, we'll make use of the `crypto` module to
compute the digest:

```javascript
const crypto = require('crypto');
const fs = requre('fs');
const path = require('path');

const hasher = crypto.createHash('sha256');

const pathname = path.resolve(__dirname, 'logs', 'logfile.log');
const rs = fs.createReadStream(pathname);

rs.on('data', data => hasher.update(data));

rs.on('end', () => {
  //Part 2 - See below
});
```

In the second part, we'll make use of the `crypto` module to compute the digest
and sign.

```javascript
const digest = hasher.digest('hex');
const privKey = fs.readFileSync('private_key.pem');
const signer = crypto.createSign('RSA-SHA256');

signer.update(digest);

const signature = signer.sign(privKey, 'base64');
```

And finally, when we need to verify our signature, we can decrypt the signature
and compare the result to the digest of the file as shown below:

```javascript
const pubKey = fs.readFileSync('public_key.pem');
const verifier = crypto.createVerify('RSA-SHA256');
const testSig = verifier.verify(pubKey, signature, 'base64');
const verified = testSig === digest;
```

[1]: https://www.npmjs.com/package/log4js
[2]: https://www.npmjs.com/package/bunyan
[3]: https://www.npmjs.com/package/morgan
[4]: https://www.npmjs.com/package/winston
[5]: https://www.npmjs.com/package/winston-daily-rotate-file
[6]: https://www.elastic.co/webinars/introduction-elk-stack
