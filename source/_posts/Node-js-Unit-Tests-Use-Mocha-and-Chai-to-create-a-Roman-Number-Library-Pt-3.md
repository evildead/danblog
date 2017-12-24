---
title: Node.js Unit Tests - Use Mocha and Chai to create a Roman Number Library Pt.3
date: 2017-12-11 01:00:43
tags:
---

In [Pt.2](http://danblog.danilocarrabino.net/2017/12/04/Node-js-Unit-Tests-Use-Mocha-and-Chai-to-create-a-Roman-Number-Library-Pt-2) we mainly focused on exception handling.

In this third part we will start converting Hindu-Arabic numbers to Roman numbers.

## Convert numbers from 1 to 5

Let's start by creating a brand new describe section in our test file:
```js
    describe('Check values', () => {
    });
```

We will check the first 5 numbers: 1, 2, 3, 4 and 5.

For any number we want to check both the value returned by _toInt_ (Hindu-Arabic number) and the one returned by _toString_ (Roman Number).
For number 1, we will test also its string form '1'.
Let's insert our six test cases inside our new _describe section_:

```js
    describe('Check values', () => {
        it('The arabic number 1 should be equal to the Roman number "I"', () => {
            let romannum = RomanNumber(1);
            romannum.toString().should.equal('I');
            romannum.toInt().should.equal(1);
        });

        it('The arabic number "1" should be equal to the Roman number "I"', () => {
            let romannum = RomanNumber('1');
            romannum.toString().should.equal('I');
            romannum.toInt().should.equal(1);
        });

        it('The arabic number 2 should be equal to the Roman number "II"', () => {
            let romannum = RomanNumber(2);
            romannum.toString().should.equal('II');
            romannum.toInt().should.equal(2);
        });

        it('The arabic number 3 should be equal to the Roman number "III"', () => {
            let romannum = RomanNumber(3);
            romannum.toString().should.equal('III');
            romannum.toInt().should.equal(3);
        });

        it('The arabic number 4 should be equal to the Roman number "IV"', () => {
            let romannum = RomanNumber(4);
            romannum.toString().should.equal('IV');
            romannum.toInt().should.equal(4);
        });

        it('The arabic number 5 should be equal to the Roman number "V"', () => {
            let romannum = RomanNumber(5);
            romannum.toString().should.equal('V');
            romannum.toInt().should.equal(5);
        });
    });
```

We need a big refactor in our code. We need to add two static methods to check:
* if a Hindu-Arabic number is valid => _isValidInt_
* if a string contains only Roman symbols => _checkOnlyRomanSymbols_
```js
romanNumber.isValidInt = function isValidInt(val) {
    if( (Number.isInteger(val)) ||
        ((typeof(val) === "string" || val instanceof String) && Number.isInteger(parseInt(val))) ) {
        
        let intVal = parseInt(val);
        if((intVal < 1) || (intVal > 3999)) {
            throw new Error('invalid range');
        }

        return true;
    }
    else {
        return false;
    }
};

romanNumber.checkOnlyRomanSymbols = function checkOnlyRomanSymbols(val) {
    if((typeof(val) !== "string") && !(val instanceof String)) {
        return false;
    }

    let romanSymbols = ['M', 'D', 'C', 'L', 'X', 'V', 'I'];

    for(let i = 0; i < val.length; i++) {
        if(romanSymbols.indexOf(val[i].toUpperCase()) < 0) {
            return false;
        }
    }

    return true;
};
```

Then we will complete or public methods _toInt_ and _toString_
```js
/**
 * toInt
 */
romanNumber.prototype.toInt = function toInt() {
    return parseInt(this.intVal);
};

/**
 * toString
 */
romanNumber.prototype.toString = function toString() {
    return this.strVal;
};
```

In the end we will rewrite our library constructor to make use of our new methods:
```js
const romanNumber = function RomanNumber(val) {
    if(!new.target) {
        return new RomanNumber(val);
    }

    if((typeof(val) === 'undefined') || (val === null) || (val === '')) {
        throw new Error('value required');
    }

    if(RomanNumber.isValidInt(val)) {
        this.intVal = parseInt(val);
        if(this.intVal == 1) {
            this.strVal = 'I';
        }
        else if(this.intVal == 2) {
            this.strVal = 'II';
        }
        else if(this.intVal == 3) {
            this.strVal = 'III';
        }
        else if(this.intVal == 4) {
            this.strVal = 'IV';
        }
        else if(this.intVal == 5) {
            this.strVal = 'V';
        }
    }
    else if(RomanNumber.checkOnlyRomanSymbols(val)) {
        
    }
    else {
        throw new Error('invalid value');
    }
};
```

As you cas see, inside the block of code executed when the value passed is a valid int, we added also the controls to convert the numbers from 1 to 5.

If we check now these new tests, we will have a correct result:
![Test 1 to 5 values conversion: passed](/content/images/2017/12/11/09-Test_FirstFiveIntValues-OK.png)

## Two-digit numbers

Let's see what happens if we try to test the number _49_.
```js
        it('The arabic number 49 should be equal to the Roman number "XLIX"', () => {
            let romannum = RomanNumber(49);
            romannum.toString().should.equal('XLIX');
            romannum.toInt().should.equal(49);
        });
```

As expected we have an error because we never handled any two-digit value:
![Test 49 value conversion: failed](/content/images/2017/12/11/10-Test_Number49-KO.png)

Let's set up a separate static method to convert _Hindu-Arabic numbers_ to _Roman Numbers_: _intToRoman_
```js
romanNumber.intToRoman = function intToRoman(val) {
    let intVal = parseInt(val);
    let finalStr = '';

    let ones = intVal % 10;
    intVal = parseInt(intVal / 10);
    let tens = intVal % 10;

    switch(ones) {
    case 1:
        finalStr = 'I';
        break;
    case 2:
        finalStr = 'II';
        break;
    case 3:
        finalStr = 'III';
        break;
    case 4:
        finalStr = 'IV';
        break;
    case 5:
        finalStr = 'V';
        break;
    case 6:
        finalStr = 'VI';
        break;
    case 7:
        finalStr = 'VII';
        break;
    case 8:
        finalStr = 'VIII';
        break;
    case 9:
        finalStr = 'IX';
        break;
    default:
        break;
    }

    switch(tens) {
    case 1:
        finalStr = 'X' + finalStr;
        break;
    case 2:
        finalStr = 'XX' + finalStr;
        break;
    case 3:
        finalStr = 'XXX' + finalStr;
        break;
    case 4:
        finalStr = 'XL' + finalStr;
        break;
    case 5:
        finalStr = 'L' + finalStr;
        break;
    case 6:
        finalStr = 'LX' + finalStr;
        break;
    case 7:
        finalStr = 'LXX' + finalStr;
        break;
    case 8:
        finalStr = 'LXXX' + finalStr;
        break;
    case 9:
        finalStr = 'XC' + finalStr;
        break;
    default:
        break;
    }

    return finalStr;
};
```

This function computes _ones_ and _tens_ values from the input, and then it handles them separately to get the appropriate Roman symbols (See how the _case 0_ is never handled because we have no Roman symbol representing it).

Now let's just replace the _isValidInt code block_ in the library constructor
```js
    ...
    if(RomanNumber.isValidInt(val)) {
        this.intVal = parseInt(val);
        this.strVal = RomanNumber.intToRoman(this.intVal);
    }
    ...
```

...and all the test cases will be passed!

## Convert any valid Hindu-Arabic number

We're now ready to go and convert any Hindu-Arabic number.

Add these test conversions for numbers 1968, "1473", 2999, 3000 and 3999.
```js
        it('The arabic number 1968 should be equal to the Roman number "MCMLXVIII"', () => {
            let romannum = RomanNumber(1968);
            romannum.toString().should.equal('MCMLXVIII');
            romannum.toInt().should.equal(1968);
        });

        it('The arabic number "1473" should be equal to the Roman number "MCDLXXIII"', () => {
            let romannum = RomanNumber("1473");
            romannum.toString().should.equal('MCDLXXIII');
            romannum.toInt().should.equal(1473);
        });

        it('The arabic number 2999 should be equal to the Roman number "MMCMXCIX"', () => {
            let romannum = RomanNumber(2999);
            romannum.toString().should.equal('MMCMXCIX');
            romannum.toInt().should.equal(2999);
        });

        it('The arabic number 3000 should be equal to the Roman number "MMM"', () => {
            let romannum = RomanNumber(3000);
            romannum.toString().should.equal('MMM');
            romannum.toInt().should.equal(3000);
        });

        it('The arabic number 3999 should be equal to the Roman number "MMMCMXCIX"', () => {
            let romannum = RomanNumber(3999);
            romannum.toString().should.equal('MMMCMXCIX');
            romannum.toInt().should.equal(3999);
        });
```

We need to enhance the static method _intToRoman_ to handle the _hundreds_ and _thousands_ too:
```js
romanNumber.intToRoman = function intToRoman(val) {
    let intVal = parseInt(val);
    let finalStr = '';

    let ones = intVal % 10;
    intVal = parseInt(intVal / 10);
    let tens = intVal % 10;
    intVal = parseInt(intVal / 10);
    let hundreds = intVal % 10;
    intVal = parseInt(intVal / 10);
    let thousands = intVal % 10;

    switch(ones) {
    case 1:
        finalStr = 'I';
        break;
    case 2:
        finalStr = 'II';
        break;
    case 3:
        finalStr = 'III';
        break;
    case 4:
        finalStr = 'IV';
        break;
    case 5:
        finalStr = 'V';
        break;
    case 6:
        finalStr = 'VI';
        break;
    case 7:
        finalStr = 'VII';
        break;
    case 8:
        finalStr = 'VIII';
        break;
    case 9:
        finalStr = 'IX';
        break;
    default:
        break;
    }

    switch(tens) {
    case 1:
        finalStr = 'X' + finalStr;
        break;
    case 2:
        finalStr = 'XX' + finalStr;
        break;
    case 3:
        finalStr = 'XXX' + finalStr;
        break;
    case 4:
        finalStr = 'XL' + finalStr;
        break;
    case 5:
        finalStr = 'L' + finalStr;
        break;
    case 6:
        finalStr = 'LX' + finalStr;
        break;
    case 7:
        finalStr = 'LXX' + finalStr;
        break;
    case 8:
        finalStr = 'LXXX' + finalStr;
        break;
    case 9:
        finalStr = 'XC' + finalStr;
        break;
    default:
        break;
    }

    switch(hundreds) {
    case 1:
        finalStr = 'C' + finalStr;
        break;
    case 2:
        finalStr = 'CC' + finalStr;
        break;
    case 3:
        finalStr = 'CCC' + finalStr;
        break;
    case 4:
        finalStr = 'CD' + finalStr;
        break;
    case 5:
        finalStr = 'D' + finalStr;
        break;
    case 6:
        finalStr = 'DC' + finalStr;
        break;
    case 7:
        finalStr = 'DCC' + finalStr;
        break;
    case 8:
        finalStr = 'DCCC' + finalStr;
        break;
    case 9:
        finalStr = 'CM' + finalStr;
        break;
    default:
        break;
    }

    switch(thousands) {
    case 1:
        finalStr = 'M' + finalStr;
        break;
    case 2:
        finalStr = 'MM' + finalStr;
        break;
    case 3:
        finalStr = 'MMM' + finalStr;
        break;
    default:
        break;
    }

    return finalStr;
};
```

Now let's check what we accomplished so far, by typing:
`$ npm test`

Everything works :)
![Test any Hindu-Arabic number conversion: passed](/content/images/2017/12/11/11-Test_AnyInt-OK.png)

