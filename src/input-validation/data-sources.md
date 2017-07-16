# Data sources

Anytime data is passed from a trusted source to a less trusted source,
integrity checks should be made.
This guarantees that the data has not been tampered with and we are receiving
the intended data. Other data source checks include:

* _Cross-system consistency checks_
* _Hash totals_
* _Referential integrity_

**Note:** In modern relational databases, if values in the primary key field
are not constrained by the database's internal mechanisms then they should be
validated.

* _Uniqueness check_
* _Table look up check_
