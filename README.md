# Static type constraints

This proposal adds static type constraints features to ECMAScript,
based on the existing syntax, just use the features of void.

It is located in the variable initial value or declares the parameter default value
or the first statement of the body of the function.

The Forms:

1. `void TYPE` an object of TYPE
1. `void [TYPE]` an Array of TYPE
1. `void [[TYPE]]` an Array of Array of TYPE
1. `void (T,Tn)` an object of T or Tn
1. `void [T,TN]` an Array of T or Tn
1. `void !TYPE` an object of TYPE, and can not be omitted
1. `void Object instanceof TYPE` an object instanceof TYPE
1. `void function(){}` a function
1. `void Function instanceof protoFunction` a function

A TYPE is an identifier that allows from an external module.

## Example

```js
import mod from 'paths';

function CustomNumber(x = void(Number, mod.BigNumber)) {
  // Declarative results type, and the default value is undefined
  void String;
  // same as void(String)
  // This function must eventually return a string

  // ...
}

class CustomClass {
  // ...
}

class PureStructure {
  constructor() {
    void {
      name: !String, // must be a name
      age: !Number,  // must be age
      email: String  // default undefined
    };
  }
}

class ClassPrototype {
  constructor() {
    void {
      name: String,
      age: Number,
      email: String
    };
  }
}

class FunctionPrototype {
  constructor() {
    void (
      (String,Number,String)
    );
  }
}

// @Flow type MetadataType = { [string]: any };

class MetadataType {
  constructor() {
    void { [String]: undefined };
  }
}

class callbackPrototypeWithDetail {
  constructor() {
    void (
      // parameters
      (
        ['currentValue', undefined, 'Optional comments'],
        ['index', Number],
        ['array', Array]
      ),
      // results
      [
        'truthy', undefined, `
          returns a value that coerces to true. callback is invoked only for
          indexes of the array which have assigned values; it is not invoked for
          indexes which have been deleted or which have never been assigned values
          `
      ]
    );
  }
}

// Conflict-free compatibility writing

function noConflict(x = void(String)||'') {
  // ...
}

// Inherit

function webComponents(x = void Object instanceof HTMLElement) {
  // ...
}

// Inline prototype function

function forEach(callback = void function(
    currentValue = void undefined,
    index = void Number,
    array = void Array){}
) {
  // ...
}

function Foreach(callback = void Function instanceof protoCallback) {
  // ...
}

function protoCallback(
    currentValue = void undefined,
    index = void Number,
    array = void Array
) {

}

let
  custom = void Number,
    // The type is fixed to Number, and custom === undefined
  one    = void(Number)||1,
    // The type is fixed to Number, and one === 1
  multi  = void(String, Number),
    // Allowed type of String or Number, and multi === undefined
  obj    = void(CustomClass)||{};
    // Allowed type of CustomClass, and default value is {}
```

## If the proposal passes

E.g: asm.js

```js
function geometricMean(start, end) {
  start = start|0; // start has type int
  end = end|0;     // end has type int
  return +exp(+logSum(start, end) / +((end - start)|0));
}
```

Proposal:

```js
import types from 'asm-types';
function geometricMean(start = void(!types.int), end = void(!types.int)) {
  void(types.int);
  return exp(logSum(start, end) / (end - start));
}

geometricMean('0','1'); // Throws TypeError: ....
geometricMean(0); // Throws TypeError: 2 argument required, but only 1 present.
geometricMean(0,1,2); // Throws TypeError: 2 argument required, but 3 present.
geometricMean(0,1); // Ok ...
```

## Compatibility

The real purpose of a void expression is not an operation.

It is description for typing, the literal semantics.

It is compatible with the old engine.

Even if it is operated, there are no side effects.

The new engine will extract the type description and will not perform operations.

So, the form `Object instanceof TYPE` is for backward compatibility.

```js
function CustomString(x = void( Object instanceof String ) || 'abc' ) {
 // x is String or inherited String
}
```

Maybe null is better.

```js
function CustomString(x = void( null instanceof String ) || 'abc' ) {
 // x is String or inherited String
}
```

For safety, it may be a good idea to use it with default values.

```js
let
  isUndefined = void Number,
    // Static type constraint does not take effect
  isNumberUndefined = void(Number)||undefined;
    // Static type constraint take effect

```

Last resort: Identifiers expressed as strings

```js
// @Flow
// let x: number = 0;
let x = void('number')||0; // You know, number !== Number
```

## PrimitiveDefaultValues

Knowing that:

```js
let x = 0, y = new Number(0);

console.log(x instanceof Number); // false
console.log(y instanceof Number); // true
console.log(x === y);             // false
```

Use the primitive default value to solve this problem.

The supported primitive default values:

1. `''`     string
1. `false`  boolean
1. `0`      number

So

```js
function PrimitiveDefaultValues(
  s = void('')||'',      // s is string, typeof s === 'string'
  n = void(0)||'',       // n is number, typeof n === 'number'
  b = void(false)||false // b is boolean, typeof b === 'boolean'
) {
}

function notSupport(b = void(true)||false) {}
```

## Pending

I'm not sure if it is necessary to support multiple levels of nesting

```js
class ConfigType {
  constructor() {
    void {
      version: Number,
      encodeNames: Boolean,
      lines: [String],
      filename: String,
      linker: {
        statics: { [String]: Number } // Need support?
      }
    };
  }
}
```
