Logging
=======

Logging should always be handled by the application and should not rely on
server configuration.

All logging should be implemented by a master routine on a trusted system, and
the developers should also ensure no sensitive data is included in the logs
(e.g. passwords, session information, system details, etc.), nor is there
any debugging or stack trace information.
Additionally, logging should cover both successful and unsuccessful security
events, with an emphasis on important log event data.

Important event data most commonly refers to:

* All input validation failures.
* All authentication attempts, especially failures.
* All access control failures.
* All apparent tampering events, including unexpected changes to state data.
* All attempts to connect with invalid or expired session tokens.
* All system exceptions.
* All administrative functions, including changes to security configuration
  settings.
* All backend TLS connection failures and cryptographic module failures.

To demonstrate this, a simple node.js application is presented. In our example
we will be using a third-party logging library called `winston`.  

```javascript
  var winston = require('winston');

  [...]

  winston.add(winston.transports.File, {filename: 'myLog.log'});

  //Check user level
  switch (RoleLevel) {
    case 1:
      //Login successfull - Log
      winston.log('info', 'User logged in!');
      //Using winston.info()
      winston.info('User logged in!')
      break;
    case 2:
      //Login unsuccessful - Log
      winston.warn("Unsuccessful login.")
      break;
    default:
      //Unspecified error
      winston.error("Login error.")
  }

  [...]

```

Our example is using the `winston` package for error logging and saving the
log to the filename `myLog.log`.

Another important aspect of logging is log rotation. To exemplify we will add
log rotation to our previous example by using the package
`winston-daily-rotate-file`.  

Using log rotation:
```javascript
var winston = require('winston');
require('winston-daily-rotate-file');

[...]

var transport = new (winston.transports.DailyRotateFile)({
  filename: './log',
  datePattern: 'yyyy-MM-dd',
  prepend: true,
  level: process.env.ENV === 'development' ? 'debug' : 'info'
})

var logger = new (winston.logger)({
  transports: [
    transport
  ]
})

//Check user level
switch (RoleLevel) {
  case 1:
    //Login successfull - Log
    logger.info('User logged in!');
    break;
  case 2:
    //Login unsuccessful - Log
    logger.warn('Unsuccessful login.')
    break;
  default:
    //Unspecified error
    logger.error('Login error.')
}

[...]
```

When using server-side javascript(`node.js`) it's not recommended to implement
you own logging routines. Instead, the best option is to use a third-party
library like the one used in the previous examples (`winston`).

More information about `winston` can be
found [here](https://github.com/winstonjs/winston).

As an alternative package for logging the following are also very popular among
node.js developers:
* [log4js][1] - https://www.npmjs.com/package/log4js
* [bunyan][2] - https://www.npmjs.com/package/bunyan

Of these libraries one of the most popular is `winston`, hence our usage in the
examples.

In case of HTTP request logging, the most popular package is `morgan`.
It's usage is very simple, as demonstrated below:
Note that this example is built upon `express`.

```javascript
var express = require('express')
var morgan = require('morgan')

var app = express()

app.use(morgan('combined'))

app.get('/', function (req, res) {
  res.send('hello, world!')
})
```

For the full documentation please consult the npm package page [here][3].

From the log access perspective, only authorized individuals should have
access to the logs.

Developers should also make sure that a mechanism that allows for log
analysis is set in place, as well as guarantee that no untrusted data will
be executed as code in the intended log viewing software or interface.

One of the most popular open source mechanisms for this is called `ELK stack`,
which consists of `Elasticsearch`, `Logstash` and `Kibana`. More information
regarding this can be
found [here](https://www.elastic.co/webinars/introduction-elk-stack)

Regarding allocated memory cleanup, Node.js has an built-in Garbage Collector and
the V8 uses two different collection algorithms.
Several resources are available online for more in-depth information regarding
memory cleanup.  
A great example of this is the article written by Gergely Nemeth
[here](https://blog.risingstack.com/node-js-at-scale-node-js-garbage-collection/).

As a final step to guarantee log validity and integrity, a cryptographic
hash function should be used as an additional step to ensure no log
tampering has taken place.

The following example shows a working example of log file signing and validating,
using `SHA256` and the `crypto` package.

//TODO: A MODIFICAR - ATENCAO  
Credits to Fissure King @ stackoverflow.com.

There are three parts to this algorithm.
  - Computing the digest.
  - Signing the digest.
  - Verifying the signature.

In the first part of our program we'll make use of the `crypto` module to compute
the digest:
```javascript
'use strict'

const crypto = require('crypto')
const fs = requre('fs')
const path = require('path')

const hasher = crypto.createHash('sha256')

const pathname = path.resolve(__dirname, 'logs', 'logfile.log')
const rs = fs.createReadStream(pathname)

rs.on('data', data => hasher.update(data))

rs.on('end', () => {
  //Part 2 - See below
})
```

In the second part we'll make use of the `crypto` module to compute the digest
and sign.

```javascript
  [...]
  const digest = hasher.digest('hex')
  const privKey = fs.readFileSync('private_key.pem')
  const signer = crypto.createSign('RSA-SHA256')

  signer.update(digest)

  const signature = signer.sign(privKey, 'base64')
  [...]
```

And finally when we need to verify our signature, we can decrypt the signature
and compare the result to the digest of the file.  
As shown below:
```javascript
  const pubKey = fs.readFileSync('public_key.pem')
  const verifier = crypto.createVerify('RSA-SHA256')
  const testSig = verifier.verify(pubKey, signature, 'base64')
  const verified = testSig === digest
```


[1]:http://
[2]:http://
[3]:https://www.npmjs.com/package/morgan
