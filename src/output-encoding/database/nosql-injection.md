NoSQL
=====

In the past years, NoSQL databases have become prevalent in some use cases.

Explaining the differences between SQL and NoSQL databases are beyond the scope of this
document, but one of the main differences between the two is that NoSQL
compromises data consistency in favor of availability, partition tolerance,
and speed.

Another big difference is the flexibility in data types. While in SQL databases
the first step before storing data is to design the schema, in NoSQL it's
possible to add/remove/change new data types without having to redesign the
schema and migrate the database to the new schema.

It's also important to know that there are various NoSQL databases that are best
suited for different purposes. We will consider [MongoDB][1] as [it is the most
popular Database Management System according to DB-Engines.com][2].

Being here you should have no doubt or at least suspect that NoSQL databases are
vulnerable to injection attacks just like any other database. This misconception
stems from the lack of support for traditional SQL syntax.

## MongoDB

In this section we will focus on injection attacks but MongoDB security is more
than this. To learn more about [MongoDB security please refer to the official
documentation][3].

As usual whenever an application accepts user input as query parameters, then
malicious content can be injected into the database unless some steps are taken
to prevent it.

Note that since there is no common language between NoSQL databases, the
injection code samples presented here are database specific and assumes that the
database engine is JavaScript _capable_.

According to the MongoDB documentation, there are three operations that allow
arbitrary JavaScript expressions to run directly on the server. These operations
are:  

* [`$where`][4]
* [`mapReduce`][5]
* [`group`][6] (_deprecated since MongoDB 3.4_)

Let's have a look at `$where`.

From the documentation page

> Use the `$where` operator to pass either a string containing a JavaScript
> expression or a full JavaScript function to the query system. The `$where`
> provides greater flexibility, but requires that the database processes the
> JavaScript expression or function for each document in the collection.
> Reference the document in the JavaScript __expression__ or __function__ using
> either `this` or `obj`.

Now let's consider the following examples where `req.query.id` represents a
value retrieved from request URL query string

```javascript
const dbQuery = {
  $where: 'this.UserID = ' + req.query.id
}

db.Users.find(dbQuery);
```

This query returns the document whose `UserID` is equal to the `req.query.id`
value.

Making `req.query.id` equals to `0; return true` will lead to the expression
`this.UserID = 0; return true` which is the NoSQL equivalent to:

```sql
SELECT * FROM Users WHERE UserID = 0 OR 1 = 1
```

allowing all users to be listed.

## Mitigation

Get used to this, you should always perform input validation and reject any
invalid input. This is more than half the job to get you safe.

Then you should always use MongoDB data types on your queries so that even the
input that passed the validation process is casted to what you're expecting

```javascript
const dbQuery = {
  $where: 'this.UserID = new Number(' + req.query.id + ')'
}

db.Users.find(dbQuery);
```

Obviously the `new Number(0; return true)` would fail, throwing a database
error. To know more about [Error Handling and Logging read the appropriate
section][7].

[1]: https://www.mongodb.com/
[2]: https://db-engines.com/en/ranking
[3]: https://docs.mongodb.com/manual/security/
[4]: https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where
[5]: https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce
[6]: https://docs.mongodb.com/manual/reference/command/group/#dbcmd.group
[7]: ../../error-handling-logging/README.md