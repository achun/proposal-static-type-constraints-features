# Static type constraints

This proposal adds static type constraints features to ECMAScript,
based on the existing syntax, just use the features of void.

It is located in the variable initial value or declares the parameter default value
or the first statement of the body of the function.

The Forms:

1. `void %PrimitiveDefaultValues%` See [PrimitiveDefaultValues](#primitivedefaultvalues)
1. `void TYPE` an object of TYPE
1. `void [TYPE]` an Array of TYPE
1. `void [[TYPE]]` an Array of Array of TYPE
1. `void (T,Tn)` an object of T or Tn
1. `void [T,TN]` an Array of T or Tn
1. `void !TYPE` an object of TYPE, and can not be omitted
1. `void TYPE++` an object instanceof TYPE

A TYPE is an identifier that allows from an external module.

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
1. `0`      integer
1. `{}`     Object and property computed with string
1. `[]`     Array, the previous article has appeared

Legal but is not responsible for implementing.

1. `+0`     unsigned integer
1. `1`      1bit
1. `2`      2bit
1. `3`      3bit
1. `4`      4bit
1. `5`      5bit
1. `6`      6bit
1. `7`      7bit
1. `8`      i8
1. `+8`     u8
1. `16`     i16
1. `+16`    u16
1. `32`     i32
1. `+32`    u32
1. `64`     i64
1. `+64`    u64
1. `128`    i128
1. `+128`   u128
1. `.0`     float
1. `.32`    float32
1. `.64`    float64
1. `.128`   float128

So

```js
function PrimitiveDefaultValues(
  s = void('')||'',       // s is string, typeof s === 'string'
  n = void(0)||'',        // n is number, typeof n === 'number'
  b = void(false)||false  // b is boolean, typeof b === 'boolean'
) {
  void {                  // results object
    name: !'',            // must be name: string
    stars: !0,            // must be stars: number
    followers: [{}]       // optional followers: [object]
  };
}
```

## Example

```js
import mod from 'paths';

function CustomNumber(
  x = void(0, mod.BigNumber)
  // x is number or mod.BigNumber, and default is undefined
) {
  void '';
  // This function results an omissible string
}

class CustomClass {
  constructor() {
    void {
      name: '',
      age: 0,
      email: ''
    };
    // properties structure
  }
}

class Interface {
  constructor() {
    void undefined;
    // Just a semantic interface,
    // usually it means that the successor must override all methods
  }
  method(
    // void ...
  ){}
}

class PureStructure {
  constructor() {
    void {
      name: !'', // must be a name
      age: !0,  // must be age
      email: ''  // default undefined
    };
  }
}

// @Flow type MetadataType = { [string]: any };

class MetadataType {
  constructor() {
    void {['']: undefined}; // same as void {}
  }
}

// Conflict-free compatibility writing

function noConflict(x = void('')||'') {
  // ...
}

// Inherit

function webComponents(x = void ++HTMLElement) {
  // ...
}

function forEach(callback = void protoCallback) {
  // ...
}

function protoCallback(
    currentValue = void undefined,
    index = void 0,
    array = void []
) {

}

let
  custom = void 0,
    // The type is fixed to number, and custom === undefined
  one    = void(0)||1,
    // The type is fixed to number, and one === 1
  multi  = void('', 0),
    // Allowed type of string or number, and multi === undefined
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
geometricMean(0);       // Throws TypeError: 2 argument required, but ...
geometricMean(0,1,2);   // Throws TypeError: 2 argument required, but ...
geometricMean(0,1);     // Ok ...
```

## Compatibility

The real purpose of a void expression is not an operation.

It is description for typing, the literal semantics.

It is compatible with the old engine.

Even if it is operated, there are no side effects.

The new engine will extract the type description and will not perform operations.

So, the form `TYPE++` is for backward compatibility.

```js
function CustomString(x = void('',String++) || 'abc' ) {
 // x is string or instanceof String and default value is 'abc'
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
// @Flow let x: number = 0;
let x = void('number')||0;
// You know, number !== Number, and number is not a built-in type in ECMAScript
```

## NotSupport

```js

function notSupportVoidTrue(
  b = void(true)||false   // throws TypeError: ...
) {}

// Multiple levels of nesting

class ConfigType {
  constructor() {
    void {
      version: 0,
      encodeNames: false,
      lines: [''],
      filename: '',
      linker: {
        statics: { ['']: 0 } // throws TypeError: ...
      }
    };
  }
}

// void function

function forEach(
  callback = void function( // throws TypeError: ...
    currentValue = void undefined,
    index = void Number,
    array = void Array){}
) {}

// void ArrowFunctionIdentifier

const ArrowFunction = (
    currentValue = void undefined,
    index = void Number,
    array = void Array
)=>{};

function forEach(
  callback = void ArrowFunction // throws TypeError: ...
) {}
```
