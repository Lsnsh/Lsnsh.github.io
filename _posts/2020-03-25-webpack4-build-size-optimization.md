---
title: webpack3.x升级4.x之后打包大小优化
title_en: webpack4-build-size-optimization
date: 2020-03-25 23:23:00
tags:
  - Vue
  - Webpack
---

## 前言

[上一篇][1]我们讲到，webpack3.x 升级 4.x 时遇到的问题、原因和解决方案，今天讲一下 `webpack 4.x` 的打包优化。

## 打包分析

在优化之前我们需要清楚项目的打包情况，`npm run build` 后 `webpack` 会将打包信息打印到终端，像这样：

![](/assets/images/webpack3升级4之后打包大小优化/1.jpg)

能够看到有好几个文件比较大，但是并不清楚具体包含哪些模块，哪些模块适合单独提取，哪些模块不适合提取，这个时候就需要用到打包分析的工具。

<!-- more -->

以下摘自 [webpack 文档][9]：

> **Bundle Anlysis - 打包分析**
>
> 一旦开始分割代码，分析输出以检查模块在哪里结束将很有用。[official analyze tool][3] 是一个很好的起点。 还有一些其他社区支持的选项：
>
> - [webpack-chart][4]: 用于 webpack 统计信息的交互式饼图
> - [webpack-visualizer][5]: 可视化和分析您的包，以查看哪些模块正在占用空间，哪些可能是重复的。
> - [webpack-bundle-analyzer][6]: 一个插件和 CLI 实用程序，将包内容表示为方便的交互式可缩放树形图。
> - [webpack bundle optimize helper][7]: 此工具将分析您的捆绑包，并为您提供切实可行的建议，以进行改进以减小捆绑包的大小。
> - [bundle-stats][8]: 生成捆绑报告（捆绑大小，资产，模块），并比较不同版本之间的结果。

这里我们使用的是 `webpack-bundle-analyzer`，交互式可缩放树形图，便于我们对打包情况有一个比较直观的了解。

安装插件

```bash
npm i -D webpack-bundle-analyzer
```

引入插件

```js
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer")
  .BundleAnalyzerPlugin;

module.exports = {
  plugins: [new BundleAnalyzerPlugin()],
};
```

然后我们再试着构建以下，看会怎么样

```bash
npm run build
```

插件会自动在浏览器中打开链接：http://127.0.0.1:8888/ ，以交互式可缩放树形图展示了当前项目的打包情况，可以清楚的看到各个文件包含的模块，针对性的处理：

![](/assets/images/webpack3升级4之后打包大小优化/2.jpg)

## 代码拆分

分析上面 👆 的树形图，归纳了以下几个可以优化的点：

- 将两个入口公共的依赖，提取出来，如：Element-UI、Vue 全家桶、Lodash 等
- 将仅某一个组件使用的依赖，提取出来，如：Swiper
- 其他的依赖，作为单独的一部分

将 `optimization.splitChunks` 属性的默认值删除，然后在 `cacheGroups` 中配置如下：

**webpack.prod.js**

```diff
// ..
const webpackConfig = merge(baseWebpackConfig, {
  // ..
  optimization: {
    splitChunks: {
-     chunks: "async",
-     minSize: 30000,
-     maxSize: 0,
-     minChunks: 1,
-     maxAsyncRequests: 5,
-     maxInitialRequests: 3,
-     automaticNameDelimiter: "~",
-     automaticNameMaxLength: 30,
-     name: true,
      cacheGroups: {
+       common: {
+         test: /[\\/]node_modules[\\/]/,
+         name: 'vendors-common',
+         minSize: 0,
+         minChunks: 2, // 最少被2个chunk引用
+         chunks: 'initial',
+         priority: 1 // 优先级，默认是0，可以为负数
+       },
+       other: {
+         test: /[\\/]node_modules[\\/]/,
+         name: 'vendors-other',
+         chunks: 'initial',
+         priority: -10
+       },
-       defaultVendors: {
-         test: /[\\/]node_modules[\\/]/,
-         priority: -10
-       },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
  // ..
});

// ..
module.exports = webpackConfig;
```

我们再次构建，看看

```bash
npm run build
```

![](/assets/images/webpack3升级4之后打包大小优化/4.jpg)
![](/assets/images/webpack3升级4之后打包大小优化/3.jpg)

可以看到拆分出了几个包含 `node_modules` 的大文件：

- vendors-common.\*\*\*.js（两个入口公共的依赖，被引用两次以上）
- vendors-other.\*\*\*.js（其他依赖，单个入口中使用，作为全局使用的）
- 5.\*\*\*.js（`swiper`，单个组件中引入的依赖，`webpack` 会自动提取）
- 其他项目中使用到并自动拆分出的依赖，文件较小在上图中没体现出来

比较一下代码拆分前后的两个入口加载文件大小的不同：

- 拆分前：_app（2.5MB） qi（1.04MB）_
- 拆分后：_app（1.88MB）qi（1.04MB）_

