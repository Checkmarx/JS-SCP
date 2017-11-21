Validation and Storing Authentication Data
==========================================

## Validation

The key subject of this section is authentication data storage, as more often
than desirable, user account databases are leaked on the Internet.
Of course, this is not guaranteed to happen to you, but in the case of such an
event, collateral damages can be avoided if authentication data (especially
passwords) are stored properly.

First, let's make it clear that "_all authentication controls should fail
securely_". You're recommended to read all Authentication and Password
Management sections, as they cover recommendations about reporting back wrong
authentication data and how to handle logging.

One other preliminary recommendation - for sequential authentication
implementations (as Google does nowadays), validation should happen only on the
completion of all data input on a trusted system (e.g. the server).

## Storing Passwords Securely: The Theory

Now let's talk about storing passwords.

You don't really need to store passwords as they are provided by users
(plaintext), however you'll need to validate each authentication to determine
whether users are providing the same token.

So, for security reasons, what you need is a one-way function, `H` so that for
every password `p1` and `p2`, `p1` is different from `p2`, and `H(p1)` is also
different from `H(p2)`[^1].

Does this sound, or look, like math?
Pay attention to this last requirement - `H` should be like a function where
there's no function `H⁻¹` so that `H⁻¹(H(p1))` is equal to `p1`. This means that
there's no way back to the original `p1`, unless you try all possible values of
`p`.

If `H` is one-way only, what's the real problem with account leakage?

Well, if you know all possible passwords, you can pre-compute their hashes and
run a rainbow table attack.

We're sure that you were already told that passwords are hard to manage from a
user's point of view and that users tend to re-use passwords which are easy to
remember, making the password universe really small. 

How can we avoid this?

The point is - if two different users provide the same password `p1` we should
store a different hashed value.
It may sound impossible, but the answer is `salt`: a pseudo-random **unique per
user password** value which is appended to `p1` so that the resulting hash is
computed as follows: `H(p1 + salt)`.

So, each entry on passwords store should keep the resulting hash and the `salt`
itself in plaintext: `salt` is not required to remain private.

Final recommendations:

* Avoid using deprecated hashing algorithms (e.g. SHA-1, MD5, etc)
* Read the [Pseudo-Random Generators section][1].

The following code-sample shows a basic example of how a `SHA256` hash with a
fixed salt value is generated:

```javascript
var crypto = require('crypto');

var inputString = "Sample to text to be hashed"
var salt = "YourSaltHere"

var hash = crypto.createHash('sha256')
hash.update(inputString + salt)

var result = hash.digest('hex');

console.log(result)
//result:
//dd893b455e9a31bd84d015de60c3a62c85fbe393792c31aa40e0df1f2a4f9286
```

This approach has several flaws and should not be used. It is given here only to
illustrate the theory with a practical example. The next section explains how to
correctly salt passwords in real life.

## Storing password securely: the practice

One of the most important adage in cryptography is: **never roll your own
crypto**. By doing so, one can put an entire application at risk. It is a
sensitive and complex topic. Hopefully, cryptography provides tools and
standards reviewed and approved by experts. Therefore, it is important to use
them instead of trying to re-invent the wheel.

In the case of password storage, the hashing algorithms recommended by
[OWASP][2] are [`bcrypt`][3], [`PDKDF2`][4], [`Argon2`][5] and [`scrypt`][6].
This takes care of hashing and salting passwords in a robust way. In Node.js,
there are packages that provide robust implementations for most of the
aforementioned algorithms.

In our example, we will be using `bcrypt`.

It can be downloaded using  `npm install`:

```
npm install bcrypt
```

The following example shows how to use `bcrypt`, which should be good enough for
most of the situations. The advantage of `bcrypt` is that it's simpler to use,
thus less error-prone.

### Encryption:

```javascript
var bcrypt = require('bcrypt');

var saltRounds = 12;
var passPlain = "SecretPasswordToHash"

bcrypt.hash(myPlaintextPassword, saltRounds, (err, hash) => {
  // Store hash in your password DB.
});
```

### Decryption:

```javascript
bcrypt.compare(passPlain, hash, (err, res) => {
  // res === true
});
```

Note that when using `bcrypt` on a server, the async mode is recommended. This
is due to the hashing being CPU intensive. So keep in mind that using
synchronous code in the server will block the event loop and "hang" the
application while it waits for results.

[^1]: Hashing functions are the subject of Collisions but recommended hashing functions have a really low collisions probability

[1]: /cryptographic-practices/pseudo-random-generators.md
[2]: https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet
[3]: https://www.npmjs.com/package/bcryptjs 
[4]: https://www.npmjs.com/package/pbkdf2
[5]: https://www.npmjs.com/package/argon2
[6]: https://www.npmjs.com/package/scrypt
