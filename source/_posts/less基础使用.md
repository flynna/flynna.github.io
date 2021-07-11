---
title: less基础使用
date: 2021-06-14 15:42:11
tags: [less基础]
# type: "categories"
categories: [大前端, CSS]
---

`less是一个CSS预处理器，可以为网站启用可自定义，可管理和可重用的样式表。 做为 CSS 的一种形式的扩展，它并没有减少 CSS 的功能，而是在现有的 CSS 语法上，为CSS加入程序式语言的特性，以便可以通过Web浏览器读取。 它提供诸如变量，函数， mixins 和操作等功能，可以构建动态CSS。`

### 特性及优缺点

- 更清晰和更可读的代码可以以有组织的方式编写。
- 我们可以定义样式，它可以在整个代码中重复使用。
- less 是基于 JavaScript 的，是超集的 CSS。
- less 是一个敏捷工具，可以排除代码冗余的问题。
- less 轻松地生成可在浏览器中工作的 CSS。
- less 使您能够使用嵌套编写更干净，组织良好的代码。
- less 通过使用变量可以更快地实现维护。
- less 使您能够通过在规则集中引用它们来轻松地重用整个类。
- less 提供使用操作，使得编码更快并节省时间。
- 由于模块之间的紧密耦合，应当采取更多的努力来重用和/或测试依赖模块。
- 与旧的预处理器(如 Sass)相比，LESS 具有较少的框架，Sass 由框架 Compass ， Gravity 和 Susy 组成。

### 安装及编译

> npm install -g less
> lessc style.less style.css

### 语法

#### 变量

- 普通变量 @变量名：变量值；

```less
// 使用时直接 @color, 这里 color即为变量
@color: #fefefe;
.bg {
  background: @color;
}
// 编译后 .bg { background: #fefefe; }
```

- 变量作为选择器 @{ 变量名 }

```less
// 在使用选择器变量时，变量名需要放在用 @ 符号前缀的花括号 {} 内
@name: bg;
.@{name}-red {
  background: red;
}
// 编译后 .bg-red { background: red; }
```

- 变量作为属性名 --- 用法同 选择器变量

- `url变量`

```less
// 将文件路径设置变量
@xkd: "https://www.7k7k.com";
.banner {
  background: url("@{xkd}/img.png");
}
// 编译后 .banner { background: url('https://www.7k7k.com/img.png'); }
```

- 多维变量 @变量名：变量值 1 变量值 2 变量值 3；

```less
@colors: #666 #e00;
@size: (10px 20px) (30px 40px 50px) (10px 20px 30px 40px);
.dv {
  color: nth(@color, 1);
  margin: nth(@size, 1);
}
```

#### 混合

- 将一组属性栋一个集合混入到另一歌规则集合中。

```less
.color {
  color: #333;
}
.dv {
  background: #f5f5f5;
  .color();
}
// 编译后 .dv { color: #333; background: #f5f5f5; }
```

#### 嵌套

- 嵌套（nesting）代替层叠或与层叠结合使用的能力

```less
.container {
  background: #f5f5f5;
  .header {
    background: #434343;
    .nav {
      height: 60px;
    }
  }
}
// 编译后 .container .header .nav {} | .container .header {} | .container {}
```

#### 运算

算术运算符 `+`、`-`、`*`、`/` 可以对任何数字、颜色或变量进行运算，若单位不一致，以最左侧操作数的单位类型为准 。

```less
// 所有操作数被转换成相同的单位
@conversion-1: 5cm + 10mm; // 结果是 6cm
@conversion-2: 2 - 3cm - 5mm; // 结果是 -1.5cm

// conversion is impossible
@incompatible-units: 2 + 5px - 3cm; // 结果是 4px

// example with variables
@base: 5%;
@filler: @base * 2; // 结果是 10%
@other: @base + @filler; // 结果是 15%

@var: 50vh/2;
width: calc(50% + (@var - 20px)); // 结果是 calc(50% + (25vh - 20px))
```

#### 转义

- 转义后，值按照原样输出，不做任何处理。语法： `~"anything"` 或 `~'anything'`

```less
@min768: ~"(min-width: 768px)"; // 从 Less 3.5 开始，可以简写为：@min768: (min-width: 768px);
.element {
  @media @min768 {
    font-size: 1.2rem;
  }
}
// 编译后 @media (min-width: 768px) {//}
```

#### 函数

- 内置函数： Less 内置了多种函数用于转换颜色、处理字符串、算术运算等。这些函数在[Less 函数手册](https://less.bootcss.com/functions/)中有详细介绍。

#### 命名空间和访问符

```less
#bundle() {
  .button {
    display: block;
    border: 1px solid black;
    background-color: grey;
    &:hover {
      background-color: white;
    }
  }
  .tab {
    ...;
  }
  .citation {
    ...;
  }
}
```

现在，如果我们希望把 `.button` 类混合到 `#header a` 中，我们可以这样做：

```less
#header a {
  color: orange;
  #bundle.button(); // 还可以书写为 #bundle > .button 形式
}
```

#### 映射

```less
#colors() {
  primary: blue;
  secondary: green;
}
.button {
  color: #colors[primary];
  border: 1px solid #colors[secondary];
}
// 输出
.button {
  color: blue;
  border: 1px solid green;
}
```

#### 作用域

- Less 中的作用域与 CSS 中的作用域非常类似
- 与 CSS 自定义属性一样，混合（mixin）和变量的定义不必在引用之前事先定义

```less
@var: red;
#page {
  @var: white;
  #header {
    color: @var; // white
  }
}
// 两者不因为 @var 的先后顺序影响 只受作用域影响
@var: red;
#page {
  #header {
    color: @var; // white
  }
  @var: white;
}
```

#### 导入

```less
@import "library"; // library.less
@import "typo.css";
```

### 使用

- 使用 less 函数完成基础的内外边距样式

```less
.box-base-loop( @count, @type, @prop )when( @count > 0 ) {
  .@{type}-@{count} {
    @{prop}: ~"@{count}px";
  }
  .@{type}lr-@{count} {
    @{prop}: 0 ~"@{count}px";
  }
  .@{type}tb-@{count} {
    @{prop}: ~"@{count}px" 0;
  }
  .@{type}l-@{count} {
    @{prop}-left: ~"@{count}px";
  }
  .@{type}r-@{count} {
    @{prop}-right: ~"@{count}px";
  }
  .@{type}t-@{count} {
    @{prop}-top: ~"@{count}px";
  }
  .@{type}b-@{count} {
    @{prop}-bottom: ~"@{count}px";
  }
  .box-base-loop((@count - 5), @type, @prop);
}

.box-base-loop(30, m, margin);
.box-base-loop(30, p, padding);
```

### 更多

- 请查看 less 进阶使用详解
