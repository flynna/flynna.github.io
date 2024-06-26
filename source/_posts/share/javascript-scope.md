---
title: Javascript 作用域理解
date: 2022-08-04 21:34:45
updated: 2022-08-04 21:34:45
tags:
  - javascript
  - 作用域
  - 预解析
  - 变量提升
  - 暂时性死区
categories:
  - 技术分享
  - JavaScript
---

### 什么是作用域？

域：范围、区域

作用域：变量起作用的一个域 --> 变量起作用的一个范围

那么，在`js`中的作用域又是怎么设定的呢？直接写在`script`脚本的最外层和写在函数体内又有啥不同？

<!-- more -->

### 作用域分类

在`es5 即 ECMAscript5`根据范围的不同，作用域分为了**全局作用域**和**局部作用域**。**这个标准中只有函数拥有局部作用域。**

在`es6 中新增了块作用域。`

#### 全局作用域

在全局作用域下声明的变量就是全局变量。而**全局作用域是唯一的，只有一个**。

你可以理解为： `<script>`标签下的最外层变量或者函数、以及**所有能够在`js`代码的任何地方能够访问的对象拥有全局作用域。**，那么不难得知：**`window`对象拥有全局作用域。**

例如代码：

```html
<script>
  var a = 'test';

  function fn1() {
    var b = 1;

    c = {};
  }

  var d = function () {
    var f = true;
  };
</script>
```

在这个代码片段中，变量`a`、函数`fn1，d`、属性`c（为什么是属性？）`，均能够在代码的任何地方进行访问（也是‘所谓的’处在最外层），都处在同一个全局作用域中。`fn1 中的 c 会不断向上寻址，直到全局作用域，如果没有任何 var 声明，那么它的赋值则会被当做是 window 的外挂属性进行设置并赋值，让使用者产生了一种不用声明的假象。`

#### 局部作用域（私有作用域）

`es5`标准中，只有函数有私有作用域。在函数体内定义的一切变量和函数，影响范围和可访问范围仅在函数体内，如上面代码片段中的变量`b 和 f`。

> **每个函数都有一个独立的私有作用域。**

#### **块级作用域**

##### **why?**

这个是`es6`提出的概念，那么为什么需要块作用域？

> 块作用域的出现，避免了内存泄漏，例如下面代码：用来计数的循环变量泄露为全局变量

```javascript
for (var i = 0; i < 10; i++) {
  // ...
}
```

> 预解析导致的误判，误判外层变量被覆盖。~~（这个问题我认为影响还没那么大，毕竟只要写的够规范，就不存在）~~

```javascript
var i = 1;

function fn() {
  i *= 10; // NaN

  if (true) {
    var i = 20;
  }
}

fn();
console.log(i);
```

大家可以猜一下结果，我先说错误答案：~~`10, 20`~~，相信有一部分同志已经绕进去了。

其实细看，在代码运行到`fn()`时，开始对`fn`函数体进行预解析，这个过程也可以理解为‘变量提升’，然后再开始执行函数体代码，此时的`i *= 10`其实是对局部变量的赋值操作。说到这就不难发现，此时的全局变量`i`值为`1`。

##### 定义和规范

任何`{}`包裹的代码都可以称之为块，而包裹在内的变量都会受块作用域的影响。而让这个块作用域生效的，正是`let 和 const`。

通过`let`和`const`声明的变量，无法在块的外部访问。而两者的区别，就是`let`的值可以被修改，`const`定义的是常量，无法被修改。

与`var`的区别：

> **let 和 const 定义的变量是会提升的，只是它们提升的时候不会进行默认初始化，使得它们无法被访问（因为这些变量在暂时性死区 TDZ【temporal dead zone】 里）**。

> **let 和 const 声明的变量，仅在‘块作用域’中生效。**

### 作用域变量的访问规则

#### 预解析(变量提升)

