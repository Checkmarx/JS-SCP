Password Policies
=================

Passwords are historical assets part of most authentication systems.
Additionally, passwords are the number one target of attackers.

Often enough, a service leaks its user database, and despite the exposure of
email addresses and other personal data, the biggest concern are passwords. Why?
Because passwords are not easy to manage nor remember. Users tend to
use weak passwords (e.g. "123456") that they can easily remember and re-use for
different services.

If your application sign-in requires a password, the best you can do is
"_enforce password complexity requirements, (...) requiring the use of
alphabetic as well as numeric and/or special characters)_". Password length
should also be enforced - "_eight characters is commonly used, but 16 is better
or consider the use of multi-word pass phrases_".

Of course that none of the previous guidelines will prevent users from re-using
the same password. The best you can do to tackle this bad practice is to
"_enforce password changes_", preventing password re-use. "_Critical systems may
require more frequent changes. The time between resets must be administratively
controlled_".

## Reset

Even if you're not applying an extra password policy, users still need to be
able to reset their password.
Such a mechanism is as critical as signup or sign-in, and you're encouraged to
follow the best practices to be sure your system does not disclose sensitive
data nor is compromised.

"_Passwords should be at least one day old before they can be changed_". This
way you'll prevent attacks on password re-use. Whenever using "_email based
resets, only send email to a pre-registered address with a temporary
link/password_" which should have a short expiration time.

Whenever a password reset is requested, the user should be notified.
Similarly, temporary passwords should be changed on next use.

A common practice for password reset is the 'Security Question', whose answer
was previously configured by the account owner. "_Password reset questions
should support sufficiently random answers_": asking for "Favorite Book?" may
lead to "The Bible" quite often which makes this reset question a bad one.
