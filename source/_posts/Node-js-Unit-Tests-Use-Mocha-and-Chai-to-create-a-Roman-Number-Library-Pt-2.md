---
title: Node.js Unit Tests - Use Mocha and Chai to create a Roman Number Library Pt.2
date: 2017-12-04 00:49:00
tags:
---

In [Pt.1](http://danblog.danilocarrabino.net/2017/12/03/Node-js-Unit-Tests-Use-Mocha-and-Chai-to-create-a-Roman-Number-Library-Pt-1) we introduced the concepts of [Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development), [Unit Test](https://en.wikipedia.org/wiki/Unit_testing), [Roman number](https://www.math.nmsu.edu/~pmorandi/math111f01/RomanNumerals.html), and we listed the requirements to build our Roman Number library.

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

Test the correct result by yourself, by typing:
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

## Requirement: invalid range value

The test cases for this requirement, will be integer values not in [1,3999]
```js
it('The constructor invoked with 0 should return an exception: invalid range', () => {
    expect(() => RomanNumber(0)).to.throw(Error, /^invalid range$/);
});

it('The constructor invoked with 10000 should return an exception: invalid range', () => {
    expect(() => RomanNumber(10000)).to.throw(Error, /^invalid range$/);
});
```

Even these test cases use _Chai expect interface_ and the method _throw_.

As for the previous requirement we'll skip the test part here and we'll go to add this piece of code to our library constructor:
```js
    if((val < 1) || (val > 3999)) {
        throw new Error('invalid range');
    }
```
As expected: all tests passed.
![Test for not in range values: passed](/content/images/2017/12/04/06-Test-not-in-range-values_No-Error.png)

## Invalid values

The next two tests will be about checking two types of invalid values:
1. A String containing non Roman symbols: 'error'
2. A String containing Roman symbols and Hindu-Arabic symbols: 'CD1X'

For the both of these cases, our roman number library will have to throw an Error exception with message: 'Invalid value'

Here are the test cases to add to testRomanNumber.js
```js
    it('The constructor invoked with "error" should return an exception: invalid value', () => {
        expect(() => RomanNumber('error')).to.throw(Error, /^invalid value$/);
    });

    it('The constructor invoked with "CD1X" should return an exception: invalid value', () => {
        expect(() => RomanNumber('CD1X')).to.throw(Error, /^invalid value$/);
    });
```

In order to pass these tests, we have to check that any string passed in input must contain only Roman symbols.

To add this little improvement a minor refactor must be performed on our constructor. Replace:
```js
    if((val < 1) || (val > 3999)) {
        throw new Error('invalid range');
    }
```
with:
```js
    if(Number.isInteger(val)) {
        if((val < 1) || (val > 3999)) {
            throw new Error('invalid range');
        }
    }
    else if((typeof(val) === "string") || (val instanceof String)) {
        let romanSymbols = ['M', 'D', 'C', 'L', 'X', 'V', 'I'];
        
        for(let i = 0; i < val.length; i++) {
            if(romanSymbols.indexOf(val[i].toUpperCase()) < 0) {
                throw new Error('invalid value');
            }
        }
    }
    else {
        throw new Error('invalid value');
    }
```

If the input is a string, we make sure that any of its characters belong to our romanSymbols array.

Test the correct result by yourself, by typing:
`$ npm test`

# Allow Hindu-Arabic numbers as String

Because we want to allow anyone using our library to pass an integer as a string (i.e. '1473' => 1473), we'll add this quick test:
```js
    it('The constructor invoked with "1473" should not return an exception', () => {
        expect(() => RomanNumber('1473')).to.not.throw();
    });
```

Let's have a look at what happens if we run the test now:
![Test for Integer string: error](/content/images/2017/12/04/07-Test-Integer-string-Error.png)

The exception is thrown because so far we assumed that any string was supposed to be in Roman symbols only.

For this reason we have must handle also this case, when a string contains Hindu-Arabic symbols (integer).

We will transform the constructor in this way:

```js
    if( (Number.isInteger(val)) ||
        ((typeof(val) === "string" || val instanceof String) && Number.isInteger(parseInt(val))) ) {
        
        let intVal = parseInt(val);
        if((intVal < 1) || (intVal > 3999)) {
            throw new Error('invalid range');
        }
    }
    else if((typeof(val) === "string") || (val instanceof String)) {
        let romanSymbols = ['M', 'D', 'C', 'L', 'X', 'V', 'I'];
        
        for(let i = 0; i < val.length; i++) {
            if(romanSymbols.indexOf(val[i].toUpperCase()) < 0) {
                throw new Error('invalid value');
            }
        }
    }
    else {
        throw new Error('invalid value');
    }
```

In this way we can treat an integer string like an integer.

Let's run the test:
`$ npm test`

All tests passed
![Test for Integer string: error](/content/images/2017/12/04/08-Test-Integer-string-No-Error.png)

## Even more to come

In [Pt.3](http://danblog.danilocarrabino.net/2017/12/11/Node-js-Unit-Tests-Use-Mocha-and-Chai-to-create-a-Roman-Number-Library-Pt-3) we will start converting numbers (from Hindu-Arabic to Roman numbers).

Check out the [Roman Library Repository](https://github.com/evildead/RomanNumber) if you cannot wait for the last chapter of this blog
