SQL Injection
=============

When using SQL databases like MySQL, exploit query parameters to execute
arbitrary SQL instructions is called SQL Injection (SQLi).

Consider the following sample [Express framework][1] router which queries a
[MySQL][2] database.

```javascript
const express = require('express');
const db = require('./db');

const router = express.Router();

router.get('/email', (req, res) => {
  db.query('SELECT email FROM users WHERE id = ' + req.query.id);
    .then(record => {
      // do stuff
      res.send(record[0]);
    })
});
```

The application gets the user `id` from the URL and queries the database to
retrieve it's email address.

There are two wrong things with this example:

1. the database query is built via String concatenation;
2. user input, which should always be handled as untrusted and unsafe data, is
   concatenated to the query;

A numerical query string `id` parameter would lead to an expectable query like
the one below

```sql
SELECT email FROM users WHERE id = 1
```

but a well crafted query string `id` parameter may dump all tables names
available in current database. Consider the following query string parameter
`id` value

```
1 UNION SELECT group_concat(table_name) FROM information_schema.tables WHERE table_name = database()
```

the resulting query would look like

```sql
SELECT email FROM users WHERE id = 1 UNION SELECT group_concat(table_name) FROM information_schema.tables WHERE table_name = database()
```

This would put on the screen the list of all database tables. Them the attacker
can keep going retrieving whatever information we wants from the database.

Ultimately, with the right permissions, the attacker can even write files to the
disk. Consider the following value to the query string parameter `id`

```
1 UNION SELECT "<h1>hello world</h1>" INTO OUTFILE "/home/website/public_html"
```

leading to the following query to be executed

```sql
SELECT email FROM users WHERE id = 1 UNION SELECT "<h1>hello world</h1>" INTO OUTFILE "/home/website/public_html"
```

## Mitigation

The first mistake we should pay attention: the application expects
`req.query.id` to be numerical but no input validation is performed. In the
[Input Validation chapter][3] you will find examples of how to do it right.
Remember that whenever the input validation fails, the input should always be
rejected and in this case, the query wouldn't be executed.

If instead of a numerical parameter the query expects a string, the input
validation may not be enough, nevertheless it should be done (e.g. against a
whitelist of allowed characters, etc.). 

Then, to secure this and any other database query we will need a simple and
single step: use Prepared Statements or Parameterized Queries in all
database queries which accepts parameters (as replacement of queries build via
string concatenation). You can read about [Parameterized Queries on Database
Security][4].

For completeness this is how our router would look like using a parameterized
query in Postgres:

```javascript
const express = require('express');
const db = require('./db');

const router = express.Router();

router.get('/email', (req, res) => {
  db.query('SELECT email FROM users WHERE id = $1', req.query.id);
    .then(record => {
      // do stuff
      res.send(record[0]);
    })
});
```

Note that no input validation was added to the above code sample. Although it
should be performed it was omitted on purpose to demonstrate that parameterized
queries would suffice to prevent the SQLi.

Some DBMS do not support parameterized queries but in most cases modules offer
"_placeholders_" as alternative. This is the case of [npm mysql package][5] as
explained on [Parameterized Queries section][4] of Database Security chapter.

If everything else fails or is unavailable what should be done is "remove the
meaning" of any special character within `req.query.id`. This operation is
known as "_escaping_" and it is also cover on [Parameterized Queries section][4].

[1]: https://expressjs.com/
[2]: https://www.mysql.com/
[3]: ../../input-validation/README.md
[4]: ../../database-security/parameterized-queries.md
[5]: https://www.npmjs.com/package/mysql