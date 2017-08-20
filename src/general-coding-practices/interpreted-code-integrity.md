Interpreted code integrity
==========================

[OWASP SCP - Quick Reference Guide][1] recommends to "_use checksums or hashes to
verify the integrity of interpreted code, libraries, executables, and
configuration files_".

Let's focus on third-party resources such as JavaScript libraries or Stylesheets
downloaded on the client-side from third-party remote hosts.

Maybe you're thinking about Content Delivery Networks (CDN). They are everywhere
and we "need" them. But what if they get compromised and resources get modified
somehow?

How can we be sure that what is downloaded by our application on client-side is
what we were expecting?

[Subresource Integrity][2] (SRI) are a great illustration of this
recommendation.

Before SRI, scripts required by our client-side applications were added to the
page as illustrated below, with no warranty that what would be downloaded was
the same that was there during development time.

```html
<script src="https://example.com/example-framework.js"</script>
```

Now with SRI we can tell the browser not only to download the resource, but also
to execute it if and only if it matches a cryptographic hash

```html
<script src="https://example.com/example-framework.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
        crossorigin="anonymous"></script>
```

This is also valid for `<link>` HTML elements.

[1]: https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide
[2]: https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity