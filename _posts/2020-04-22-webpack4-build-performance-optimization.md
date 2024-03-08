---
title: webpack3.x升级4.x之后打包速度优化
title_en: webpack4-build-performance-optimization
date: 2020-04-22 19:18:56
tags:
  - Vue
  - Webpack
---

## 前言

[上一篇][1]我们讲到，webpack3.x 升级 4.x 后打包大小优化，今天讲一下 `webpack 4.x`（webpack 4.43.0） 的打包速度优化，其实在升级了 `webpack4` 之后对于打包速度就已经有了很大的提升，但是查找时间（缩小范围）、`loader` 处理时间（多进程）、二次打包时间（缓存）仍有可优化的空间。

## 打包分析

在优化之前我们需要清楚项目打包的性能情况，这里我们使用 [speed-measure-webpack-plugin][2] 插件来进行分析

**webpack.base.js**

```diff
+const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');

+const smp = new SpeedMeasurePlugin({
+  outputFormat: 'humanVerbose'
+});

const webpackConfig = merge(baseWebpackConfig, {
  // ..
});

-module.exports = webpackConfig;
+module.exports = smp.wrap(webpackConfig);
```

打包看下，本次耗时 `62,361 ms`，列出了 `Plugins` 和 `Loaders` 具体耗时的细节：

图比较长但是基本能看出，其中耗时较多的是 `vue-loader` 和 `ts-loader`

<!-- more -->

![](/assets/images/webpack3升级4之后打包速度优化/1.jpg)

## 查找时间优化-exclude/include

`webpack` 从入口文件开始，根据依赖关系查找模块，我们要尽可能少的处理模块，最常见的就是排除 `exclude: /node_modules/`，`exclude` 和 `include` 同时使用时 exclude 优先级更高

**webpack.base.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: "vue-loader",
      },
      {
        test: /\.js$/,
        loader: "babel-loader",
        // 确保 node_modules 中 Vue 单文件组件的 <script> 部分能被转译
        exclude: (file) => /node_modules/.test(file) && !/\.vue\.js/.test(file),
      },
      {
        test: /\.tsx?$/,
        loader: "ts-loader",
        exclude: /node_modules/,
        options: {
          // 禁用类型检查，可以通过插件的方式使用
          transpileOnly: true,
          experimentalWatchApi: true,
          appendTsSuffixTo: [/\.vue$/],
        },
      },
    ],
  },
};
```

> `ts` 类型检查，可以单独使用插件： [ForkTsCheckerWebpackPlugin][4]

## loader 处理时间优化-thread-loader

> Each worker is a separate node.js process, which has an overhead of ~600ms. There is also an overhead of inter-process communication.
> 每个 `worker` 都是一个单独的 `node.js` 进程，其开销约为 `600` 毫秒。进程间通信也有开销。[查看更多][11]

**webpack.base.js**

```js
// ...
const isProduction = process.env.NODE_ENV === "production";

if (isProduction) {
  const threadLoader = require("thread-loader");
  // 预热 worker
  threadLoader.warmup(
    {
      // pool options, like passed to loader options
      // must match loader options to boot the correct pool
    },
    [
      // modules to load
      // can be any module
      "babel-loader",
      "ts-loader",
      "vue-loader",
      "sass-loader",
    ]
  );
}

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.vue$/,
        use: isProduction ? ["thread-loader", "vue-loader"] : ["vue-loader"],
      },
      {
        test: /\.js$/,
        use: isProduction
          ? ["thread-loader", "babel-loader"]
          : ["babel-loader"],
        exclude: (file) => /node_modules/.test(file) && !/\.vue\.js/.test(file),
      },
      {
        test: /\.tsx?$/,
        loader: "ts-loader",
        exclude: /node_modules/,
        use: isProduction
          ? [
              "thread-loader",
              {
                loader: "ts-loader",
                options: {
                  happyPackMode: true,
                  transpileOnly: true,
                  experimentalWatchApi: true,
                  appendTsSuffixTo: [/\.vue$/],
                },
              },
            ]
          : [
              {
                loader: "ts-loader",
                options: {
                  transpileOnly: true,
                  experimentalWatchApi: true,
                  appendTsSuffixTo: [/\.vue$/],
                },
              },
            ],
      },
      {
        test: /\.s?css$/,
        use: isProduction
          ? [
              MiniCssExtractPlugin.loader,
              "css-loader",
              {
                loader: "thread-loader",
                options: {
                  workerParallelJobs: 2,
                },
              },
              "sass-loader",
            ]
          : ["vue-style-loader", "css-loader", "sass-loader"],
      },
    ],
  },
};
```

### 与 ts-loader 使用时报错

```log
ERROR in ./src/main.ts
Module build failed (from ./node_modules/thread-loader/dist/cjs.js):
Thread Loader (Worker 0)
Cannot read property 'errors' of undefined
    at PoolWorker.fromErrorObj (/Users/yourProjectPath/node_modules/thread-loader/dist/WorkerPool.js:262:12)
    at /Users/yourProjectPath/node_modules/thread-loader/dist/WorkerPool.js:204:29
    at mapSeries (/Users/yourProjectPath/node_modules/neo-async/async.js:3625:14)
    at PoolWorker.onWorkerMessage (/Users/yourProjectPath/node_modules/thread-loader/dist/WorkerPool.js:170:35)
    at successfulTypeScriptInstance (/Users/yourProjectPath/node_modules/ts-loader/dist/instances.js:119:28)
    at Object.getTypeScriptInstance (/Users/yourProjectPath/node_modules/ts-loader/dist/instances.js:34:12)
    at Object.loader (/Users/yourProjectPath/node_modules/ts-loader/dist/index.js:17:41)
