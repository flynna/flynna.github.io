---
title: Decorator 装饰器的实现原理及实践
date: 2023-02-02 16:17:54
updated: 2023-02-02 16:17:54
tags:
  - decorator
  - 装饰器
  - javascript
  - es7
categories:
  - 技术分享
  - ECMAScript
---

### `Decorator` 是什么？

因为之前一个 `Vue` 项目中需要对一个 `Watch` 监听添加去抖动功能，最后经过深思熟虑，决定使用 `decorator` 进行实现。

简单来讲，`decorator`就是`ES7`的一个语法糖，类似`wrapper`，它作用于一个目标函数，对这个目标函数添加一些额外的操作，然后返回一个新的函数。形如：

```typescript
@myDecorator
function newFn() {
  return console.log('这是被装饰器包裹的内部函数');
}
```

`decorator` 的实现依赖于 `ES5` 的 `Object.defineProperty` 方法。

<!-- more -->

### 了解 `Object.defineProperty`

`defineProperty` 简单来说就是为调用他的对象添加新的属性，或者对已知某个已存在的属性进行修改。

#### 语法

```typescript
Object.defineProperty(obj, prop, descriptor);
```

它有三个参数：

- `obj`: 目标对象
- `prop`: 属性名，键
- `descriptor`: 属性描述符，针对该属性的描述

#### `descriptor` 属性描述符

对象里目前存在的属性描述符有两种主要形式：**数据描述符**和**存取描述符**。

**两种描述符只能是其中之一，不能同时使用。且都是对象**

- 数据描述符：描述属性值是可写还是不可写。对应独有的可选键：`value(属性值)/writable(可写)`
- 存取描述符：由 `getter` 和 `setter` 函数所描述的属性。对应独有的可选键：`get(属性访问方法)/set(属性设置方法)`

- 两者共有的可选键：`configurable(可变更[删除])/enumerable(可枚举)`

