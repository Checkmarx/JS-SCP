Database Connections
====================

## The concept

FIXME


## Connection string protection

FIXME: revelant


## Database Credentials

You should use different credentials for every trust distinction and level:
* User
* Read-only user
* Guest
* Admin
That way if a connection is being made for a read-only user, they could never
mess up with your database information because the user actually can only read
the data.