入口 2 没有变化，入口 1 的加载文件大小减少的 `0.62MB`，缩小将近 `25%`，当然这是没有进行 `gzip` 压缩的情况，在 `nginx` 服务器上启用 `gzip` 压缩整体大小会更小，这里就不展开了

如果你想单独将某些依赖，单独提取出来，可以看看官方的这个[例子][13]，如果你的项目依赖较少，你完全可以按你喜欢的组合，手动拆分你的项目依赖到多个文件中。

## 缓存（快取）

当访问网页时，请求的文件会存储在浏览器的缓存中，当下次再访问时，如果满足[HTTP 缓存机制][10]，浏览器可以从缓存中读取文件，而不必从服务器读取，甚至不用发起请求，可以显著提升加载速度并减少流量消耗。

但是如果项目重新构建，而用户浏览器中的缓存还未过期，想要快速更新并继续保留部分缓存，该怎么做呢？

![](/assets/images/webpack3升级4之后打包大小优化/5.jpg)

> 更多细节请看，[HTTP 缓存-废弃和更新缓存的响应][11]

对此 `webpack` 打包需要优化一下几点：

- 控制输出文件的名称 - contentHash
- 提取运行时代码 - runtimeChunk
- 指定模块标识符 - moduleId

**webpack.prod.js**

```diff
// ..
const webpackConfig = merge(baseWebpackConfig, {
  // ..
  output: {
    // ..
-   filename: utils.assetsPath('js/[name].[chunkhash].js')
+   // 根据文件内容生成的 hash，文件内容改变时 hash 也随之改变
+   filename: utils.assetsPath('js/[name].[contenthash].js')
  },
  optimization: {
+   // 运行时块包含模块间的引用关系，将它提取出来作为单独的文件，引用关系发生改变时，不会导致无关的文件内容发生改变
+   runtimeChunk: 'single',
+   // 因为使用 hashed，当本地新增或减少依赖时，不会影响其他模块的 ID（默认情况下 ID 的值是根据解析顺序递增的）
+   moduleIds: 'hashed'
    // ...
  }
});

// ..
module.exports = webpackConfig;
```

> 更多细节请看，[webpack 文档-缓存（快取）][12]

## 总结

代码拆分的过程中，有尝试过基于现有配置的基础上，单独拆分出指定依赖（如 `Element-UI`，因为不常改动且文件较大），但是会破坏现有的拆分，多次尝试之后无果，最接近的时候是这样的，入口 1 会将其包含在文件中，但是入口 2 不会，当前项目有两个入口：

![](/assets/images/webpack3升级4之后打包大小优化/6.jpg)

采用如下配置：

```diff
// ..
const webpackConfig = merge(baseWebpackConfig, {
  // ..
  optimization: {
    splitChunks: {
      cacheGroups: {
        common: {
-         test: /[\\/]node_modules[\\/]/,
+         test: /[\\/]node_modules[\\/]((?!(element-ui)).)+[\\/]/,
          name: 'vendors-common',
          minSize: 0,
          minChunks: 2, // 最少被2个chunk引用
          chunks: 'initial',
          priority: 1 // 优先级，默认是0，可以为负数
        },
        other: {
-         test: /[\\/]node_modules[\\/]/,
+         test: /[\\/]node_modules[\\/]((?!(element-ui)).)+[\\/]/,
          name: 'vendors-other',
          chunks: 'initial',
          priority: -10
        },
+       'element-ui': {
+         test: /[\\/]node_modules[\\/](element-ui)[\\/]/,
+         name: 'vendor-element-ui',
+         chunks: 'initial',
+         priority: -15
+       },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
  // ..
});

// ..
module.exports = webpackConfig;
```

后续有处理方案后会更新，到这里打包大小优化就告一段了，下一篇会讲讲如何对打包速度进行优化。

[1]: {% post_url 2019-11-18-webpack3-upgrade-webpack4-issue-logs %}
[2]: https://router.vuejs.org/zh/guide/advanced/lazy-loading.html
[3]: https://github.com/webpack/analyse
[4]: https://alexkuz.github.io/webpack-chart/
[5]: https://chrisbateman.github.io/webpack-visualizer/
[6]: https://github.com/webpack-contrib/webpack-bundle-analyzer
[7]: https://webpack.jakoblind.no/optimize
[8]: https://github.com/bundle-stats/bundle-stats
[9]: https://webpack.js.org/guides/code-splitting/#bundle-analysis
[10]: https://yq.aliyun.com/articles/606283
[11]: https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#%E5%BA%9F%E5%BC%83%E5%92%8C%E6%9B%B4%E6%96%B0%E7%BC%93%E5%AD%98%E7%9A%84%E5%93%8D%E5%BA%94
[12]: https://v4.webpack.js.org/guides/caching/
[13]: https://v4.webpack.js.org/plugins/split-chunks-plugin/#split-chunks-example-3
