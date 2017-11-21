Database
========

Databases are also a common subject of security vulnerabilities due to lack of
proper output encoding and security best practices.

One way to compromise a database is through query manipulation, exploiting user
input data used as query parameters.

Before getting into detail of specific Database Management Systems (DBMS), lets
illustrate this using an abstract language.

Consider a robot which is capable of executing the following instruction as soon
as user provides the required instruction parameters, filling the blanks;

```
GET CAR NUMBER ___ FROM GARAGE ___ AND PAINT IT ___.
```

A user may request car number `1` from garage `3` to be painted `red`. So the
instruction sent to the robot will look like;

```
GET CAR NUMBER 1 FROM GARAGE 3 AND PAINT IT red.
```

What if a user submits something he wasn't supposed to? Well, the robot
will execute anything as long as it is syntactically valid.

Let's keep the car number (`1`) and the color (`red`) but let's play a little
with the garage number. This time let's try something like `3 and remove it's
wheels`. How would the full instruction look like?

```
GET CAR NUMBER 1 FROM GARAGE 3 and remove it's wheels AND PAINT IT red.
```

As long as the robot recognizes the given parameter `and remove it's wheels` as
a valid command and the full query is syntactically valid, it will execute it.
What you will get? The same red car but without wheels.

This is exactly how a database injection performs whether the database engine
is SQL or NoSQL.
