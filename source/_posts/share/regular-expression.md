---
title: 如何构建你想要的正则表达式?
date: 2022-08-30 20:32:44
updated: 2022-08-30 20:32:44
tags:
  - 正则表达式
categories:
  - 技术分享
  - 正则表达式
---

### 什么是正则表达式

正则表达式`(regular expression)`描述了一种字符串匹配的模式`(pattern)`，可以用来检查一个串是否含有某种子串、将匹配的子串替换或者从某个串中取出符合某个条件的子串等。

> 正则表达式是由普通字符（例如字符 a 到 z）以及特殊字符（称为"元字符"）组成的文字模式。模式描述在搜索文本时要匹配的一个或多个字符串。正则表达式作为一个模板，将某个字符模式与所搜索的字符串进行匹配。

<!-- more -->

### 正则表达式的创建

正则表达式是引用数据类型，又称规则表达式。可以通过内置构造函数创建，亦可以通过字面量方式创建。

#### 字面量方式创建

> 语法：`const reg = /xxx/`双斜杠包夹的内容就是正则表达式。

#### 内置构造函数创建

> 语法：`const reg = new RegExp('xxx')` 通过`new`关键字创建正则实例。

`so：`如果写一个正则表达式`xxx`，来先了解正则表达式的一些比较特殊的符号意义所在？

### 元字符

#### `\d`出现一个数字

```javascript
const reg = /\d/;
const str = 'sdasdna126sdfd6550sad';
const res = reg.test(str); // true
```

其他字符同理，就不单独写例子了，通过`test`则返回`true`.

#### `\D`出现一个非数字

#### `\s`出现一个空格

#### `\S`出现一个非空格

#### `\w`出现一个数字字母下划线

#### `\W`出现一个非数字字母下划线

#### `.`出现一个除了换行以外的字符

实际使用过程中，称之为任意字符也不为过。

### 限定符

#### `*`

> 出现的次数为`0 - infinite`(正无穷)，`eg.` `/a*/`表示出现 0 - 无穷次~~（因为限定的次数最低可以是 0，所以理解为所有字符亦可）😂😂😂~~

#### `+`

> 出现的次数为`1 - infinite`(正无穷)，`eg.` `/\d+/`表示数字出现一次以上

#### `?`

> 出现的次数为`0 - 1`，`eg:` `/\d?/`可以不出现数字，也可以出现数字（仅匹配出现的第一次）

#### `{n} 和 {n,}`

> 单类型字符连续出现`n`个，`eg.` `/\d{3}/`表示连续三个或三个以上数字在一块 ，只要出现一次即为`true`.

#### `{n,m}`

> 单类型字符连续出现**至少**`n - m`次（意味着也可以超过），`eg.`

```javascript
const str = 's354ada1sdf22515553sg4d5sf';
console.log(/\d{5,7}/.test(str)); // true
```

估计有些童鞋已经蒙圈了，既然都是至少这么多次，那它们直接有啥区别？比如`{n,} {n,m}`，又该怎么使用？，别急来先看**边界符**：

### 边界符

写了边界符后，在使用`test`检测时，搭配限定符，就可实现区分`{n, m} 和 {n, }`，前者代表整个长度只能在`n - m`，后者是长度至少为`n`.

#### `^ 以...开头`

#### `$ 以...结尾`

```javascript
const reg1 = /^\d$/;
const str1 = '1';
const str2 = '123';
console.log(reg1.test(str1)); //	true
console.log(reg1.test(str2)); //	false（\d只能出现一个）

const reg2 = /^\w{4,9}$/;
const str3 = '_as123';
const str4 = '_as1234567';
console.log(reg2.test(str3)); //	true
console.log(reg2.test(str4)); //	false（指定了\w 出现4-9次 长度  字符串长度不能超过 9，同时 只能是\w字符）
```

### 特殊符号

#### `\`转译符

意指把 **原本没有特殊含义的东西 加上 \就没了特殊含义**；~~把原本就有特殊含义的东西加上`\`就没有特殊含义了~~

#### `()`把一块东西当成整体

#### `|`或

> 注意：`||`逻辑或 `|`占位或 两者的区别

`eg.` 简版邮箱`/^\w{6,12}@(qq|163|sina)\.(com|cn|net)/`

#### `[]`取中间任意**一个**

只能取中括号内的一个字符，注意字符长度，`eg.`

```js
const reg = /^[abcd]$/;
const str = 'a';
const str2 = 'abcd';
console.log(reg.test(str)); // true
console.log(reg.test(str2)); // false
```

#### `-`到..至..

`eg. ` 对于正则 `/^[a-zA-Z0-9_]$/`，字符`M`就符合.

#### `[^]`非

对`[]`的正则取非，`eg. /^[^a-z]$/`，指非`a-z`的字符，长度为 1.

再然后要说的就是标识符了，它写在`//`的后面，具体使用如下：

