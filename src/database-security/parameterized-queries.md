Parameterized Queries
=====================

Prepared Statements (with Parameterized Queries) are the best and most secure
way to protect against SQL Injections.

In some reported situations, prepared statements could harm performance of the
web application. Therefore, if for any reason you need to stop using this type
of database queries, we strongly suggest to read [Input Validation][1] and
[Output Encoding][2] sections.

## Vulnerable code

Before to start, a gentle reminder of a code vulnerable to SQL injection.

```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1
var sql    = 'SELECT * FROM users WHERE id = ' + evilUserData;
client.query(sql, function (error, results, fields) {
  if (error) throw error;
  // ...
});
```

An attacker is able to modify the SQL request in order to exfiltrate information or modify the database.
When performing SQL queries, using String concatenation is the worst thing to do!

Let's have a look to [Postgres] [3] and [MySQL] [4] modules in order to avoid this situation.

## Postgres
The [`postgres`] [5] module supports parameterized queries. It is really easy to use. 
Looking at the previous vulnerable code, we just need to do the following:
```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1

var sql    = 'SELECT * FROM users WHERE id = $1';

client.query(sql, evilUserData, function (error, results, fields) {
  if (error) throw error;
  // ...
});
```
## MySQL
The [`mysql`] [6] module doesn't support parameterrized queries, but the module gives different methods to escape parameters such as `mysql.escape()`, `connection.escape()` and `pool.escape()`.
Here is an example:
```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1

var sql    = 'SELECT * FROM users WHERE id = ' + connection.escape(evilUserData);

connection.query(sql, function (error, results, fields) {
  if (error) throw error;
  // ...
});
``` 

Or you can also use `?` characters as a placeholder in order to escape values automatically by the module without using the escaping functions.
Another example with placeholders:
```javascript
// evilUserData parameter is an input given by a user
// For the example it could be someting like that: or 1=1

var sql    = 'SELECT * FROM users WHERE id = ?';

connection.query(sql, [evilUserData], function (error, results, fields) {
  if (error) throw error;
  // ...
});
```

[1]: /input-validation/README.md
[2]: /output-encodeing/README.md
[3]: https://node-postgres.com/features/queries#parameterized-query
[4]: https://github.com/mysqljs/mysql#escaping-query-values
[5]: https://www.npmjs.com/package/pg
[6]: https://www.npmjs.com/package/mysql