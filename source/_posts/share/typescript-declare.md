---
title: 需要了解的 typescript 知识点
date: 2022-09-14 11:02:26
updated: 2022-09-14 11:02:26
tags:
  - typescript
categories:
  - 技术分享
  - TypeScript
---

> 前段时间项目忙，加之疫情原因，就没有更新。😊😊😊

> 这篇文章主要是类型声明过程中遇到的需求和问题做的记录。

> 持续更新中...

<!-- more -->

### 如何编写`.d.ts`声明文件？

`.d.ts`文件是在编写`typescript`项目中必不可少的，它是让我们能在`ts`中调用的`js`的声明文件。比如有很多主流的`npm库`都是基于`js`编写的，那么在你的`ts`项目中引用时，会提示你找不到对应包或函数的声明，此时并不需要我们用`ts`对组件重写，只需在你的项目中编写包含该库类型声明的`.d.ts`文件即可。

通常在定制化编写组件时，`.d.ts`会统一放到`@types`或者`typings`文件夹下，当然你也可以不写`.d.ts`，而通过`tsc`对`ts`文件做编译生成对应文件的`.d.ts`。

- [`declare var`](https://ts.xcatliu.com/basics/declaration-files.html#declare-var) 声明全局变量
- [`declare function`](https://ts.xcatliu.com/basics/declaration-files.html#declare-function) 声明全局方法
- [`declare class`](https://ts.xcatliu.com/basics/declaration-files.html#declare-class) 声明全局类
- [`declare enum`](https://ts.xcatliu.com/basics/declaration-files.html#declare-enum) 声明全局枚举类型
- [`declare namespace`](https://ts.xcatliu.com/basics/declaration-files.html#declare-namespace) 声明（含有子属性的）全局对象
- [`interface` 和 `type`](https://ts.xcatliu.com/basics/declaration-files.html#interface-和-type) 声明全局类型
- [`export`](https://ts.xcatliu.com/basics/declaration-files.html#export) 导出变量
- [`export namespace`](https://ts.xcatliu.com/basics/declaration-files.html#export-namespace) 导出（含有子属性的）对象
- [`export default`](https://ts.xcatliu.com/basics/declaration-files.html#export-default) ES6 默认导出
- [`export =`](https://ts.xcatliu.com/basics/declaration-files.html#export-1) commonjs 导出模块
- [`export as namespace`](https://ts.xcatliu.com/basics/declaration-files.html#export-as-namespace) UMD 库声明全局变量
- [`declare global`](https://ts.xcatliu.com/basics/declaration-files.html#declare-global) 扩展全局变量
- [`declare module`](https://ts.xcatliu.com/basics/declaration-files.html#declare-module) 扩展模块
- [`/// `](https://ts.xcatliu.com/basics/declaration-files.html#san-xie-xian-zhi-ling) 三斜线指令

`eg. 扩展 axios 模块请求的 config 配置。` ~~axios.interceptors.response. errors.config 亦可拿到~~

```typescript
declare module 'axios' {
  export interface AxiosRequestConfig {
    metadata?: {
      start: number;
    };

    /**
     *  用于控制该请求出现错误时， 是否由默认message显示错误信息
     *
     * @type {boolean}
     * @memberof AxiosRequestConfig
     */
    suppressError?: boolean;

    /**
     * 用于请求在401状态码时， 是否自动重定向到登陆页。
     *
     * @type {boolean}
     * @memberof AxiosRequestConfig
     */
    suppressUnauthRedirect?: boolean;
  }
}
```

<div class="warning">

> 声明语句中只能定义类型，切勿在声明语句中定义具体的实现

</div>

#### `///`三斜线指令

> /// \<reference path="..." \/\>

三斜线引用告诉编译器在编译过程中要引入的额外的文件。**三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释。**

#### `declare namespace`详解

`namespace`中文称命名空间，现在已经不建议再使用`ts`中的`namespace`，而推荐使用`ES6`的模块化方案了，~~虽然目前仍能使用~~。

`eg. jQuery`提供了一个`jQuery.ajax`方法可以调用，那么我们就应该使用`declare namespace jQuery`来声明这个拥有多个子属性的全局变量。

```typescript
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;
  const version: number;
  class Event {
    blur(eventType: EventType): void;
  }
  enum EventType {
    CustomClick,
  }
}
```

注意，在`declare namespace`内部，我们直接使用`function ajax`来声明函数，而不是使用`declare function ajax`。类似的，也可以使用 `const`, `class`, `enum`等语句`

- 嵌套的命名空间

使用场景： 诸如 jQuery.fn.extend({})

```typescript
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;
  namespace fn {
    function extend(object: any): void;
  }
}
```

- 当然你也可以这样写：

```typescript
// 这样写有很多局限性， 比如 jQuery不仅仅有 fn属性的时候
declare namespace jQuery.fn {
  function extend(object: any): void;
}
```

### `type`和`interface`如何抉择？

推荐任何时候都是用`type`，`type`使用起来更像一个变量，与`interface`相比，`type`的特点如下：

- 表达功能更强大，不局限于`object/class/function`
- 要扩展已有`type`需要创建新`type`，不可以重名
- 支持更复杂的类型操作

基本上所有用`interface`表达的类型都有其等价的`type`表达。

### 给函数添加属性申明

```typescript
interface FuncWithAttachment {
  (param: string): boolean;
  someProperty: number;
}

const testFunc: FuncWithAttachment = (param) => {};
const result = testFunc('mike'); // 有类型提醒
testFunc.someProperty = 3; // 有类型提醒
```

等价的**声明合并**写法:

```typescript
// 组合多个声明语句，它们不会产生冲突
type FuncWithAttachment = (param: string) => boolean;
type FuncWithAttachment = {
  someProperty: number;
};
```

### 枚举类型声明及类型使用

```typescript
// 联合类型
type A = 'a' | 'b' | 'c';

// 枚举类型
enum A {
  'a' = 'a',
  'b' = 'b',
  'c' = 'c',
}
```

> **注意 (type)A 等价 keyof typeof (enum 的 keys)A，如果想使用 (enum 的 values 的联合类型) 直接使用 A 即可**

### 泛型的其他使用场景

`列举除了泛型函数外的一些常见使用场景：`

- 接口返回数据类型约束

```typescript
// 有这么一个 interface ,显然， 很多接口返回数据都是这种结构，所以这里使用了 泛型约束
interface A<T> {
  page: number;
  size: number;
  results: T[];
  total: number;
}

interface B {
  name: string;
  age?: number;
}

const res = await request<A<B>>('/api/xx');
```

- `Promise` 类型

在做异步操作时我们经常使用 `async` 函数，函数调用时会 `return` 一个 `Promise` 对象，可以使用 `then` 方法添加回调函数。

`Promise` 是一个泛型类型，`T` 泛型变量用于确定使用 `then` 方法时接收的第一个回调函数（`onfulfilled`）的参数类型。

```typescript
// 某 props 接收异步处理函数
onTaskRequest?: (args: API.X[]) => Promise<API.X[]>;
```

- `event` 事件类型

```typescript
// click 使用 React.MouseEvent 加 dom 类型的泛型
// HTMLInputElement 代表 input标签 另外一个常用的是 HTMLDivElement
const onClick = (e: React.MouseEvent<HTMLInputElement>) => {};
```

- 作为组件 `Props` 及 `hooks` 参数

```tsx
interface FnComProps {}
const FnCom: React.FC<FnComProps> = (props) => {
  return <>组件</>;
};
```

...

### 动态更新 `Object` 的 `key`

```typescript
type Person = {
  name: string;
  age?: number;
  [k: string]: any; // 指定 k 可以为任意的 string
};
```

### 反向使用 typeof

```typescript
// NewsCard 组件接收的 props 类型
interface NewsCardProps {
  // ficusService.getNews: (params: API.NewsQuery) => Promise<API.PaginationNewsResult<API.News>>
  // 通过 typeof 反向获取类型
  provider: typeof ficusService.getNews;
}
```

### 巧用`Partial`

使用 `Partial` 将所有的 `props` 属性都变为可选值。------ 这对我们在接口定义时很有用

```typescript
type Partial<T> = { [P in keyof T]?: T[P] };
```

```typescript
// 在 A 接口中, 我们使用 interface X 完成了对某数据的返回类型定义。
// 在 B 接口中, 我们可能会对 X 中定义的字段的部分字段进行更新， 但在 A 接口 为必选字段， B 接口为可选

export const conditionService = {
  async query(query: Query) {
    const res = await axios.get<PageResult<ConditionTemp>>(`/v1.0/condition`, {
      params: query,
    });
    return res.data;
  },
  async update(data: Partial<ConditionTemp>) {
    await axios.put(`/v1.0/condition`, data);
  },
};
```

### 为 Window 添加参数

使用第三方库时（ga）,ga 是全局方法，在使用时会提示" 类型“Window”上不存在属性“ga”

```typescript
interface Window {
  ga: (
    command: 'send',
    hitType: 'event' | 'pageview',
    fieldsObject: GAFieldsObject | string,
  ) => void;
  reloadAuthorized: () => void;
}
```

不想在 Window 中增加，但是想要全局使用，比如通过 define 注入的参数，我们通过 `declare` 关键字在 `/src/typings.d.ts` 注入。

```typescript
declare const REACT_APP_ENV: 'test' | 'dev' | 'pre' | false;
```

### 装包没有相对应的 `@types`

- 安装包对应的`@types`文件包，如`@types/react`、`@types/react-dom`

- 自定义其为`any`。

```typescript
declare module 'xxx'; // d.ts
import xxx from 'xxx'; // tsx
```

### @ts-ignore

```typescript
// @ts-ignore
xxx;
```

遇到动态性比较强的代码，不妨使用 `as unknown as XXX`

### [typescript 其他使用姿势](https://ts.xcatliu.com/basics/declaration-files.html)
