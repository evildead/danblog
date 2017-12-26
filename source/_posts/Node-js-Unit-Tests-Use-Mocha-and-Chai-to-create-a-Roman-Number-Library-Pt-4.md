---
title: Node.js Unit Tests - Use Mocha and Chai to create a Roman Number Library Pt.4
date: 2017-12-24 15:06:07
tags:
---

In [Pt.3](http://danblog.danilocarrabino.net/2017/12/11/Node-js-Unit-Tests-Use-Mocha-and-Chai-to-create-a-Roman-Number-Library-Pt-3) we implemented the conversion from _Hindu-Arabic_ numbers to _Roman_ numbers.
In this final chapter, we will complete our library by adding the conversion from _Roman_ numbers to _Hindu-Arabic_ numbers.

## Convert the first five Roman numbers

We will set up the test cases the way we did for the other conversions: we will check both the method toInt() and the method toString().
We will test the first five Roman numbers: 'I'. 'II'. 'III'. 'IV' and 'V':
```js
        it('The Roman number "I" should be equal to the arabic number 1', () => {
            let romannum = RomanNumber('I');
            romannum.toString().should.equal('I');
            romannum.toInt().should.equal(1);
        });

        it('The Roman number "II" should be equal to the arabic number 2', () => {
            let romannum = RomanNumber('II');
            romannum.toString().should.equal('II');
            romannum.toInt().should.equal(2);
        });

        it('The Roman number "III" should be equal to the arabic number 3', () => {
            let romannum = RomanNumber('III');
            romannum.toString().should.equal('III');
            romannum.toInt().should.equal(3);
        });

        it('The Roman number "IV" should be equal to the arabic number 4', () => {
            let romannum = RomanNumber('IV');
            romannum.toString().should.equal('IV');
            romannum.toInt().should.equal(4);
        });

        it('The Roman number "V" should be equal to the arabic number 5', () => {
            let romannum = RomanNumber('V');
            romannum.toString().should.equal('V');
            romannum.toInt().should.equal(5);
        });
```

To pass these very first conversion tests, we just need to update the library constructor in this way:
```js
    ...
    if(RomanNumber.isValidInt(val)) {
        this.intVal = parseInt(val);
        this.strVal = RomanNumber.intToRoman(this.intVal);
    }
    else if(RomanNumber.checkOnlyRomanSymbols(val)) {
        this.strVal = val.toUpperCase();
        
        let patterns = new Map();
        patterns.set('I', 1);
        patterns.set('II', 2);
        patterns.set('III', 3);
        patterns.set('IV', 4);
        patterns.set('V', 5);
        
        this.intVal = patterns.get(this.strVal);
    }
    else {
        throw new Error('invalid value');
    }
    ...
```

In the code block relative to _checkOnlyRomanSymbols_ we used a [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) structure, where we matched Roman numbers (as keys) to Hindu-Arabic numbers (as values).

All the tests passed correctly as shown below:
![Test first five Roman numbers: passed](/content/images/2017/12/24/12-Test_FirstFiveRomanNum_OK.png)

## Convert Roman numbers composed of _simple_ patterns

The next tests we are going to perform, are about 'simple' Roman numbers: for _simple_, we mean those numbers represented by the following patterns:
```js
/**
 *  Patterns:
 * 
 *              |   1   |   2   |   3   |   4   |   5   |   6   |   7   |   8   |   9   |
 *  Ones:       |   I   |   II  |  III  |   IV  |   V   |   VI  |  VII  |  VIII |   IX  |
 * 
 *              |   10  |   20  |   30  |   40  |   50  |   60  |   70  |   80  |   90  |
 *  Tens:       |   X   |   XX  |  XXX  |   XL  |   L   |   LX  |  LXX  |  LXXX |   XC  |
 * 
 *              |  100  |  200  |  300  |  400  |  500  |  600  |  700  |  800  |  900  |
 *  Hundreds:   |   C   |   CC  |  CCC  |   CD  |   D   |   DC  |  DCC  |  DCCC |   CM  |
 * 
 *              |  1000 |  2000 |  3000 |   -   |   -   |   -   |   -   |   -   |   -   |
 *  Thousands:  |   M   |   MM  |  MMM  |   -   |   -   |   -   |   -   |   -   |   -   |
 */ 
```

In particular we will test: 'XXX', 'XL', 'XC', 'CC', 'CD', 'DCCC' and 'MMM'.
```js
        it('The Roman number "XXX" should be equal to the arabic number 30', () => {
            let romannum = RomanNumber('XXX');
            romannum.toString().should.equal('XXX');
            romannum.toInt().should.equal(30);
        });

        it('The Roman number "XL" should be equal to the arabic number 40', () => {
            let romannum = RomanNumber('XL');
            romannum.toString().should.equal('XL');
            romannum.toInt().should.equal(40);
        });

        it('The Roman number "XC" should be equal to the arabic number 90', () => {
            let romannum = RomanNumber('XC');
            romannum.toString().should.equal('XC');
            romannum.toInt().should.equal(90);
        });

        it('The Roman number "CC" should be equal to the arabic number 200', () => {
            let romannum = RomanNumber('CC');
            romannum.toString().should.equal('CC');
            romannum.toInt().should.equal(200);
        });

        it('The Roman number "CD" should be equal to the arabic number 400', () => {
            let romannum = RomanNumber('CD');
            romannum.toString().should.equal('CD');
            romannum.toInt().should.equal(400);
        });

        it('The Roman number "DCCC" should be equal to the arabic number 800', () => {
            let romannum = RomanNumber('DCCC');
            romannum.toString().should.equal('DCCC');
            romannum.toInt().should.equal(800);
        });

        it('The Roman number "MMM" should be equal to the arabic number 3000', () => {
            let romannum = RomanNumber('MMM');
            romannum.toString().should.equal('MMM');
            romannum.toInt().should.equal(3000);
        });
```

In order to pass the previous tests, we first need to update our library constructor to its final value:
```js
    if(RomanNumber.isValidInt(val)) {
        this.intVal = parseInt(val);
        this.strVal = RomanNumber.intToRoman(this.intVal);
    }
    else if(RomanNumber.checkOnlyRomanSymbols(val)) {
        this.strVal = val.toUpperCase();
        this.intVal = RomanNumber.romanToInt(this.strVal);
    }
    else {
        throw new Error('invalid value');
    }
```

Then we must add the final static method, named _romanToInt_:
```js
romanNumber.romanToInt = function romanToInt(val) {
    let patternsArray = [
        ["I", 1],    ["II", 2],    ["III", 3],   ["IV", 4],   ["V", 5],   ["VI", 6],   ["VII", 7],   ["VIII", 8],   ["IX", 9],    // Ones
        ["X", 10],   ["XX", 20],   ["XXX", 30],  ["XL", 40],  ["L", 50],  ["LX", 60],  ["LXX", 70],  ["LXXX", 80],  ["XC", 90],   // Tens
        ["C", 100],  ["CC", 200],  ["CCC", 300], ["CD", 400], ["D", 500], ["DC", 600], ["DCC", 700], ["DCCC", 800], ["CM", 900],  // Hundreds
        ["M", 1000], ["MM", 2000], ["MMM", 3000]                                                                                  // Thousands
    ];
    let patterns = new Map(patternsArray);

    let finalInt = patterns.get(val);

    return finalInt;
};
```

In this method we implemented the patterns seen previously, by using a Map structure initialized with an array of mapping values.

Check the results by yourself, you won't be disappointed!

## Convert any valid Roman number

We're now ready to mix the patterns we've just set up in order to convert any valid Roman number.

This is the test-case list: "CDXXIX", "MCDLXXXII", "MCMLXXX", "MMMCMXCIX":
```js
        it('The Roman number "CDXXIX" should be equal to the arabic number 429', () => {
            let romannum = RomanNumber('CDXXIX');
            romannum.toString().should.equal('CDXXIX');
            romannum.toInt().should.equal(429);
        });

        it('The Roman number "MCDLXXXII" should be equal to the arabic number 1482', () => {
            let romannum = RomanNumber('MCDLXXXII');
            romannum.toString().should.equal('MCDLXXXII');
            romannum.toInt().should.equal(1482);
        });

        it('The Roman number "MCMLXXX" should be equal to the arabic number 1980', () => {
            let romannum = RomanNumber('MCMLXXX');
            romannum.toString().should.equal('MCMLXXX');
            romannum.toInt().should.equal(1980);
        });

        it('The Roman number "MMMCMXCIX" should be equal to the arabic number 3999', () => {
            let romannum = RomanNumber('MMMCMXCIX');
            romannum.toString().should.equal('MMMCMXCIX');
            romannum.toInt().should.equal(3999);
        });
```

Here is how we will have to update the _romanToInt_ method:
```js
romanNumber.romanToInt = function romanToInt(val) {
    let patternsArray = [
        ["I", 1],    ["II", 2],    ["III", 3],   ["IV", 4],   ["V", 5],   ["VI", 6],   ["VII", 7],   ["VIII", 8],   ["IX", 9],    // Ones
        ["X", 10],   ["XX", 20],   ["XXX", 30],  ["XL", 40],  ["L", 50],  ["LX", 60],  ["LXX", 70],  ["LXXX", 80],  ["XC", 90],   // Tens
        ["C", 100],  ["CC", 200],  ["CCC", 300], ["CD", 400], ["D", 500], ["DC", 600], ["DCC", 700], ["DCCC", 800], ["CM", 900],  // Hundreds
        ["M", 1000], ["MM", 2000], ["MMM", 3000]                                                                                  // Thousands
    ];
    let patterns = new Map(patternsArray);

    let i = 0;
    let finalInt = 0;

    // this loop is used to read the entire Roman string
    while(i < val.length) {
        let tmpPattern = '';
        // this loop is used to build the next pattern
        while(i < val.length) {
            if(tmpPattern.length == 0) {
                tmpPattern += val[i++];
            }
            else {
                let tmpPatternVal = patterns.get(tmpPattern + val[i]);
                if(typeof(tmpPatternVal) !== 'undefined') {
                    tmpPattern += val[i++];
                }
                else {
                    break;
                }
            }
        }

        if(tmpPattern.length > 0) {
            let tmpNum = patterns.get(tmpPattern);
            finalInt += tmpNum;
        }
    }

    return finalInt;
};
```

The first _while_ loop represents a way to read all the string's symbols, and the second one will be used to look for a single pattern. Any time a pattern is found, we'll add the relative integer value to the _finalInt_.

Let's finally check the tests:
`$ npm test`

...and here we go!
![Test any valid Roman number: passed](/content/images/2017/12/24/13-Test_AnyValidRomanNum_OK.png)

## Make our library more resilient

Now that our Roman numbers library is able to correctly convert any valid Roman number, we just want to make it better by adding the last checks for invalid cases:
* No more that three equal consecutive symbols can be found;
* A pattern relative to a certain set (e.g. _hundreds_) cannot be found after a higher or equal set (e.g. _thousands_ or _hundreds_): for example the invalid Roman number "MIM" contains the pattern "M", belonging to _thousands_, after "I", belonging to _ones_.

Here are the very last test cases to be added inside _Check exceptions_:
```js
        it('The constructor invoked with "IIII" should return an exception: invalid value', () => {
            expect(() => RomanNumber('IIII')).to.throw(Error, /^invalid value$/);
        });

        it('The constructor invoked with "MMMMCMXCIX" should return an exception: invalid value', () => {
            expect(() => RomanNumber('MMMMCMXCIX')).to.throw(Error, /^invalid value$/);
        });
```

and

```js
        it('The constructor invoked with "MMMMDMXCIX" should return an exception: invalid value', () => {
            expect(() => RomanNumber('MMMMDMXCIX')).to.throw(Error, /^invalid value$/);
        });

        it('The constructor invoked with "MMMDMXCIX" should return an exception: invalid value', () => {
            expect(() => RomanNumber('MMMDMXCIX')).to.throw(Error, /^invalid value$/);
        });

        it('The constructor invoked with "MIM" should return an exception: invalid value', () => {
            expect(() => RomanNumber('MIM')).to.throw(Error, /^invalid value$/);
        });

        it('The constructor invoked with "MDCVV" should return an exception: invalid value', () => {
            expect(() => RomanNumber('MDCVV')).to.throw(Error, /^invalid value$/);
        });
```

Even though all these new test cases should throw an _invalid value_ exception, at the current state they still don't:
![Test particular invalid Roman numbers: failed](/content/images/2017/12/24/14-Test_ParticularInvalidRomanValues_KO.png)

Update the library method _romanToInt_ to its final value and repeat the tests:
```js
/**
 * Static method romanToInt
 * @param {Roman String} val: must be a string composed of Roman symbols only
 */
romanNumber.romanToInt = function romanToInt(val) {
    let patternsArray = [
        ["I", 1],    ["II", 2],    ["III", 3],   ["IV", 4],   ["V", 5],   ["VI", 6],   ["VII", 7],   ["VIII", 8],   ["IX", 9],    // Ones
        ["X", 10],   ["XX", 20],   ["XXX", 30],  ["XL", 40],  ["L", 50],  ["LX", 60],  ["LXX", 70],  ["LXXX", 80],  ["XC", 90],   // Tens
        ["C", 100],  ["CC", 200],  ["CCC", 300], ["CD", 400], ["D", 500], ["DC", 600], ["DCC", 700], ["DCCC", 800], ["CM", 900],  // Hundreds
        ["M", 1000], ["MM", 2000], ["MMM", 3000]                                                                                  // Thousands
    ];
    let patterns = new Map(patternsArray);

    let i = 0;
    let finalInt = 0;
    let numConsecutives;
    let lastPatternFound = '';
    let lastPatternNumFound = 0;

    // this loop is used to read the entire Roman string
    while(i < val.length) {
        let tmpPattern = '';
        // this loop is used to build the next pattern and to check for more than 3 consecutive symbols 
        while(i < val.length) {
            // check for more than 3 consecutive symbols ////////////
            if((i > 0) && (val[i] == val[i-1])) {
                if(numConsecutives == 3) {
                    throw new Error('invalid value');
                }
                else {
                    numConsecutives++;
                }
            }
            else {
                numConsecutives = 1;
            }
            /////////////////////////////////////////////////////////

            if(tmpPattern.length == 0) {
                tmpPattern += val[i++];
            }
            else {
                let tmpPatternVal = patterns.get(tmpPattern + val[i]);
                if(typeof(tmpPatternVal) !== 'undefined') {
                    tmpPattern += val[i++];
                }
                else {
                    break;
                }
            }
        }

        if(tmpPattern.length > 0) {
            let tmpNum = patterns.get(tmpPattern);
            if(typeof(tmpNum) !== 'undefined') {
                if(lastPatternFound.length > 0) {
                    // new pattern must belong to a lower category than the previous one
                    // if a previous value found was 400, the following value must be from 1 to 99
                    if(tmpNum.toString().length >= lastPatternNumFound.toString().length) {
                        throw new Error('invalid value');
                    }
                }
                lastPatternFound = tmpPattern;
                lastPatternNumFound = tmpNum;
                finalInt += tmpNum;
            }
            else {
                throw new Error('invalid value');
            }
        }
    }

    return finalInt;
};
```

All tests passed:
![Test particular invalid Roman numbers: failed](/content/images/2017/12/24/15-Test_ParticularInvalidRomanValues_OK.png)

## Thank you very much :)

Thank you so much for your attention and your interest towards these blog posts :)

Check out the [Roman Library Repository](https://github.com/evildead/RomanNumber) to have a look at the final code.

Pay a visit to my web page: [MyPortfolio Danilo Carrabino](http://myportfolio.danilocarrabino.net/portfolios/danilo.carrabino)
