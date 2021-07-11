---
title: 初步接触typescript
date: 2021-06-15 13:52:33
tags: [ts初识]
# type: "categories"
categories: [大前端, TypeScript]
---

### 语言特性

` TypeScript 是一种给 JavaScript 添加特性的语言扩展。 `

- 类型批注和编译时类型检查、类型推断、类型擦除、接口、枚举、`Mixin`、泛型编程、名字空间、元组、Await、类、模块、lambda 函数的箭头语法、可选参数以及默认参数。



### 安装

> npm install -g typescript
>
> tsc -v // 查看版本
>
> 其他编译参数  https://www.runoob.com/typescript/ts-basic-syntax.html



### 基础类型

  `TypeScript`支持与JavaScript几乎相同的数据类型，此外还提供了实用的枚举类型方便我们使用。

- 数字：number；
- 字符串： string；
- 布尔类型：boolean；
- 数组类型：元素类型后跟 number[]；或者使用数组泛型 Array< number>;
- 元组类型： 允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。如： [string, number]; 
- 枚举类型：用于定义数值集合

```typescript
enum Color { Red, Creen, Blue }

let c: Color = Color.Blue;
```

- any类型：任意类型，可以赋予任意类型的值；
- void类型：与 any 相反，表示没有任何类型；
- null 和 undefined 类型：只能指 null 与 undefined 元素本身；
- never类型： never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值 



### 声明变量

- 指定类型时，变量类型就必须保持一致。类型未设置时，值可以是任意类型

```typescript
let [变量名] : [类型] = 值;
let name: string = '张三';
```

- 类型断言

```typescript
// 此时 oldName 未指定类型 可以是任意类型 (当然这里只是举例， 正常情况下基础类型会通过类型推断得出)
const oldName = '张三';

// 方式一 <类型>值
let name: string = <string>oldName;

// 方式二 值 as 类型
let nName: string = oldName as string;
```

- 缺省声明

```typescript
// [变量名]?: [类型]; or [变量名]!: [类型];
// 通常用于 函数参数声明或对象属性声明; ? 可能存在  !一定存在
```

- 变量作用域： 除了全局和局部变量，这里重点说类作用域

```typescript
// 类变量声明在一个类里头，但在类的方法外面。 该变量可以通过类的对象来访问。类变量也可以是静态的，静态的变量可以通过类名直接访问。
class Person {
  name = '张三';
  static love = true;
}

const p = new Person();
console.log(p.name); // 张三
console.log(Person.love); // true
```

- 函数声明

