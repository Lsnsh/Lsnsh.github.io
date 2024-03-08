---
title: 使用 JavaScript 实现人机对战的井字棋游戏
title_en: javascript-implement-tic-tac-toe-human-machine-game
date: 2021-11-23 10:57:03
tags:
  - JavaScript
  - Game
---

## 前言

在今年早些时候，参加了极客时间的前端进阶训练营，接触到了人机对战的井字棋游戏，说要写一篇博客记录一下，今天填坑来了属于是。

井字棋、或者说三子棋相信大家都不陌生，学生时代在本子上画一个“井”字和同学一起下棋，或者直接在带格子的语文作业本上下。

话不多说，我们先来看看，实现的效果：

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/1.gif)

<!-- more -->

具体如何实现，主要分为以下几部分：

- [棋盘绘制](#棋盘绘制)
- [用户落子](#用户落子)
- [检查胜负](#检查胜负)
- [模拟落子（一层）](#模拟落子（一层）)
- [最佳选择（递归）](#最佳选择（递归）)
- [电脑落子](#电脑落子)

## 棋盘绘制

- 使用 `CSS inline-block` 实现 `3 * 3` 布局
- 定义一维数组 `pattern` ，模拟 `3 * 3` 棋盘，记录落子情况
  - 数组元素值定义： `0` 表示未落子，`1` 表示 `O` 落子，`2` 表示 `X` 落子
- `render`：遍历一维数组，根据数组元素的值，动态绘制棋盘

```html
<!DOCTYPE html>

<style>
  #board {
    width: 306px;
    margin: 0 auto;
  }

  .cell {
    width: 100px;
    height: 100px;
    color: red;
    display: inline-block;
    border: 1px solid black;
    vertical-align: middle;
    line-height: 100px;
    font-size: 50px;
    text-align: center;
  }
</style>

<!-- 可以不写 body 标签 -->
<div id="board"></div>

<!-- 静态 -->
<script>
  let pattern = [2, 0, 0, 0, 1, 0, 0, 0, 0];

  function render() {
    const board = document.getElementById("board");
    // 绘制前，清空棋盘
    board.innerHTML = "";

    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        const cellValue = pattern[i * 3 + j];
        const cell = document.createElement("div");

        cell.classList.add("cell");
        cell.innerText = cellValue === 2 ? "X" : cellValue === 1 ? "O" : "";

        board.appendChild(cell);
      }
      board.appendChild(document.createElement("br"));
    }
  }

  render();
</script>
```

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/2.jpg)

## 用户落子

- 定义标识变量 `color` ，记录最近落子一方的元素值
- `render`：遍历一维数组，给棋格添加点击事件，将当前棋格的”坐标“作为参数传递
- `setCellValue`：定义落子方法，处理一维数组和标识更新、以及棋盘重绘

```html
<!-- 可点击 -->
<script>
  let pattern = [2, 0, 0, 0, 1, 0, 0, 0, 0];
  let color = 1;

  function render() {
    // ...
    cell.addEventListener("click", () => setCellValue(i, j));
    // ...
  }

  function setCellValue(i, j) {
    pattern[i * 3 + j] = color;
    color = 3 - color;
    render();
  }

  render();
</script>
```

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/3.gif)

## 检查胜负

- `setCellValue`：用户落子后，检查游戏胜负，有一方获胜，游戏结束
- `checkWinOrLose`：遍历棋盘，依次检查三纵、三横、两斜的胜负情况

```html
<!-- 可点击+可以判定胜负 -->
<script>
  let pattern = [0, 0, 0, 0, 0, 0, 0, 0, 0];
  let color = 1;

  function render() {
    //...
  }

  // 落子
  function setCellValue(i, j) {
    pattern[i * 3 + j] = color;
    render();
    if (checkWinOrLose(pattern, color)) {
      alert(`${color === 2 ? "X" : color === 1 ? "O" : ""} is winner!`);
      // 游戏结束
      return;
    }
    color = 3 - color;
  }

  // 检查胜负
  function checkWinOrLose(pattern, color) {
    let win = false;

    // 三横
    for (let i = 0; i < 3; i++) {
      win = true;
      for (let j = 0; j < 3; j++) {
        if (pattern[i * 3 + j] !== color) {
          win = false;
        }
      }
      if (win) {
        return win;
      }
    }

    // 三纵
    for (let i = 0; i < 3; i++) {
      win = true;
      for (let j = 0; j < 3; j++) {
        if (pattern[j * 3 + i] !== color) {
          win = false;
        }
      }
      if (win) {
        return win;
      }
    }

    // 两斜
    // 正斜线 /
    {
      win = true;
      for (let i = 0; i < 3; i++) {
        if (pattern[i * 3 + 2 - i] !== color) {
          win = false;
        }
      }
      if (win) {
        return win;
      }
    }

    // 反斜线 \
    {
      win = true;
      for (let i = 0; i < 3; i++) {
        if (pattern[i * 3 + i] !== color) {
          win = false;
        }
      }
      if (win) {
        return win;
      }
    }
    return false;
  }

  render();
</script>
```

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/4.gif)

## 模拟落子（一层）

- `setCellValue`：任意一方落子之后，模拟落子（一层），预判胜方
  - `predictWin`：遍历棋盘，将棋盘数组拷贝，模拟即将落子方落子，得出获胜的落子位置
- `clone`：拷贝一维数组

```html
<script>
  //...
  function render() {
    //...
  }

  // 落子
  function setCellValue(i, j) {
    // ...
    if (predictWin(pattern, color)) {
      alert(`${color === 2 ? "X" : color === 1 ? "O" : ""} will win!`);
    }
  }

  // 检查胜负
  function checkWinOrLose(pattern, color) {
    //...
  }

  // 模拟落子（一层）
  function predictWin(pattern, color) {
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        // 跳过已经落子的位置
        if (pattern[i * 3 + j]) {
          continue;
        }
        let temp = clone(pattern);
        temp[i * 3 + j] = color;
        if (checkWinOrLose(temp, color)) {
          return [i, j];
        }
      }
    }
    return null;
  }

  function clone(value) {
    return Object.create(value);
  }

  render();
</script>
```

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/5.gif)

## 最佳选择（递归）

- `setCellValue`: 任意一方落子之后，寻找最佳选择（递归），预判局势
  - `bestChoice`：
    - 执行流程：
      - 调用 `predictWin` 方法模拟落子（一层），如果返回获胜的落子位置，终止执行，返回结果
      - 遍历棋盘，我方（`color`）模拟落子后，以对手方（（`3 - color`））视角，代入参数**递归调用** `bestChoice` 方法，得出对手方的最佳结果
      - 对手方的最佳结果取反（`-r`），即是我方的最差结果，比较后更新对于我方最好的落子位置（`point`）和结果（`result`），直到获胜（`result = 1`），跳出遍历
      - 遍历结束，返回结果
    - 返回值定义：
      - color：落子方
      - point：落子“坐标”
      - result：`-2` 表示未开始，`-1` 表示输，`0` 表示和局，`1` 表示赢

```html
<script>
  //...
  function render() {
    //...
  }
  // 落子
  function setCellValue(i, j) {
    //...
    let choice = bestChoice(pattern, color);
    console.log(
      `${
        choice.color === 2 ? "X" : choice.color == 1 ? "O" : ""
      } best choice: point=${JSON.stringify(choice.point)}, result=${
        choice.result
      }`
    );
  }
  // 检查胜负
  function checkWinOrLose(pattern, color) {
    //...
  }
  // 模拟落子（一层）
  function predictWin(pattern, color) {
    //...
  }

  // 最佳选择（递归）
  function bestChoice(pattern, color) {
    let point = predictWin(pattern, color);
    if (point) {
      return {
        color: color,
        point: point,
        result: 1,
      };
    }

    // 起步结果
    let result = -2;
    outer: for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        // 跳过已经落子的位置
        if (pattern[i * 3 + j]) {
          continue;
        }
        let temp = clone(pattern);
        temp[i * 3 + j] = color;
        // 根据「对手方」目前情况下未来「最好的结果」，换位得出我方「最差的结果」（-r）
        let r = bestChoice(temp, 3 - color).result;
        // 比较后更新落子位置和结果
        if (-r >= result) {
          result = -r;
          point = [i, j];
        }
        // 剪枝
        if (result === 1) {
          break outer;
        }
      }
    }
    return {
      color: color,
      point: point,
      result: point ? result : 0,
    };
  }
  //...

  render();
</script>
```

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/6.gif)

## 电脑落子

- `userSetCellValue`：用户落子后，电脑落子
  - `computerSetCellValue`：调用 `bestChoice` 得到最佳选择（包含落子位置和结果），然后落子

```html
<script>
  let pattern = [0, 0, 0, 0, 1, 0, 0, 0, 0];
  let color = 2;

  function render() {
    // ...
    cell.addEventListener("click", () => userSetCellValue(i, j));
    // ...
  }

  // 用户落子
  function userSetCellValue(i, j) {
    // ...
    computerSetCellValue();
  }

  // 电脑落子
  function computerSetCellValue() {
    let choice = bestChoice(pattern, color);
    if (choice.point) {
      pattern[choice.point[0] * 3 + choice.point[1]] = color;
      render();
      if (checkWinOrLose(pattern, color)) {
        alert(`${color === 2 ? "X" : color === 1 ? "O" : ""} is winner!`);
        // 游戏结束
        return;
      }
      color = 3 - color;
    }
  }

  // 检查胜负
  function checkWinOrLose(pattern, color) {
    //...
  }
  // 模拟落子（一层）
  function predictWin(pattern, color) {
    //...
  }
  // 最佳选择（递归）
  function bestChoice(pattern, color) {
    //...
  }
  //...

  render();
</script>
```

![](/assets/images/11-JavaScript%20实现人机对战的井字棋游戏/7.gif)

## 总结

回顾一下，在这个人机对战的井字棋游戏中，机器操作的主体逻辑在 `bestChoice - 最佳选择` 方法中，基于遍历和递归，机器模拟机器下一棋后，再模拟人下一棋，循环往复，直到得出 `bestChoice - 最佳选择`。

说白了就是利用计算机的算力，穷举出所有的可能性，找规则内的最优解。

井字棋还不够计算机算力塞牙缝的，五子棋用穷举法好像会有点小卡，到围棋好像就没法用穷举法应对所有情况了。

欢迎点赞评论，游戏主体部分有了，但是还有诸多不完善的地方，在线链接：

https://codepen.io/lsnsh-the-bashful/full/wvrBpqw

[1]: https://codepen.io/lsnsh-the-bashful/full/wvrBpqw