> [了解更多 Object.defineProperty 相关](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

### 装饰者模式

`概念：允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。`

你可以理解为就是一层`wrapper`，`React`的高阶函数组件就是装饰者模式的一个实践。比如下面这段代码：

```tsx
import type { IRouteComponentProps } from 'umi';
import { PageContainer } from '@ant-design/pro-layout';
import styles from './index.less';

const withLayout = (Page: React.FC<any>) => {
  console.log('这里可以添加一些公用逻辑...');

  // 返回一个新的 wrapper 组件
  return (props: IRouteComponentProps) => {
    console.log('这里可以通过 props，处理一些特殊逻辑...');

    return (
      <PageContainer className={styles.pageContainer}>
        <Page {...props} />
      </PageContainer>
    );
  };
};
```

### 写一个 `decorator`

> `@`装饰器，后面紧跟一个装饰器函数，该函数有三个形参：`target, propertyKey, descriptor`，最后返回一个新的`descriptor`。该函数即是目标函数的`wrapper`实现，当然该装饰器函数也可以是工厂函数，通过`fn(...args)`返回。

**装饰器`decorator`只能在类中使用**

**在类方法上使用时，target 就是类的 prototype，propertyKey 就是方法名**

**在类上使用时，`descriptor` 接受的第一个参数 `target` 就是类的 `prototype`**

#### 在类的一个方法上使用

`该例使用的是` **数据描述符** `的方式对 descriptor 进行修改。最后面还有一个 debounce 的例子，准备使用` **存取描述符** `实现`

```typescript
// log 装饰器 --- 可接受参数
function log(fnName: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value; // 拿到目标函数

    descriptor.value = function (...args) {
      console.log(`这就是装饰器为${fnName}函数添加的日志功能`);
      original.apply(this, args);
    };

    return descriptor;
  };
}

class Test {
  constructor() {}

  @log('myFn')
  myFn() {
    // console.log('这是一个函数');
  }
}
```

在该例子中，我在`Test`中定义了一个`myFn`的方法，同时为其添加了`log`的装饰器修饰。`log`的作用就是在`myFn`执行的时候，添加 `console.log` 的日志打印记录。装饰器函数最后返回一个包含新的`value(也就是这个函数体)`的`descriptor`。它的原理等价下面实现：

```typescript
const defaultDescriptor = {
  value: function () {
    // console.log('这是一个函数');
  },
  enumerable: false,
  configurable: true,
  writable: true,
};

// 装饰器的作用：通过 @function：function 返回新的 descriptor
const descriptor = log('myFn')(Test.prototype, 'myFn', defaultDescriptor) || defaultDescriptor;

// 将新的 descriptor 作用到目标方法
Object.defineProperty(Test.prototype, 'myFn', descriptor);
```

这样是不是更容易理解`@`究竟都干了些什么，总结下来，其实自己实现一个装饰器也就不那么难了。

#### 在类上使用

在类上使用，可以实现很多有意思的功能：**添加静态成员、继承、MixIn...**

```typescript
function addType(target: Cat) {
  target.type = 'Animal'; // 添加类的静态成员 static
  return target;
}

@addType
class Cat {
  public name: string;

  constructor({ name }: { name: string }) {
    this.name = name;
  }

  public say() {
    console.log(`我是一只${Cat.type}， 我的名字叫${this.name}`);
  }
}
```

通过在`Cat`上添加`@addType`，所有用到该装饰器的类，都会多一个静态成员`type`。那么继承又该怎么实现呢？

```typescript
@Student
class Man {
  constructor(name) {
    this.name = name;
  }
  say() {
    console.log('my name is ' + this.name);
  }
}

function Student(target: Student) {
  return class extends target {
    constructor(opt) {
      super(opt);
    }

    // 方法重写
    say() {
      console.log('我不要你觉得，我要我觉得');
    }
  };
}
```

通过`@Student`可以生成`Man`的子类`Student`，当然例子仅表示能这么实现，实际业务中**不推荐使用这种方式进行继承**，~~`new Man()`生成的实例居然是`Student`的实例，什么？父类的实例居然是子类的实例？想想就有些受不了~~。

那么，有没有能接受的方式？下面定义的`Mixin`，用到该装饰器的类，均可挂载该装饰器类的所有方法，如下：

```typescript
class A {
  runA() {
    console.log('我是runA!');
  }
}

class B {
  runB() {
    console.log('我是runB!');
  }
}

@mixin(A, B)
class C {}

function mixin(...args) {
  return function (target) {
    // 遍历原型上的方法，并挂载到目标类
    for (const cls of args) {
      //! 注意: 通过这种方式混入到目标类，仅仅是完成了将方法函数拷贝到目标类的效果，并未挂载实例对象的基本属性，并未实现真正的多继承
      for (const key of Object.getOwnPropertyNames(cls.prototype)) {
        if (key !== 'constructor') {
          Object.defineProperty(
            target.prototype,
            key,
            Object.getOwnPropertyDescriptor(cls.prototype, key),
          );
        }
      }
    }
  };
}

const c = new C();
c.runA(); // 我是runA!
c.runB(); // 我是runB!
```

实际项目中又怎么用呢？

### `Vue2`装饰器语法中的运用

这个是之前给`watcher`添加`debounce`的时候加的装饰器，实现如下：

```typescript
function debounce(fn, time) {
  // store timeout value for cancel the timeout
  let timeoutRef = null;

  return function (...args) {
    clearTimeout(timeoutRef);

    timeoutRef = setTimeout(() => {
      fn.apply(this, args);
    }, time);
  };
}

function Debounce(limit = 0) {
  return function debounceDesc(target: any, key: string, descriptor: PropertyDescriptor) {
    return {
      configurable: true,
      enumerable: descriptor.enumerable,
      get: function () {
        // Attach this function to the instance (not the class)
        Object.defineProperty(this, key, {
          configurable: true,
          enumerable: descriptor.enumerable,
          value: debounce(descriptor.value, limit),
        });
        return (this as any)[key];
      },
    };
  };
}
```
