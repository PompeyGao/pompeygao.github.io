---
layout:     post
title:      "JS 常见的 6 种继承方式"
subtitle:   ""
date:       2020-09-12
author:     "pompeygao"
header-img: "img/js.jpg"
tags:
    - JS

---



#### 一、继承的概念

------

继承是面向对象的，使用这种方式我们可以更好地复用以前的开发代码，缩短开发的周期、提升开发效率。

举个一个经典的例子。

先定义一个类（Class）叫汽车，汽车的属性包括颜色、轮胎、品牌、速度、排气量等，由汽车这个类可以派生出“轿车”和“货车”两个类，那么可以在汽车的基础属性上，为轿车添加一个后备厢、给货车添加一个大货箱。这样轿车和货车就是不一样的，但是二者都属于汽车这个类，这样从这个例子中就能详细说明汽车、轿车以及卡车之间的继承关系。

继承可以使得子类具有父类的各种方法和属性，比如上面的例子中“轿车” 和 “货车” 分别继承了汽车的属性，而不需要再次在“轿车”中定义汽车已经有的属性。在“轿车”继承“汽车”的同时，也可以重新定义汽车的某些属性，并重写或覆盖某些属性和方法，使其获得与“汽车”这个父类不同的属性和方法。

#### 二、JS实现继承的几种方式

---

##### 1. 原型链继承

​	原型链继承是较为常见的继承方式之一，其中涉及的构造函数、原型和实例，三者之间存在着一定的关系，即每一个构造函数都有一个原型对象，原型对象包含一个指向构造函数的指针，而实例则包含一个原型对象的指针。

代码示例：

```js
function Parent1() {
    this.name = "Parent1";
    this.play = [1,2];
}

function Child1() {
    this.type = "Child1";
}

Child1.prototype = new Parent1();
console.log(new Child1()); 
// {
//	type: "Child1", 
//	__proto__: {
//		name: "Parent1"
//		play: [1, 2]}
// }
```

上面的代码虽然对父类的方法和属性都能访问，但存在一个潜在的问题，

```js
const c1 = new Child1();
const c2 = new Child1();
c1.play.push(3);
console.log(c1.play); // [1, 2, 3]
console.log(c2.play); // [1, 2, 3]
```

改变`c1.play`的值，`c2.play`的值也跟着改变。**因为两个实例使用的是同一个原型对象，它们的内存空间是共享的，当一个发生变化，另一个也会随之变化，这就是使用原型链继承方式的一个缺点**。



##### 2. 构造函数继承（借助call）

代码示例：

```js
function Parent() {
	this.name = "Parent";
}
Parent.prototype.getName = function (){
    return this.name;
}

function Child(){
    Parent.call(this);
    this.type = "Child";
}

let child = new Child();
console.log(child); // {name: "Parent", type: "Child"}
console.log(child.getName()); // error: child.getName is not a function
```

可以看到最后打印的 child 在控制台显示，除了 Child 的属性 type 之外，也继承了 Parent 的属性 name。这样写的时候子类虽然能够拿到父类的属性值，解决了第一种继承方式的弊端，但问题是，**父类原型对象中一旦存在父类之前自己定义的方法，那么子类将无法继承这些方法**。构造函数实现继承的优缺点，**它使父类的引用属性不会被共享，优化了第一种继承方式的弊端；但是随之而来的缺点也比较明显——只能继承父类的实例属性和方法，不能继承原型属性或者方法**。



##### 3. 组合继承（1 和 2 组合）

代码示例：

```js
function Parent() {
    this.name = "P";
    this.play = [1,2];
}
Parent.prototype.getName = function (){
    return this.name;
}
function Child() {
    // 第二次调用Parnent()
    Parent.call(this);
    this.type = "C";
}
// 第一次调用Parent()
Child.prototype = new Parent();
// 手动挂载构造器，指向自己的构造函数
Child.prototype.constructor = Child;
const c3 = new Child();
const c4 = new Child();
c3.play.push(3);
console.log(c3.play); // [1, 2, 3]
console.log(c4.play); // [1, 2]
console.log(c3.getName()); // P
console.log(c4.getName()); // P
```

但是这里又增加了一个新问题：通过注释我们可以看到 Parent 执行了两次，第一次是改变Child 的 prototype 的时候，第二次是通过 call 方法调用 Parent 的时候，那么 Parent 多构造一次就多进行了一次性能开销，这是我们不愿看到的。

那么是否有更好的办法解决这个问题呢？请你再往下学习，下面的第6种继承方式可以更好地解决这里的问题。

上面介绍的更多是围绕着构造函数的方式，那么对于 JavaScript 的普通对象，怎么实现继承呢？

##### 4. 原型式继承

这里要用到 ES5 里面的 `Object.create` 方法，这个方法接收两个参数：一是用作新对象原型的对象、二是为新对象定义额外属性的对象（可选参数）。

代码示例：

