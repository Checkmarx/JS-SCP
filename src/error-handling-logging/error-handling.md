Error Handling
==============

In this section of the document, we cover the best practices for error handling
in JavaScript. Like with any robust application, error handling is essential.

The first part consists of an overview of how JavaScript handles errors.
In the second part, we will provide code examples for different types of error
handling in JavaScript.

## Overview

JavaScript has an `Error` constructor that creates an error object. Per usual,
when a runtime exception occurs, an error is thrown. The syntax is the
following:

```javascript
new Error([message[, filename[, lineNumber]]])
```

The resulting error object can also be used for user-defined exception.

An important distinction to keep in mind is related to the nature of the error.
Errors can be the result of `Operational Errors` or `Programmer errors`.

`Operational errors` are errors related to operations which _might_ fail.
This means that the application logic is correct, but an unexpected error
occurred.
Examples of this include Input/Output (I/O) errors, network problems, out of
memory, etc.

The other types of errors are called `Programmer errors`, and these are related
to logic bugs in the application's code. These errors are a result of a problem
in the application's code. Yet despite being able to correct the issue by
changing the code, they can't be handled properly since the problem stems from
mistakes in the code.
Examples of this include accessing properties of an "undefined" variable, type
confusion (passing to a function an unexpected data type), etc.  

In Node.js, there are three main ways to deliver errors:

* `throw` the error (exception)
* Passing the error to a `callback()`, a function used to handle errors and the
  results of asynchronous operations
* Using an `EventEmitter` to emit an `error` event

Also, it is important to understand the difference between `error` and
`exception`. Errors can be constructed then passed directly to another
function or _thrown_. If an `error` is _thrown_, it becomes an exception.

In plain synchronous JavaScript, the syntax of an exception tends to be the
following:

```javascript
throw new Error('Some error.');
```

Throwing exceptions is not a common pattern of JavaScript or Node.js's core.
Instead, most native APIs pass `Error` as an argument to the callback function

````javascript
callback(new Error('Some error'));
```

The usage of `callback()` is due to the asynchronous nature of JavaScript and
the associated errors that can occur.

In synchronous operations like I/O errors, all exceptions should be caught not
only due to good practices, but also to ensure that if the application fails, it
fails on a controlled fashion.

As an example, consider this __incorrect__ way to read a file:

```javascript
const fs = require('fs');

const content = fs.readFileSync('/tmp/missing-file.txt');
```

Notice that there is no error catching. By ignoring errors, if the application
fails, there's no way to ensure it failed in a controlled way, since we have
no way of knowing if anything went wrong.

So be careful and always catch exceptions in synchronous operations.

The correct way to write the previous code in order to catch exceptions is:

```javascript
const fs = require('fs');

try {
  const content = fs.readFileSync('/tmp/missing-file.txt');
} catch (e) {
  console.error('ups... something went wrong');
}
```

ES6 introduced `promises`. The simplest way to look at `promises` is to see them
as "a proxy for a value not necessarily known when the promise is created."

What this means is that by using `promises`, there are several advantages for
the developer.

Mainly:

* No more callback pyramids aka "callback hell"
* No more error handling every second line
* No more reliance on external libraries for simple operations like getting
  the result of a loop

__IMPORTANT NOTE__: Not everything is good news, one of the biggest caveats that
developers have to keep in mind when using `promises` is that any exception
thrown within a `then` handler, a `catch` handler or within the function passed
to `new Promise`, will be silently disposed unless handled manually.

Consider the following code:

```javascript
function validateUser () {
  return new Promise ((resolve, reject) => {
    getUser().then(user => {
      getUserPassword(user).then(userpassword => {
        checkPassword(userpassword).then(validated => {
          resolve(validated)
        });
      });
    });
  });
}
```

By using promises, we can re-write our previous code in the following manner.
Please note that we are still not handling errors:

```javascript
function validateUser () {
  return getUser()
    .then(getUserPassword)
    .then(checkPassword)
}
```

Now the same code, using promises and handling errors:

```javascript
function validateUser () {
  return getUser()
    .then(getUserPassword)
    .then(checkPassword)
    .catch(fallbackForRequestFail); // error handling
}
```

[ES8/ES2017][1] introduced the `async`/`await`, which allows developers to write
asynchronous code in a synchronous fashion

```javascript
app.post('/login', (req, res, next) => {
  const user = req.body.user;
  const password = req.body.password;

  // input validation omitted for brevity

  db.Users.find({user: user, password: password}, (err, user_acc) => {
    if (err) {
      logger.error(err);
      return res.status(400).send('Login failed');
    }

    user_acc.getUserDetails(user_acc, (err, user_acc) => {
        if (err) {
          logger.error(err);
          return res.status(400).send('Login failed');
        }

        user_acc.userLoginLog(user_acc, (err) => {
          if (err) {
            logger.error(err);
            return res.status(400).send('Login failed');
          }

          res.send(user_acc);
        });
    });
  });
});
```

Can now be written as:

```javascript
app.post('/login', async (req, res, next) => {
  const user = req.body.user;
  const password = req.body.password;

  try {
    const user_acc = await db.Users.find({user: user, password: password});
    const user_acc.details = await user_acc.getUserDetails(user_acc);

    await user_acc.userLoginLog(user_acc);

    res.send(user_acc);
  } catch (err) {
    logger.error(err);
    return res.status(400).send('Login failed');
  }
});
```

It is also good practice to implement custom error messages or custom error
pages as a way to make sure that no information is leaked when an error occurs.

The following examples implement a custom error page for web applications
based in Express Node.js web application framework

```javascript
const express = require('express')
const app = express();

app.enable('verbose errors');
app.use(app.router);

app.use((req, res, next) => {
  res.status(404);

  // respond with html page
  if (req.accepts('html')) {
    res.render('404', { url: req.url });
    return;
  }

  // respond with json
  if (req.accepts('json')) {
    res.send({ error: 'Not found' });
    return;
  }

  // default to plain-text. send()
  res.type('txt').send('Not found');

  // Routes
});
```

The final way in which we will demonstrate to capture errors is by using
`EventEmitter`.

If you're using event oriented programming or code that deals with streams,
an error handler should be present. `EventEmitters` fire an error event that can
be captured. The following is an example of `EventEmitter` using `promises`:

```javascript
const EventEmitter = require('events');

class Emitter extends EventEmitter {}

const myEmitter = new Emitter();
const logger = console;

myEmitter.on('error', (err) => {
  logger.error('Unexpected error.');
});

myEmitter.emit('error', new Error('Something went wrong.'));

// In case of an unhandled exception, terminate gracefully.
// Using promises.
process.on('uncaughtException', (error, promise) => {
  logger.error('Uncaught exception!', { error: error, promise: promise });
  process.exit(1);
});
```

Another important detail to keep in mind is to guarantee that no sensitive
information is within the error responses. This includes no system details,
session identifiers, account information, stack traces or debug information.

Finally, it is necessary to ensure that in case of an error associated with the
security controls, by default, access is denied.

[1]: https://www.ecma-international.org/publications/standards/Ecma-262.htm

