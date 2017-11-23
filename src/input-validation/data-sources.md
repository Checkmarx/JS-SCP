# Data Sources

Modern web applications have multiple data sources, such as user interfaces and
third party services via APIs. These **data sources should be identified and
classified as trusted and untrusted. Any time data is passed from a trusted
source to a less trusted one, integrity checks should be made** in order to
guarantee that the data has not been tampered with.

While classifying data sources, do not assume that they can be trusted because
of the size or importance of the entity ruling them. If the data source is out
of your control, you are better classifying them as untrusted. Otherwise, your
application can be compromised through them.

Other data source checks include:

* Cross-system consistency checks
* Hash totals
* Referential integrity

**Note** - in modern relational databases, if values in the primary key field
are not constrained by the database's internal mechanisms, they should be
validated.

* Uniqueness check
* Table look up check
