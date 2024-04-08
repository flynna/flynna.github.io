---
title: 深入了解事件循环
date: 2024-04-03 11:03:15
updated: 2024-04-03 11:03:15
tags:
  - javascript
  - 事件循环
  - promise链式操作
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

### 关于 `Promise` 对事件循环的的一些影响

#### 代码块内不存在 `resolve 和 reject` 调用

> **即使 `Promise` 内部代码没有 `resolve 和 reject` 的相关调用，也不会阻塞程序以及抛出异常。**

当 `Promise` 相关代码块回调执行完成后（含其内产生的微任务和宏任务完成），并且自身无相关引用（未赋值给某个外部变量等情况）时，即 `promise` 对象被垃圾回收释放前，其状态会变更为 `fulfilled`。由浏览器进行垃圾回收，避免造成内存泄露。

> **关于是否会内存泄露的验证，参考：[既未 Resolve 又未 Reject 的 Promise 对象会导致内存泄漏吗？](https://k8w.cn/post/67)**

eg.

```js
function fn() {
  console.log(1);
  new Promise((resolve, reject) => {
    console.log(2);
    setTimeout(() => {
      console.log(4);
    });
  }).then(() => {
    console.log(3); // 不会被执行
  });
}
fn();
```

上面例子，在 `fn()` 执行后，依次执行 `log(1) 和 log(2)`，然后将 `console.log(4);` 所在的匿名添加到宏任务队列（延时队列）中，此时 `fn` 内的主执行栈清空，由于微任务队列里不存在任务，则继续等待延时队列中的 `log(4)` 执行，执行完成后，`fn` 所处线程代码执行完成。

其内的 `promise` 虽然没有 `resolve` 和 `reject` 的调用，但此时状态依然会被置为 `fulfilled` 结束，且其后的 `.then 和 .catch` 不会被执行。因为：

> **（一般情况：不存在异常抛出的前提下）只有调用 `resolve` 或者 `reject`，并且等待后续同步代码执行完成（含异步任务入队列）后才会将其自身的 `.then` 或 `.catch` 的匿名回调函数添加到微任务队列中**

为了验证上面的结论，下面看另一种添加了 `async/await` 的情况：

```js
async function fn() {
  console.log(1);
  await new Promise((resolve, reject) => {
    console.log(2);
    setTimeout(() => {
      console.log(4);
    });
  });
  console.log(3); // 不会被执行;
}
fn();
new Promise((resolve, reject) => {
  console.log(5);
  resolve();
}).then(() => {
  console.log(6);
});
// 1 2 5 Promise {<fulfilled>: undefined} 6 4
```

这个例子，使用了 `ES7` 的 `async/await` 语法糖，根据最终的输出结果可以看到 `log(3)`（即 `promise.then`）的微任务并没有产生和执行，且状态会被置为 `fulfilled`。

反推：**如果** `await` 关键字会产生新的微任务，那么它一定会在 `log(6)` 所属 `then` 微任务之前被放进微任务队列中，且 `log(6)` 会被阻塞（**这是假设的错误结论**）。但根据结果不难看出：

> **即时使用了 `ES7` 的 `async/await` 语法糖，同样是存在 `resolve/reject` 调用，才会将 `（await 后的代码即 .then）/.catch` 放入微任务队列**

<div class="success">

> **结论：**
>
> **（一般情况：不存在异常抛出的前提下）如果 `promise` 没有调用 `resolve` 或者 `reject`，那么它的所有 `.then（包括 await 后的代码块）` 和 `.catch` 都不会放入微任务队列执行**

</div>

#### 代码块内同时存在 `resolve 和 reject` 调用

> **（一般情况：不存在异常抛出的前提下）`promise` 模块内如果同时调用了 `resolve` 和 `reject`，那么先被调用（执行）的生效，后调用的则不会生效，也不会执行后者所触发的 `then` 或者 `catch`**

所以上面 `eg.1` 例子的 `setTimeout 代码中的 reject` 并不会被触发，也不会执行其 `.catch（如果代码有写的话）`。

#### 代码块内的 `resolve 或 reject` 并非在回调的末尾

> **不管是 `resolve` 还是 `reject`，如果其所在位置并非在 `promise` 回调函数的末尾，而是在中间（表示其后还存在别的同步代码 or 新的 `promise`），那么会优先执行后面的同步代码，如果后面的代码存在异步任务，会优先将其放入（宏/微）任务队列中，最后再将（`resolve 或 reject`）自身的回调追加到微任务队列末尾。**
>
> **待本轮所有的宏/微任务加入到队列后，才会从微任务队列中取出首个任务继续向后执行**，如后面的 `eg.3`

#### 关于 `promise` 的链式调用

形如：

> `Promise.resolve().then(() => {}).catch(() => {}).then(() => {})....`

<div class="success">

> **结论：**
>
> **和 `resolve 或 reject` 是否同时调用无关和是调用 `resolve` 还是 `reject` 无关，（只要满足其中一个被调用过，进入了 `promise` 链式调用操作后，其后的所有不管是 `.catch` 还是 `.then` 都会生成微任务参与事件循环）。**
>
> **每一次的 `.then` 或者 `.catch` 都是一个新的微任务，按顺序执行，且仅前一个 `.then 或 .catch` 的回调函数执行完成后（这里的执行完成代表：执行完成其内部的同步代码，并且将产生的新微任务（不会立即执行）追加到微任务队列末尾）才会继续向后执行。**
>
> **值得注意的是，尽管链式中例如 `.catch` 的回调函数不会被触发执行（尽管它不会被触发且处在链式的第一个），但它同样会参与到微任务队列中占位（你可以理解为它会生成微任务，但是任务的回调函数为空代码块而非实际的回调，从而影响微任务队列的排列顺序）**

</div>

对比下面的 `eg.3` 和 `eg.4` 的 `6 输出位置变化`，不难得出上面结论，特别是结论的最后一点。

`eg.3`

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
        console.log(10);
        reject();
      }, 3 * 1000);
      resolve();
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
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
        console.log(22);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 31 32 4 5 6 7 22 9 8  10 110
