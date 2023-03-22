---
title: javascript 中的符号集合
date: 2022-09-15 15:12:50
updated: 2022-09-15 15:12:50
tags:
  - javascript
categories:
  - 技术分享
  - JavaScript
---

逆水行舟，不进则退。

随着`ECMAScript`标准的不断更新迭代，你会发现在`js`代码中，符号越来越多，稍不学习就会不懂其含义，更别谈使用了。

本文就个人在项目中使用到的符号结合代码进行介绍 😮‍💨😮‍💨😮‍💨~~当然类似基础运算、幂运算、自增自减、大小比较、三目运算符等等就忽略了~~...

<!-- more -->

### 逻辑位运算`|、&、~、^、<<、>>`

将需要运算的两个值转为二进制数，进行位**或、与、非、异或、左移、右移**运算。 对应衍生的有`~=、&=、|=、^=、<<=、>>=`。🙄🙄
🙄~~这个相对来说用的很少，但你必须要了解。~~

`eg.`

```javascript
let a = 5;
a &= 3; // a = a & 3 = 5 & 3 = 1;
```

> `>>` 和 `<<` 右移左移运算，将操作数转为二进制后，返回移动对应位数后的新值，移动过程中，超出部分去除，补充部分补 0

```javascript
const a = 5; //  00000000000000000000000000000101
const b = 2; //  00000000000000000000000000000010
console.log(a >> b); //  00000000000000000000000000000001
```

除此之外还有无符号右位移 `>>>`，与右位移 `>>`的区别就是前者的符号位始终为 0，即正数。

### 逗号操作符

对逗号表达式进行求值，并返回最后一项的值。~~这个目前也不是特别常见~~

```javascript
console.log((2, 3)); // 3
let i = 1;
a = (i++, i); // a = 2
```

### 解构赋值运算符`...`

比较常见的一个需求，同时在面试的时候也有可能会被问到。**如何对未知 keys 的对象解构，拿到除属性 xx 外的其他属性集合？**

有的童鞋可能马上就想到`copy 再 delete obj.xx`...~~当然我不建议这么干~~

```javascript
const obj = { a: 1, b: 'b', c: false };
const { a, ...other } = obj;
console.log(other); // { b: 'b', c: false }

const arr = [1, 2, 3, 4, 5];
const [arr0, ...arrOther] = arr;
console.log(arr0); // 1
console.log(arrOther); // [2, 3, 4, 5]
```

### 可选链操作符`?.`

**很常用，必须掌握**。允许读取位于连接对象链深处的属性的值，作用类似`.`区别就是前者`?.`允许对象为`null/undefined`，当访问的对象为`null/undefined`时，不会执行`.`后面的逻辑，且不会抛出异常（此时返回`undefined`）。除此外，`?.`也可作用于数组、函数...`eg.`

```javascript
const obj = {}; // 假设 obj 存在一个 子属性 hh: { a: 11 } （比如通过接口返回的一个字段属性，可能有值，可能没有）
const fn = void 0; // 假设这个函数是 undefined，（比如某些通过 props 传递 fn 场景，可能为 undefined）

console.log(obj.hh?.a); // 此处 hh 没有值，返回一个 undefined， 如果不写 ?. 直接 obj.hh.a 会抛异常 a of undefined
console.log(fn?.(args)); // 函数调用，fn 为空，返回一个 undefined，（）内传入函数实参 args

// 数组值访问 (数组可能为 null 或 undefined) 同上
// arr?.[index]?.xxx;
// 支持链式操作
// obj?.xx?.xx?.xxx
```

### `??`空值合并运算符

当左侧操作数为`null`或`undefined`时，其返回右侧的操作数，否则返回左侧的操作数 **(注意`""、0、false、NaN`等值会返回左侧值)**

- 短路运算

与逻辑或`||`运算符不同，逻辑或会在左操作数为`false`值时返回右侧操作数。也就是说，如果你使用`||`来为某些变量设置默认的值时，你可能会遇到意料之外的行为。

```typescript
const a = null || 'saasf';
console.log(a); // saasf

const b = 0 ?? 1;
console.log(b); // 0
```

- 与`&&`或`||`操作符共用

若空值合并运算符`??`直接与 `AND（&&）`和`OR（||）`操作符组合使用`??`是不行的。这种情况下会抛出`SyntaxError`, 所以在使用时需要显示表明优先级。

```typescript
(null || undefined) ?? 'foo'; // 返回 "foo"
```

- 与`?.`联动

一般用于接口数据使用或者`props`传递等场景

```tsx
interface PageAProps {
  title?: string;
}

const PageA = (props: PageAProps) => {
  return <div>{props?.title ?? '页面A'}</div>;
};
// props?.title ?? '页面A' 联动设置默认的值
```
