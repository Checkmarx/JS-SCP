Concurrency
===========

Shared resources bring concurrency problems such as [deadlocks][1] and
[resources starvation][2]. Techniques for solving these problems are already
well-known and documented - [semaphores][3] and [mutexes][4] are two of them.

This is an important topic which deserves to be covered here, because JavaScript
is known to be single threaded and takes advantage of its `Event Loop` for
performance. The point is that JavaScript concurrency model is precisely based
on its `Event Loop`.

You can read about JavaScript concurrency model in detail on [Mozilla Developer
NetworK "Concurrency model and Event Loop"][5].

The following few lines are meant just to remind you that your database is a
shared resource subject of concurrent connections and operations. Thankfully,
you don't have to handle database concurrency. Nevertheless, when your
application performs read/write operations on the file system or even a single
file, you may be subject to concurrency problems.

Let's walk through a common and simple example.
You have written a _middleware_ for [Express Node.js web application
framework][6] which counts visits, using a `counter.txt` file as storage.

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');

const app = express();
const file = path.resolve('./counter.txt');

const counterMiddleware = (req, res, next) => {
  fs.readFile(file, (err, data) => {
    if (!err) {
      let counter = +data.toString();
      counter++;

      fs.writeFile(file, counter, (err) => {
        next();
      });
    }

    next();
  });
};
```

On every single request, the `counter.txt` file is read and written.

As you may know and certainly expect, the Express server answers multiple
requests at the same time and, for a busy web site, two different requests may
try to access the file at the same time[^1] or write operations do not happen in
the expected order messing with the counter.

This is where synchronization mechanisms and concurrency controls come in.

You may want to have a look at one of these npm packages or other with the same
purpose, according to your needs:

* [rwlock][7]
* [semaphore][8]
* [mutexify][9]

Rule of thumb: concurrency is something you should always care about because
when it comes to scalability, the first approach will be to increase the number
of processes to handle a specific task.

[^1]: is is gonna happen more frequently if you have multiple processes attending HTTP requests

[1]: https://en.wikipedia.org/wiki/Deadlock
[2]: https://en.wikipedia.org/wiki/Starvation_(computer_science)
[3]: https://en.wikipedia.org/wiki/Semaphore_(programming)
[4]: https://en.wikipedia.org/wiki/Lock_(computer_science)
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop
[6]: https://expressjs.com/
[7]: https://www.npmjs.com/package/rwlock
[8]: https://www.npmjs.com/package/semaphore
[9]: https://www.npmjs.com/package/mutexify
