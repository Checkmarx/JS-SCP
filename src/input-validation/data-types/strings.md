# Strings

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

**Important** you should always avoid to right your own validation routine. A
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

Prior to the introduction of [TypedArray][8] in ECMAScript 2015 (ES6) Node.JS
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

## _Validate all input against a "white" list of allowed characters_

Whenever possible this is an important first step from security standpoint as
it will contribute to prevent [Injections][^3] nevertheless if any potential
hazardous characters must be allowed as input, 



// TODO redirect to Output Encoding section due to hazardous characters

   formats[^2].

  Use [safe-regex](https://www.npmjs.com/package/safe-regex) to avoid
  [regular expression denial of service](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS) attacks

* [`utf8`][9] package implements functions and constants to support text
  encoded in UTF-8. It includes functions to translate between runes and UTF-8
rse
  byte sequences.

  Validating UTF-8 encoded runes:
    * [`Valid`](https://golang.org/pkg/unicode/utf8/#Valid)
    * [`ValidRune`](https://golang.org/pkg/unicode/utf8/#ValidRune)
    * [`ValidString`](https://golang.org/pkg/unicode/utf8/#ValidString)

  Encoding UTF-8 runes:
    * [`EncodeRune`](https://golang.org/pkg/unicode/utf8/#EncodeRune)

  Decoding UTF-8:
    * [`DecodeLastRune`](https://golang.org/pkg/unicode/utf8/#DecodeLastRune)
    * [`DecodeLastRuneInString`](https://golang.org/pkg/unicode/utf8/#DecodeLastRuneInString)
    * [`DecodeRune`](https://golang.org/pkg/unicode/utf8/#DecodeLastRune)
    * [`DecodeRuneInString`](https://golang.org/pkg/unicode/utf8/#DecodeRuneInString)


**Note**: `Forms` are treated by Go as `Maps` of `String` values.

Other techniques to ensure the validity of the data include:

* _Whitelisting_ - whenever possible validate the input against a whitelist
  of allowed characters. See [Validation - Strip tags][1].
* _Boundary checking_ - both data and numbers length should be verified.
* _Character escaping_ - for special characters such as standalone quotation
  marks.
* _Numeric validation_ - if input is numeric.
* _Check for Null Bytes_ - `(%00)`
* _Checks for new line characters_ - `%0d`, `%0a`, `\r`, `\n`
* _Checks forpath alteration characters_ - `../` or `\\..`
* _Checks for Extended UTF-8_ - check for alternative representations of
  special characters

**Note**: Ensure that the HTTP request and response headers only contain
ASCII characters.

Third-party packages exist that handle security in Go:

* [Gorilla][6] - One of the most used packages for web
  application security.
  It has support for `websockets`, `cookie sessions`, `RPC`, among
  others.

* [Form][7] - Decodes `url.Values` into Go value(s) and Encodes Go value(s)
  into `url.Values`.
  Dual `Array` and Full `map` support.

* [Validator][8] - Go `Struct` and `Field` validation, including `Cross Field`,
  `Cross Struct`, `Map` as well as `Slice` and `Array` diving.

[^1]: [HTTP/2][1], the first new version of HTTP since HTTP/1.1, is now a binary protocol intead of textual.
[^2]: "_Browserify lets you `require('modules')` in the browser by bundling up all of your dependencies._" ([source][11])
[^3]: Injection is an OWASP TOP 10 security vulnerability. This topic is covered in detail on [Output Encoding section][13]).

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
