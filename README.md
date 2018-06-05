# Static type constraints

ECMAScript 静态类型约束表达式. 该提案以我的需求和习惯进行设计.

接受建议但最终选择在我, 因为即便合理但可能不符合我的习惯.

首先, 这里给出的方法不是银弹, 毕竟 ECMAScript 是非静态类型语言.

目标:

1. 使用合法的 ECMAScript 语法实现无副作用的静态类型描述
1. 配合额外的工具实现静态类型检查

方案:

```text
在变量初始化, 形参缺省值, 函数主体的第一句位置加入无副作用的静态类型约束表达式.
该表达式会被引擎计算, 但它不产生副作用, 加上或去掉都不影响执行结果.
理论上只要使用得当任何运算符都可以实现这一目标.
本方案选用一元运算符 `void`.
```

如果需要, 可以使用额外的工具移除静态类型约束表达式.

赞美 `void`:

```js
let
  v = 0,              // v 有值, 缺乏类型约束
  o = void('')||'',   // o 有值, 表达式 void('') 表示类型为 string
  i = void(0)||0,     // i 有值, 表达式 void(0) 表示类型为 number
  d = void(false)||0, // d 有值, 表达式 void(false) 表示类型为 boolean
  x = void(0);        // 只声明了类型为 number, 值依然是 undefined
```

增加的类型描述在默认值之前, 并用 `||` 运算连接, 这个小技巧保证了无副作用,
除非故意使用有副作用的表达式.

形式:

