---
layout:     post
title:      "JavaScript数据类型"
subtitle:   "JavaScript类型：关于类型，有哪些你不知道的细节？"
date:       2020-07-23
author:     "pompeygao"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - JavaScript
    - 数据类型
---

## 数据类型

最新的 ECMAScript 标准定义了 8种数据类型:

- 6种原始类型，可以由`typeof`运算符检查

  - `Undefined`: `typeof instance === "undefined"`
  - `Boolean`: `typeof instance === "boolean"`
  - `Number`: `typeof instance === "number"`
  - `String`: `typeof instance === "string"`
  - `Symbol`: `typeof instance === "symbol"`
  - `BigInt`: `typeof instance === "bigint"`

- `Null`: `typeof instance === "object"`

  > 特殊的原始类型

- `Object`: `typeof instance === "object"`



#### Undefined、Null

- `Undefined`类型的值只有一个 `undefined`。任何变量在赋值前都是`Undefined`类型、值为`undefined`。一般我们可以用全局变量`undefined`来表达这个值，或者用`void`运算把任意一个表达式变成`undefined`值。

  *但是在JavaScript中 `undefined` 是一个变量，不是关键字，所以为了避免无意中被篡改，可以使用 `void 0` 来获取 `undefined` 值*

  ```javascript
  // 可能在非全局作用域中被当作标识符（变量名）来使用
  (function() {
  var undefined = 'foo';
  console.log(undefined, typeof undefined)
  })()
  // 输出：foo string
  ````

- `Null`类型也只有一个值，就是 `null`，它的语义表示空值，与 `undefined `不同，`null`是JavaScript关键字，所以在任何代码中，你都可以放心用 `null `关键字来获取` null `值。

- `Undefined `跟 `Null `有一定的表意差别，`Null `表示的是：“定义了但是为空”。所以，在实际编程时，我们一般不会把变量赋值为 `undefined`，这样可以保证所有值为` undefined `的变量，都是从未赋值的自然状态。

#### Boolean

- 只有两个值 `true`和`false`。

#### Number

- 根据 ECMAScript 标准，JavaScript 中只有一种数字类型：基于 IEEE 754 标准的双精度 64 位二进制格式的值（-(2^53 -1) 到 2^53 -1）。**它并没有为整数给出一种特定的类型**。除了能够表示浮点数外，还有一些带符号的值：`+Infinity`，`-Infinity` 和 `NaN` (非数值，Not-a-Number)。

- 要检查值是否大于或小于 `+/-Infinity`，你可以使用常量 [`Number.MAX_VALUE`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_VALUE) 和 [`Number.MIN_VALUE`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MIN_VALUE)。另外在 ECMAScript 6 中，你也可以通过 [`Number.isSafeInteger()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isSafeInteger) 方法还有 [`Number.MAX_SAFE_INTEGER`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER) 和 [`Number.MIN_SAFE_INTEGER`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MIN_SAFE_INTEGER) 来检查值是否在双精度浮点数的取值范围内。 超出这个范围，JavaScript 中的数字不再安全了。

- 为什么在 JavaScript 中，0.1+0.2 不等于0.3 

  ```js
  0.1 + 0.2  === 0.3  // false
  ```
  
  计算机中的数字都是以二进制存储的，如果要计算 0.1 + 0.2 的结果，**计算机会先把 0.1 和 0.2 分别转化成二进制，然后相加，最后再把相加得到的结果转为十进制** 。 所以0.1+0.2在的计算过程如下:
  
  - 先将0.1和0.2转化为二进制，但有一些浮点数在转化为二进制时，会出现无限循环 。比如， 十进制的 0.1 转化为二进制，会得到如下结果：
  
  ```javascript
  0.0001 1001 1001 1001 1001 1001 1001 1001 …（1001无限循环）
  ```
  
  而存储结构中的尾数部分最多只能表示`53`位。为了能表示 0.1，只能模仿十进制进行四舍五入了，但二进制只有 0 和 1 ， 于是变为 0 舍 1 入 。 因此，0.1 在计算机里的二进制表示形式如下：
  
  
  ```javascript
  0.0001100110011001100110011001100110011001100110011001101
  ```
  
  - 将十进制的0.1和0.2相加，在计算浮点数相加时，需要先进行 “对位”，将较小的指数化为较大的指数，并将小数部分相应右移：
  
     ![0.1+0.2](/img/in-post/data-types-js/add.png)
  
  - 最后使用`JS`将二进制结果使用十进制表示
  
     ```
     (-1)**0 * 2**-2 * (0b10011001100110011001100110011001100110011001100110100 * 2**-52); //0.30000000000000004
     console.log(0.1 + 0.2) ; // 0.30000000000000004
     ```
  
  - 从上面的计算过程可以看出，0.1 和 0.2 在转换为二进制时就发生了一次精度丢失，而对于计算后的二进制又有一次精度丢失 。因此，得到的结果是不准确的。
  
