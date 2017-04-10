#bcx-expression-evaluator

Safely evaluate an Javascript like expression in given context.

In Buttonwood, we heavily use meta-data (JSON format) to deliver business logic from backend to front-end. We don't want to design a meta-data format too complex to maintain, this tool allows us to define some light logic in pure string, way more flexible than rigid meta-data, much safer and more maintainable than passing js function as string (we did that) from backend to front-end.

This tool was mainly extracted, modified and extended from the expression parser of [aurelia-binding](https://github.com/aurelia/binding).

## Install

    npm install --save bcx-expression-evaluator

## Document

`function evaluate(expression, context, helper, opts)`

  * `expression`: the expression string to be evaluated
  * `context`: the input model object
  * `helper`: optional helper object
  * `opts`: optional hashmap, current only support `rejectAssignment` and `stringInterpolationMode`
    * `rejectAssignment` rejects assignment in expression
    * `stringInterpolationMode` treats the whole expression like if it's in backticks \`expression\`

`function evaluateStringInterpolation` is a short-cut to call evaluate with stringInterpolationMode option.

## Usage (in es6 syntax)

    import {evaluate, evaluateStringInterpolation} from 'bcx-expression-evaluator';

    const context = {
      a: 1,
      b: 2,
      c: {
        one: 'one',
        two: 'two'
      },
      avg: function() { return (this.a + this.b) / 2; }
    };

    evaluate('avg() > a ? c.one : c.two', context);  // => 'one';


### use some helper

    const helper = {
      limit: 5,
      sum: (v1, v2) => v1 + v2
    };

    evaluate('sum(a, b) > limit', context, helper);  // => false;

### access context object itself with special `$this` variable

    evaluate('$this', context); // => the context object
    evaluate('$this.a', context); // => 1

### explicitly access helper object with special `$parent` variable
(carried over from aurelia-binding, might change $parent to $helper in future releases.)

    evaluate('a', {a:1}, {a:2}); // => 1
    evaluate('$this.a', {a:1}, {a:2}); // => 1
    evaluate('$parent.a', {a:1}, {a:2}); // => 2

### support es6 string interpolation

    evaluate('`${a+1}`', {a:1}); // => '2'

You can evaluate a string interpolation without backtick "`"

    evaluate('${a+1}', {a:1}, null, {stringInterpolationMode: true}); // => '2'
    evaluateStringInterpolation('${a+1}', {a:1}); // => '2'

You don't have to escape backtick in stringInterpolationMode

    evaluate('`\\`${a+1}\\``', {a:1}); // => '`2`', beware you need double escape as we are writing expression in string quotes
    evaluate('`${a+1}`', {a:1}, null, {stringInterpolationMode: true}); // => '`2`'
    evaluateStringInterpolation('`${a+1}`', {a:1}); // => '`2`'

### safe. It is not an eval in Javascript, doesn't have access to global javascript objects

    evaluate('parseInt(a, 10)', {a:"7"}) // => undefined

    // only have access to context and helper
    evaluate('parseInt(a, 10)', {a:"7"}, {parseInt: parseInt}) // => 7

### silent most of the time

    evaluate('a.b', {}) // => undefined, no error throwed
    evaluate('a.b || c', {c: 'lorem'}) // => 'lorem', no error throwed

### you can use assignment to mutate context object (or even helper object)

    let obj = {a: 1, b: 2};
    evaluate('a = 3', obj); // obj is now {a: 3, b: 2}
    evaluate('b > 3 ? (a = true) : (a = false)', obj); // obj is now {a: false, b: 2}

### disable assignment if you don't need it
This doesn't eliminate side effect, it would not prevent any function you called in the expression to mutate something.

    evaluate('a=1', {a:0}, null, {rejectAssignment: true}); // throws error

## Difference from real Javascript expression
The expression looks like Javascript expression, but there are some difference.

### wrong reference results undefined instead of error

    let obj = {a: 1};
    obj.b.a // => error
    evaluate('b.a', obj); // => undefined

### default result for +/- operators

    undefined + 1 // => NaN
    1 + undefined // => NaN
    null + 1 // => 1
    1 + null // => 1
    undefined + undefined // => NaN
    null + null // => 0

    // in our expression, + and - ignores undefined/null value,
    // if both left and right parts are (evaluated to) undefined/null, result default to 0
    evaluate('undefined + 1'); // => 1
    evaluate('1 + undefined'); // => 1
    evaluate('null + 1'); // => 1
    evaluate('1 + null'); // => 1
    evaluate('undefined + undefined'); // => 0
    evaluate('null + null'); // => 0

### no function expression

    // all would not work in expression
    (function(){return 1})()
    (() => 1)()
    arr.sort((a, b) => a > b)

    // but this would work
    arr.sort(aHelperFunc)


[BUTTONWOODCX™ PTY LTD](http://www.buttonwood.com.au).
