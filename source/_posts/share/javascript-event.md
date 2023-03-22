---
title: javascript中的事件组成、分类、绑定与监听
date: 2022-07-19 22:35:26
updated: 2022-07-19 22:35:26
tags:
  - javascript
  - 事件冒泡
  - 事件委派
categories:
  - 技术分享
  - JavaScript
---

### What?

主要是介绍事件相关知识。涵盖事件的组成部分、常见事件分类、以及当我们为元素绑定事件监听以后，如何阻止事件冒泡（捕获）、阻止默认事件、委派、如何解绑...

<!-- more -->

### 事件是由哪些元素构成的？

#### 事件源

源：源头，来源。描述事件的产生源头，例如给一个按钮绑定了点击事件，那么这个按钮就是事件源 —— 调用者。

> btn.onclick = function(){}，btn 即为事件源

#### 事件类型

事件类型，比如已知的点击事件、 移入移出事件、表单提交事件、浏览器事件...

> 注意：类似于 onclick 这种方式的事件绑定，它的类型是 click。on 前缀只是绑定事件的一种方式 onclick 不是类型。

#### 事件处理函数

事件被触发以后，所执行的函数体，一般情况下是匿名函数。

> function(){}

### 事件对象 `e`

#### 事件对象获取

> window.event 以前只能在 IE 中能使用

> Chrome 和 Firefox 提供了另一种方法 即 提供一个 参数。（事件处理函数的第一个默认参数）

存在兼容性，兼容处理：

```javascript
btn.onclick = function (e) {
  const e = e || window.e;
  // ...
};
```

#### 事件目标元素获取

兼容性处理：

> target = e.target || e.srcEle (IE)

#### 获取触发事件的鼠标按键

获取：非`IE`下为`e.button`，`IE`下为`e.which`。

对应值：

| 环境  | 鼠标左键 | 鼠标滚轮 | 鼠标右键 |
| ----- | -------- | -------- | -------- |
| IE    | 1        | 2        | 3        |
| 非 IE | 0        | 1        | 2        |

#### 获取光标位置

- `e.clientX/clientYY`获取到的是**触发点**相对浏览器可视区域左上角距离，不随页面滚动而改变。
- `e.pageX/pageY`获取到的是**触发点**相对文档区域左上角距离，会随着页面滚动而改变

**拓展，注意 display 为 none 时， 宽高均为 0，拿数据的时候注意判断**，`ele`指的元素。

- `ele.offsetX/offsetY`获取到是**触发点**相对被触发`dom`的左上角距离，不过左上角基准点在不同浏览器中有区别，其中在`IE`中以内容区左上角为基准点不包括边框，如果**触发点**在边框上会返回负值，而`chrome`中以边框左上角为基准点。
- `ele.layerX/layerY`获取到的是**触发点**相对被触发`dom`左上角的距离，数值与`offsetX/offsetY`相同，这个变量就是`firefox`用来替代`offsetX/offsetY`的。（有个前提条件就是，被触发的 dom 需要是有定位的元素，否则会返回相对`html`文档区域左上角的距离）

- `ele.offsetWidth/offsetHeight`**元素**的宽度/高度，包括`border`。

- `ele.offsetLeft/offsetTop`**元素**到有定位的父元素的左边/上边的距离 ，如果父元素都没有定位 ，那就是相对于 body 上边的距离。

- `ele.offsetParent`拿到有定位的父元素，都没定位，那么拿到的就是`body`。

- `ele.clientWidth/clientHeight`元素的宽度/高度，不含`border`。

- `ele.clientLeft/clientTop`元素的左/上边框宽度

**火狐以外的所有浏览器使用（兼容 IE）**

- `window.screenLeft`浏览器窗口相对于显示器(屏幕)左边的距离

- `window.screenTop`浏览器窗口相对于显示器(屏幕)上边的距离

**火狐浏览器使用（不兼容 IE）**

- `window.screenX`浏览器窗口相对于显示器(屏幕)左边的距离

- `window.screenY`浏览器窗口相对于显示器(屏幕)上边的距离

#### 键盘事件对象的属性

> ctrlkey ctrl 按钮是否被按下，值为 boolean

> shiftkey shift 按钮是否被按下，值为 boolean

> altkey alt 按钮是否被按下，值为 boolean

...

`keyCode`获取触发`kayup`事件的键盘按钮编号，可以通过`keyCode`为具体的某一个按键添加对应的功能逻辑。

---

### **常见的几种事件类型**

#### 鼠标事件

- `click`---鼠标左键单击。
- `dblclick`---鼠标双击。
- `contextmenu`---鼠标右键点击事件。
- `mousedown`---鼠标按下。
- `mouseup`---鼠标抬起(松开按键)。
- `mousemove`---鼠标移动。
- `mouseenter`---鼠标移入。
- `mouseover`---鼠标移入。
- `mouseleave`---鼠标离开。
- `mouseout`---鼠标离开。

##### `mouseenter`和`mouseover`的区别：

