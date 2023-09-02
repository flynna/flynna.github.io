---
title: typescript 中的符号集合
date: 2022-09-15 16:57:42
updated: 2022-09-15 16:57:42
tags:
  - typescript
categories:
  - 技术分享
  - TypeScript
---

在 `typescript` 代码日常编写过程中，会经常性的遇到一些符号，这里对一些符号进行一个汇总。

例如：`?、??、?.、!、_`等一系列的符号，其中，除了`!`，其他都是`js`提供的，更方便的服务于`ts`。

### 类型查询操作符

`typeof T； [类型查询操作符] 结果为 T 上已知的 公共属性类型的 联合。 'XXX' | 'YYY'...`

<!-- more -->

### 索引类型查询操作符

`keyof T； [索引类型查询操作符] 结果为 T 上已知的 公共属性名的 联合。 'XXX' | 'YYY'...`

```typescript
// 例：枚举已知属性的集合
type Keys = 'A' | 'B' | 'C';
or;
interface KeysProps {
  A: string;
  B: string;
  C: string;
}
type Keys = keyof KeysProps;
```

### 索引访问操作符

`T[K]; [索引访问操作符] 可以通过索引的方式 取到 T上属性为 K 的 Type`

```typescript
// 例：
interface A {
  b: boolean;
  c: string;
}
type C = A['b'] ---------- C = boolean;
```

### 映射类型操作符

`in T 以及 in keyof T [映射类型]`

```typescript
// 例1  in T   [此时的 T 应该为联合类型] 当需要为指定的 key 集合约束类型时 很有用
type C = 'aa' | 'bb' | 'cc';
type D = { [k in C]: string };

//↓ 得到的结果

type D = {
  aa: string;
  bb: string;
  cc: string;
};

// 例2 in keyof T [此时的 T 为 interface 对象类型] 在已知 key集合的情况下 重约束 key的类型
interface A {
  aa: string;
  bb: number;
  cc: string;
}
type C = { [k in keyof A]: boolean };

//↓ 得到的结果

type C = {
  aa: boolean;
  bb: boolean;
  cc: boolean;
};
```

### `!`非空断言操作符

`！断言只是忽略TS检查，实际运行过程中， 该报错还是会报错, 此外 !是放在变量后的，注意非运算符区别`

- 忽略`undefined`和`null`类型 （ 比如：**x! 将从 x 值域中排除 null 和 undefined** ）

```typescript
function myFunc(maybeString: string | undefined | null) {
  const onlyString: string = maybeString; // Error
  const ignoreUndefinedAndNull: string = maybeString!; // Ok
}
```

- 调用函数时忽略`undefined`

```typescript
type NumGenerator = () => number;
function myFunc(numGenerator: NumGenerator | undefined) {
  const num1 = numGenerator(); // Error
  const num2 = numGenerator!(); //OK
}
```

### `?:`可选属性

```typescript
interface A {
  name: string;
  age?: number;
}
const a: A = { name: '张三' }; // age 可选
```

### 类型运算符`&、 |`

`用于类型运算： & 将多个类型合并为一个类型， | 取值可以为多种类型中的一种 `

> type A = xx & xxx 交叉类型

> type B = xx | xxx 联合类型

### `_`数字分隔符

你可以把一个下划线作为数字内部的分隔符来分组数字， 比如

```typescript
const inhabitantsOfMunich = 1_464_301;
const distanceEarthSunInKm = 149_600_000;
const fileSystemPermission = 0b111_111_000;
const bytes = 0b1111_10101011_11110000_00001101;
```

编译后，会生成以下 ES5 代码：

```typescript
'use strict';
var inhabitantsOfMunich = 1464301;
var distanceEarthSunInKm = 149600000;
var fileSystemPermission = 504;
var bytes = 262926349;
```

`注意： 只能在两个数字之间添加_分割符，不能连续使用多个 `\_`分隔符`

```typescript
// Numeric separators are not allowed here.(6188)
3_.141592 // Error
3._141592 // Error

// Numeric separators are not allowed here.(6188)
1_e10 // Error
1e_10 // Error

// Cannot find name '_126301'.(2304)
_126301  // Error
// Numeric separators are not allowed here.(6188)
126301_ // Error

// Cannot find name 'b111111000'.(2304)
// An identifier or keyword cannot immediately follow a numeric literal.(1351)
0_b111111000 // Error

// Numeric separators are not allowed here.(6188)
0b_111111000 // Error

// Multiple consecutive numeric separators are not permitted.(6189)
123__456 // Error
```

- 数字分隔符的解析：

```typescript
// 不能使用 Number() parseInt() parseFloat() 解析
// 自定义转换函数解析出实际数字
// 如  str.replace 方法 替换掉 _
```

### `<Type> 及 as` 断言

- 尖括号语法(`React`中不支持)

```typescript
const a: any = 'asf';
const n = (<string>a).length;
```

- `as`语法

```typescript
const a: any = 'asf';
const n = (a as string).length;
```
