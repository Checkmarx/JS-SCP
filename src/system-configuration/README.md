System Configuration
====================

Keeping things updated is key in security. So, with that in mind, developers
should keep JavaScript framework updated to the latest version as well as external packages and
frameworks used by the web application.


## Directory listings

If a developer forgets to disable directory listings (OWASP also calls it
[Directory Indexing][4]), an attacker could check for sensitive files navigating
through directories.


## HTTP Headers
On production environments, remove all functionalities and files that you don't
need. Any test code and functions not needed on the final version
(ready to go to production), should stay on the developer layer and not in a
location everyone can see - _aka_ public.

HTTP Response Headers should also be checked. Removing the headers which
disclose sensitive information like:

* OS version
* Webserver version
* Framework or programming language version

FIXME

This information can be used by attackers to check for vulnerabilities in the
versions you disclose, therefore, it is advised to remove them.

By default, this is not disclosed by Go. However, if you use any type of external
package or framework, don't forget to double-check it.

Try to find something like:



## Implement better security

Put your mindset hat on and follow the [least privilege principle][1] on the web
server, processes and service accounts.

Take care of your web application error handling. When exceptions occur, fail
securely. You can check [Error Handling and Logging][5] section in this guide
for more information regarding this topic.

Prevent disclosure of the directory structure on your `robots.txt` file.
`robots.txt` is a direction file and __NOT__ a security control.
Adopt a white-list approach:

```
User-agent: *
Allow: /sitemap.xml
Allow: /index
Allow: /contact
Allow: /aboutus
Disallow: /
```

The example above will allow any user-agent or bot to index those specific
pages and disallow the rest. This way you don't disclose sensitive folders or
pages - like admin paths or other important data.

Isolate the development environment from the production network. Provide the
right access to developers and test groups, and better yet, create additional
security layers to protect them. In most cases, development environments are
easier targets to attacks.

Lastly, but still very important, is to have a software change control system to
manage and record changes in your web application code (development and
production environments). There are numerous Github host-yourself clones that
can be used for this purpose.



A more in-depth analysis of this implementation can be found [here][5].

[1]: https://www.owasp.org/index.php/Least_privilege
[2]: https://godoc.org/golang.org/x/net/webdav
[4]: https://www.owasp.org/index.php/OWASP_Periodic_Table_of_Vulnerabilities_-_Directory_Indexingi
