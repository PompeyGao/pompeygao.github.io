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
  
  







