#### String

- JavaScript的字符串类型用于表示文本数据。它是一组16位的无符号整数值的“元素”。在字符串中的每个元素占据了字符串的位置。第一个元素的索引为0，下一个是索引1，依此类推。字符串的长度是它的元素的数量。
- JavaScript 字符串是不可更改的。这意味着字符串一旦被创建，就不能被修改。但是，可以基于对原始字符串的操作来创建新的字符串。例如：
  - 获取一个字符串的子串可通过选择个别字母或者使用 [`String.substr()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/substr).
  - 两个字符串的连接使用连接操作符 (`+`) 或者 [`String.concat()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/concat).

#### Symbol

- **symbol** 是一种基本数据类型 （[primitive data type](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)）。`Symbol()`函数会返回**symbol**类型的值，该类型具有静态属性和静态方法。它的静态属性会暴露几个内建的成员对象；它的静态方法会暴露全局的symbol注册，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法："`new Symbol()`"。

- 当使用 JSON.stringify() 时，以 symbol 值作为键的属性会被完全忽略：

  ```js
  JSON.stringify({[Symbol("foo")]: "foo"});                 
  // '{}'
  ```

#### BigInt

- [`BigInt`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt)类型是 JavaScript 中的一个基础的数值类型，可以用任意精度表示整数。使用 BigInt，您可以安全地存储和操作大整数，甚至可以超过数字的安全整数限制。BigInt是通过在整数末尾附加 `n `或调用构造函数来创建的。

- 通过使用常量[`Number.MAX_SAFE_INTEGER`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER)，您可以获得可以用数字递增的最安全的值。通过引入 BigInt，您可以操作超过[`Number.MAX_SAFE_INTEGER`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER)的数字。您可以在下面的示例中观察到这一点，其中递增[`Number.MAX_SAFE_INTEGER`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER)会返回预期的结果:

  ```
  > const x = 2n ** 53n;
  9007199254740992n
  > const y = x + 1n; 
  9007199254740993n
  ```

  可以对`BigInt`使用运算符`+、``*、``-、``**`和`%`，就像对数字一样。BigInt 严格来说并不等于一个数字，但它是松散的。

  在将`BigInt`转换为`Boolean`时，它的行为类似于一个数字：`if、``||、``&&、``Boolean 和``!。`

  `BigInt`不能与数字互换操作。否则，将抛出[`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)。

#### Object

- JavaScript 中的几个基本类型，都在对象类型中有一个“亲戚”。它们是：Number；String；Boolean；Symbol。所以 3 与 new Number(3) 是完全不同的值，它们一个是 Number 类型， 一个是对象类型。
- Number、String 和 Boolean，三个构造器是两用的，当跟 new 搭配时，它们产生对象，当直接调用时，它们表示强制类型转换。
- Symbol 函数比较特殊，直接用 new 调用它会抛出错误，但它仍然是 Symbol 对象的构造器。



#### 装箱 拆箱

- 每一种基本类型 Number、String、Boolean、Symbol 在对象中都有对应的类
  - 所谓装箱转换，正是把基本类型转换为对应的对象，它是类型转换中一种相当重要的种类。
  - 拆箱转换，把对象类型转换到基本类型
- 装箱机制会频繁产生临时对象，在一些对性能要求较高的场景下，我们应该尽量避免对基本类型做装箱转换。

