1. `void %PrimitiveDefaultValues%` 参见 [原始值](#primitivedefaultvalues)
1. `void TYPE`      表示类型为 `TYPE`
1. `void [TYPE]`    表示类型为 数组, 元素类型为 `TYPE`
1. `void [[TYPE]]`  表示类型为 数组, 元素类型为 `TYPE` 的数组
1. `void (T,Tn)`    表示类型为 `T` 或 `Tn`
1. `void [T,TN]`    表示类型为 数组, 元素类型为 `T` 或 `Tn`
1. `void !TYPE`     表示类型为 `TYPE` 且必须有值, 即拒绝 `undefined`
1. `void !(T,TN)`   表示类型为 `T` 或 `Tn`, 且必须有值, 注意 `!` 的位置
1. `void TYPE++`    表示类型为 `TYPE` 或它的继承者

其中 `TYPE` 可以是: 按优先级

1. [原始值](#primitivedefaultvalues)
1. [符号值](#nonletterbeginning)
1. 标识符  包括形式 mod.identifier
1. 注册类型 使用额外方法注册的类型, 具体细节由实现者决定
1. 未知的   忽略, 因为它是无副作用的, 所有可以直接忽略

## PrimitiveDefaultValues

周知:

```js
let x = 0, y = new Number(0);

console.log(x instanceof Number); // false
console.log(y instanceof Number); // true
console.log(x === y);             // false
```

现实中几乎见不到 `Number` 类型, 常用的是不存在的 `number` 类型.

为了达到语义上的一致, 下列原始值表示特定的类型.

1. `''`     string
1. `false`  boolean
1. `0`      integer
1. `{}`     Object 并且访问计算属性 `[property]` 中的 `property` 类型为 string
1. `[]`     Array 并且不限制元素类型, 除非明确定义了元素类型

例:

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

## NonLetterBeginning

下列非字母开头的值表示特定的类型, 是考虑与 [WASM] 融合, 具体处理方式由实现负责.

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

例: asm.js

```js
function geometricMean(start, end) {
  start = start|0; // start has type int
  end = end|0;     // end has type int
  return +exp(+logSum(start, end) / +((end - start)|0));
}
```

例: 本方案写法

```js
function geometricMean(start = void !0, end = void !0) {
  void !0; // 必须返回 integer 类型
  return exp(logSum(start, end) / (end - start));
}
geometricMean('0','1'); // Throws TypeError: ....
geometricMean(0);       // Throws TypeError: 2 argument required, but ...
geometricMean(0,1,2);   // Throws TypeError: 2 argument required, but ...
geometricMean(0,1);     // Ok ...
```

## Identifier

标识符写法.

例:

```js
import asm from 'asm'; // 假设此模块描述了类型

function geometricMean(start = void !asm.int, end = void !asm.int) {
  void !asm.int;
  return exp(logSum(start, end) / (end - start));
}

geometricMean('0','1'); // Throws TypeError: ....
geometricMean(0);       // Throws TypeError: 2 argument required, but ...
geometricMean(0,1,2);   // Throws TypeError: 2 argument required, but ...
geometricMean(0,1);     // Ok ...
```

如果使用者没有 import 相关模块, 具体的写法将影响代码是否会抛出错误.

可以用非空字符串表示标识符. 出于兼容性考虑, 即便没有引入相关模块也不会出错.

例: 这和上面的写法在语义上是等价的, 但不依赖引入的模块.

```js
function geometricMean(start = void !'asm.int', end = void !'asm.int') {
  void !'asm.int';
  return exp(logSum(start, end) / (end - start));
}
```

## NotSupport

下列用法不被支持.

```js
// 不支持 true

function notSupportVoidTrue(
  b = void(true)||false   // throws TypeError: ...
) {}

// 不支持嵌套 inline 类型

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

// 不支持 inline 函数

function forEach(
  callback = void function( // throws TypeError: ...
    currentValue = void undefined,
    index = void Number,
    array = void Array){}
) {}

// 不支持箭头函数, 目的是防止在箭头函数中误用 this.

const ArrowFunction = (
    currentValue = void undefined,
    index = void Number,
    array = void Array
)=>{};

function forEach(
  callback = void ArrowFunction // throws TypeError: ...
) {}

// 具有二义性的强制符号 `!` 用法

function results() {
  void (!0,''); // throws TypeError: ...
  // 正确写法: void !(0,'');
}
```

## Example

下列用法并不全面, 欢迎提交遗漏.

```js
import mod from 'paths';

function CustomNumber(
  x = void(0, mod.BigNumber)
  // 类型为 number 或 mod.BigNumber, 缺省值为 undefined
) {
  void '';
  // 无返回值或返回值类型为 string
}

class CustomClass {
  constructor() {
    void {
      name: '',
      age: 0,
      email: ''
    };
    // 描述拥有的字段的类型
    // 未来 ECMAScript 可能会支持新的语法来声明字段
    // 即便新的语法出台, 本方案保持不变, 没有必要搞两套
  }
}

class AbstractClass {
  constructor() {
    void undefined;
    // 固定用法, 位于构造函数且没有其它语句, 表示该类是抽象类
  }

  any () {
    void undefined;
    // 这个方法必须被覆盖, 但返回什么类型由继承者决定
  }

  exUndefined() {
    void !undefined;
    // 这个方法必须被覆盖, 且继承者必须声明返回类型
  }
}

class Class extends AbstractClass {
  any () {
    void undefined;
  }

  exUndefined() {
    void '';
    // 明确了类型为 string, 但还是允许无返回值
    // void undefined; // throws TypeError: ...
  }
}

class PureStructure {
  constructor() {
    void {
      name: !'', // 必须是 string 类型
      age: !0,   // 必须是 integer 类型
      email: ''  // string 类型, 允许(不存在)值为 undefined
    };
  }
}

// @Flow type MetadataType = { [string]: any };

class MetadataType {
  constructor() {
    void {['']: undefined};
    // 因为默认 {} 计算属性类型是 string, 该写法可以简化为: void {}
  }
}

// Conflict-free compatibility writing

function noConflict(
  x = void('')||''
  // 字符串类型, 且设置了默认值, 注意默认值表达式是 ECMAScript 原有的规范.
) {}

// Inherit

function webComponents(
  x = void HTMLElement++
  // 表示 x instanceof HTMLElement
) {}

function forEach(
  callback = void protoCallback
  // 因为 protoCallback.prototype 没有更多的属性, 所以这里表示函数类型
  // 判断函数类型的算法:
  // Object.getOwnPropertyNames(fn.prototype).join(',') === 'constructor'
) {}

function protoCallback(
    currentValue = void undefined,
    index = void 0,
    array = void []
) {

}

function mustBeResults() {
  void !(0,'');
  // 返回类型必须为 integer 或 string
}

let
  custom = void 0,
  one    = void(0)||1,
  multi  = void('', 0),
  obj    = void(CustomClass)||{};
```

## License

MIT License

Copyright (c) 2018 YU HengChun<achun.shx at qq.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

[WASM]: https://webassembly.org/