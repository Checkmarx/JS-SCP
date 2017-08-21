Dependencies
============

With Node.js advent came [npm][2] - Node Package Manager and more recently
[yarn][3] by Facebook. Both promote reusable JavaScript code, going beyond
Node.js scope: following the best practices and with tools like [Browserify][4],
now developers write code once and can use it both server and client side.

Together with other tool and services like the popular [Mocha test framework][5]
or Github free repositories for Open Source projects and it's integration with
[Travis CI][6], there was an important overall increase on code quality.

This is the perfect match with OWASP SCP recommendation "_use tested and
approved managed code rather than creating new unmanaged code for common tasks".

Nevertheless, while writing their own applications developers should handle
third-party dependencies carefully as they will be part of their applications.

We will focus on [npm][2] as it is still the most used package manager. In
general [yarn][3] is full compatible with [npm][2] what doesn't mean that
discussed issues do also apply to it. In fact [yarn][3] came to solve some of
them.

To install a dependency using [npm][2] we do:

```shell
$ npm install [PACKAGE-NAME] --save
```

Then `PACKAGE-NAME` is downloaded and stored locally at `node_modules` folder
and the `package.json` file is updated to track the new dependency.

By default, issuing the [npm install command][7] as we did above, downloads the
most recent `PACKAGE-NAME` version and `package.json` is updated with an entry
like `"package-name": "^1.2.4"` in the `dependencies` section.

Assuming you're using a version control system like [Git][8], `node_modules` is
something you exclude. So, whenever you checkout your application from Git, you
have to install its dependencies, what you running:

```shell
$ npm install
```

And here is the problem.
Right now you're not sure that you got `package-name` version `1.2.4`.

Maybe in the meanwhile, `package-name` received some fixes and the actual
version is something like `1.2.10`. Or some features were improved or new ones
added and now it is at version `1.3.0`.

1. What if these changes done to `package-name` break your application?
2. What if these changes introduces security vulnerabilities?

Let's answer these questions, starting from the second one.

You should always audit your dependencies. As they will be part of your
application it is your responsibility to do it. More, it is your duty.

Some developers rely on Github stars to assess package's "quality", but if you
think about Github stars as Facebook like, probably we agree that they don't
say much.

Hopefully your task won't be a hard one. There are interesting projects
assessing npm packages security. You have, for examples, [Snyk][9] and the [snyk
npm package][10] which helps you find, fix and monitor for known
vulnerabilities. This is a tool you can easily integrate on your CI (Continuous
Integration) pipeline or run as a [package.json script][10] (as you probably do
to run linter and tests).

[Node Security Platform][11] and [its nsp tools][12] is another option. Both
will mitigate the risk of using insecure dependencies.

To shed some light on how to prevent insecure dependencies to reach your
repository, remember that also Git (and most of the version control systems) do
provide hooks. [Git, for example, has a pre-commit hook][13] which can be used
to run [snyk][10] or [nps][12] (or both, why not?), aborting in case of failure.

Time to answer the first question: "_What if these changes done to
`package-name` break your application?_".

Well [npm shrinkwrap][14] answers the question "_locking down dependency
versions for publication_".

After running all your tests and security assessment it is time to run:

```shell
$ npm shrinkwrap
```

If you don't have one, a `npm-shrinkwrap.json` file will be created from the
`package.json` one with something like:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "dependencies": {
    "package-name": {
      "version": "1.2.4",
      "from": "package-name@latest",
      "resolved": "https://registry.npmjs.org/sax/-/package-name-1.2.4.tgz"
    }
  }
}
```

When running `npm install` to install your application dependencies,
`npm-shrinkwrap` will take precedence despite of the fact `package-name` may
have been updated.

A final and important note about "typo-squatting" attacks.
With a huge database of packages which are identified by their names, attackers
have been publishing malicious packages using names similar to some popular
packages. 

Double check what you type and always validate that you've got what you've
looking for from your package manager.

[2]: https://www.npmjs.com/
[3]: https://yarnpkg.com/en/
[4]: http://browserify.org/
[5]: https://mochajs.org/
[6]: https://travis-ci.org/
[7]: https://docs.npmjs.com/cli/install
[8]: https://git-scm.com/
[9]: https://snyk.io/
[10]: https://docs.npmjs.com/misc/scripts
[11]: https://nodesecurity.io/
[12]: https://www.npmjs.com/package/nsp
[13]: https://git-scm.com/book/gr/v2/Customizing-Git-Git-Hooks#_committing_workflow_hooks
[14]: https://docs.npmjs.com/cli/shrinkwrap