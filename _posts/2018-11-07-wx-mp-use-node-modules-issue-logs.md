---
title: 微信小程序中使用npm包遇到的问题
title_en: wx-mp-use-node-modules-issue-logs
date: 2018-11-07 02:38:39
tags:
  - Log
  - NPM
  - MiniPrograme
---

## 前言

最近写了个[CNode 社区的微信小程序版本][4]，把在微信小程序中使用 npm 包，踩的坑记录一下，希望能给遇到类似问题的小伙伴，提供一些思路和方向。

## npm 支持

从小程序基础库版本 2.2.1 或以上、及开发者工具 1.02.1808300 或以上开始，小程序支持使用 npm 安装第三方包。

## 踩坑之路

<!-- more -->

1. 由于项目中需要格式化一些日期数据，于是选择了[moment][3]，一款 JavaScript 日期处理类库

2. 首先使用命令行，安装`moment`

   ```shell
   # 打开小程序根目录
   cd src
   npm install --production moment
   ```

3. 然后按照[小程序文档][1]往下操作，直到**构建完成**

4. 于是我们迫不及待的开始使用`moment`

   ```js
   import moment from "moment";

   let sFromNowText = moment(new Date() - 360000).fromNow();
   console.log(sFromNowText);

   // 控制台输出："6 minutes ago"
   ```

5. `moment`能够正常工作，但是很快我们发现英文不是我们想要的

6. 于是我们找到`moment`的[国际化配置][2]，并`设置全局语言为`中文

   ```js
   import moment from "moment";
   moment.locale("zh-cn");

   let sFromNowText = moment(new Date() - 360000).fromNow();
   console.log(sFromNowText);

   // 可是控制台，依然输出："6 minutes ago"
   ```

7. 尝试输出`moment.locale`执行后的返回值

   ```js
   // ...
   let sLang = moment.locale("zh-cn");
   console.log(sLang);

   // 控制台输出："en"
   ```

8. 发现设置语言环境失败了，经过排查和翻阅[小程序文档][1]后，发现通过微信开发者工具`构建npm`后，并不会将语言环境相关文件拷贝到`miniprogram_npm`目录下，仅将入口 js 文件及其依赖的模块进行打包处理，进而导致加载中文语言环境失败。以下是摘自小程序文档的一段话：

   ```
   构建打包分为两种：小程序 npm 包会直接拷贝构建文件生成目录下的所有文件到 miniprogram_npm 中；其他 npm 包则会从入口 js 文件开始走一遍依赖分析和打包过程（类似 webpack）。
   ```

9. 快速查看了一下`moment`源码，发现`moment.locale`方法，会从`./locale/`目录下加载语言环境包，于是尝试手动从`node_modules/moment/`目录下，将中文语言环境包，拷贝到`miniprogram_npm`目录下

   ```js
   // moment.js
   // ...
   function loadLocale(name) {
     // ...
     var aliasedRequire = require;
     aliasedRequire("./locale/" + name);
     // ...
   }
   // ...
   ```

10. 经过调试发现，`moment`定义语言环境时出错，原来是由于`构建npm`导致入口文件(`moment.js`)经过打包后更名为`index.js`导致：`Error: module "miniprogram_npm/moment/moment" is not defined`

    ```js
    // zh-cn.js
    (function (global, factory) {
      typeof exports === "object" &&
      typeof module !== "undefined" &&
      typeof require === "function"
        ? factory(require("../moment"))
        : typeof define === "function" && define.amd
        ? define(["../moment"], factory)
        : factory(global.moment);
    })(this, function (moment) {
      "use strict";

      var zhCn = moment.defineLocale("zh-cn", {
        // ...
      });

      return zhCn;
    });
    ```

11. 果然手动将`'../moment'`统一改为`'../index'`，然后重新执行

    ```js
    import moment from "moment";
    moment.locale("zh-cn");

    let sFromNowText = moment(new Date() - 360000).fromNow();
    console.log(sFromNowText);

    // 现在控制台，输出："6 分钟前"
    ```

12. 大功告成，nice! 但也别忘了回过头来，总结一下导致这几个问题的原因：
    1. `构建npm`会将 npm 包的入口文件（eg: moment.js），打包后更名为`index.js`
    2. `构建npm`仅将 npm 包的入口 js 文件及其依赖的模块进行打包处理，除此之外的文件（eg: moment 的语言环境包）并不会做打包处理

> `miniprogram_npm` 、`构建npm`及更多 npm 支持相关信息，请翻阅[小程序文档-npm 支持][1]

[1]: https://developers.weixin.qq.com/miniprogram/dev/devtools/npm.html
[2]: http://momentjs.cn/docs/#/i18n/
[3]: http://momentjs.cn
[4]: https://github.com/Lsnsh/cnode-wechat-applet
