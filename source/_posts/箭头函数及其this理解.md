---
title: 箭头函数及其this理解
date: 2021-05-16 17:00:00
tags: [ES6, this, 箭头函数]
# type: "categories"
categories: [大前端, JavaScript]
---

`简单的说，箭头函数就是对匿名函数的简化`

<font color='#a4d3ef'>作为ES6的一种新规范，箭头函数的优点不仅在于写法上的简化，而且能够根据情况，与匿名函数之间切换使用，使this指向不同的值。</font>

那么，箭头函数如何使用呢？ 

- <a href='#use-target'>用法及格式</a>
- <a href='#this-target'>this指向理解</a>
- <a href='#difference-target'>与普通函数区别</a>


<h5 id='use-target'>用法及格式</h5>
一般形式上的箭头函数长这样  （看了案例还是不明白的，可以运行代码看一下结果，或许你就明白了）
```javascript
() => {
	return 
}
```
- 单参数及单语句时 (单参数，在箭头函数中参数的括号可以省略)
```javascript
x => x*2  
// (x) => x*2  这样写也是可以的
// 普通匿名函数
function (x) {
	return x*2
}
```
- 多参数单语句 （多个参数，参数必须由括号括起来）

```javascript
(x, y) => x*y  
// 普通匿名函数
function (x, y) {
	return x*y
}
```
- 单参数多执行语句时
```javascript
x => {
	if (x > 5) {
		x *= 5		// x = x * 5
	} else {
		x = 5
	}
	return x
} 
// 普通匿名函数
function (x) {
	if (x > 5) {
		x *= 5		// x = x * 5
	} else {
		x = 5
	}
	return x
}
```
- 多参数多执行语句

```javascript
(x, y) => {
	if (x > 5) {
		x *= 5		// x = x * 5
		y = x
	} else {
		y *= 5
		x = y
	}
	return x + y
} 
// 普通匿名函数
function (x, y) {
	if (x > 5) {
		x *= 5		// x = x * 5
		y = x
	} else {
		y *= 5
		x = y
	}
	return x + y
}
```
- 没有参数时 （参数位置必须加括号） 多语句情况同上

```javascript
() => 99
// 普通匿名函数
function () {
	return 99
}
```

- 可变参数 （利用rest取参数值）
这里扩展了知识点  rest，可以利用... 对个数不明参数的参数进行囊括。函数内，可以通过数组的方式。对rest取值，即可拿到实参。
```javascript
(x, y, ...rest) =>{
    let i,sum = x+y
    for (i=0; i < rest.length; i++){
        sum += rest[i]
    }
    return sum
}

// 例子 表达式函数
let fn = (x, y, ...rest) => {
	let i, sum = x + y             // sum = 4
	console.log(rest.length)        // 3
	for (i = 0; i < rest.length; i++) {
		sum += rest[i]
		console.log(sum)           
        // sum += 4; sum += 5;sum += 6  4/8/13/19
		console.log(typeof sum)    // number
	}
	console.log(sum)               // 19
	return sum
}
const result = fn(1,3,4,5,6)
console.log(result)                // 19
```
- 如果函数返回值是一个对象
```javascript
// 这样写会出错
x => {foo: x} // 这和函数体{}有冲突
// 以下三种均可
x => { return {foo: x}}
// 或者
x => x = {foo: x}
// 再或者
x => ({foo: x})
```
<font color='#7171c6'>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这些就是常见的箭头函数的写法，可用于一般类型返回值使用。 看完是不是感觉，原来箭头函数这么简单，对没错...就是如此简单。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;箭头函数在一定形式上与匿名函数没有太大的差异，写法上除了函数头尾部，函数体依旧是逻辑性的代码。

<h5 id='this-target'>this指向理解</h5>
<font color='#1c6666'>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;箭头函数的this和外部保持一致。是不是感觉很抽象？ 换句话说，在箭头函数内使用this时，它的this 和在箭头函数外的this指向同一个对象。看代码：</font>

```javascript
console.log(this)	// window
let fun = () => {
	console.log(this)		
}
box.onclick = fun		// fun 函数内 this 指向 window

box.onclick = function(){
    let fun = () => {
		console.log(this)	// this 指向 box
	} 
	console.log(this)		// this 指向 box
    fun()	
}

// 普通匿名函数
box.onclick = function(){
    let fun = function () {
		console.log(this)	// this 指向window
	}
	console.log(this)		// this 指向 box
    fun()	
}
```
- 从上面的代码，我们可以看到，在触发类型事件内，定义了一个箭头函数，它的this 指向了 事件源 box，而在该触发类型事件内定义匿名函数，它的this指向的是 全局window，这是为什么呢?  
- 从概念上看，箭头this的指向和外部保持一致，此时定义的箭头函数外部，即为box的事件函数内部，而事件函数内部的this此时是指向的 box，箭头函数的this也指向了box，完成与外部保持一致，所以console.log两次都是 box。而通过匿名函数方式定义的函数的this，和定义的位置没有关系，和它的调用者有关，fun函数是通过fun()调用的，那么它的this指向的就是window，不明白没关系。 0.0， window.fun() 现在明白了没。

<h5 id='difference-target'>与普通函数的区别</h5>
- 箭头函数形式上做了改变，简化了函数体。
- 普通函数支持通过arguments 获取未知个数的实参，而箭头函数不支持arguments用法。
- 箭头函数在es6标准下可使用，普通函数则没有这个限制。
- this指向的修改。箭头函数的this指向外部，常在对类的方法进行构造时使用，使函数体内的this始终指向这个类。如果需要this指向当前源，则可使用普通函数。