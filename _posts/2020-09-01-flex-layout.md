---
title: Flex布局
title_en: 2020-09-01-flex-layout
date: 2020-09-01 17:35:12
tags:
  - CSS
---

## 前言

从第一代 `DIV + CSS` 正常流的排版方式，到 `Flexbox` 弹性盒模型，再到 `Grid` 的排版方式，以及最新的可以利用浏览器渲染引擎的 [CSS Houdini][1]。今天我们先来讲一下 `Flexbox` 弹性盒模型的基础以及如何[实现常见布局](#实现常见布局)，并对比传统 `DIV + CSS` 布局。

[caniuse flexbox](https://caniuse.com/#feat=flexbox)

<!-- more -->

## 容器

想要使用 `Flex` 布局，我们先要找一个元素将它指定为容器，然后就可以通过不同的容器属性，规定容器内 `item` 项目（子元素）的水平垂直排列（默认是从上到下，从左到右）、对齐方式等一些较为大力度的规则。容器内的项目，通过项目属性，可以进一步达到更细粒度的控制，后面会讲到。

```
display: flex/inline-flex;
```

注意，设为 Flex 布局以后，子元素的 `float`、`clear` 和 `vertical-align` 属性将失效。

## 容器属性

### flex-direction

主轴方向，项目排列方向

- row - 水平方向，从左到右
- row-reverse - 水平方向，从右到左
- column - 垂直方向，从上到下
- column-reverse - 垂直方向，从下到上

```
flex-direction: row | row-reverse | column | column-reverse;
```

### flex-wrap

项目超出容器后的换行方式

- nowrap - 不换行
- wrap - 换行，下一行从左往右排列
- wrap-reverse - 换行，下一行从右往左排列

```
flex-wrap: nowrap | wrap | wrap-reverse;
```

### flex-flow

`flex-direction` 和 `flex-wrap` 的组合

```
flex-flow: <flex-direction> || <flex-wrap>;
```

### justify-content

项目在主轴（横向轴）上的对齐方式，默认从左侧开始排列

- flex-start - 从左往右
- flex-end - 从右侧开始，从右往左
- center - 从左往右，中间对齐
- space-between - 从左往右，元素间均摊容器剩余空间
- space-around - 从左往右，元素间均摊容器剩余空间，第一个和最后一个元素两侧都会均摊剩余空间，不会贴边

```
justify-content: flex-start | flex-end | center | space-between | space-around;
```

### align-items

项目在交叉轴上的对齐方式

- flex-start
- flex-end
- center
- space-between
- space-around

```
align-items: flex-start | flex-end | center | space-between | space-around;
```

### align-content

- flex-start
- flex-end
- center
- space-between
- space-around
- stretch

```
align-content: flex-start | flex-end | center | space-between | space-around | stretch;
```

## 项目属性

### order

项目排序，从小到大排列

```
order: <integer>;
```

### flex-grow

项目增长，占据剩余空间的份数，默认是 `0`，不占据剩余空间

```
flex-grow: <number>; /* default 0 */
```

示例图片：

![](/assets/images/Flex布局/css_flex_flex-grow.html.png)

示例代码：

```html
<style>
  section {
    display: flex;
    background-color: #000;
    color: #fff;
    font-size: 30px;
  }
</style>

<section>
  <div style="flex-grow: 1;background-color: crimson;">1</div>
  <div style="width: 100px;">2</div>
  <div>333</div>
  <div style="flex-grow: 1;background-color: green;">4</div>
</section>
```

### flex-shrink

项目收缩时，占收缩总量的份数，默认是 1，最小收缩至项目内容占据的最小宽度

```
flex-shrink: <number>; /* default 1 */
```

示例图片：

![](/assets/images/Flex布局/css_flex_flex-shrink.html.png)

上图所示的容器，保持内容完整的最小宽度为：`100 + 100 + 54 + 18 = 272`

当浏览器窗口宽度缩小 `15px` 时，设置了 `flex-shrink: 2;` 的项目 1 的宽度由 `100px` 减少到了 `90px`，减少的宽度占本次减少宽度的 `2/3`；

而未设置 `flex-shrink` 的项目 2，宽度由 `100px` 减少到了 `95px`，减少的宽度占本次减少宽度的 `1/3`。

示例代码：

```html
<style>
  section {
    display: flex;
    background-color: #000;
    color: #fff;
    font-size: 30px;
  }
</style>

<section>
  <div style="width: 100px;flex-shrink: 2;background-color: crimson;">1</div>
  <div style="width: 100px;">2</div>
  <div>333</div>
  <div style="background-color: green;">4</div>
</section>
```

那么问题来了，细心的同学可能发现了，项目 1 和项目 2 的减少占比为什么不是 `2/5` 和 `1/5` 呢？

答案是：项目 3 和项目 4，没法再减少了。

在没有主动设置项目宽度的情况下，项目宽度由其内容所占据的宽度决定，默认情况下就是最小宽度。

### flex-basis

项目的起始宽度，默认值为 `auto`，表示起始宽度由内容决定

```
flex-basis: <length> | auto; /* default auto */
```

### flex

`flex-grow`、`flex-shrink` 和 `flex-basis` 组合

```
flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
```

### align-self

项目自身的对齐方式，默认继承容器 `align-items` 的值

- auto
- flex-start
- flex-end
- center
- baseline
- stretch

```
align-self: auto | flex-start | flex-end | center | baseline | stretch;
```

## 实现常见布局

### 水平居中

![](/assets/images/Flex布局/css_layout_1.png)

传统布局：

```css
.parent {
  position: relative;
}
.children {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

`Flex` 布局：

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### 圣杯布局

![](/assets/images/Flex布局/css_layout_2.png)

传统布局：

```css
.parent {
  min-height: 100vh;
  font-size: 30px;
  text-align: center;
}
.header {
  height: 100px;
  background-color: #efef;
}
.main {
  min-height: calc(100vh - 200px);
}
.main:after {
  display: block;
  content: "";
  clear: both;
}
.left {
  float: left;
  width: 200px;
  min-height: calc(100vh - 200px);
  background-color: red;
}
.center {
  min-height: calc(100vh - 200px);
  margin: 0 200px;
  background-color: green;
}
.right {
  float: right;
  width: 200px;
  min-height: calc(100vh - 200px);
  background-color: blue;
}
.footer {
  height: 100px;
  background-color: #efef;
}
```

```html
<div class="parent">
  <div class="header">header</div>
  <div class="main">
    <div class="left">left</div>
    <div class="right">right</div>
    <div class="center">center</div>
  </div>
  <div class="footer">footer</div>
</div>
```

`Flex` 布局：

```css
.parent {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
  font-size: 30px;
  text-align: center;
}
.header {
  height: 100px;
  background-color: #efef;
}
.main {
  display: flex;
  flex: 1;
}
.left {
  flex-basis: 200px;
  background-color: red;
}
.center {
  flex: 1;
  background-color: green;
}
.right {
  flex-basis: 200px;
  background-color: blue;
}
.footer {
  height: 100px;
  background-color: #efef;
}
```

```html
<div class="parent">
  <div class="header">header</div>
  <div class="main">
    <div class="left">left</div>
    <div class="center">center</div>
    <div class="right">right</div>
  </div>
  <div class="footer">footer</div>
</div>
```

相较于传统基于盒模型的布局方案，`Flex` 布局语义清晰，更为简洁，但是也不是万能的，如果要实现元素嵌套、样式细节比较多场景，还是传统的布局方式更灵活、可读，需要在适合的场景灵活运用，才能事半功倍。

## 参考链接

- http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
- https://blog.poetries.top/css-reference/flexbox/

[1]: https://developer.mozilla.org/en-US/docs/Web/Houdini
