# Post-validation Actions

According to Data Validation's best practices, the input validation is only
the first part of the data validation guidelines. As such,
_Post-validation Actions_ should also be performed.
The _Post-validation Actions_ used vary with the context and are divided in
three separate categories:

* **Enforcement Actions**

  Several types of _Enforcement Actions_ exist in order to better secure our
  application and data.

  * inform the user that submitted data has failed to comply with the
    requirements and therefor the data should be modified in order to comply
    with the required conditions.

  * modify user submitted data on the server side without notifying the user of
    said changes. This is most suitable in systems with interactive usage.

  **Note:** The latter is used mostly in cosmetic changes (modifying sensitive
  user data can lead to problems like truncating, which incur in data loss).
* **Advisory Action**

  Advisory Actions usually allow for unchanged data
  to be entered, but the source actor is informed that there were issues with
  said data. This is most suitable for non-interactive systems.
* **Verification Action**

  Verification Action refer to special cases in
  Advisory Actions. In these cases, the user submits the data and the source
  actor asks the user to verify said data and suggests changes. The user then
  accepts these changes or keeps his original input.

A simple way to illustrate this is a Billing address form, where the user
enters his address and the system suggests addresses associated with the
account. The user then accepts one of these suggestions or ships to the address
that was initially entered.
