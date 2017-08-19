NoSQL
=====

In the past years, NoSQL databases have become prevalent in some use cases.

Explaining the differences between SQL and NoSQL is beyond the scope of this
document, but one of the main differences between the two is that NoSQL
compromises data consistency in favor of availability, partition tolerance,
and speed.

Another big difference is the flexibility in data types. While in SQL databases
the first step before storing data is to design the schema, in NoSQL it's
possible to add/remove/change new data types without having to redesign the
schema and migrate the SQL DB to the new schema.

It's also important to know that there are various NoSQL databases that are
suited for different purposes.



## MongoDB

According to [DB-Engines.com][4] the most popular NoSQL databases in use is
`MongoDB`.  

In this section we will look at the security risks involving `MongoDB`.
For more information regarding other aspects of MongoDB see the
[MongoDB documentation][1]

As is usual if the application accepts user input as query parameters, then
malicious content can be injected into the database unless some steps are taken
to prevent it.

There are several aspects of security to keep in mind if using MongoDB. In this
section of the document we will focus on NoSQL injection.

Before diving into injection, it's important to clarify that that NoSQL
databases are vulnerable to injection attacks just like any other database.
This misconception stems from the lack of support for traditional SQL
syntax.

__Remember__: NoSQL databases are also susceptible to injection attacks.

Another important aspect to keep in mind is that since there is no common
language between NoSQL databases, the injection code presented here is database
specific and assumes the middleware to be Javascript.

### BSON:

### Javascript:

According to the MongoDB documentation, there are three operations that allow
arbitrary Javascript expressions to run directly on the server.

These operations are:  

`$where` - https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where  

`mapReduce` - https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce

`group` - https://docs.mongodb.com/manual/reference/command/group/#dbcmd.group

(_Note: `group` has been deprecated since MongoDB 3.4_)

Let's have a look at each.

### $where

First, the `$where` operation:
> Use the `$where` operator to pass either a string containing a JavaScript
expression or a full JavaScript function to the query system. The `$where`
provides greater flexibility, but requires that the database processes the
JavaScript expression or function for each document in the collection.
Reference the document in the JavaScript __expression__ or __function__ using
either `this` or `obj` .

Now let's consider the following examples.

Javascript expression using `this`:

```javascript
var DBQuery = {
  $where: "this.UserID = " + post_UserID
}

db.Users.find(DBQuery);
```

This query returns the user whose `UserID` equals the `request.body.UserID`
value, which is coming from an HTTP `POST` Request.

If the `post_UserID` value is `'0; return true` then the query
will be evaluated as:  

`this.UserID = 0; return true`  

which is the `NoSQL` equivalent to:     

`SELECT User FROM Users WHERE UserID = '0' OR 1=1`

__________________________

Another possible injection is to use the `$or` - Logical OR operator.

> The `$or` operator performs a logical OR operation on an array of two or more <expressions> and selects the documents that satisfy at least one of the <expressions>.
The $or has the following syntax:

>```json
{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
```

If we consider a an HTTP request with the following parameters in the URL -
`Username=cx&Password=123456`.

And that our Javascript middleware reads these parameters as arguments to DB
query.

If attacker injects the `$or` operator in the URL so it
becomes:

 `Username=0', $or: [ {}, {'a':'a&password=' }]`

then MongoDB will process the request as:

`{username: 'user1', $or: [ {}, {'a' : 'a' , password: '' } ] }`

Which is the SQL equivalent of :

`SELECT * FROM Users WHERE username = "user1" AND (TRUE OR (‘a’=’a’ AND password = ‘’))`

Which will return all the `Users` whose Username is `user1` and or not empty.

//TODO: Check ALL Code

__________________________

### Map Reduce:

Another common feature of NoSQL databases is the `MapReduce` operation. `Map`
is a function used to transform some data in a list, and returns the same list
after applying the desired transformation.

`Reduce` is another function that is used to "gather" the data in a list,
performing some operation on _all_ of the data, and reducing them to a _single_
value.

A simple example is to consider a dataset with keywords related to countries. If
we wanted to count each occurrence of a keyword related to each country and
afterwards "reduce" our total count of all country related keyword occurrences,
to a single value with the total number of occurrences. Using `MapReduce`
would be a good way to achieve it.

Generally `mapReduce` is not suited to be used as a regular interface, but rather
to generate things in the backgrounds, such as reports or data caching.

This a very simple explanation of `mapReduce`. Further information regarding this can
be found on the following [page][5].

Let's have a look at an example:
This code _maps_ each user to a country, and then _reduces_ the results
to get the total of users per country.

a sample of the data in the JSON is the following:
```json
[
  {
    "userID": "1",
    "name" : "John",
    "email": "john@example.com",
    "country": "Portugal"
  }
]
```

```javascript
var mapper = function {emit(this.contry), 1};

var reducer = function (country, count) {
   return Array.sum(count)
 };

db.sourceData.mapReduce(
  mapper,
  reducer,
  {
    out: "results"
  }
);

db.results.find(function (err, res) {
  if (err) console.log (err);
  console.log(res);
});
```

//TODO mapReduce Injection example!


Note that the `$where` operator, `mapReduce` and `group` (deprecated since
  MongoDB 3.4) command __cannot__ access certain global functions or properties
  such as `db` and others.
For more information please consult the MongoDB documentation.

[1]: http://LINK_DOCS
[2]: https://zanon.io/posts/nosql-injection-in-mongodb
[3]: https://github.com/cr0hn/nosqlinjection_wordlists
[4]: https://db-engines.com/en/ranking
