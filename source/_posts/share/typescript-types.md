---
title: typescript 高级类型介绍
date: 2022-09-15 14:51:52
updated: 2022-09-15 14:51:52
tags:
  - typescript
categories:
  - 技术分享
  - TypeScript
---

`typescript`支持与`javascript`几乎相同的数据类型，此外还提供了其他实用的类型方便我们使用。

<!-- more -->

### 基础类型回顾

- 数字：`number`；
- 字符串：`string`；
- 布尔类型：`boolean`；
- 数组类型：元素类型后跟 `number[]`；或者使用数组泛型 `Array<number>`;
- 元组类型： 允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。如：`[string, number]`;
- 枚举类型：用于定义数值集合
- `any`类型：任意类型，可以赋予任意类型的值；
- `void`类型：与`any`相反，表示没有任何类型；
- `null`和`undefined`类型：只能指`null`与`undefined`元素本身；
- `never`类型：`never`是其它类型（包括`null 和 undefined`）的子类型，代表从不会出现的值

### 交叉类型

`type C = A & B; 类型 C 为同时包含类型 A、B 的新类型`

```typescript
interface A {
  name: string;
  age?: number;
}

interface B {
  eat(): void;
  say?: (name: string) => void;
}

// 交叉类型实现方式 1： 继承
interface C extends A, B {}

// 交叉类型实现方式 2： & (正常情况下)
type C = A & B;

// 交叉类型实现方式 3：映射(类似遍历赋值...不推荐)
```

### 联合类型

`type C = A | B; 类型 C 为 A 或 B 其中一种类型`

```typescript
// 定义如上
// ....

// 正常使用：
const c: C = { age: 18, name: '张三' } as A;

// or
const fn = (arg: C) => {
  // 通过访问 arg.name 属性，隐式判断当前 type 类型。A 中 name 为必选项
  if (arg.name) {
    console.log(arg.name);
  } else {
    arg.eat();
  }
};
```

### 联合类型之类型保护

- in 关键字

```typescript
interface A {
  name: string;
  age: number;
}
interface B {
  name: string;
  date: Date;
}
type UnknownT = A | B;

function printT(ar: UnknownT) {
  console.log('Name: ' + ar.name);
  if ('age' in ar) {
    console.log('age: ' + ar.age);
  }
  if ('date' in ar) {
    console.log('date: ' + ar.date);
  }
}
```

- typeof 关键字

```typescript
const fn = (arg: number | string) => {
  if (typeof arg === 'number') return '是类型 number';
  if (typeof arg === 'string') return '是类型 string';
};
```

- instanceof 关键字

```typescript
interface A {
  gitValue(): string;
}

class Aa implements A {
  constructor(private val: string) {}
  gitValue() {
    return this.val;
  }
}

class Ab implements A {
  constructor(private val: number) {}
  gitValue() {
    return this.val + '';
  }
}

const a = new Aa('aaa');
console.log(a instanceof Aa); // true
```

- 字面量类型保护

```typescript
type Foo = {
  kind: 'foo'; // 字面量类型
  foo: number;
};

type Bar = {
  kind: 'bar'; // 字面量类型
  bar: number;
};

function doStuff(arg: Foo | Bar) {
  if (arg.kind === 'foo') {
    console.log(arg.foo); // ok
    console.log(arg.bar); // Error
  } else {
    // 一定是 Bar
    console.log(arg.foo); // Error
    console.log(arg.bar); // ok
  }
}
```

- 自定义类型保护的类型谓词

```typescript
function isNumber(x: any): x is number {
  return typeof x === 'number';
}
```

### 通用类型

```typescript
// 接口通用
interface A {
  [k: string]: any;
}
```

### 字面量类型

```typescript
// color 的值只可以是字符串 red、blue、yellow
type color = 'red' | 'blue' | 'yellow';
```

### 接口数据类型

任何一个项目都离不开对数据和接口的处理，拼接数据和接口是形成业务逻辑也是前端的主要工作之一，将接口返回的数据定义 TypeScript 类型可以减少很多维护成本和查询 api 的时间 。