```js
let parent = {
    name: "p",
    play: [1, 2],
    getName: function() {
        return this.name;
    }
};

let p1 = Object.create(parent);
p1.name = "Tom";
p1.play.push("p1");

let p2 = Object.create(parent);
p2.play.push("p2");

console.log(p1.name);// "Tom"
// p1 继承了 parent 的 name 属性，然后又进行重新赋值
console.log(p1.name === p1.getName()); // true
// p1 继承的 getName 方法检查自己的 name 是否和属性里的值一样
console.log(p2.name); // "p"
// p2 继承了 parent 的 name 属性
console.log(p1.play); // [1, 2, "p1", "p2"]
// 
console.log(p2.play); // [1, 2, "p2", "p2"]
```

从上面的代码中可以看到，通过 `Object.create` 这个方法可以实现普通对象的继承，不仅仅能继承属性，同样也可以继承 getName 的方法。 缺点也很明显，多个实例的引用类型属性指向相同的内存，存在篡改的可能

##### 5. 寄生式继承

使用原型式继承可以获得一份目标对象的浅拷贝，然后利用这个浅拷贝的能力再进行增强，添加一些方法，这样的继承方式就叫做寄生式继承。

虽然其优缺点和原型式继承一样，但是对于普通对象的继承方式来说，寄生式继承相比于原型式继承，还在父类基础上添加了更多的方法。

代码示例：

```js
let parent = {
    name: "p",
    play: [1, 2],
    getName: function(){
        return this.name;
    }
}

function clone(original){
    let clone = Object.create(original);
    clone.getPlay = function () {
        return this.play;
    }
    return clone;
}
let p1 = clone(parent);

console.log(p1.getName()); // "p"
console.log(p1.getPlay()); // [1, 2]
```

从最后的输出结果中可以看到，p1通过 clone 的方法，增加了 getPlay的方法，从而使 p1这个普通对象在继承过程中又增加了一个方法，这样的继承方式就是寄生式继承。



在上面第三种组合继承方式中提到了一些弊端，即两次调用父类的构造函数造成浪费，下面要介绍的寄生组合继承就可以解决这个问题。

##### 6. 寄生组合式继承

结合第四种中提及的继承方式，解决普通对象的继承问题的 Object.create 方法，我们在前面这几种继承方式的优缺点基础上进行改造，得出了寄生组合式的继承方式，这也是所有继承方式里面相对最优的继承方式。

代码示例：

```js
function clone(parent, child) {
  // 这里改用 Object.create 就可以减少组合继承中多进行一次构造的过程
  child.prototype = Object.create(parent.prototype);
  child.prototype.constructor = child;
}

function Parent6() {
  this.name = "parent6";
  this.play = [1, 2, 3];
}
Parent6.prototype.getName = function() {
  return this.name;
};
function Child6() {
  Parent6.call(this);
  this.friends = "child5";
}

clone(Parent6, Child6);

Child6.prototype.getFriends = function() {
  return this.friends;
};

let person6 = new Child6();
console.log(person6);
console.log(person6.getName());
console.log(person6.getFriends());
```

##### ES6 的 extends 关键字实现逻辑

我们可以利用 ES6 里的 extends 的语法糖，使用关键词很容易直接实现 JavaScript 的继承，但是如果想深入了解 extends 语法糖是怎么实现的，就得深入研究 extends 的底层逻辑。

代码示例：

```js
class Person {
    constructor(name) {
        this.name = name;
    }
    // 原型方法
    // 即Person.prototype.getName = function (){}
    // 下面可以简写为 getName() {}
    getName = function () {
        return this.name
    }
}
class Gamer extends Person {
    constructor(name, age) {
        // 子类中存在构造函数，则需要在使用 this 之前首先调用super()
        super(name);
        this.age = age;
    }
}
const Yi = new Gamer("Master Yi", 20);
Yi.getName();
```

利用 babel 将 ES6 的代码编译成 ES5 , 如下：

```js
function _possibleConstructorReturn(self, call) {
  // ...
  return call && (typeof call === "object" || typeof call === "function")
    ? call
    : self;
}

function _inherits(subClass, superClass) {
  // 这里可以看到
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true,
    },
  });
  if (superClass)
    Object.setPrototypeOf
      ? Object.setPrototypeOf(subClass, superClass)
      : (subClass.__proto__ = superClass);
}

var Parent = function Parent() {
  // 验证是否是 Parent 构造出来的 this
  _classCallCheck(this, Parent);
};

var Child = (function (_Parent) {
  _inherits(Child, _Parent);
  function Child() {
    _classCallCheck(this, Child);
    return _possibleConstructorReturn(
      this,
      (Child.__proto__ || Object.getPrototypeOf(Child)).apply(this, arguments)
    );
  }
  return Child;
})(Parent);
```

从上面编译完成的源码中可以看到，它采用的也是寄生组合继承方式，因此也证明了这种方式是较优的解决继承的方式。






#### 参考资料

1. [JavaScript 核心原理精讲](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=601&sid=20-h5Url-0#/detail/pc?id=6176)

