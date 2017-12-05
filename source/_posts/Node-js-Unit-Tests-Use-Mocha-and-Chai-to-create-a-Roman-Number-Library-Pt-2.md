---
title: Node.js Unit Tests - Use Mocha and Chai to create a Roman Number Library Pt.2
date: 2017-12-04 00:49:00
tags:
---

In [Pt.1](http://localhost:4000/2017/12/03/Node-js-Unit-Tests-Use-Mocha-and-Chai-to-create-a-Roman-Number-Library-Pt-1) we introduced the concepts of [Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development), [Unit Test](https://en.wikipedia.org/wiki/Unit_testing), [Roman number](https://www.math.nmsu.edu/~pmorandi/math111f01/RomanNumerals.html), and we listed the requirements to build our Roman Number library.

## The second requirement: invoke library as a function (without new)

Let's create the test for this requirement by adding this code to our testRomanNumber.js:

```js
...
it('The constructor should be callable without new', () => {
    let romanNumber = RomanNumber(1);
    assert.isObject(romanNumber);
});
...
```

If we run this new test by typing:
`$ npm test`

we'll see this error:
![Test Invoke without new: error](/content/images/2017/12/04/02-Test-InvokeAsFunction_Error.png)

In order to pass this test we may use the property [_new.target_](https://developer.mozilla.org/it/docs/Web/JavaScript/Reference/Operators/new.target) added in ECMAScript 2015 (ES6)

The _new.target_ property lets you detect whether a function or constructor was called using the _new_ operator. In constructors and functions instantiated with the _new_ operator, _new.target_ returns a reference to the constructor or function. In normal function calls, _new.target_ is undefined.

Let's add this piece of code inside RomanNumber constructor
```js
    if(!new.target) {
        return new RomanNumber(val);
    }
```
Then we run the test
`$ npm test`

And, as if by magic, the error disappears:
![Test Invoke without new: passed](/content/images/2017/12/04/03-Test-InvokeAsFunction_No-Error.png)

## The third requirement: add toInt and toString() methods

Add two test cases for this requirement, inside the _describe_ "Check exceptions"
```js
it('The object should contain method toInt', () => {
    let romanNumber = RomanNumber(1);
    assert.isFunction(romanNumber.toInt);
});

it('The object should contain method toString', () => {
    let romanNumber = RomanNumber(1);
    assert.isFunction(romanNumber.toString);
});
```
These test cases are bases on the _isFunction_ method of _Chai assert interface_

If we run the test with:
`$ npm test`

we'll see that only the toInt() method does not exist:
![Test presence of methods toInt and toString: error](/content/images/2017/12/04/04-Test-toInt-Method-DoesNotExist.png)

This is because in Javascript every object has a [toString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString) method

So, in order to pass the test, we just have to add this code to RomanNumber.js:
```js
/**
 * toInt
 */
romanNumber.prototype.toInt = function toInt() {
    return 1;
};
```

Test the result by yourself, by typing:
`$ npm test`

## Requirement: value passed not null or empty

We have to create three test cases to check _null_ value, _empty_ value, and _no_ value.

```js
it('The constructor invoked with null should return an exception: value required', () => {
    expect(() => RomanNumber(null)).to.throw(Error, /^value required$/);
});

it('The constructor invoked with empty string should return an exception: value required', () => {
    expect(() => RomanNumber('')).to.throw(Error, /^value required$/);
});

it('The constructor invoked with no value should return an exception: value required', () => {
    expect(() => RomanNumber()).to.throw(Error, /^value required$/);
});
```

These test cases use _Chai expect interface_ and the method _throw_.

In this way we can check that, given a particular value, the library constructor throws an Error with a particular message.

We already know that no exception will be thrown, so let's step forward and add the code to our library constructor:
```js
    if((typeof(val) === 'undefined') || (val === null) || (val === '')) {
        throw new Error('value required');
    }
```

Let's run our test
`$ npm test`

and... All tests passed! Very good job indeed so far!

![Test for null or empty values: passed](/content/images/2017/12/04/05-Test-for-null-or-empty-values_No-Error.png)
