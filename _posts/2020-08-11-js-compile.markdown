---
layout:     post
title:      "JS引擎解析执行过程"
subtitle:   ""
date:       2020-08-11
author:     "pompeygao"
header-img: "img/js.jpg"
tags:
    - JS

---



#### 一、分词 tokenize、解析 Parser

------

##### 词法分析 Lexical Analysis

> 读取代码，将JavaScript代码通过词法分析解析成一个个的token流。

词法分析器的作用，是将一行行的源码拆解成一个个（token）。首先，词法分析器会扫描（scanning）代码，提取token；然后，会进行评估（evaluating），判断token属于哪一类的值。

```js
n * n;

// 词法分析后的结果
[
  { type: { ... }, value: "n", start: 0, end: 1, loc: { ... } },
  { type: { ... }, value: "*", start: 2, end: 3, loc: { ... } },
  { type: { ... }, value: "n", start: 4, end: 5, loc: { ... } },
]
// 每一个 type 有一组属性来描述该令牌：
{
  type: {
    label: 'name',
    keyword: undefined,
    beforeExpr: false,
    startsExpr: true,
    rightAssociative: false,
    isLoop: false,
    isAssign: false,
    prefix: false,
    postfix: false,
    binop: null,
    updateContext: null
  },
  ...
}
```

上面代码中，源代码经过词法分析后，返回一组词义单位，以及它们各自的词类。

##### 语法分析 Syntactic Analysis

> 对生成的token根据语法规则进行语法分析，组装成抽象语法树AST。如果源码符合语法规则，这一步就会顺利完成，生成一个抽象语法树；如果源码存在语法错误，这一步就会终止，抛出一个“语法错误”。

```js
function square(n) {
  return n * n;
}
// 生成的AST
{
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  params: [{
    type: "Identifier",
    name: "n"
  }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "*",
        left: {
          type: "Identifier",
          name: "n"
        },
        right: {
          type: "Identifier",
          name: "n"
        }
      }
    }]
  }
}
```

通常，这一步是整个JavaScript代码执行过程中最慢的。

> AST 是非常重要的一种数据结构，在很多项目中有着广泛的应用。其中最著名的一个项目是 Babel。Babel 是一个被广泛使用的代码转码器，可以将 ES6 代码转为 ES5 代码。Babel 的工作原理就是先将 ES6 源码转换为 AST，然后再将 ES6 语法的 AST 转换为 ES5 语法的 AST，最后利用 ES5 的 AST 生成 JavaScript 源代码。
>
> 除了 Babel 外，还有 ESLint 也使用 AST。ESLint 是一个用来检查 JavaScript 编写规范的插件，其检测流程也是需要将源码转换为 AST，然后再利用 AST 来检查代码规范化的问题。

#### 二、生成字节码

---

- 通过解释器Ignition将AST转换成字节码。
- 字节码就是介于 AST 和机器码之间的一种代码。但是与特定类型的机器码无关，字节码需要通过解释器将其转换为机器码后才能执行。
![bytecode](/img/in-post/js-compile/bytecode.png)

#### 三、执行阶段

----

通常，如果有一段第一次执行的字节码，解释器 Ignition 会逐条解释执行。到了这里，相信你已经发现了，解释器 Ignition 除了负责生成字节码之外，它还有另外一个作用，就是解释执行字节码。在 Ignition 执行字节码的过程中，如果发现有热点代码（HotSpot），比如一段代码被重复执行多次，这种就称为热点代码，那么后台的编译器 TurboFan 就会把该段热点的字节码编译为高效的机器码，然后当再次执行这段被优化的代码时，只需要执行编译后的机器码就可以了，这样就大大提升了代码的执行效率。



其实字节码配合解释器和编译器是最近一段时间很火的技术，比如 Java 和 Python 的虚拟机也都是基于这种技术实现的，我们把这种技术称为即时编译（JIT）。具体到 V8，就是指解释器 Ignition 在解释执行字节码的同时，收集代码信息，当它发现某一部分代码变热了之后，TurboFan 编译器便闪亮登场，把热点的字节码转换为机器码，并把转换后的机器码保存起来，以备下次使用。

JIT 的工作过程：

![JIT](/img/in-post/js-compile/JIT.png)

#### 四、垃圾回收

----

##### 引用计数

此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

> 限制：循环引用
>
> 该算法有个限制：无法处理循环引用的事例。在下面的例子中，两个对象被创建，并互相引用，形成了一个循环。它们被调用之后会离开函数作用域，所以它们已经没有用了，可以被回收了。然而，引用计数算法考虑到它们互相都有至少一次引用，所以它们不会被回收。

##### 标记-清除

这个算法把“对象是否不再需要”简化定义为“对象是否可以获得”。

这个算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。垃圾回收器将定期从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和收集所有不能获得的对象。

这个算法比前一个要好，因为“有零引用的对象”总是不可获得的，但是相反却不一定，参考“循环引用”。

从2012年起，所有现代浏览器都使用了标记-清除垃圾回收算法。所有对JavaScript垃圾回收算法的改进都是基于标记-清除算法的改进，并没有改进标记-清除算法本身和它对“对象是否不再需要”的简化定义。

> 循环引用不再是问题了
>
> 在上面的示例中，函数调用返回之后，两个对象从全局对象出发无法获取。因此，他们将会被垃圾回收器回收。第二个示例同样，一旦 div 和其事件处理无法从根获取到，他们将会被垃圾回收器回收。
>
> 限制: 那些无法从根对象查询到的对象都将被清除
>
> 尽管这是一个限制，但实践中我们很少会碰到类似的情况，所以开发者不太会去关心垃圾回收机制。


#### 参考资料

1. [高级前端基础-JavaScript抽象语法树AST](https://segmentfault.com/a/1190000018532745)
2. [编译器和解释器：V8是如何执行一段JavaScript代码的？](https://time.geekbang.org/column/article/131887)
3. [内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)

