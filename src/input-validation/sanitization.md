Sanitization
============

Sanitization refers to the process of removing or replacing submitted data.
When dealing with data, after the proper validation checks have been made, an
additional step which tends to be taken in order to strengthen data safety is
sanitization.

The [validator.js package][1] introduced for [Strings validation][2] also 
[offers a sanitizer API][3] which will be used in this section.

The most common uses of sanitization are as follows:

## Convert Single Less-Than Characters `<` to Entity

The characters `<` and `>` have a special meaning. For example, on an HTML
context as the represent respectively the start and end of a tag. If user
input is intended to be outputted on an HTML context, it should be sanitized as
soon as possible.

The [validator.js `escape(input)`][3] method, replaces `<`, `>`, `&`, `'`, `"`
and `/` by their correspondent [HTML entities][4], making it safe for output
(to know more about output safety  please refer to the [Output Encoding
section][5])

```javascript
'use strict';

var sanitizer = require('validator');

const sampleSourceCode = '<script>alert("Hello World");</script>';
const sanitizedSampleSourceCode = sanitizer.escape(sampleSourceCode);

console.log(sanitizedSampleSourceCode);
// &lt;script&gt;alert(&quot;Hello World&quot;);&lt;&#x2F;script&gt;
```

Note that [validator.js `unescape(input)`][3] does it the other way around,
replacing back HTML encoded entities.

## Remove Line Breaks, Tabs and Extra White Space

The control sequence `\r\n` is used as headers delimiter on some textual
protocols such as HTTP and mail.

If you're setting headers from input value, you have to remove all of these
characters in order to avoid, for example, [HTTP Splitting][6].

[validator.js `stripLow(input \[, keep_new_lines\])`][3] does just that:

> Remove characters with a numerical value less than 32 and higher than 127,
> mostly control characters. If keep_new_lines is true, newline characters are
> preserved (\n and \r, hex 0xA and 0xD). Unicode-safe in JavaScript.

`ltrim(input \[, chars\])` and `rtrim(input \[, chars])` functions are also of
interest to remove leading and trailing characters (JavaScript native `trim()`
function only handles characters that represent white spaces).

```javascript
'use strict';

var sanitizer = require('validator');

const string = '\tHello\r\nWorld.\n\nThis  is a test!';
const safe = sanitizer.stripLow(string);

console.log(safe);
// HelloWorld.This  is a test!
```

## URL Request Path

The dot-dot-slash characters sequence was already [referred to as a subject of
validation][7] - **if you're not handling them deliberately, any input
containing the sequence should be rejected.**

Be sure to canonicalize all URLs, converting them to an unified form;
usually absolute URLs/paths are a good option.

[1]: https://github.com/chriso/validator.js
[2]: ./data-types/strings.md
[3]: https://github.com/chriso/validator.js#sanitizers
[4]: https://www.w3schools.com/html/html_entities.asp
[5]: ../output-encoding/README.md
[6]: https://www.owasp.org/index.php/HTTP_Response_Splitting
[7]: ./data-types/strings.html#check-for-special-characters
