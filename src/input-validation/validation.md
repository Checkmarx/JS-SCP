Validation
==========

"Input Validation" section on [OWASP Secure Coding Practices Guide][1] is a
sixteen items checklist, which along with the "Output Encoding" section, is one
of the most important not only regarding application security but also data
consistency and integrity since data is usually used across a variety of
systems and applications.

**A single and centralized Input Validation routine should be used by the
application, running on a trusted system**, most often the server but in
general any non-exploitable system which should be out-of-control from
unauthorized people (performing client-side validations are in general a bad
idea, since who's providing the input is also in control of the system).

Such routine not only reduces security risks but also reduces maintainability
costs as implementation errors will be fixed once while keeping data integrity
along all application workflow.

To avoid problems caused by unhandled characters or unsupported character
encodings, **the validation routine should use a single and proper character
set, such as UTF-8, to all sources of input**. So as soon as data reaches the
trusted system where `Input Validation` is performed and even before validation,
data should be encoded. How to perform this task is demonstrated on
[Data Types String section][2].

For Web Applications, **HTTP request and response headers should also be subject
of validation e.g. headers should only contain ASCII characters**. Some
security problems like [HTTP response splitting][3] are caused by failure on
proper input validation/sanitization. Remember that HTTP headers are also
untrusted data sources e.g. Cookies.
If you're curious about how Node.js validates HTTP headers
[have a look at the source code][4].

Also remember to **validate data from redirects as "_an attacker may submit
malicious content directly to the target of the redirect, thus circumventing
application logic and any validation performed before the redirect_"**.

[1]: https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf
[2]: ./data-types/string.md
[3]: https://www.owasp.org/index.php/HTTP_Response_Splitting
[4]: https://github.com/nodejs/node/blob/9e4ab6c2065229c5f79b6cc663d554f9de16f703/lib/_http_common.js#L331