`API.d.ts` `d.ts` 结尾的文件会被 TypeScript 默认导入到全局 , 无需再手动导出。

```typescript
// 不能使用 import 语法，如果需要引用需要使用三斜杠
declare module API {
  type A = '';
  interface B {}
}
// or
declare namespace CAPI {
  interface CurrentUser {
    avatar?: string;
    name?: string;
    title?: string;
  }
}

// 使用 API.A | API.B |  CAPI.CurrentUser
```

### 泛型

类型变量`T` 。通过类型变量来帮助捕获用户传入的类型 ，以此来适配多个类型使用。

你可以这样：

```typescript
function uniquefn(args: any): any {
  // ...   大可不必
}
```

很明显，去重函数的参数可以是任意的类型，使用`any`类型会导致这个函数可以接收任何类型的`arg`参数，这样就丢失了一些信息 。因此，我们需要一种方法使返回值的类型与传入参数的类型是相同的。

```typescript
// 举栗 数组去重
function uniquefn<T>(args: T[]): T[] {
  // ...
}

// 使用
const arr1 = uniquefn<{ name: string; age?: number }>([
  { name: '张三', age: 18 },
  { name: '李四', age: 20 },
]);
// or 利用了类型推论 —— 即编译器会根据传入的参数自动地帮助我们确定T的类型
uniquefn([
  { name: '张三', age: 18 },
  { name: '李四', age: 20 },
]);
```

### 预定义类型

`TypeScript 2.8在lib.d.ts里增加了一些预定义的有条件类型：`

- `NonNullable<T>` -- 从 T 中剔除 null 和 undefined。
- `ReturnType<T>` -- 获取函数返回值类型。
- `InstanceType<T>` -- 获取构造函数类型的实例类型。

```typescript
// typescript 标准库内的一些实现
type Proxy<T> = {
  get(): T;
  set(value: T): void;
};
```

### 其他内置类型

- `Required` 。与 `Partial` 相反

`-?` 的功能就是把可选属性的 `?` 去掉，使该属性变成必选项，对应的还有 `+?` ，作用与 `-?` 相反，是把属性变为可选项。

```typescript
type Partial<T> = { [P in keyof T]?: T[P] };
type Required<T> = { [P in keyof T]-?: T[P] };
```

- Exclude<T,U>

  从 `T` 中排除那些可以赋值给 `U` 的类型。 (T ∩ U) 在 T 的 补集

```typescript
type Exclude<T, U> = T extends U ? never : T;
type T = Exclude<1 | 2 | 3 | 4 | 5, 3 | 4>; // T = 1|2|5
```

- Extract<T,U>

  从 `T` 中提取那些可以赋值给 `U` 的类型。 类型 T ∩ U

```typescript
type Extract<T, U> = T extends U ? T : never;
type T = Extract<1 | 2 | 3 | 4 | 5, 3 | 4>; // T = 3|4
```

- Pick<T, K>

  从 对象`T` 中取出一系列 `K` 的属性

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

interface Person {
  name: string;
  age: number;
  sex: string;
}
const person: Pick<Person, 'name' | 'age'> = {
  name: '小王',
  age: 21,
};
```

- Record<K, T>

  将 `K` 中所有的属性的值转化为 `T` 类型。

```typescript
type Record<K extends string, T> = {
  [P in K]: T;
};

const person: Record<keyof Person, string> = {
  name: '小王',
  age: '12',
};
```

- 更多的内置------> typescript 操作符及元素类型

### Omit<T,K>（没有内置）

从对象 `T` 中排除 `key` 是 `K` 的属性 。 与 Pick 相反

```typescript
type Omit<T, K> = Pick<T, Exclude<keyof T, K>>;
```

```typescript
interface Person {
  name: string;
  age: number;
  sex: string;
}
const person: Omit<Person, 'name'> = {
  age: 1,
  sex: '男',
};
```
