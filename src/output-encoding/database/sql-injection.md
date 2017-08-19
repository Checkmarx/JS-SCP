SQL Injection
=============

When using SQL databases one of the most common attacks is called SQL injection.
Essentially it relies on database query manipulation through expected parameters
before sending it to the database engine to be executed.

As a simple analogy, consider a robot that's ready to execute the following
instruction as soon as user provides the instruction parameter, filling the
blanks

```
GET CAR NUMBER ___ FROM GARAGE ___ AND PAINT IT ___.
```

A user may request car number `1` from garage `3` to be painted `red`. So the
instruction sent to the robot will look like

```
GET CAR NUMBER 1 FROM GARAGE 3 AND PAINT IT red.
```

What if a user submits something that he wasn't supposed to? Well, the robot
will execute anything as long as it is syntactically valid.

Let's keep the car number (`1`) and the color (`red`) but let's play a little
with the garage number. This time let's try something like `3 and remove it's
wheels`. How would the full instruction look like?

```
GET CAR NUMBER 1 FROM GARAGE 3 and remove it's wheels AND PAINT IT red.
```

As long as robot understands the injected `and remove it's wheels` and the full
query is syntactically valid it will execute it and you will get the same red
car but without wheels.

This is exactly how a SQL/No-SQL injection looks like but of course using their
own syntax.

## Node.js and MySQL

Now for something more hands-on, a simple Node.js application using MySQL to
explain SQL injection.

The application gets a `username` from the URL and then queries the database for
all the information regarding this user.

The problem comes from the URL parameter being used to make the SQL query. Since
the parameter in the URL is user input, unless some precautions are taken, an
attacker could manipulate the URL parameter to perform an injection attack.

The appplication code:

```javascript
var http = require('http')
var url = require('url')
var mysql = require('mysql')

var con = mysql.createConnection({
  host: "localhost",
  user: "dbadmin",
  password: "dbpass",
  database: "exampledb"
});

function getEmail(data, callback){
  con.query("SELECT * FROM users WHERE username = " + param , function(err, rows){
      if (err){
        callback(err, null)
      } else {
        callback(null, rows[0].)
      }
  });
}

http.createServer(function (req, res){
  var param = url.parse(req.url, true).query

  con.connect(function(err){
    if (err) throw err;
    getEmail(param, function(err, content) {
      if (err) console.log(err)
      else {
        res.end(JSON.stringify(content));
      }
    })
  });


}).listen(3000);
console.log("Server running at http://localhost:3000");
```

The attacker could change the URL parameter to `1' or 1=1--` which would "trick"
the database into returning all the users in the table.

### How to fix?

There are to ways to approach this problem.
One is by encoding special characters to their hexadecimal representation. This
is usually referred to as `escaping` and the following methods can be used:  

  1 - `mysql.escape()`  
  2 - `connection.escape()`  
  3 - `pool.escape()`  

The charaters considered "special characters" are the following:

![special_chars](images/special_chars.png)  
[Source - dev.mysql.com][1]

Alternatively you can use the `?` as character placeholders for identifiers to
be escaped. If using placeholders then the SQL query of the vulnerable app
presented above would be:
```javascript
[...]
  con.query("SELECT * FROM users WHERE username = ?" + param, function(err, rows){
    //do stuff
  });
[...]
```  

Another way to prevent this attack is to use `prepared statements`.
There are some benefits to using this method.

From the MySQL documentation:
> - Less overhead for parsing the statement each time it is executed. Typically,
database applications process large volumes of almost-identical statements, with
only changes to literal or variable values in clauses such as `WHERE` for queries
and deletes, `SET` for updates, and `VALUES` for inserts.

> - Protection against SQL injection attacks. The parameter values can contain
unescaped SQL quote and delimiter characters.

If using prepared statements, then our previously vulnerable query would now be
written as:
```javascript
  var sql = "SELECT * FROM ?? WHERE ?? = ?";
  var inserts = ['users', 'username', param]
  sql = mysql.format(sql, inserts);
  con.query(sql, function(err, rows){
    //do stuff
  });
```

By following this, the result is a valid and escaped query that can safely be
sent to the database.

//TODO: TESTAR CODIGO
[1]:http://TODO