> `over`会给子元素同时绑定有冒泡的行为

> `enter`没有冒泡，不会给其子元素绑定事件

#### 键盘事件

- `keydown`按下键盘键
- `keyup`紧接着`keydown`事件触发（只有按下字符键时触发）。
- `keypress`释放键盘键。

#### 浏览器事件

- `load`---页面资源加载结束时执行, 通过这个函数可以使`js`放在`head`中不影响正常加载。
- `scroll`---浏览器滚动时执行，与滚轮无关，只和浏览器的滚动条是否滚动有关系。
- `resize`---浏览器的尺寸发生变化时触发执行。~~（可以用`js`实现，响应式布局）不咋用~~

```javascript
window.onload = function () {};
```

#### 触摸事件

**`touchstart`会先于`click`事件执行，如果不想执行`click`，可以阻止默认事件。**

- `touchstart`手指触摸屏幕时触发，即使已经有手指在屏幕上也会触发。
- `touchmove`手指在屏幕滑动时触发。
- `touchend`手指从屏幕时移开时触发。

```javascript
ele.ontouchstart = function () {};
```

#### 表单事件

- `change`---焦点前后比较，发生了改变则触发该函数。
- `input`---只要输入了内容就会触发，一直输入一直触发。
- `focus`---获取焦点。
- `blur`---失去焦点。
- `submit`---提交事件。

---

### **事件绑定与事件解绑**

#### 添加一个事件监听

##### `addEventListener`

**注意：通过这种方式，可以同时绑定多个同类型事件。**

- 第一个参数：事件的类型，不需要加`on`前缀。
- 第二个参数：是函数执行的内容。
- 第三个参数：冒泡还是捕获`false`为冒泡，`true`为捕获，默认为冒泡`false`。

```javascript
ele.addEventListener('click', function () {
  console.log('我是绑定的第一个事件');
});

ele.addEventListener('click', function () {
  console.log('我是绑定的第二个事件');
});
```

##### `attachEvent`

- 第一个参数：`on` + 事件类型。
- 第二个参数：是函数，需要执行的代码。

```javascript
ele.attachEvent('onclick', function () {
  console.log('我是绑定的第一个事件');
});

ele.attachEvent('onclick', function () {
  console.log('我是绑定的第二个事件');
});
```

##### 两者区别

存在兼容性的问题，可以自己通过封装函数解决兼容问题。

> addEventListener：-ie 8 以上版本的浏览器支持，**顺序绑定 -> 顺序执行**

> attachEvent：**-ie 10 和 ie 9，顺序绑定 顺序执行。-ie 8 及以下是顺序绑定 -> 倒叙执行。**

#### 解绑事件

##### `removeEvent`('事件类型'，fn)

> addEventListener -> removeEventListener 解绑的时候，事件类型和事件处理函数(**地址**)必须是一样的。

**注意：需要解绑的时候，事件处理函数 fn 不能是匿名函数，否则无法正常解绑。fn 在绑定和解绑过程中，书写结构要完全保持一致。**

```javascript
// 注意：由于匿名 fn地址不一样 这种方式 解绑不了
ele.addEventListener('click', function () {
  console.log('object');
});

// Error
document.onclick = function () {
  ele.removeEventListener('click', function () {
    console.log('object');
  });
};

// 采用这种方式进行函数解绑
function fn() {
  console.log('object');
}

ele.addEventListener('click', fn);

document.onclick = function () {
  ele.removeEventListener('click', fn);
};
```

##### `detachEvent`('事件类型'，fn)

要求同上。

> attachEvent -> detachEvent

---

### **阻止默认事件**

比如`a`标签有`href`时，会自动跳转，如果要阻止这个行为：

> 方式一：e.parentDefault() 非 IE 浏览器

> 方式二：return false; 使用 return false 阻止默认事件，只能将 return false 放在函数的最后面

> 方式三：e.returnValue = false；

### **事件冒泡与事件捕获**

**冒泡：从子元素到根元素，从小到大。捕获：从根元素到子元素，从大到小。**

```javascript
/* 

​      inner 

​        center 

​          outer

​            body

​              html

​                documwnt

​                  window

​    事件执行机制 冒泡 捕获

​    */
```

#### 阻止冒泡（存在低版本兼容）

> e.stopPropagation(); or e.cancelBubble = true;

只有绑定了同类型事件才会触发冒泡。

阻止冒泡机制，只针对事件源本身，如果通过事件委派的形式为其子元素添加事件，那么就不能对其子元素实行阻止冒泡

---

### **事件委派**

把这个事件绑定在父元素身上，然后由父元素委派给子元素。

```javascript
box.addEventListener('click', function (e) {
  // 通过 e.target 去判断 目标源元素是哪一个
  // console.log(e.target)

  if (e.target.className == 'dv') {
    console.log('点到小的了');

    // 比如这里，设置阻止冒泡就会失效
    //e.target.stopPropagation()
  }
});
```
