# Strings

On web development, all user input that reaches the server over HTTP is handled
as `String`, but internally applications may expect to handle other data types
such as `Number`s or `Boolean` values, what requires data type conversion.

Even when expecting a `String` (e.g user's name or address) it should be
handled carefully as character set may compromise security.

This section covers `String` validation and manipulation in general. How to
convert `String` to `Number` is addressed on [Numbers section][1].

// TODO

* `strings` package contains all functions that handle strings and its
  properties.
    * [`Trim`](https://golang.org/pkg/strings/#Trim)
    * [`ToLower`](https://golang.org/pkg/strings/#ToLower)
    * [`ToTitle`](https://golang.org/pkg/strings/#ToTitle)
* [`regexp`][4] package support for regular expressions to accommodate custom
   formats[^2].
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

[1]: ./numbers.md
