# Data sources

Modern Web Applications have multiple data sources such as user interfaces and
third party services via APIs.

These **data sources should be identified and classified as trusted and
untrusted**. **Anytime data is passed from a trusted source to a less trusted
one, integrity checks should be made** in order to guarantee that the data has
not been tampered with.

While classifying data sources don't take them as trusted because of the size
or importance of the entity ruling them. If the data source is out of your
control you are better classifying them as untrusted, otherwise your application
can be compromised through them.

Other data source checks include:

* _Cross-system consistency checks_
* _Hash totals_
* _Referential integrity_

**Note:** In modern relational databases, if values in the primary key field
are not constrained by the database's internal mechanisms then they should be
validated.

* _Uniqueness check_
* _Table look up check_
