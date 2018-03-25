Pseudo-Random Generators
========================

In OWASP Secure Coding Practices, you'll find what seems to be a really complex
guideline: "_All random numbers, random file names, random GUIDs, and random
strings should be generated using the cryptographic module’s approved random
number generator when these random values are intended to be un-guessable_", so
let's talk about those "random numbers".

Cryptography relies on some randomness, but for the sake of correctness, what
most programming languages provide out-of-the-box is a pseudo-random number
generator. [JavaScript's Math.random()][1] is not an exception.

An example of pseudo-random number generation using `Math.random()`:

```javascript
function getRandomArbitrary (min, max) {
  const _min = Math.ceil(min);
  const _max = Math.floor(max);

  return Math.floor(Math.random() * (_max - _min) + _min);
}

const max = 1000
const min = 0

console.log(getRandomArbitrary(min,max))
```

Because [Math.random()][1] is a pseudo-random number generator they use a Seed,
like many others.
This Seed is **solely** responsible for the randomness of the pseudo-random
number generator -- if it is known or predictable, the same will happen to
generated number sequence.

You should carefully read the documentation when it states that:

> `Math.random()` does not provide cryptographically secure random numbers. Do
> not use them for anything related to security. Use the `Web Crypto API`
> instead, and more precisely the `window.crypto.getRandomValues()` method.

In Node.js, we can use the `crypto.randomBytes` to generate pseudo-random
numbers. But this requires some precautions, namely, the direct usage of it.

So, why does the direct usage lead to problems? Due to a possible "bias" in
random value generation.
A great article with additional information written by user joepie91 on
github.com can be found [here][2].

So, how can we generate a cryptographically secure random values? There are two
ways we can approach this problem, each more suited for a specific purpose.

First, using the UUID version 4. UUID stands for `Universally unique
identifiers` and consists of a 128 bit number.

In Node.js, there are several packages that can be used to generate a UUID. The
most popular is `uuid` and can be found [here][3].

The package is very easy to use, as shown in the sample below:

```javascript
const uuid = require('uuid/v4');
const uuid1 = uuid();

console.log(uuid1);
// a61903ff-bb29-4846-849d-0da018892da4
```

Another option is to generate a cryptographically secure value using the
`crypto` module. In this module, there's a method called `randomBytes` that
returns a buffer with randomly generated values.

Looking at the documentation we can see we need some parameters being passed
along to generate our random value: 

```
crypto.randomBytes(size[, callback])
```

So let's look at an usage example taken from the documentation:  

```javascript
const crypto = require('crypto');

// Generate a 256 byte random value
crypto.randomBytes(256, (err, buf) => {
  if (err) {
    throw err;
  }

  console.log(`${buf.length} bytes of random data: ${buf.toString('hex')}`);
  // 256 bytes of random data: fc1eb987fb77455d4062630629feb88aafeaa1bc5dbfb66616cf186a3d9e051222e440f28c14b2b75fb66e8bc31a89f41722f1eaa4f77739d5ead03342534d589d317b79157a28a31f47f456df62d20fc0befbff40e18bb5a26422d0952c380ea8d8e4368918c0e2ced3b75afac4cef5a215b3af33a93b3f043cd4fcbf2ed8333a046ee726f569dab1a6a15ee6a10ca1ece7c40b1b0adddd6bed04f48dbf4791bb63307855394d167dd4356738e31fbce3ce381c824b8be3b8822d8f5d4f8ff29923ce490d18779a8d448096b252ae0e107268f215d59dc5742e23db21d141db51fae7be63a36252a344fd05f4ac0e122e89bfb5981a024d40a5b17f31f571e4
});
```

Note that although in the example we chose the encoding to be `hex` it could
also be `base64`, `ascii`, `utf-8`, `utf16le`/`ucs2` or `binary`.

You may notice that running [crypto.randomBytes][4] is slower than
[Math.random][1] but this is expected - the fastest algorithm isn't always the
safest. Crypto's `randomBytes` is also safer to implement; an example of this,
is the fact that you *CANNOT* seed crypto/rand, the library uses OS-randomness
for this, preventing developer misuse.

If you're curious about how using `Math.random` can lead to problems, a great
article has been written about a problem in Chrome's V8 Engine related to its
use. You can read it [here][5].

So remember, if randomness is crucial (like in cryptography or token
generation), don't use `Math.random()`. Instead, use `crypto.randomBytes()`.

Finally, it's important to establish and use a policy and process for
cryptographic key management. For example, a specialist is in charge of the key
generation and management, as well as responsible for deploying them to
production servers. So, if possible, consider a central key management system
with distributed execution.

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
[2]: https://gist.github.com/joepie91/7105003c3b26e65efcea63f3db82dfba
[3]: https://www.npmjs.com/package/uuid
[4]: https://nodejs.org/dist/latest-v6.x/docs/api/crypto.html#crypto_crypto_randombytes_size_callback
[5]: https://medium.com/@betable/tifu-by-using-math-random-f1c308c4fd9d
