---
title: typescript 操作符
date: 2021-07-02 15:34:23
tags: [ts操作符]
# type: "categories"
categories: [大前端, TypeScript]
---

### 操作符

#### 索引类型查询操作符

`keyof T； [索引类型查询操作符] 结果为 T 上已知的 公共属性名的 联合。 'XXX' | 'YYY'...`

```typescript
// 例：枚举已知属性的集合
type Keys = "A" | "B" | "C";
or;
interface KeysProps {
  A: string;
  B: string;
  C: string;
}
type Keys = keyof KeysProps;
```

#### 索引访问操作符

`T[K]; [索引访问操作符] 可以通过索引的方式 取到 T上属性为 K 的 Type`

```typescript
// 例：
interface A {
  b: boolean;
  c: string;
}
type C = A['b'] ---------- C = boolean;
```

#### 映射类型操作符

`in T 以及 in keyof T [映射类型]`

```typescript
// 例1  in T   [此时的 T 应该为联合类型] 当需要为指定的 key 集合约束类型时 很有用
type C = "aa" | "bb" | "cc";
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

### 元素类型

#### 交叉类型

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

#### 联合类型

`type C = A | B; 类型 C 为 A 或 B 其中一种类型`

```typescript
// 定义如上
// ....

// 正常使用：
const c: C = { age: 18, name: "张三" } as A;

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

#### 通用类型

```typescript
// 接口通用
interface A {
  [k: string]: any;
}
```

#### 枚举类型

#### 类类型

#### 泛型类型



#### 预定义类型

`TypeScript 2.8在lib.d.ts里增加了一些预定义的有条件类型：`

- `Exclude<T, U>` -- 从 T 中剔除可以赋值给 U 的类型。
- `Extract<T, U>` -- 提取 T 中可以赋值给 U 的类型。
- `NonNullable<T>` -- 从 T 中剔除 null 和 undefined。
- `ReturnType<T>` -- 获取函数返回值类型。
- `InstanceType<T>` -- 获取构造函数类型的实例类型。

```typescript
// typescript 标准库内的一些实现
type Proxy<T> = {
  get(): T;
  set(value: T): void;
};
type Partial<T> = {
  [P in keyof T]?: T[P];
};
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
type Record<K extends string, T> = {
  [P in K]: T;
};
```
