Validation
==========

The Input Validation section on [OWASP Secure Coding Practices Guide][1] is a
sixteen-item checklist, which goes hand in hand with the [Output Encoding][5]
section. It is one of the most important sections when it comes to application
security, data consistency and integrity, as data tends to be used across an
array of systems and applications.

Single and centralized [Input Validation][6] routines should be used by
applications running on trusted systems, typically in the server but generally
on any non-exploitable system that is out of reach to unauthorized people.
Additionally, performing client-side validations is a bad idea since those
providing the info are controlling the system. Routines as such reduce risks and
maintainability costs, as implementation errors will be fixed as data integrity
if kept throughout the application’s workflow.

To avoid problems caused by unhandled characters or unsupported character
encodings, **the validation routine should use a single and proper character
set, such as UTF-8, to all sources of input**. Therefore, as soon as the data
reaches the trusted system where Input Validation is performed, and before the
validation stage, the data should be encoded. A how-to on performing this task
is demonstrated in the [Data Types String][2] section.

For web applications, **HTTP request and response headers should be subject of
validation (e.g. headers should only contain ASCII characters)**. Security
problems such as [HTTP response splitting][3]  are caused by failure of proper
input validation/sanitization. Remember that HTTP headers are also untrusted
data sources (e.g. cookies). If you're curious about how Node.js validates HTTP
headers [have a look at the source code][4].

Also, remember to **validate data from redirects as "an attacker may submit
malicious content directly to the target of the redirect, thus circumventing
application logic and any validation performed before the redirect"**.

[1]: https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf
[2]: ./data-types/string.md
[3]: https://www.owasp.org/index.php/HTTP_Response_Splitting
[4]: https://github.com/nodejs/node/blob/9e4ab6c2065229c5f79b6cc663d554f9de16f703/lib/_http_common.js#L331
[5]: ../output-encoding/README.md
[6]: README.md