### 标识符

`g`全局匹配，`i`忽略大小写，`eg.`

```javascript
const reg = /m/gi;
let str = 'MasnonasmsdfMasm';
str = str.replace(reg, '*');
console.log(str); // *asnonas*sdf*as*
```

当然标识符不仅仅是上面介绍的这两个，[https://regex101.com/](https://regex101.com/)可以在这个网站上测试及学习...

### 正则表达式原型上的方法

`eg.`正则表达式`reg = /xxx/`.

#### 对字符串进行验证

`reg.test(str)`. 符合规则就返回`true`否则返回`false`.

```javascript
const reg = /abc/;
const str = 'ahsjdhaabbc';
const res = reg.test(str); // false
```

#### 捕获字符串里面符合规则的内容

`reg.exec(str)`第一次是从头开始匹配，如果找到符合规则的字符，那么返回一个数组，数组的第 0 项就是捕获到的符合规则的字符，第二次应该从第一次结束的位置开始捕获，如果找到也是返回一个数组，如果没有找到符合规则的字符，就返回一个`null`，下次执行又是从头开始重新捕获.~~（用的很少）~~

#### 数据转换

`reg.toString()`将正则表达式转换为字符串。

### 字符串与正则相关方法

#### 字符串基于正则完成替换

`str.replace(regexp|substr, newSubStr|function)`. `reg：`一个`RegExp`对象或者其字面量，该正则所匹配的内容会被第二个参数的返回值替换掉~~（该方法的具体参数使用请参阅文档，这里仅介绍正则相关）~~

```javascript
const reg = /m/gi;
let str = 'MasnonasmsdfMasm';
str = str.replace(reg, '*'); // *asnonas*sdf*as*
```

和`replace`相近的还有`replaceAll`，与前者差别就是后者匹配所有符合正则的子串并替换，而前者则需要通过指定`g`参数实现。

#### 提取字符串中符合正则的匹配项

`str.match(reg)`返回一个数组，将所有符合的子串提取出来。

```javascript
const str = 'Ma123sdf4565dsfsdf789sdf35';
console.log(str.match(/\d{2}/g));
// ["12", "45", "65", "78", "35"]
```

#### 查找符合规则的字符串子串

`str.search(reg)`查找符合规则的字符，返回相对应的下标，如果有多个符合，那么就返回第一个，作用同`indexOf`，不过后者无法传入`reg`正则查找.

### 贪婪模式和非贪婪模式

在`javascript`中的正则表达式的匹配方式默认是贪婪模式。

`eg. `

#### 贪婪模式

```javascript
// 指匹配 b 开头，后续出现 0 -任意次数的 a 的子串
const reg = /ba*/g;
const str = 'baaaabab';
console.log(str.match(reg)); // ['baaaa', 'ba', 'b']
```

正是因为贪婪匹配（尽可能多的匹配符号条件的子串），导致了匹配出的第一项是`baaaa`（从前向后匹配，一直到匹配到的字符串不符合条件时停止）, 而非`b`.

#### 非贪婪模式

当然有贪婪也就有非贪婪，如何指定？

> 使用字符`?`标识正则使用非贪婪模式进行匹配(这是`?`的另外一种用法含义) ~~看你怎么理解~~

对上面例子的正则改造一下：

```javascript
const reg = /ba*?/g;
const str = 'baaaabab';
console.log(str.match(reg)); // ['b', 'b', 'b']
```

`/ba*`为原有的正则表达式，在其后添加`?`指明非贪婪，匹配到的结果自然就是`['b', 'b', 'b']`.

以为这样就结束了？`no no no...` 正则在匹配时还有另外一条规则...**比贪婪规则的优先级更高**，`so`在你使用时,如果不注意...你会怀疑自己的...

#### 匹配的优先级

> 最先开始的匹配拥有最高的优先权 —— The match that begins earliest wins。

怎么解释，匹配优先匹配左边的**第一个字符**，然后尽可能少的去匹配后续字符，而不是忽略第一个字符，去找第二个...😋😋😋 ~~匹配结果不会从左侧缩减字符，而只会进行右侧的懒惰匹配~~ `eg.`

```javascript
const reg = /a*?b/g;
const str = 'baaaabab';
console.log(str.match(reg)); // ['b', 'aaaab', 'ab']
```

如果你未理解该规则，那么按照非贪婪的理解，打印的结果应该是`['b', 'b', 'b']`，**这是错误的**.

**正确结果**应该是`['b', 'aaaab', 'ab']`.