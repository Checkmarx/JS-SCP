Numbers
=======

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
`String` argument, returning respectively an _integer_ and _float_ numbers.

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
returns its "integer part" (leading white spaces are removed), but if the first
character after removing any leading white spaces is other than `-`
(HYPHEN-MINUS), `+` (PLUS SIGN) or a digit, we will get `NaN` -
**N**ot-**a**-**N**umber

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
User's age upper limit was omitted for code sample briefness.

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

Even at this point we can't be sure that what was entered was a decimal integer
number: providing `32,5` will end up being parsed as `32`

```javascript
const userInputAge = '32,5';
const userAge = Number.parseInt(userInputAge, 10);

if (!Number.isInteger(userAge) || userAge <= 0) {
    throw new Error('invalid age');
}

console.log('You are %d years old', userAge);
// You are 32 years old
```

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
  +octalString;                 // 10
  ```

## Safe Integer

ECMAScript 2015 (6th Edition) introduces the concept of "Safe Integer": an
integer that "_can be exactly represented as an IEEE-754 double precision
number and whose IEEE-754 representation cannot be the result of rounding any
other integer to fit the IEEE-754 representation_" ([source][6]).

Why is this important? Let's have a look at some simple _integer_ arithmetic

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

Yes, this is an infinite loop. `MIN` is not a "Safe Integer", in fact the
last "Safe Integer" is exactly `MIN - 1` which you can get from
`Number.MAX_SAFE_INTEGER` (2⁵³-1): although there's no representation for
`MIN` and we're doing an _integer_ operation, JavaScript won't throw any error.

You may expect that `Number.MAX_SAFE_INTEGER` is the highest number that
JavaScript can handle, but no, `Number.MAX_VALUE` is the highest one:

```javascript
Number.MAX_VALUE > Number.MAX_SAFE_INTEGER; // true
Number.MAX_VALUE;                           // 1.7976931348623157e+308
```

## Division by Zero

Don't worry, you'll never get close to a "division by zero" error. Instead you
will get... _infinity_ as per the [IEEE 754-2008 standard][3]

```javascript
1/0;  // Infinity
-1/0; // -Infinity
```

## Precision

This is not a JavaScript only problem. In fact this is an issue you will find
in most programming languages as it is a limitation of the already mentioned
[IEEE 754 specification][3]. It is better to be aware of this as rounding
errors may lead rockets to miss theirs targets[^3]

```javascript
0.1+0.2;    // 0.30000000000000004
```

## Converting

### to `boolean`

JavaScript has a native Boolean type, consisting of primitive `true` and
`false` values, nevertheless some `Number`s evaluate to `false` and others to
`true`

```javascript
-1? true : false;           // true
1? true : false;            // true
Infinity? true : false;     // true
-Infinity? true : false;    // true
0? true : false;            // false
-0? true : false;           // false
NaN? true : false;          // false
```

The conversion can be done using double logical NOT operator `!`

```javascript
const number = -1;

if (!!number === true) {
    console.log('true');
} else {
    console.log('false');
}
```

### to `String`

Type coercion is commonly used to convert a number into string and it does the
trick when you're using a decimal base

```javascript
''+1;             // '1'
''+0.1;           // '0.1'
''+Math.pow(4,3); // '64'
  ```

but if you're using a non-decimal base like Octal or Hexadecimal, the result
may not be what you're expecting as you will get a string representation of
number's decimal representation

```javascript
''+012;   // '10'
''+0xA;   // '10'
```

To avoid mistakes, use always the same pattern when getting a textual
representation of a `Number`: use the [Number.toString()][10] method,
specifying the _radix_ (if not present or _undefined_, the `Number` 10 is used
by default)

```javascript
const n1 = 10;
const n2 = 012;
const n3 = 0xA;

n1.toString();          // '10'
n1.toString(10);        // '10'
n1.toString(undefined); // '10';
n2.toString(8);         // '12';
n3.toString(16);        // 'a';
```

This is also the closer you have to decimal bases conversion as you can get an
octal representation from a decimal _integer_ or from an hexadecimal value

```javascript
const decimalInt = 10;
const hexValue = 0xA;

console.log(decimalInt.toString(8));    // '12';
console.log(hexValue.toString(8));      // '12';
```

## Conclusion

As said before, over HTTP what you get server-side is always a String.
Because of that, before converting `String` to `Number`

1. _always validate the input against a "white" list of allowed characters_
  (e.g. for decimal integers `^(0|[1-9][0-9]*)$`;
2. then _validate for expected data types_ (e.g. parsing `String` to `Number`);
3. and last, _validate data range_;

You can read more about [How numbers are encoded in JavaScript][14] by
Dr. Axel Rauschmayer at 2ality.com.

[^1]: The implementation is shared with globals `parseInt()` and `parseFloat()` functions: `parseInt === Number.parseInt && parseFloat === Number.parseFloat // true`)
[^2]: "_In computing, a locale is a set of parameters that defines the user's language, region and any special variant preferences that the user wants to see in their user interface._" ([source][13])
[^3]: https://en.wikipedia.org/wiki/MIM-104_Patriot#Failure_at_Dhahran

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
[13]: https://en.wikipedia.org/wiki/Locale_(computer_software)
[14]: http://2ality.com/2012/04/number-encoding.html
