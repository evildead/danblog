---
title: Node.js Unit Tests - Use Mocha and Chai to create a Roman Number Library Pt.1
date: 2017-12-03 13:47:56
tags:
---

[Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) is a software development process that relies on the repetition of a very short development cycle: Requirements are turned into very specific test cases, then the software is improved to pass the new tests.

1. Add a test
2. Run all tests and see if the new test fails
3. Write the code
4. Run tests
5. Refactor code
6. Repeat

Thereby the cornerstone of this process is represented by the [Unit Test](https://en.wikipedia.org/wiki/Unit_testing).


## Mocha and Chai

* In Node.js a wide used test framework is [Mocha](https://mochajs.org/): it makes synchronous and asynchronous testing simple and straightforward, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases.
* The assertion library we will use is [Chai](http://chaijs.com/): it has several interfaces that allow the developer to choose the most comfortable (_should_, _expect_, _assert_).


## Roman Numbers

A [Roman number](https://www.math.nmsu.edu/~pmorandi/math111f01/RomanNumerals.html) represents an integer (Hindu-Arabic numerals) using a small set of symbols:

Roman Numeral | Hindu-Arabic Equivalent
------------- | -----------------------
I | 1
V | 5
X | 10
L | 50
C | 100
D | 500
M | 1000

There are a few rules for writing numbers with Roman numerals.

* Repeating a numeral up to three times represents addition of the number. For example, III represents 1 + 1 + 1 = 3. Only I, X, C, and M can be repeated; V, L, and D cannot be, and there is no need to do so.
* Writing numerals that decrease from left to right represents addition of the numbers. For example, LX represents 50 + 10 = 60 and XVI represents 10 + 5 + 1 = 16.
* To write a number that otherwise would take repeating of a numeral four or more times, there is a subtraction rule. Writing a smaller numeral to the left of a larger numeral represents subtraction. For example, IV represents 5 - 1 = 4 and IX represents 10 - 1 = 9. To avoid ambiguity, the only pairs of numerals that use this subtraction rule are:

Roman Numeral | Hindu-Arabic Equivalent
------------- | -----------------------
IV | 4 = 5 - 1
IX | 9 = 10 - 1
XL | 40 = 50 - 10
XC | 90 = 100 - 10
CD | 400 = 500 - 100
CM | 900 = 1000 - 100

Following these rules, every Hindu-Arabic number between 1 and 3999 (MMMCMXCIX) can be represented as a Roman numeral.


## Requirements

The Roman Number Library to build will have to take a value in input (a Roman Number, or an Hindu-Arabic number), and will have to return an object containing two methods:
* toInt()
* toString()

```js
// Example
let romanNumber1 = new RomanNumber('MMMCMXCIX');
let romanNumber2 = new RomanNumber(1);

console.log(romanNumber1.toInt()); // => 3999
console.log(romanNumber1.toString()); // => 'MMMCMXCIX'
console.log(romanNumber2.toInt()); // => 1
console.log(romanNumber2.toString()); // => 'I'
```

* If the value passed is null or empty, it should throw a 'value required' exception error (e.g. 'throw new
Error('value required');' ).
* If the value passed is outside of 1 to 3999, it should throw an 'invalid range' exception error.
* If the value passed is invalid, it should throw an 'invalid value' exception error.
* If the library is called as a function (i.e. without the new prefix), it must still pass back a new object.

## Kick-off

Let's kick our application off!