```

`eg.4`:

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
        console.log(10);
      }, 3 * 1000);
      resolve();
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
    })
      .catch(() => {
        console.log(0);
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
        console.log(22);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 31 32 6 4 5 7 22 9 8  10 110
```

#### 代码块内存在异常（`throw` or `reject`）抛出

##### 仅 `reject` 调用，且存在 `catch` 在链式的首项

<div class="success">

> **若代码自身无异常，仅存在 `reject` 调用用来触发后续 `.catch` 回调，则 `reject` 调用位置后的代码（当前 `promise` 包裹的内部代码）不会被影响（应该生成宏/微任务的继续生成宏/微任务放入队列）**

</div>

`eg.5`:

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
        console.log(10);
      }, 3 * 1000);
      reject();
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
    }).catch(() => {
      console.log(0);
    });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 31 32 0 6  10 110
```

从 `eg.5` 的输出不难看出，尽管 `reject` 优先执行，但是其后的 `setTimeout` 和 `Promise` 的执行并不会被影响。

##### 仅 `reject` 调用，且存在 `catch` 不在链式的首项

<div class="success">

> **若代码自身无异常，仅存在 `reject` 调用用来触发后续 `.catch` 回调，但是 `.catch` 并非在链式的首项，在其之前存在 `n 个 .then` 的回调，那么程序不会抛出异常，一直向后找到链式中的第一个 `.catch`执行（即时它不在链式的首项，在它之前的所有 `.then` 仅在微任务队列中占位（前面有提到，可以将之当成空回调在队列中占位进而影响排列顺序））。**同理，若是 `resolve` 则向后找第一个 `.then`。

</div>

`eg.6`:

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
        console.log(10);
      }, 3 * 1000);
      reject();
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
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
        console.log(22);
      })
      .catch(() => {
        console.log(0);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 31 32 6 0 8  10 110
```

从 `eg.6` 的输出可以看到，除了前面说的不影响后面代码执行和入异步队列外，`.catch` 前的所有 `.then` 都没有执行。此外从 `0 在 6 之后输出`（对比 `eg.5`），可以看出， 其前面的 `.then` 会参与（看作空回调）到微任务队列影响执行顺序。

##### 仅 `reject` 调用，但链式中不存在 `.catch`

<div class="success">

> **若代码自身无异常，仅存在 `reject` 调用，但后续链式中不存在 `.catch` 时，当前 `promise` 包裹的内部代码会继续执行（该放入异步任务队列的继续），但当前 `promise` 产生的所有 `.then` 链式都不会执行，且产生异常提示（并不会阻塞程序执行，微任务队列中已存在的任务会继续进行事件循环）**

</div>

`eg.7`:

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
        console.log(10);
      }, 3 * 1000);
      reject();
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
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
        console.log(22);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 31 32 6  10 110 ---- Uncaught (in promise) 异常提示
```

从 `eg.7` 输出可以看到 `log(6) 和 setTimeout` 异步任务并没有收到影响，但后面链式的所有 `.then` 都没有执行。

##### 仅 `throw new Error()`，不执行 `reject`

在前面的例子基础上，仅将 `reject()` 的部分替换为 `throw new Error()`，其他 `.catch` 的部分保留，则：

<div class="success">

> **若仅存在 `throw new Error 或 程序代码自身的异常`，不存在 `reject` 调用，则 `throw new Error` 的位置后的代码（属于当前 `promise` 包裹的内部代码部分）都不会执行，也不会加入异步任务队列（不会影响其他 `promise` 的宏/微任务和已产生的宏/微任务）。**若其链式中存在 `.catch` ，`.catch` 执行时机同前面 `reject` 的情况。
>
> **简单的说，`throw new Error` 和 `reject` 区别就是，前者代码位置后的代码不会被执行，后者会执行。其他关于 `.catch` 表现完全一致。**

</div>

`eg.8`:

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
        console.log(10);
      }, 3 * 1000);
      throw new Error();
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
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
        console.log(22);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 6  10 ---- Uncaught (in promise) Error 异常
```

