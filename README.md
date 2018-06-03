# Static type constraints

This proposal adds static type constraints features to ECMAScript,
based on the existing syntax, just use the features of void.

It is located in the variable initial value or declares the parameter default value
or the first statement of the body of the function.

The Forms:

1. `void TYPE` an object of TYPE
1. `void [TYPE]` same as `void TYPE`
1. `void [[TYPE]]` an Array of TYPE
1. `void [[[TYPE]]]` an Array of Array of TYPE
1. `void [T,Tn]` an object of T or Tn
1. `void [[T,TN]]` an Array of Array of T or Tn
1. `void [!TYPE]` an object of TYPE, and can not be omitted
1. `void Object instanceof TYPE` an object instanceof TYPE
1. `void function(){}` a function
1. `void Function instanceof protoFunction` a function

A TYPE is an identifier that allows from an external module.

## Example

```js
import mod from 'paths';

function CustomNumber(x = void([Number, mod.BigNumber])) {
  // Declarative results type, and the default value is undefined
  void String;
  // same as void [String]
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
    void [[
      String,
      Number,
      String
    ]];
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
    void [
      // parameters
      [
        ['currentValue', undefined, 'Optional comments'],
        ['index', Number],
        ['array', Array]
      ],
      // results
      [
        'truthy', undefined, `
          returns a value that coerces to true. callback is invoked only for
          indexes of the array which have assigned values; it is not invoked for
          indexes which have been deleted or which have never been assigned values
          `
      ]
    ];
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

// Prototype function

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
  multi  = void [String, Number],
    // Allowed type of String or Number, and multi === undefined
  obj    = void(CustomClass)||{};
    // Allowed type of CustomClass, and default value is {}
```
