Strings
=======

The user input data transmitted over HTTP reaches the server and is `String`,
thus the protocol itself is handled as textual[^1]. Of course, internally,
applications may be expected to handle other data, `Numbers` or `Boolean` values
requiring data type conversion (which occurs after validation).

Even when a String is expected (e.g. user's name or address), it should be
handled carefully as characters encoding may compromise the application's
security.
The validation routine requirements and data source classification are
discussed in the Validation and Data Sources sections, respectively.

We’d like to enforce that **all data coming from untrusted data source such as
parameters, URLs and HTTP header content, should be subject of validation prior
to processing and validation failure should result in input rejection**.

This section covers string validation and manipulation in general. How to
convert a String to a Number is covered in the [Numbers][4] section.

**Important** - you should always avoid writing your own validation routine. A
well-tested and actively maintained validation routine, such as
[Validator.js][5], should be used instead.

## Encode Data to a Common Character Set

Best known as Canonicalization, this aims to convert all data to a standard or
canonical form so that the system can handle data with different representations
using the same routine. How Strings are handled internally is well documented on
the [ECMAScript <sup>&reg;</sup> 2015 Language Specification][7]:

> A String value is a member of the String type. Each integer value in the
> sequence usually represents a single 16-bit unit of UTF-16 text. However,
> ECMAScript does not place any restrictions or requirements on the values
> except for that they must be 16-bit unsigned integers.

Prior to the introduction of [TypedArray][8] in ECMAScript 2015 (ES6), Node.js
introduced the [Buffer class][9], allowing you to handle and convert strings
encoding easily:

```javascript
// read user input into a Buffer
const userInputName = Buffer.from(req.body.name);

// actually conver user input to UTF-8
const userName = userInputName.toString('utf8');

console.log('Hello %n', userName);
```

If you want to do the same on client-side, you can use the Buffer module which
is backed by Typed Arrays. In fact, these are the modules used by
[Browserify][11][^2]. However, if you're not using it, you can require the
module directly:

```javascript
// Note the trailling slash
const Buffer = require('buffer/').Buffer;

// read user input into a Buffer
const userInputName = Buffer.from(req.body.name);

// actually convert user input to UTF-8
const userName = userInputName.toString('utf8');

console.log('Hello %s', userName);
```

## Validate All Input Against A Whitelist of Allowed Characters

Whenever possible, this is an important first step from a security standpoint as
it contributes to prevent injections[^3]. Nevertheless, if any potential
hazardous characters must be allowed as input, you should implement additional
controls such as Output Encoding.

Regarding the whitelist of allowed characters, you may feel tempted to do so
using Regular Expressions. However, you should be aware that they are the real
source of problems. Not only are they hard to write, but also they are the
subject of a Denial of Service attack called Regular Expression Denial of
Service (ReDoS)[^4].

The following is a simple example of an evil regex to validate decimal numbers
and the correspondent payload (you can [test it on regex101.com][16]).

* **Regex:** `^\d*[0-9](|.\d*[0-9]|)*$`
* **Payload:** `1111111111111111111111111!`

There's no doubt that Regular Expressions are a valuable and powerful mechanism,
but they should be used carefully. **For validation purposes, the first options
should always be actively maintained validation modules** like
[validator.js][17].

```javascript
const validator = require('validator');

const number1 = '10.5';
const number2 = '10,5';

validator.isDecimal(number1);   // true
validator.isDecimal(number2):   // false
```

[validator.js][17] has a more generic `isWhitelisted(str, chars)` method which
can be used to perform a this validation:

```javascript
const validator = require('validator');

const allowedChars = '0123456789.';
const decimal1 = '10.5';
const decimal2 = '10,5';

validator.isWhitelisted(decimal1, allowedChars);    // true
validator.isWhitelisted(decimal2, allowedChars);    // false
```

The second option should be well tested and actively maintained (like those
presented in the [OWASP Validation Regex Repository][18]). If you have no choice
but to write your own Regular Expression, we recommend you make use of [Safe
Regex Package][19] to validate it.

## Validate Data Length

Strings that are long enough may negatively affect your system and will create
security issues. Although overflows are not a common problem in JavaScript[^5],
certain database fields may have a fixed length.

After converting input strings to the system default character encoding,
you should validate its length:

```javascript
const addressInput = '123 Main St Anytown, USA';
validator.isLength(string, {min: 3, max: 255}); // true
```

## Check for _special characters_

If the standard validation routine cannot address the following inputs, then you
should check them specifically:

* null bytes (`%00`)
* new line (`%0d`, `%0a`, `\r`, `\n`)
* "dot-dot-slash" (`../` or `..\`)

Remember that alternative representations such as `%c0%ae%c0%ae/` should be covered.
Canonicalization should be used to address double encoding or other forms of
obfuscation attacks.

[^1]: [HTTP/2][1], the first new version of HTTP since HTTP/1.1, is now a binary protocol instead of textual.
[^2]: "_Browserify lets you `require('modules')` in the browser by bundling up all of your dependencies._" ([source][11])
[^3]: Injection is an OWASP TOP 10 security vulnerability. This topic is covered in detail on [Output Encoding section][13]).
[^4]: ["Regular Expression Denial of Service"][15] by Alex Roichamn and Adar Weidman from Checkmarx
[^5]: ["ngineering Heap Overflow Exploits with JavaScript"][20] by Mark Daniel, Jake Honoroff and Charlie Miller

[1]: https://en.wikipedia.org/wiki/HTTP/2
[4]: ./numbers.md
[5]: https://github.com/chriso/validator.js
[7]: http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-string-value
[8]: http://www.ecma-international.org/ecma-262/6.0/#sec-typedarray-objects
[9]: https://nodejs.org/dist/latest-v6.x/docs/api/buffer.html#buffer_buffer
[11]: http://browserify.org/
[13]: ../../output-encoding/README.md
[15]: https://www.checkmarx.com/wp-content/uploads/2015/03/ReDoS-Attacks.pdf
[16]: https://regex101.com/r/7192zY/1
[17]: https://github.com/chriso/validator.js
[18]: https://www.owasp.org/index.php/OWASP_Validation_Regex_Repository
[19]: https://www.npmjs.com/package/safe-regex
[20]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.145.4704&rep=rep1&type=pdf
