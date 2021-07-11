---
title: javascript 的几种继承方式及优缺点
date: 2021-05-16 15:05:00
tags: [继承]
# type: "categories"
categories: [大前端, JavaScript]
---

`本文主要介绍前端通过JS实现继承的几种方法。`

    - 介绍
    - 实现方式
    	- 实现
    	- 优缺点
    	- 注意点
    - 结语

##### 介绍

- 继承： 任何一门面向对象的语言都有继承，继承简单的说 **即为子类继承父类的属性和方法，并且在继承以后可以对这些属性和方法进行操作及使用。**
  `举个栗子：`（你的父亲很富有，你继承你父亲的财产以后，你也就变得和他一样富有。）

- JS 虽然是一门弱类型语言，但是却给我们提供了一种很好的实现继承的方法——基于原型实现 不知道原型的盆友，戳这里 →[原型](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Object_prototypes)

##### 实现方式

- JS 中的继承可以分为以下六种： - <a href ='#title1' style="color:#00F; text-decoration:none"
   onMouseOver="this.style.color='#F00';this.style.textDecoration='underline'" 
  onMouseOut="this.style.color='#00F';this.style.textDecoration='none'">拷贝继承</a> - <a href ='#title2' style="color:#00F; text-decoration:none"
   onMouseOver="this.style.color='#F00';this.style.textDecoration='underline'" 
  onMouseOut="this.style.color='#00F';this.style.textDecoration='none'">原型链继承<a> - <a href ='#title3' style="color:#00F; text-decoration:none"
   onMouseOver="this.style.color='#F00';this.style.textDecoration='underline'" 
  onMouseOut="this.style.color='#00F';this.style.textDecoration='none'">构造继承</a> - <a href ='#title4' style="color:#00F; text-decoration:none"
   onMouseOver="this.style.color='#F00';this.style.textDecoration='underline'" 
  onMouseOut="this.style.color='#00F';this.style.textDecoration='none'">组合继承</a> - <a href ='#title5' style="color:#00F; text-decoration:none"
   onMouseOver="this.style.color='#F00';this.style.textDecoration='underline'" 
  onMouseOut="this.style.color='#00F';this.style.textDecoration='none'">寄生继承</a> - <a href ='#title6' style="color:#00F; text-decoration:none"
   onMouseOver="this.style.color='#F00';this.style.textDecoration='underline'" 
  onMouseOut="this.style.color='#00F';this.style.textDecoration='none'">ES6 继承(语法糖)</a>

定义一个公用父类 People：

```javascript
function People (name) {
	this.name = name
	this.say = fucntion () {
		console.log('My name is ' + this.name)
	}
}
// 原型上的方法
People.prototype.eat = function (food) {
	console.log(this.name + "正在吃" + food)
}
```

<h6 id='title1'>1.拷贝继承</h6>

- 枚举父类实例的方法及属性，拷贝到子类的原型上

```javascript
function Man(name) {
  let people = new People(name);
  for (const v in people) {
    Man.prototype[v] = people[v];
  }
}
// Test Code
let pp = new Man("JACK");
console.log(pp.name); // JACK
pp.say(); // My name is JACK
console.log(pp instanceof People); // false
console.log(pp instanceof Man); // true
```

- 优缺点：
  ==支持多继承== -效率较低，内存占用较高。（因为要拷贝父类的属性） -无法获取父类不可枚举的方法。（不可枚举方法，不能使用 for in 访问）
- 不建议使用

<h6 id='title2'>2.原型链继承</h6>

- 子类的原型指向父类的实例

```javascript
function Man() {}
// 1.继承传入参数，子类就不能在自身构造函数内添加相应字段，同时所有实例均为同一属性值
Man.prototype = new People("JACK"); // Man.prototype.name = 'JACK'
let jack = new Man();
console.log(jack.name); // JACK

// 2.继承以后重构 为子类添加属性及方法
function Man(name, age) {
  this.name = name;
  this.age = age;
  this.sayHi = function () {
    console.log("hello " + this.name + ", my age is " + this.age);
  };
}
Man.prototype = new People();
let jack = new Man("JACK", 28);
// 继承的方法
jack.say(); // My name is JACK
// 添加的方法
jack.sayHi(); // hello JACK, my age is 28
```

- 优缺点：
  `1.非常纯粹的继承关系，实例是子类的实例，也是父类的实例。 2.父类新增原型方法/原型属性，子类都能访问到。 3.简单，易于实现。`
  第一种方式，要想为子类新增属性和方法，必须要在 new Animal()之后执行。
  ==无法实现多继承。==

- 注意点：
  来自原型的所有引用属性是所有实例共享的。

<h6 id='title3'>3.构造继承</h6>

- 将父类的构造类当成函数进行使用, 通过`call、apply` 修改父类的 this 指向实现继承。

```javascript
function Man(name) {
  People.call(this, name);
}
// Test Code
let jack = new Man("JACK");
console.log(jack.name); // JACK
jack.say(); // My name is JACK
console.log(jack instanceof People); // false
console.log(jack instanceof Man); // true
```

- 优缺点：
  `解决子类实例共享父类引用属性的问题. 可以实现多继承（call多个父类对象）`
  实例并不是父类的实例，只是子类的实例
  只能继承父类的实例属性和方法，不能继承原型属性/方法
  无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

<h6 id='title4'>4.组合继承</h6>

- 原型继承和构造继承的组合体

```javascript
function Man(name) {
  People.call(this, name);
}
Man.prototype = new People();
Man.prototype.constructor = Man;
// Test Code
let jack = new Man("JACK");
console.log(jack.name); //JACK
jack.say(); // My name is JACK
console.log(jack instanceof People); // true
```

- 优缺点
  `可以继承实例属性/方法，也可以继承原型属性/方法； 实例既是父类的实例，也是子类的实例；可实现复用 `
  调用了两次父类构造函数，生成了两份实例.

<h6 id='title5'>5.寄生继承</h6>

- 比组合多了一个立即执行函数, 消除多余的实例

```javascript
function Man(name) {
  People.call(this);
  this.name = name;
}
(function () {
  // 创建一个没有实例方法的类
  let Super = function () {};
  Super.prototype = People.prototype;
  // 将实例作为子类的原型
  Man.prototype = new Super();
})();

// Test Code
let jack = new Man("JACK");
console.log(jack.name); // JACK
jack.say(); // My name is JACK
console.log(jack instanceof People); // true
console.log(jack instanceof Man); //true
```

- 优缺点
`作为继承来讲，已经足够优秀了`
实现相对较复杂一点
<h6 id='title6'>6.ES6语法糖继承</h6>

- ES6 提供的一种很好的继承方式，使用方法也较简单

```javascript
class People {
  constructor(name) {
    this.name = name;
  }
  say() {
    console.log("My name is " + this.name);
  }
}
class Man extends People {
  constructor(name) {
    super(name);
  }
}
// Test Code
let jack = new Man("JACK");
jack.say(); // My name is JACK
console.log(jack instanceof People); // true
console.log(jack instanceof Man); // true
```

- 优缺点
  `ECMAscript6标准的继承方案`

##### 结语

_<font>继承能够在一定程度上帮助我们快速便捷的使用父类的属性及方法，在 js 中实现继承的方案大致就这六种，基本都是依托原型建立的继承关系。其中的拷贝继承个人不推荐使用，ES6 继承能用则用。</font>_
