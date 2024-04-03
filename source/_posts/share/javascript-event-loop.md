---
title: 深入了解事件循环
date: 2024-04-03 11:03:15
updated: 2024-04-03 11:03:15
tags:
  - javascript
  - 事件循环
categories:
  - 技术分享
  - JavaScript
---

### 前言

#### `JS` 是单线程

在此之前，请注意进程和线程的区别。单线程的特性，决定了`JS` 引擎在任何时候都只有一个主线程来执行任务。

为什么是单线程？

> `js` 的主要作用是和用户互动以及操作 `dom`.
>
> 想象一个场景，在浏览器环境下，我们需要使用 `js` 来操作 `dom`，它可能是 **插入 dom** 或者 **删除 dom**，如果 `js` 是多线程，那么可能存在两种操作同时执行的情况，这是自相矛盾的...

然而，有些任务在执行的过程中是非常耗时的，比如定时器或者监听事件...它们会阻塞主线程的执行，导致页面卡顿。

#### `JS` 是非阻塞性的

为了避免主线程被阻塞，`JS` 引擎将任务分为两种：**同步任务（代码）和异步任务（代码）**

常见的异步代码：

> `setTimeout、setInterval、AJAX/Fetch、事件绑定监听、promise.then、promise.catch...`

**请注意：`promise` 本身是同步任务（代码），仅 `then` 和 `catch` 是异步的**

协调同步任务和异步任务执行，从而使 `JS` 的执行是非阻塞的，就需要事件循环（`Event Loop`）机制来实现。

<!-- more -->

### 什么是事件循环

同步任务是由 `JS` 引擎发起并立即执行，异步任务则是通过宿主环境（浏览器、`node`）在正确时机发起并执行。

#### 执行栈

在我们调用某一个方法时，`js` 会生成对应的执行环境（执行上下文 `context`）和私有作用域。如果该方法中又存在另一个方法的调用，则需要等待另一个方法执行完毕后，再执行当前方法。依次类推，这个不停等待内部函数执行的过程就执行栈，秉持先进后出的原则，等执行栈为空时，则表示当前函数执行完成。

如果是其他的同步代码执行，例如变量赋值而非函数调用，则直接入栈并立即执行，执行完成后出栈。

```javascript
const a = 1;
function b() {
  c();
}
function c() {
  console.log('c');
}
b();
console.log('a');
```

> a 声明并赋值（入栈出栈）-> b() 入栈开始执行 -> c() 入栈开始执行 -> c() 执行完毕出栈 -> b() 执行完毕出栈 -> console.log('a')（入栈出栈）-> 执行栈清空

例如上面的代码，为了方便说明，我省略了函数回调注册的过程， 同时只写了同步代码，实际应用中，同步异步代码都会有。

#### 任务队列

上面有提到，对于异步任务，`js` 引擎不会一直等待其执行完成再继续其他任务。

当代码执行过程中，如果遇到**同步（任务）代码立即入栈并执行，执行完成后出栈。在这个过程中，如果存在异步（任务）代码，则先将异步任务放入宿主环境等待正确执行时机，时机到了则入异步任务执行队列，等待执行栈清空，主线程空闲后依次执行**。

```javascript
function a() {
  console..log('a');
}
new Promise((res, rej) => {
  console.log('promise');
}).then(() => {
  console.log('promise then');
}).then(() => {
  console.log('promise then then');
});
new Promise((res, rej) => {
  console.log('promise2');
}).then(() => {
  console.log('promise2 then');
}).then(() => {
  console.log('promise2 then then');
});
a();
```

> console.log('promise') 入执行栈并出栈 -> promise then 入异步任务队列 --> promise2 then 入异步任务队列 -> console.log('promise2') 入执行栈并出栈 -> a() 入栈并出栈 -> 执行栈清空，主线程空闲 --> promise then 入执行栈执行(出队列)，promise then then 入异步队列 --> promise2 then 入执行栈执行(出队列)，promise2 then then 入异步队列 ---> promise then then 入执行栈(出队列) ---> promise2 then then 入执行栈(出队列) ---> 执行栈和异步任务队列均清空

上面例子，我刻意没有使用类似 `setTimeout` 这样的异步方法，因为它本身是支持定时到某个时间间隔后执行。又或者类似点击事件，它也只有在被点击的时候才加入到异步任务执行队列里。

为了区分这类异步任务，`js` 引擎将异步任务分为两类：**宏任务（`macro-task`）队列和微任务（`micro-task`）队列**

#### 宏任务

| #                     | 浏览器 | Node |
| --------------------- | ------ | ---- |
| I/O                   | ✅     | ✅   |
| setTimeout            | ✅     | ✅   |
| setInterval           | ✅     | ✅   |
| setImmediate          | ❌     | ✅   |
| requestAnimationFrame | ✅     | ❌   |

`script` 主线程代码其实也是宏任务，此外还有注入事件监听、点击事件等等都可以算作是宏任务。

> 常见的微任务：

#### 微任务

| #                          | 浏览器 | Node |
| -------------------------- | ------ | ---- |
| process.nextTick           | ❌     | ✅   |
| MutationObserver           | ✅     | ❌   |
| Promise.then catch finally | ✅     | ✅   |

根据上面列出的常见宏任务和微任务，不难看出：类似不确定的执行时机的 `api` 都是宏任务，反之则是微任务。（~~方便记忆 😋😋😋~~）

#### 事件循环的产生

不管是宏任务还是微任务还是主线程的同步代码，他们都是 `JS` 代码，都需要满足单线程的前提...

