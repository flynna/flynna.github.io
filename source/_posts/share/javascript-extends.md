---
title: Javascript 中的继承方案及其实现
date: 2022-08-05 18:30:31
updated: 2022-08-05 18:30:31
tags:
  - javascript
  - 继承
categories:
  - 技术分享
  - JavaScript
---

### 继承(extends) 导语

任何一门面向对象的语言都有继承，继承简单的说 即为子类继承父类的属性和方法，并且在继承以后可以对这些属性和方法进行操作及使用。

举个栗子：（你的父亲很富有，你继承你父亲的财产以后，你也就变得和他一样富有。）

`js`虽然是一门弱类型语言，但是却给我们提供了一种很好的实现继承的方法——基于原型实现 不知道原型的盆友，[戳这里](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Object_prototypes)

<!-- more -->

### 继承的六种实现

`主要对继承熟悉掌握，所以没有单独写测试用例进行测试。`

定义基类(父类)`Man`：

```javascript
function Man(params) {
  this.name = params.name;

  this.say = function () {
    console.log('my name is ' + this.name);
  };
}

// 原型上的方法
Man.prototype.eat = function (food) {
  console.log(this.name + '正在吃' + food);
};
```

#### 拷贝继承

> 实现：枚举父类实例的方法及属性，拷贝到子类的原型上

```javascript
function Student(params) {
  const man1 = new Man(params);

  for (const key in man1) {
    // 需要拷贝基类原型上的属性和方法，所以移除了 hasOwnProperty 判断
    // if (Object.hasOwnProperty.call(man1, key)) {
    Student.prototype[key] = man1[key];
    // }
  }
}

// test
const jack = new Student({ name: 'jack' });

console.log(jack.name); // jack

jack.say(); // my name is jack
jack.eat('零食'); // jack正在吃零食

console.log(jack instanceof Man); // false
console.log(jack instanceof Student); // true
```

> 优缺点：

- 支持多继承

- 效率较低，内存占用较高。（因为要拷贝父类的属性）

- 无法获取父类不可枚举的方法。（不可枚举方法，不能使用 `for in` 访问）

#### 原型链继承

> 实现：子类的原型指向父类的实例

```javascript
function Student(params) {
  this.name = params.name;
  this.age = params.age;
  this.sex = params.sex;

  this.sayHi = function () {
    console.log('hello ' + this.name + ', my age is ' + this.age);
  };
}

Student.prototype = new Man({});

// test
const rose = new Student({ name: 'rose', age: 18 });

console.log(rose.name); // rose
rose.say(); // my name is rose
rose.sayHi(); // hello rose, my age is 18
```

> 优缺点：

- 非常纯粹的继承关系，实例是子类的实例，也是父类的实例。

- 父类新增原型方法/原型属性，子类也能访问到。

- 简单易实现。

- 无法实现多继承，且来自原型上的引用属性所有实例共享！

- 子类需要对自身(非原型)属性重载，因为基类(非原型)属性无法在继承时被实例化-基于原型链继承导致。

#### 构造继承

> 实现：

```javascript
function Student(params) {
  Man.call(this, params);
}

// test
const m1 = new Student({ name: 'mm' });

m1.say(); // my name is mm
console.log(m1 instanceof Man); // false
console.log(m1 instanceof Student); // true
```

> 优缺点：

- 解决子类实例共享父类引用属性的问题. 可以实现多继承（`call`多个父类对象）。

- 实例并不是父类的实例，只是子类的实例。

- 只能继承父类的实例属性和方法，不能继承原型属性/方法

- 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

#### 组合继承

> 实现：

```javascript
function Student(params) {
  Man.call(this, params);
}

Student.prototype = new Man({});
Student.prototype.constructor = Student;

// test
let m2 = new Student({ name: 'mm2' });

console.log(m2.name); // mm2
m2.say(); // my name is mm2
console.log(m2 instanceof Man); // true
console.log(m2 instanceof Student); // true
```

> 优缺点：

- 可以继承实例属性/方法，也可以继承原型属性/方法。

- 实例既是父类的实例，也是子类的实例。

- 可实现函数方法的复用

- 调用了两次父类构造函数，生成了两份实例.

#### 寄生继承

`比组合多了一个立即执行函数, 消除多余的实例`

> 实现：

```javascript
function Student(params) {
  Man.call(this, params);
}

(function () {
  // 创建一个没有实例方法的类
  const Super = function () {};

  Super.prototype = Man.prototype;

  // 将实例作为子类的原型
  Student.prototype = new Super();
})();
```

> 优缺点：

- 实现相对较复杂一点。

#### `ES6`语法糖

> 实现：

```javascript
class Man {
  constructor(name) {
    this.name = name;
  }
  say() {
    console.log('my name is ' + this.name);
  }
}
class Student extends Man {
  constructor(name) {
    super(name);
  }
}

// test
const m3 = new Student('mm3');

m3.say(); // my name is mm3
console.log(m3 instanceof Man); // true
console.log(m3 instanceof Student); // true
```

> 优缺点：

- `ECMAscript6`标准的继承方案

`继承能够在一定程度上帮助我们快速便捷的使用父类的属性及方法，在 js 中实现继承的方案大致就这六种，基本都是依托原型建立的继承关系。其中的拷贝继承个人不推荐使用，ES6继承能用则用`