```

`ts-loader` 设置 `happyPackMode: true`，[查看更多][5]

### 与 node-sass 使用

看 [webpack 文档][8]有提到说 `node-sass` 搭配 `thread-loader` 使用的时候需要设置 `workerParallelJobs: 2`，亲测发现现在不用这样设置了，速度会快一些

> `thread-loader@2.1.3` `node-sass@4.13.1` `node@12.16.2`

### 与 sass-loader 使用时报错

报错信息如下：

```log
Module build failed (from ./node_modules/thread-loader/dist/cjs.js):
Thread Loader (Worker 3)
this.getResolve is not a function
```

暂时没有解决方案，将 `sass-loader` 降级到 `v7.3.1` 可以正常运行，[相关 issue][3]

## 二次打包时间优化-cache

使用 `webpack4` 二次打包的时候，我们会发现比第一次快来不少，这是因为 `webpack` 内置 `terser-webpack-plugin` 来最小化 JS 文件，默认会启用多进程和缓存，第二次的时候直接读取项目下 `node_modules/.cache/terser-webpack-plugin` 目录，所以较第一次打包会快上不少

同样的我们在使用 `babel-loader` 的时候，也可以将缓存启用，设置 `cacheDirectory: true` 即可

如果你想缓存其他 `loader` 的处理结果，你可以使用 [cache-loader][6]

<!-- 因为 `worker` 的启动和进程间通信都有开销，所以 -->

## 总结

项目不够大，服务器逻辑核数（物理核心 \* 每个核心的线程数）不够多，不要使用多进程打包，反向优化说的就是我，使用 `thread-loader` 多进程打包后，在本地构建反而增加了近 `40s` 的时间，如下图：

![](/assets/images/webpack3升级4之后打包速度优化/2.jpg)

这篇文章拖了很久，半个多月才写完，总结一下拖延的点：

- 一是中途看到[【Webpack】538- 打包速度提升指南][7]这篇文章，觉得写的很好，一度都不想开始优化了，觉得写了没别人写得好，还写出来干啥。后来硬着头皮优化完了，发现文章有过时（`HappyPack`）和冗余（`terser-webpack-plugin` 默认会启用 多进程）的部分，而且也没有实践的部分，顿时就觉得有了自己的（这篇文章）存在有了意义；
- 二是发现优化完之后发现效果不好，在本地构建甚至反向优化，一度怀疑还有没有写的必要，好在在测试服务器上通过 `CI/CD` 构建速度中位数 `50s` 左右，还算可以，抱着有始有终的态度写完了

最后还是提一下，`worker` 启动和进程间通信的开销都不小，结合项目实际情况使用，不要为了用而用，项目质量比性能更重要。

## 参考链接

- [【Webpack】538- 打包速度提升指南][7]
- [webpack - Build Performance][9]
- [vue-loader 如何从 v14 迁移 至 v15][10]

[1]: {% post_url 2020-03-25-webpack4-build-size-optimization %}
[2]: https://www.npmjs.com/package/speed-measure-webpack-plugin
[3]: https://github.com/webpack-contrib/sass-loader/issues/761#issuecomment-554608680
[4]: https://www.npmjs.com/package/fork-ts-checker-webpack-plugin
[5]: https://www.npmjs.com/package/ts-loader#happypackmode
[6]: https://www.npmjs.com/package/cache-loader
[7]: https://cloud.tencent.com/developer/article/1603799
[8]: https://v4.webpack.js.org/guides/build-performance/#sass
[9]: https://v4.webpack.js.org/guides/build-performance/
[10]: https://vue-loader.vuejs.org/zh/migrating.html
[11]: https://www.npmjs.com/package/thread-loader
