Parameterized Queries
=====================

Prepared Statements (with Parameterized Queries) are the best and most secure
way to protect against SQL Injections.

In some reported situations, prepared statements could harm the performance of
the web application. Therefore, if for any reason you need to stop using this
type of database query, we strongly suggest to read the [Input Validation][1]
and [Output Encoding][2] sections.

## Vulnerable Code

Before we start, a gentle reminder of code vulnerable to an SQL injection.

```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1
const sql = 'SELECT * FROM users WHERE id = ' + evilUserData;
client.query(sql, (error, results, fields) => {
  if (error) {
    throw error;
  }

  // ...
});
```

An attacker is able to modify the SQL request in order to exfiltrate
information or modify the database.
When performing SQL queries, using a String concatenation is the worst thing to
do!

Let's have a look at [Postgres][3] and [MySQL][4] packages in order to avoid
this situation.

## Postgres

The [postgres package][5] supports parameterized queries. It is really easy to
use.
Looking at the vulnerable code above, we just need to do the following:

```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1
const sql = 'SELECT * FROM users WHERE id = $1';

client.query(sql, evilUserData, (error, results, fields) => {
  if (error) {
    throw error;
  }

  // ...
});
```

## MySQL

Although [mysql package][6] doesn't support parameterized queries, it offers
placeholder `?`

```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1
const sql = 'SELECT * FROM users WHERE id = ?';

connection.query(sql, [evilUserData], (error, results, fields) => {
  if (error) {
    throw error;
  }

  // ...
});
```

This is almost the same as escaping `evilUserData` yourself with a great
advantage - you will never forget to escape it.


```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1
const sql = 'SELECT * FROM users WHERE id = ' + connection.escape(evilUserData);

connection.query(sql, (error, results, fields) => {
  if (error) {
    throw error;
  }

  // ...
});
```

[1]: /input-validation/README.md
[2]: /output-encodeing/README.md
[3]: https://node-postgres.com/features/queries#parameterized-query
[4]: https://github.com/mysqljs/mysql#escaping-query-values
[5]: https://www.npmjs.com/package/pg
[6]: https://www.npmjs.com/package/mysql