## Refactor intToRoman

Even though our intToRoman method performs correctly, it really needs refactoring (for style's sake).

The idea is to group the Roman symbols, based on the ones used for _ones_, _tens_, _hundreds_ and _thousands_.
In fact, the pattern used is always the same: it's just the symbols that change.

Here is the final version of our conversion method _intToRoman_:
```js
/**
 * static method intToRoman
 * @param {Integer} val: must be an integer between 1 and 3999 (even in the form '1' to '3999')
 * 
 * The patterns for ones, tens, hundreds and thousands are the same:
 * only sumbols change:
 * 
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
romanNumber.intToRoman = function intToRoman(val) {
    let intVal = parseInt(val);
    let onesSym       = ['I', 'V', 'X'];
    let tensSym       = ['X', 'L', 'C'];
    let hundredsSym   = ['C', 'D', 'M'];
    let thousandsSym  = ['M', '-', '-'];
    let finalStr = '';
    
    // Retrieve units, tens, hundreds and thousands from val
    for(let i = 0; i < 4; i++) {
        let tmpSym;
        let tmpVal;
        if(i == 0) {        // ones
            tmpSym = onesSym;
        }
        else if(i == 1) {   // tens
            tmpSym = tensSym;
        }
        else if(i == 2) {   // hundreds
            tmpSym = hundredsSym;
        }
        else {              // thousands
            tmpSym = thousandsSym;
        }
        tmpVal = intVal % 10;
        intVal = parseInt(intVal / 10);

        switch(tmpVal) {
        case 1:
            finalStr = tmpSym[0] + finalStr;
            break;
        case 2:
            finalStr = tmpSym[0] + tmpSym[0] + finalStr;
            break;
        case 3:
            finalStr = tmpSym[0] + tmpSym[0] + tmpSym[0] + finalStr;
            break;
        case 4:
            finalStr = tmpSym[0] + tmpSym[1] + finalStr;
            break;
        case 5:
            finalStr = tmpSym[1] + finalStr;
            break;
        case 6:
            finalStr = tmpSym[1] + tmpSym[0] + finalStr;
            break;
        case 7:
            finalStr = tmpSym[1] + tmpSym[0] + tmpSym[0] + finalStr;
            break;
        case 8:
            finalStr = tmpSym[1] + tmpSym[0] + tmpSym[0] + tmpSym[0] + finalStr;
            break;
        case 9:
            finalStr = tmpSym[0] + tmpSym[2] + finalStr;
            break;
        default:
            break;
        }
    }

    return finalStr;
};
```

If you just relaunch the tests, you will see that any of them is correctly passed.

## Even more to come (we're going to make it)

In Pt.4 we will complete our library by adding conversion facility from Roman numbers to Hindu-Arabic numbers.

Check out the [Roman Library Repository](https://github.com/evildead/RomanNumber) if you cannot wait for the last chapter of this blog
