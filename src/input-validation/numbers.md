# Numbers

[JavaScript specification][1] makes a clear differentiation between three
`Number` related terms/concepts:

* [Number value][2]: the number representation "_corresponding to a
  double-precision 64-bit binary format [IEEE 754-2008][3] value_";
* [Number type][4]: "_set of all possible `Number` values including the special
  "Not-a-Number (NaN) value, positive infinity and negative infinity_";
* [Number object][5]: "_member of the Object type that is an instance of the
  standard built-in Number constructor_".

We will use `Number` interchangeably to refer either the representation, value
and the `Number` constructor.

An introductory note regarding JavaScript internals related to `Number`s:

* All numbers are internally handled and stored as double-precision 64-bit
  binary format, following the IEEE 754-2008 specification (yes, in JavaScript
  every number is a floating numbers);
* `NaN`, `+Infinity` (the same than `Infinity`) and `-Infinity` are all
  `Number`s
  ```javascript
  typeof NaN;               // 'number'
  typeof +Infinity;         // 'number'
  +Infinity === Infinity;   // true
  typeof -Infinity;         // 'number'
  ```
* JavaScript has both `+0` (or `0`) and `-0`: but they are the same
  ```javascript
  0 === -0  // true
  ```

## From `String` to `Number`

Although user input that reaches the server over HTTP is handled as `String`,
internally, applications may expect numeric values (e.g. user's age).

Looking at [`Number` constructor properties][8] we find the
[`Number.parseInt`][9] and [`Number.parseFloat`][10][^1] methods: both expect a
`String` argument, returning respectively an _integer_ and _float_ number.

`Number.parseInt` also accepts a _radix_ which when not specified or
_undefined_ defaults to 10 (decimal base) except when the `String` argument
begins with `0x` or `0X` in which case a _radix_ of 16 (hexadecimal base) is
assumed.

How the expected `String` argument is parsed is fully detailed in the
specification but issues may arise here. Let's see how it works!

### Requesting user's age

We're expecting an _integer_, but over HTTP the value will arrive as `String`.
Let's parse it

```javascript
const userInputAge = '32';
const userAge = Number.parseInt(userInputAge);

console.log('User is %d years old', userAge);
// User is 32 years old
```

That's OK, but what if user's input looks like `32 years old`

```javascript
const userInputAge = '32 years old';
const userAge = Number.parseInt(userInputAge);

console.log('User is %d years old', userAge);
// User is 32 years old
```

Despite of the fact that input string is alphanumeric, `Number.parseInt`
returns its "integer part" (leading white spaces are removed). If the first non
white space character is other than `-` (`HYPHEN-MINUS), `+` (`PLUS SIGN`) // TODO

```javascript
const userInputAge = 'thirty 2';
const userAge = Number.parseInt(userInputAge);

console.log('User is %d years old', userAge);
// User is NaN years old
```

There were no parsing errors, but we know that we are not ready to go with
user's age. Testing whether the `Number.parseInt(userInputAge)` result is a
`Number` won't suffice as `NaN` is itself a `Number`.

```javascript
const userInputAge = 'thrity 2';
const userAge = Number.parseInt(userInputAge);

if (typeof userAge !== 'number') {
    throw new Error('invalid age');
}

console.log('User is %d years old', userAge);
// User is NaN years old
```

Let's enforce that parsed user's age is in fact an _integer_ bigger than zero

```javascript
const userIntputAge = 'thirty 2';
const userAge = Number.parseInt(userInputAge);

if (!Number.isInteger(userAge) || userAge <= 0) {
    throw new Error('invalid age');
}

console.log('User is %d years old');
```

**Note**: [`Number.isInteger`][11] returns `false` for `Infinity`/`-Infinity`.
User's age upper limit was omitted for code sample biefness.

Now that we've received a validation error it may look safe but...
What if user provides his age using hexadecimal base (at least he will look
younger)?

```javascript
const userInputAge = '0x20';
const userAge = Number.parseInt(userInputAge);

if (!Number.isInteger(userAge) || userAge <= 0) {
    throw new Error('invalid age');
}

console.log('You are %d years old', userAge);
// User is 32 years old
```

Surprisingly or not, `0x20` did validate as integer. Why?
In fact, `Number.parseInt('0x20');` returns the _integer_ number `32`:
although the '0x20' string is parsed as hexadecimal due to the '0x' prefix
(`0X` is also valid), internally all numbers are stored as decimal.

As said before `Number.parseInt` accepts a _radix_ as second argument. So, to
enforce `userInputAge` to be given as a decimal number, we just have to provide
a _radix_ equal to `10`

```javascript
const userInputAge = '0x20';
const userAge = Number.parseInt(userInputAge, 10);

if (!Number.isInteger(userAge) || userAge <= 0) {
    throw new Error('invalid age');
}

console.log('You are %d years old', userAge);
```

And as expected we have the validation error.

### Requesting weight

Weight is a good example of a _float_ number, so let's ask users to input
theirs.


```javascript
const userInputWeight = '80.5';
const userWeight = Number.parseFloat(userInputWeight);

console.log('User\'s weight is %d Kg', userWeight);
// User's weight is 80.5 Kg
```

Depending on user's locale[^2], one should use `,` (comma) as a decimal
separator. What difference does it make?

```javascript
const userInputWeight = '80,5';
const userWeight = Number.parseFloat(userInputWeight);

console.log('User\'s weight is %d Kg', userWeight);
// User's weight is 80 Kg
```

Exactly `0.5` Kg (`Number.parseFloat` returns `80`): per the specification,
`Number.parseFloat` uses `.` (dot) as decimal separator.

## Type Coercion

Quite often `String` to `Number` conversion is done using the
[Unary `+` Operator][12], forcing _type coercion_

```javascript
+'';    // 0
+'0';   // 0
+'-0';  // -0
+'NaN'; // NaN
+' 1';  // 1
+'-1';  // -1
+'0.1'; // 0.1
```

but this may not lead to the expected results

* type coercion and `Number.parseFloat` inconsistency
  ```javascript
  const userInput = '80,5';
  
  parseFloat(userInput);  // 80
  +userInput;             // NaN
  ```
* octal representation
  ```javascript
  const octal = 012;
  const octalString = '012';
  
  +octal;         // 10
  +octalString;   // 12
  ```

  To get the expected result, `octalString` should be equal to `0o12`

  ```javascript
  const octalString = '0o12';
  +octalString;   // 10
  ```

## Safe Integer

ECMAScript 2015 (6th Edition) introduces the concept of "Safe Integer": an
integer that "_can be exactly represented as an IEEE-754 double precision
number and whose IEEE-754 representation cannot be the result of rounding any
other integer to fit the IEEE-754 representation_" ([source][6]).

Why is this important? Let's have a look at some simple _integer_ arithmethics

```javascript
const N = 9007199254740992;

N + 1;  // 9007199254740992
N + 2;  // 9007199254740994
```

Is it `9007199254740992` safe?

```javascript
Number.isSafeInteger(9007199254740992); // false
```

Again, why is this so important?

```javascript
const MIN = 9007199254740992;
const MAX = 9007199254740994;

for (let i = MIN; i < MAX; i++) {
    console.log(i);
}
```

Yes, this is an infinte loop. `MIN` is not a "Safe Integer", in fact the
last "Safe Integer" is exactly `MIN - 1` which you can get from
`Number.MAX_SAFE_INTEGER` (2⁵³-1): although there's no representation for
`MIN` and we're doing an _integer_ operation, JavaScript won't throw any error.

You may expect that `Number.MAX_SAFE_INTEGER` is the highest number that
JavaScript can handle, but no, `Number.MAX_VALUE` is the highest one and

```javascript
Number.MAX_VALUE > Number.MAX_SAFE_INTEGER; // true
Number.MAX_VALUE;                           // 1.7976931348623157e+308
```

## Division by Zero

You can get `Infinity`/`-Infinit` dividing by `0` (zero):
```javascript
1/0;  // Infinity
-1/0; // -Infinity
```

## Precision

```javascript
0.1+0.2;    // 0.30000000000000004
```

To know how to convert Numbers between bases, please refer to // TODO

* [`Boolean`][7] JavaScript has a native Boolean type, consisting of primitive
  `true` and `false` values.

  In JavaScript any non empty string evaluates to `true`, what may not suit
  your expectations specially when you're handling form data coming from client
  side e.g.

  ```javascript
  ''? true : false;      // false
  '0'? true : false;     // true
  'false'? true : false; // true
  '1'? true : false;     // true
  'true'? true : false;  // true
  ```

  To do it safely, strings should be tested for equality using the
  [strict equality operator][8] `===` e.g.

  ```javascript
  const YES = 'yes';
  const NO = 'no';

  const value = 'yes';
  if (value === YES) {
    // do stuff
  }
  ```

### Number conversion to:

* [String][9] type coercion is commonly used to convert a number into string
  and it does the trick when you're using a decimal base
  ```javascript
  ''+1;             // '1'
  ''+0.1;           // '0.1'
  ''+Math.pow(4,3); // '64'
  ```

  but if you're using a non-decimal base like Octal or Hexadecimal, the result
  may not be what you're expecting

  ```javascript
  ''+012;   // '10'
  ''+0xA;   // '10'
  ```

  In this case, you should use the [Number.toString()][10] method, specifying
  the _radix_ (if not present or _undefined_, the Number 10 is used by default)

  ```javascript
  Number(012).toString(8);  // '12'
  Number(0xA).toString(16); // 'a';
  ```

* [Boolean][7]: any non-zero `Number` evaluates to `true` (no matter what the
  Number base is) and zero evaluates to `false`.

  The easiest way to get a boolean value from a Number is using the
  [strict equality operator][8] `===`

  ```javascript
  const v1 = 2;
  const v2 = 02;
  const v3 = x2;
  const v4 = 0;
  const v5 = 00;
  const v6 = 0x0;

  v1 !== 0; // true
  v2 !== 0; // true
  v3 !== 0; // true
  v4 !== 0; // false
  v5 !== 0; // false
  v6 !== 0; // false
  ```

* other bases: it worth mentioning that JavaScript stores 
  * Octal to Decimal
    ```javascript
    const octalNumber = 0777;
    const decimalNumber = octalNumber.toNumber(10); // 255
    ```

  * Hexadecimal to Decimal
    ```javascript
    const hexNumber = 0xa;
    const decimalNumber = parseInt(hexNumber, 16); // 10
    ```

You can read more about [How numbers are encoded in JavaScript][] at
http://2ality.com/2012/04/number-encoding.html


[^1]: The implementation is shared with globals `parseInt()` and `parseFloat()` functions: `parseInt === Number.parseInt && parseFloat === Number.parseFloat // true`)
[^2]: "_In computing, a locale is a set of parameters that defines the user's language, region and any special variant preferences that the user wants to see in their user interface._" ([source][locale])

[1]: http://www.ecma-international.org/ecma-262/6.0/
[2]: http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-number-value
[3]: https://standards.ieee.org/findstds/standard/754-2008.html
[4]: http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-number-type
[5]: http://www.ecma-international.org/ecma-262/6.0/#sec-number-object
[6]: https://developer.mozilla.org/pt-PT/docs/Web/JavaScript/Reference/Global_Objects/Number/isSafeInteger
[7]: http://www.ecma-international.org/ecma-262/6.0/#sec-number.issafeinteger
[8]: http://www.ecma-international.org/ecma-262/6.0/#sec-properties-of-the-number-constructor
[9]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-number.parseint
[10]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-number.parsefloat
[11]: http://www.ecma-international.org/ecma-262/6.0/#sec-number.isinteger
[12]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-unary-plus-operator
[locale]: https://en.wikipedia.org/wiki/Locale_(computer_software)



[1]: https://nodejs.org/en/
[2]: https://www.npmjs.com/

[5]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-number-objects

[7]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-terms-and-definitions-boolean-type
[8]: http://www.ecma-international.org/ecma-262/6.0/#sec-strict-equality-comparison 
[9]: http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-string-type
[10]: http://www.ecma-international.org/ecma-262/6.0/#sec-number.prototype.tostring