从 `eg.8` 输出可以看到，微任务 `log(6)` 和宏任务 `log(10)` 并没有被影响，但同步任务 `log(31)` 和微任务 `log(32)` 并没有执行。

##### 若同时存在 `throw new Error` 和 `reject 或者 resolve`

<div class="success">

> **若 `throw new Error` 在 `reject 或者 resolve` 之前，则同前面所说 `reject 或者 resolve` 不会被执行。**
>
> **若 `throw new Error` 在 `reject 或者 resolve` 之后，则 `throw new Error` 后面的代码不会被执行，同时 `reject 或者 resolve` 会生效并触发其回调 `.catch 或者 .then` 的执行。**

</div>

`eg.9`:

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
        console.log(10);
      }, 3 * 1000);
      resolve();
      throw new Error('aaa');
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
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
        console.log(22);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 4 5 6 7 22 9 8  10
```

从 `eg.9` 输出可以看到，`log(110)、log(31)、log(32)` 未执行，其他代码执行正常。

##### 异常影响总结

<div class="info">

> **总的来说：**
>
> **不管是 `throw err`、`resolve`、`reject` 如何组合，`promise` 内包裹的代码块，皆是从上向下执行，遇到错误 `throw err` 时终止。**
>
> **在其遇到错误终止前的所有代码均会生效（同步代码则立即执行，异步任务则放入异步执行队列中），错误终止后的代码无效（尽管它是 `resolve` or `reject`调用）。**
>
> **若是在代码块执行完成（遇到代码错误、没有错误并执行到 `promise` 包裹的代码块末尾）前，若是存在 `resolve` 调用，则继续向下执行 `.then`，若是存在 `reject` 或者遇到代码错误（后者 `throw err` 后的代码不执行，前者其后的代码会执行），则继续向下执行 `.catch`。**
>
> **`promise` 内（包括其链式调用中的回调函数）若是存在 `throw err` 异常抛出，不会阻塞程序或者外部函数的向下运行，此外若是链式调用中回调（包括 `.then` 内的）若有异常抛出，则在链式中当前异常抛出函数之后，继续向下找到第一个 `.catch` 执行。**
>
> **`.then 或者 .catch` 中的回调若是没有异常抛出，则会继续向下找 `.then` 执行（当然这个过程也需要在任务队列中排队）。**
>
> **一旦进入链式回调中（即便是通过 `throw err` 进入的，而非调用 `resolve 或 reject`），链式中的所有符合条件的回调都将被执行。**参考 `eg.11`

</div

`eg.10`:

```js
function fn() {
  console.log(-1);
  new Promise((res) => {
    console.log(0);
    throw new Error('aaa');
    console.log(1);
  }).then(() => {
    console.log(3);
  });
  console.log(2);
}
fn();
// -1 0 2
function fn1() {
  console.log(-1);
  new Promise((res) => {
    console.log(0);
    res();
  }).then(() => {
    throw new Error('aaa');
  });
  console.log(3);
}
fn1();
// -1 0 3
```

从 `eg.10` 的输出结果来看，`promise` 内的 `throw err` 并不会阻塞外部代码或者函数运行（即时没有 `.catch` 捕获）。

`eg.11`:

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
        console.log(10);
      }, 3 * 1000);
      throw new Error('aaa');
      setTimeout(() => {
        console.log(110);
      }, 3 * 1000);
      new Promise((res) => {
        console.log(31);
        res();
      }).then(() => {
        console.log(32);
      });
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
      .catch(() => {
        console.log(22);
      })
      .then(() => {
        console.log(23);
      })
      .then(() => {
        console.log(8);
      });
  })
  .then(() => {
    console.log(6);
  });
// 1 2 3 6 22 23 8  10
```

从 `eg.11` 的输出结果可以看出 `log(6)` 在 `log(22)` 之前被执行，**尽管没有调用 `resolve 或 reject`，而是通过 `throw err` 进入的回调，其后首个 `.then` 虽然不会执行，但是同样会参与到微任务队列占位排队（看作空，而非实际回调）。从这一点看，它和 `reject` 的表现完全一致**

#### 大胆猜想？

对于已经被执行和生成的 `promise`，尽管会因为一些原因（没有异常抛出，没有 `resolve 和 reject` 调用）导致其后的 `.then 和 .catch` 链式回调不被执行，但是它（当前已被执行的 `promise` 自身的 `.then or .catch`）同样会被放入微任务队列中（会当做空任务处理，仅参与微任务队列的占位排队，从而影响后续子任务的执行先后顺序）。

可能是通过某个标记位来确认后续的链式操作（`.then` 或`.catch`）是执行还是占位，默认标记为空表示两种类型回调都是占位？以至于没有异常抛出，没有 `resolve 和 reject` 调用的 `promise` 后面的所有链式操作没有执行（从表现上来看）。

比如调用过 `resolve` 则将默认值为空的标记修改为 `success`，表示链式中 `then` 需要执行而非占位，反之（抛出异常）亦然。

当然这些都是从目前表现上引发的猜想，实际实现需要阅读官方文档规范或者源码才能得知...
