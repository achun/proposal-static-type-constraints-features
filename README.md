# Static type constraints

This proposal adds static type constraints features to ECMAScript,
based on the existing syntax, just use the features of void.

It is located in the variable initial value or declares the parameter default value
or the first statement of the body of the function.

The Forms:

1. `void TYPE` an object of TYPE, and default value is undefined
1. `void [TYPE]` same as `void TYPE`
1. `void [[TYPE]]` an Array of TYPE, and default value is undefined
1. `void [[[TYPE]]]` an Array of Array of TYPE, and default value is undefined
1. `void [T,Tn]` an object of T or Tn, and default value is undefined
1. `void [literal-value,TYPE]` an object of TYPE, and default value is literal-value
1. `void [literal-value,T,Tn]` an object of T or Tn, and default value is literal-value

A TYPE is an identifier that allows from an external module.

## Example

```js
import mod from 'paths';

let
  custom = void Number,
    // The type is fixed to Number, and custom === undefined
  one    = void [1,Number],
    // The type is fixed to Number, and one === 1
  multi  = void [String,Number],
    // Allowed type of String or Number, and multi === undefined
  obj    = void [{},CustomClass];
    // Allowed type of CustomClass, and default value is {}

function CustomName(x = void ['', Number, mod.BigNumber]) {
  // Declarative results type
  void String; // This function must eventually return a string
  // ...
}

class CustomClass {
  // ...
}
```