在代码被执行之前，会在当前的作用域内做一次预解析，该过程会将该作用域内 `var` 和 `function` 等关键字声明的变量和函数创建并赋值为 `undefined`，并在 `var or function` 代码被执行时正式赋值，如果在声明前访问，则会提示为 `undefined`（`let 和 const` 除外）。

**函数关键字声明的变量提升优先级最高**，意味着如果存在**同名的声明**，那么在其实际声明的代码位置之前访问，会返回函数，同名声明位置之后访问，则返回新的值（可以理解为在**实际执行前访问是函数**，**实际执行时函数所属代码块不执行也不编译，所以会被覆盖**）:

```js
console.log(fn); // function
var fn = 'aaa';
function fn() {};
console.log(fn); // aaa

// or
console.log(f); // function
function f() {};
var f = 'aaa';
console.log(f); // aaa

```

**只有关键字声明（`var/let/const/function`）的变量才会被预解析创建**，如下示例，没有通过关键字声明则会抛出异常：

```javascript
console.log(a); // a is not defined
a = 1;
```

#### 暂时性死区 `TDZ`

前面提到，`let 和 const` 声明的变量，在其作用域内同样会经历预解析的过程，但不同于 `var`，`let 和 const` 在其声明语句前的区域该变量不可用，而这块区域，就被称之为暂时性死区（`TDZ...temporal dead zone`）。

暂时性死区是 `JavaScript` 中的一种行为，当使用 `let` 和 `const` 关键字声明变量时发生，但不影响 `var`。在 `ECMAScript 6` 中，在声明之前（其作用域范围内）访问 `let or const` 变量会导致 `ReferenceError`。

#### 作用域链

当我们访问某个变量时，会优先在该作用域内查找（`如果在声明之前访问，则结果同上预解析过程和暂时性死区解释`），如果当前作用域没有该变量的预解析声明，则继续向上一级作用域查找，依次查找直到全局作用域。找到则使用，未找到则抛出异常 `xxx is not defined`。

**注意： 变量查找，只能往上查找，不能往下。总结就是：全局不能访问局部，局部可以拿到全局。**

```javascript
let n = 10,
  m = 20;

function fn2(n) {
  console.log(n); // 15
  console.log(m); // 20
}

fn2(15);
```

`n`会优先从局部变量读取，`m`局部没有声明，向上寻址，在全局作用域中找到。

### 作用域变量的赋值规则

注意：**赋值的这个变量的寻址过程需要遵循作用域变量的访问规则**。

> 如果自己的作用域，有这个变量，那么直接给自己作用域的这个变量赋值。

> 如果自己的作用域没有这个变量，那就往上一级查找，如果找到，那就赋值，如果没有找到，就继续往上一级查找...直到全局作用域，如果找到，那就赋值，找不到就会被当做是 window 的一个属性，并进行赋值。

**注意这个寻找的过程，只有 var 变量允许在声明之前赋值，let 不允许。**

### 作用域变量的生命周期

`就是作用域变量在内存之中存活的时间。`

> 全局变量：生命周期是和程序同步的， 程序不关闭，变量就一直存在。

> 局部变量：生命周期是和函数执行同步的，函数执行结束变量就被删除了。

> 块作用域变量：仅存活与块代码执行时，执行结束内存就会释放。

综上，全局变量的大量使用会导致程序变得更重。如果代码逻辑写的不够严谨，很容易造成内存泄漏吗（例如：不声明变量直接赋值，会挂载到 window 对象），影响到我们程序的运行效率。~~如果可能，还是少设计一点全局变量吧！对大家都好...hh~~

> 建议：任何一个独立的 script 标签下的代码，都应用匿名函数包裹，并自调用。避免污染全局，并造成其他未知的程序错误。

例如：

```javascript
(function () {
  // ...
})();

// 或者：这是自调用的几种写法，推荐用上面那种，下面的仅做了解
~function () {};
!function () {};
+function () {};
```
