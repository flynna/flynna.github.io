---
title: javascript[ES6] 中的箭头函数学习及this理解
date: 2022-07-20 23:13:06
updated: 2022-07-22 23:54:15
tags:
  - javascript
  - 箭头函数
  - es6
categories:
  - 技术分享
  - ECMAScript
---

### 箭头函数是什么？

简单的说，箭头函数就是对匿名函数的简化。

作为`ES6`的一种新规范，箭头函数的优点不仅在于写法上的简化，而且能够根据情况，与匿名函数之间切换使用，使`this`指向不同的值。

`so`，箭头函数如何使用呢？

<!-- more -->

### 格式用法

一般形式上的箭头函数长这样~~（看了案例还是不明白的，可以运行代码看一下结果，或许你就明白了）~~

```javascript
() => {
  return;
};
```

#### **单参数单返回语句**

> 单参数：在箭头函数中参数的括号可以省略，但不建议~~一般项目规范箭头函数，参数位置必须加括号~~

> 单返回语句：函数体 {} 括号可以省略。（**特殊说明：注意返回对象时的书写格式，避免与函数体的 {} 冲突**，~~错误示范：y => { y };~~）

```javascript
x => x*2;
y => ({ y });

// 等价写法：
(x) => x*2;
(x) => { return x*2 }
(y) => y = { y }; // (y) => (y = { y }); // (y) => y = ({ y });
(y) => { return { y } }

// 等价的匿名函数写法
function(x) {
	return x*2;
}
function(y) {
	return { y };
}
```

> 当只有一个执行语句，且无需返回数据时，可以返回可执行代码：**此时返回的执行语句必须用小括号包裹**

```javascript
let y;
(x) => (y = x);

// 等价的匿名函数写法
function(x) {
  y = x;
}
```

#### **单参数多执行语句**

```javascript
x => {
  const flag = x > 5;
  return flag ? x : x * 5;
}

// 等价写法：
(x) => {
  return x > 5 ? x : x * 5;
}

// 等价的匿名函数写法
function(x) {
	return x > 5 ? x : x * 5;
}
```

#### **多参数单返回语句**

与单参数单返回语句的**唯一区别**就是参数必须用小括号包裹。

```javascript
(x, y) => x*y;

// 等价写法：
(x, y) => { return x*y; }

// 等价的匿名函数写法
function(x, y) {
	return x*y;
}
```

#### **多参数多执行语句**

与单参数多执行语句的**唯一区别**就是参数必须用小括号包裹。

```javascript
(x, y) => {
  const flag = x > 5;
  return flag ? y : x * y;
}

// 等价的匿名函数写法
function(x, y) {
	return x > 5 ? y : x * y;
}
```

#### **没有参数**

**参数位置必须加括号**，其他单/多语句执行的书写格式同上。

```javascript
() => 99;

// 等价的匿名函数写法
function() {
	return 99;
}
```

#### **可变参数**

利用`rest`变量取值，可以利用...对个数不明参数的参数进行囊括，得到变量组成的数组。

```javascript
const fn = (x, y, ...rest) => {
  let sum = x + y;

  console.log(rest.length); // 3

  for (let i = 0; i < rest.length; i++) {
    sum += rest[i];

    console.log(sum);
    // sum += 4; sum += 5;sum += 6  4/8/13/19
  }

  return sum;
};

const result = fn(1, 3, 4, 5, 6);
console.log(result); // 19
```

看完是不是感觉，原来箭头函数这么简单，对没错...就是如此简单。

箭头函数在一定形式上与匿名函数没有太大的差异，写法上除了函数头尾部，函数体依旧是逻辑性的代码。

---

### 与普通匿名函数的区别

- 箭头函数形式上做了改变，简化了函数体。

- 普通函数支持通过`arguments`获取未知个数的实参，而箭头函数不支持`arguments`用法，究其原因：箭头函数没有自身的`this、arguments`，但是...如若它的父级存在，那么在箭头函数内获取的就是其父级对应的`this、arguments`。如果在箭头函数中有这个获取自身的`arguments`需求，可以用`...rest`替代。

- 箭头函数在`ES6`标准下可使用，普通函数则没有这个限制。

- `this`指向的修改。箭头函数的`this`指向外部，常在对类的方法进行构造时使用，使函数体内的`this`始终指向这个类。如果需要 `this`指向当前源，建议使用普通函数。

<div class="primary">

> 补充说明：

</div>

> **箭头函数除了没有自身的 this、arguments 外，new.target、super 也没有**。（注意只是自身没有，它会从它的外部获取，就近原则，如果外部还是没有自身的，继续向外...）；另：**不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数**。

> **箭头函数有作用域**（词法作用域），词法作用域简单来讲就是，一切变量（包括 this）都根据作用域链来查找。

> 至于为啥箭头函数自身没有？这玩意儿连 ES6 规范原文里都没写原因（[https://262.ecma-international.org/6.0/#sec-arrow-function-definitions](https://262.ecma-international.org/6.0/#sec-arrow-function-definitions)），只要知道是规范标准就行，也可以理解为是箭头函数的特性...

### 箭头函数中的`this`

> 请记住：箭头函数自身没有 this，如若你在箭头函数中使用了它，那么它其实是**箭头函数父级**的 this。

怎么理解这个**箭头函数的父级**？可以参考**位置**，它的外层的`console.log`的`this`值即为它的`this`值。

```javascript
// 事件函数中：
// 普通匿名函数中的 this，指向了事件源 ele
ele.onclick = function () {
  console.log(this); // ele

  const fn = function () {
    // 普通匿名函数 this，执行了它的调用者 window
    console.log(this); // window
  };

  fn();
};

// 下面的箭头函数外部，或者可以理解为其 父级
console.log(this);

ele.onclick = () => {
  // 箭头函数内部
  console.log(this); // 与上面那个 log 一致，都是指向的 window
};
```

又比如：

```javascript
function parentFn() {
  // 这是普通匿名函数的内部，它的调用者是 window，所以下面这句 log 就是 window
  console.log(this);

  const fn = () => {
    // 这里是箭头函数内部，上面那句 log 就是外部，而外部 this 指向了 window，所以下面这句 log 的值也是 window
    console.log(this);
  };

  fn();
}

parentFn(); // window.parentFn()
```

结合上面两种方式：

```javascript
ele.onclick = function () {
  // 这里是普通匿名函数的内部，this 指向了它的调用者：ele
  console.log(this);

  const fn = () => {
    // 这里的 this 取决于外面的 log，同为 ele
    console.log(this);
  };
};
```

#### 谈谈我对箭头函数与普通匿名函数的`this`区分理解

从上面的代码，我们可以看到一个现象：就是只要有箭头函数存在，那么它的`this`，始终都是和外面一层打印的`this`一样。而普通的匿名函数，不管是否处于其他匿名函数/箭头函数内部（不管层级多深），它自身的`this`都只会指向它的调用者。

**普通匿名函数的调用者又是个啥？**

> 通过 fn() 方式调用：函数在通过函数名调用时，调用者就是 window，可以理解为 window.fn()，只不过 window 省略了。

> 通过 obj.fn() 方式调用：此时这个普通匿名函数是对象身上一个方法，对象调用自身的方法，调用者就是这个 obj。

> 如果是事件处理函数，例如 ele.onclick 触发的事件，虽然没有显式的调用者，但是却由 ele 引起，它的调用者可以理解为 ele。

<div class="success">

> 总结：箭头函数 this 取决于其函数体的位置（与调用者无关），值始终与其外层的 this 一致。普通匿名函数 this 取决于其调用者（与位置无关），值始终是调用者。

</div>