同一时间不能执行多个任务，此外宏任务和微任务本身又可以产生新的同步代码或者新的宏任务和微任务，所以这些任务应该按照怎样的规律去执行呢？事件循环机制因此诞生...

**`script` 主线程代码可以看做是一个宏任务，从这个角度分析，其实代码产生的宏任务和微任务都是由(`script`)宏任务产生，说其先执行宏任务再执行微任务也不算离谱...**

抛开这个观点来看，其实执行顺序无非就是：主线程同步代码 ---> 前者产生的所有微任务（根据生成顺序入队列，包含微任务产生的新微任务）执行 ---> 主线程执行完成输出 undefined（步骤可忽略） ---> 宏任务执行队列（**和微任务不同的是，宏任务需要正确的时机才会执行，根据时机满足的先后顺序入执行队列**，所以这里是执行队列而非任务产生的顺序队列。此外前者微任务产生的新宏任务将根据执行时机追加到宏任务执行队列末尾）

**注意：宏任务（内部代码）的执行也需要严格按照先同步再所有微任务再宏任务的顺序执行下一个宏任务**，意味着尽管宏任务 1 和 2 都是主线程依次产生，但是也需要先执行完宏任务 1 的所有微任务（如果这个过程中产生新的宏任务，则加入宏任务队列等待正确时机加入执行队列的末尾），再执行宏任务 2

> 宏任务的同步代码 ---> 宏任务的所有微任务执行(根据生成顺序入队列) ---> 下一个宏任务的同步代码 ---> 下一个宏任务的所有微任务执行.....

依次类推，这种循环执行的过程被称之为事件循环。

> 如点击或者监听事件被触发，被触发的瞬间加入执行队列等待执行...

#### 规律总结

主线程(同步代码立即执行并出栈，异步代码分别入微任务队列、宏任务队列)

---> 微任务队列(由 js 引擎发起并执行的任务： 先进先执行，微任务产生的新的微任务追加到本轮微任务队列末尾，在宏任务之前执行【执行完本轮所有的微任务】)

---> 宏任务队列(由宿主环境【浏览器、node】发起并执行的任务：等待正确的时机【如定时器(从加入到宏任务队列开始计时，如果时间相同则按加入顺序放入)】，

【如点击或者监听事件被触发】时机到了后将执行代码放入宏任务执行队列【时机是否正确的监听判断，不需要等待微任务 or 主线程同步代码执行完成】)

---> 执行本轮宏任务产生的异步任务队列(按顺序取出一个任务，执行其同步代码，以及产生的所有微任务，然后取出下一个异步任务...)

一次事件循环：

```
主线程所有同步代码（宏任务执行）--->
主线程产生的微任务 1 执行 ---->
主线程产生的微任务 2 执行 ---->
主线程产生的微任务 1 产生的微任务执行 ---->
主线程产生的微任务 2 产生的微任务执行 ---->
主线程产生的微任务 1 产生的微任务的微任务执行（依次类推，执行完主线程所有的微任务） --->

undefined（主线程执行完成打印） --->
```

将上面的‘主线程’文字替换为（‘宏任务’）同样适用，等全部完成后，表示该轮宏任务执行结束，开启下一个宏任务继续上面的循环执行...

### 练习

[https://www.jsv9000.app/](https://www.jsv9000.app/)

`eg.1`

```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
setTimeout(function () {
  console.log('setTimeout');

  new Promise((res) => {
    console.log('promise1');
    res();
  })
    .then(() => {
      console.log('promise1 then');
    })
    .then(() => {
      console.log('promise1 then then');
    });

  new Promise((res) => {
    console.log('promise2');
    res();
  })
    .then(() => {
      console.log('promise2 then');
    })
    .then(() => {
      console.log('promise2 then then');
    });

  setTimeout(() => {
    console.log('setTimeout setTimeout2');
  }, 0);
}, 0);
async1();

new Promise(function (resolve) {
  console.log('promise');
  resolve();
}).then(function () {
  console.log('promise then');
});
console.log('script end');
```

```
script start
async1 start
async2
promise
script end
async1 end
promise then
undefined
setTimeout
promise1
promise2
promise1 then
promise2 then
promise1 then then
promise2 then then
setTimeout setTimeout2
```

`eg.2`

```js
new Promise((resolve, reject) => {
  console.log(1);
  resolve();
})
  .then(() => {
    console.log(2);
    new Promise((resolve, reject) => {
      console.log(3);
      setTimeout(() => {
        reject();
      }, 3 * 1000);
      resolve();
    })
      .then(() => {
        console.log(4);
        new Promise((resolve, reject) => {
          console.log(5);
          resolve();
        })
          .then(() => {
            console.log(7);
          })
          .then(() => {
            console.log(9);
          });
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
```

```
1
2
3
4 (这里的then要比6的then先加入队列，因为4所在的then加入到微任务队列后，才表示2所在then同步代码执行结束，6继续被加入微任务队列，在所在then微任务后面)
5
6
7 (同理，7所在then被加入微任务队列后，4所在then同步代码执行结束，此时8所在then被追加到微任务队列，在7所在then之后)
8
9 (7所在then被执行后，9所在then被追加到微任务队列末尾，在8所在then之后)
```

**注意：`promise` 内部如果没有调用 `resolve` 或者 `reject` 那么 `promise` 内部代码（含新产生的微任务和宏任务）都执行完成后，此时 `promise` 状态会自动终止（执行栈清空，该模块微任务宏任务队列清空，由引擎终止回收，而非继续等待）**

**`promise` 模块内如果同时调用了 `resolve` 和 `reject`，那么先被调用的生效，后调用的则不会生效，也不会触发 `then` 回或者 `catch`**，所以上面例子的 `reject` 不会被触发。
