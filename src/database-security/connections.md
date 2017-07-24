Database Connections
====================

## Connection string protection

To keep your connection strings secure, it's always a good practice to put the authentication details on a separated configuration file outside public access.

Instead of placing your configuration file at `/home/public_html/`, consider `/home/private/config.json` (should be placed in a protected area).

```json
{
  db: {
    user: 'f00',
    pass: 'f00?bar#ItsP0ssible',
    host: 'localhost',
    port: '3306'
  }
}
```

Then you can load your `config.json` file on your JavaScript application:

```js
var cfg = require('./config.json');
var dbUser = cfg.db.user;
var dbPass = cfg.db.pass
```

Of course, if the attacker has root access, he could see the file. Which brings
us to the most cautious thing you can do - encrypt the file

## Database Credentials

You should use different credentials for every trust distinction and level:
* User
* Read-only user
* Guest
* Admin

That way if a connection is being made for a read-only user, they could never
mess up with your database information because the user actually can only read
the data.


