Strings
=======

The user input data transmitted over HTTP reaches the server and it is handled
as `String` thus the protocol itself is textual[^1].
Of course that internally applications may expect to handle other data types
such as `Number`s or `Boolean` values requiring data type conversion (what
should happen after validation).

Even when a `String` is expected (e.g user's name or address) it should be
handled carefully as characters encoding may compromise applications security.

The validation routine requirements and data sources classification were
discussed on the [Validation][2] and [Data Sources][3] sections, respectively.

As already said but to enforce it **all data coming from untrusted data source
such as parameters, URLs and HTTP header content, should be subject of
validation prior to processing** and **validation failures should result in
input rejection**.

This section covers `String` validation and manipulation in general. How to
convert `String` to `Number` is covered on [Numbers section][4].

**Important**: you should always avoid to right your own validation routine. A
a well tested and actively maintained one like [Validator.js][5] should be used
instead.

## Encode data to a common character set

This is known as [Canonicalization][6] and it aims to convert all data to a
standard or canonical form so that the system can handle data with different
representations using the same routine.

How `String`s are handled internally is well documented on the [ECMAScript
<sup>&reg;</sup> 2015 Language Specification][7]

> A String value is a member of the String type. Each integer value in the
> sequence usually represents a single 16-bit unit of UTF-16 text. However,
> ECMAScript does not place any restrictions or requirements on the values
> except that they must be 16-bit unsigned integers.

Prior to the introduction of [TypedArray][8] in ECMAScript 2015 (ES6) Node.js
introduced the [Buffer class][9], allowing you to handle and convert strings
encoding easily

```javascript
'use strict';

// read user input into a Buffer
const userInputName = Buffer.from(req.body.name);

// actually conver user input to UTF-8
const userName = userInputName.toString('utf8');

console.log('Hello %n', userName);
```

If you want to do the same on client-side you can use the [Buffer module][10]
which is backed by Typed Arrays. In fact this is the modules used by
[Browserify][11][^2] but you're not using it, you can `require` the modules
directly

```javascript
'use strict';

// Note the trailling slash
var Buffer = require('buffer/').Buffer;

// read user input into a Buffer
const userInputName = Buffer.from(req.body.name);

// actually convert user input to UTF-8
const userName = userInputName.toString('utf8');

console.log('Hello %s', userName);
```

## Validate all input against a "white" list of allowed characters

Whenever possible this is an important first step from security standpoint as
it will contribute to prevent Injections[^3] nevertheless **if any potential
hazardous characters must be allowed as input you should implement additional
controls like [Output Encoding][13]**.

Regarding the _"white" list of allowed characters_, you may feel tempted to do
it using Regular Expressions but you should be aware that they are a real
source of problems. Not only they are hard to write but also they are subject
of a denial of service attack called Regular expression Denial of Service
(ReDoS)[^4].

Below a simple example of a evil Regex to validate decimal numbers and the
correspondent payload (you can [test it on regex101.com][16]).

* **Regex:** `^\d*[0-9](|.\d*[0-9]|)*$`
* **Payload:** `1111111111111111111111111!`

There's no doubt that Regular Expressions are a valuable and powerful
mechanism but they should be used carefully.

**For validation purposes the first options should always be actively
maintained validation modules** like [validator.js][17].

```javascript
'use strict';

var validator = require('validator');

const number1 = '10.5';
const number2 = '10,5';

validator.isDecimal(number1);   // true
validator.isDecimal(number2):   // false
```

[validator.js][17] has a more generic `isWhitelisted(str, chars)` method which
can be used to perform a this validation

```javascript
'use strict';

var validator = require('validator');

const allowedChars = '0123456789.';
const decimal1 = '10.5';
const decimal2 = '10,5';

validator.isWhitelisted(decimal1, allowedChars);    // true
validator.isWhitelisted(decimal2, allowedChars);    // false
```

The second options should be well tested and activily maintained Regular
Expressions like the ones present in the
[OWASP Validation Regex Repository][18].

If there's no other chance and you have to write your own Regular Expression,
make use of [Safe Regex package][19] to validate it.

## Validate data length

Long enough strings may impact negatively on your system, creating security
issues. Although _overflows_ are not a common problem in JavaScript (still they
can happen[^5]) some database fields may have fixed length.

So, after converting input strings to the system default character encoding,
validate its length:

```javascript
'use strict';

const addressInput = '123 Main St Anytown, USA';
validator.isLength(string, {min: 3, max: 255}); // true
```

## Check for _special characters_

If the standard validation routine can not address the following inputs, then
you should check them specifically:

* null bytes (`%00`)
* new line (`%0d`, `%0a`, `\r`, `\n`)
* "dot-dot-slash" (`../` or `..\`)

Remember that alternative representations like `%c0%ae%c0%ae/` should be
covered. Canonicalization should be used to address double encoding or other
forms of obfuscation attacks.

[^1]: [HTTP/2][1], the first new version of HTTP since HTTP/1.1, is now a binary protocol intead of textual.
[^2]: "_Browserify lets you `require('modules')` in the browser by bundling up all of your dependencies._" ([source][11])
[^3]: Injection is an OWASP TOP 10 security vulnerability. This topic is covered in detail on [Output Encoding section][13]).
[^4]: ["Regular Expression Denial of Service"][15] by Alex Roichamn and Adar Weidman from Checkmarx
[^5]: ["ngineering Heap Overflow Exploits with JavaScript"][20] by Mark Daniel, Jake Honoroff and Charlie Miller

[1]: https://en.wikipedia.org/wiki/HTTP/2
[2]: ./../validation.md
[3]: ./../data-sources.md
[4]: ./numbers.md
[5]: https://github.com/chriso/validator.js
[6]: https://en.wikipedia.org/wiki/Canonicalization
[7]: http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-string-value
[8]: http://www.ecma-international.org/ecma-262/6.0/#sec-typedarray-objects
[9]: https://nodejs.org/dist/latest-v6.x/docs/api/buffer.html#buffer_buffer
[10]: https://www.npmjs.com/package/buffer
[11]: http://browserify.org/
[12]: https://www.owasp.org/index.php/Top_10_2017-A1-Injection
[13]: ../../output-encoding/README.md
[14]: http://www.regular-expressions.info/
[15]: https://www.checkmarx.com/wp-content/uploads/2015/03/ReDoS-Attacks.pdf
[16]: https://regex101.com/r/7192zY/1
[17]: https://github.com/chriso/validator.js
[18]: https://www.owasp.org/index.php/OWASP_Validation_Regex_Repository
[19]: https://www.npmjs.com/package/safe-regex
