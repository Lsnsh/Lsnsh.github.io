---
title: webpack3.x升级4.x时遇到的问题
title_en: webpack3-upgrade-webpack4-issue-logs
date: 2019-11-18 15:46:07
tags:
  - Log
  - Vue
  - Webpack
---

## 背景

在 `vue-cli3` 发布之前，使用 `vue-cli2` 构建的 `vue` 项目是基于 `webpack3.x` 的，伴随着项目的版本迭代，功能逐渐增多，文件也逐渐增多，打包时间从最初的 `4.5` 分钟，最久的时候 `17` 分钟。

因为使用 `ts` 的缘故，每次打包 `ts-loader` 需要进行类型检查，导致耗时剧增。权衡利弊下，将 `ts-loader` 的类型检查禁用后（`transpileOnly` 选项设置为 `true`），打包时间锐减到 `1` 分钟。

随着时间的推移，项目依赖逐渐落后，打包时间也到了 `2.5` 分钟，恰逢 `webpack4` 对于打包性能的提升，于是就有了这篇升级日志：

## 起步

整体思路：

1. 升级项目依赖
2. 跑通开发环境打包
3. 跑通生成环境打包

## 升级项目依赖

一开始的项目依赖升级的思路是，先升级 `webpack` 到最新的版本，然后逐步升级 `loader` 、`plugin` 、`babel` 等依赖。但是基于之前其他项目升级 `webpack4` 的经验，发现逐步升级会耗费一部分时间在**新旧依赖**的问题上，于是决定激进一点，使用 `ncu` 一次性将 `package.json` 中包含的项目依赖升级到最新版本，考虑到不确定性，可以选择性的修改 `package.json` 不升级某些依赖，比如主要版本号有变动的依赖（eg: 0.x.x => 1.x.x）。

```bash
# 安装 npm-check-updates
npm i -g npm-check-updates

# 检查项目依赖的版本与最新稳定版本，列出清单（beta版本可能不会列出来）
ncu

# 将 `package.json` 中包含的依赖的版本号，更新到最新
ncu -u

# 删除旧的依赖，安装新的依赖
rm -rf node_modules
rm package-lock.json
npm i
```

<!-- more -->

## 跑通开发环境打包

```log
npm run dev
```

**运行过程中遇到的问题：**

1.  ### TypeError: proxyMiddleware is not a function

    **报错日志：**

    ```log
    /Users/yourProjectPath/build/dev-server.js:47
      app.use(proxyMiddleware(options.filter || context, options));
            ^

    TypeError: proxyMiddleware is not a function
        at /Users/yourProjectPath/build/dev-server.js:47:11
        at Array.forEach (<anonymous>)
        at Object.<anonymous> (/Users/yourProjectPath/build/dev-server.js:42:25)
        at Module._compile (module.js:643:30)
        at Object.Module._extensions..js (module.js:654:10)
        at Module.load (module.js:556:32)
        at tryModuleLoad (module.js:499:12)
        at Function.Module._load (module.js:491:3)
        at Function.Module.runMain (module.js:684:10)
        at startup (bootstrap_node.js:187:16)
        at bootstrap_node.js:608:3
    ```

    **原因：**

    因为 [http-proxy-middleware][96] 升级到 `v1.x` 导致的：

    ```diff
    - "http-proxy-middleware": "~0.17.3",
    + "http-proxy-middleware": "~1.0.3",
    ```

    `v1.x` 起，需要通过主动导入 `createProxyMiddleware` 的方式使用中间件：

    ```js
    // javascript

    const express = require("express");
    const { createProxyMiddleware } = require("http-proxy-middleware");

    const app = express();

    app.use(
      "/api",
      createProxyMiddleware({
        target: "http://www.example.org",
        changeOrigin: true,
      })
    );
    app.listen(3000);

    // http://localhost:3000/api/foo/bar -> http://www.example.org/api/foo/bar
    ```

    **如何解决：**

    主动导入 `createProxyMiddleware`

    ```diff
    - const proxyMiddleware = require('http-proxy-middleware');
    + const { createProxyMiddleware } = require('http-proxy-middleware');
    ```

    将 `proxyMiddleware` 改为 `createProxyMiddleware`

    ```diff
    - app.use(proxyMiddleware(options.filter || context, options));
    + app.use(createProxyMiddleware(options.filter || context, options));
    ```

2.  ### vue-loader was used without the corresponding plugin. Make sure to include VueLoaderPlugin in your webpack config.

    [vue-loader 如何从 v14 迁移至 v15][1]

    **报错日志：**

    ```log
    ERROR in ./src/App.vue
    Module Error (from ./node_modules/vue-loader/lib/index.js):
    vue-loader was used without the corresponding plugin. Make sure to include VueLoaderPlugin in your webpack config.
    @ ./src/main.ts 2:0-28 45:23-26
    @ multi ./build/dev-client ./src/main.ts
    ```

    ```log
    ERROR in ./src/page/supplier/preferential/full-discount/view.vue?vue&type=script&lang=ts&
    Module not found: Error: Can't resolve '../../../../interface/coupon.coupon_code_view' in '/Users/yourProjectPath/src/page/supplier/preferential/full-discount'
    @ ./src/page/supplier/preferential/full-discount/view.vue?vue&type=script&lang=ts& 19:0-70
    @ ./src/page/supplier/preferential/full-discount/view.vue
    @ ./src/router/supplier/preferential.ts
    @ ./src/router/supplier/index.ts
    @ ./src/router/index.ts
    @ ./src/main.ts
    @ multi ./build/dev-client ./src/main.ts
    ```

    **原因：**

    `vue-loader` 升级到 `v15.x` 以后，需要搭配 `VueLoaderPlugin` 使用

    **如何解决：**

    ```js
    // webpack.base.confg.js
    const { VueLoaderPlugin } = require("vue-loader");

    module.exports = {
      // ...
      plugins: [new VueLoaderPlugin()],
    };
    ```

3.  ### Component template requires a root element, rather than just text.

    **报错日志：**

    ```log
    Module Error (from ./node_modules/vue-loader/lib/loaders/templateLoader.js):
    (Emitted value instead of an instance of Error)

    Errors compiling template:

    Component template requires a root element, rather than just text.

    1   |
        |
    2   |  #app
        |  ^^^^
    3   |    router-view
        |  ^^^^^^^^^^^^^

     @ ./src/App.vue?vue&type=template&id=7ba5bd90&lang=pug& 1:0-238 1:0-238
     @ ./src/App.vue
     @ ./src/main.ts
     @ multi ./build/dev-client ./src/main.ts
    ```

    ```log
    Module Error (from ./node_modules/vue-loader/lib/loaders/templateLoader.js):
    (Emitted value instead of an instance of Error)

    Errors compiling template:

    text "div
    el-card" outside root element will be ignored.
    ```

    **原因：**

    引用自 `vue-loader` 文档：

    > 注意有些模板的 loader 会导出一个编译好的模板函数而不是普通的 HTML，诸如 pug-loader。为了向 Vue 的模板编译器传递正确的内容，你必须换用一个输出普通 HTML 的 loader。例如，为了支持 `<template lang="pug">`，你可以使用 pug-plain-loader：

    **如何解决：**

    项目中使用到了 `pug` 语法，由于 `vue-loader` 的升级，现在需要引入 `pug-plain-loader`

    ```bash
    npm i -D pug-plain-loader
    ```

    ```js
    // webpack.base.confg.js

    module.exports = {
      // ...
      module: {
        rules: [
         // ...
          {
            test: /\.pug$/,
            loader: "pug-plain-loader"
          }
        ];
      }
    };
    ```

4.  ### Cannot find module '@babel/core'

    **报错日志：**

    ```log
    ERROR in ./src/store/mutation/util.js
    Module build failed (from ./node_modules/babel-loader/lib/index.js):
    Error: Cannot find module '@babel/core'
    babel-loader@8 requires Babel 7.x (the package '@babel/core'). If you'd like to use Babel 6.x ('babel-core'), you should install 'babel-loader@7'.
        at Function.Module._resolveFilename (module.js:538:15)
        at Function.Module._load (module.js:468:25)
        at Module.require (module.js:587:17)
        at require (internal/module.js:11:18)
        at Object.<anonymous> (/Users/yourProjectPath/node_modules/babel-loader/lib/index.js:10:11)
        at Module._compile (module.js:643:30)
        at Object.Module._extensions..js (module.js:654:10)
        at Module.load (module.js:556:32)
        at tryModuleLoad (module.js:499:12)
        at Function.Module._load (module.js:491:3)
        at Module.require (module.js:587:17)
        at require (internal/module.js:11:18)
        at loadLoader (/Users/yourProjectPath/node_modules/loader-runner/lib/loadLoader.js:18:17)
        at iteratePitchingLoaders (/Users/yourProjectPath/node_modules/loader-runner/lib/LoaderRunner.js:169:2)
        at runLoaders (/Users/yourProjectPath/node_modules/loader-runner/lib/LoaderRunner.js:365:2)
        at NormalModule.doBuild (/Users/yourProjectPath/node_modules/webpack/lib/NormalModule.js:295:3)
    @ ./src/store/mutation/nav.ts 3:0-48 54:41-63 59:41-63 63:41-63 70:37-59
    @ ./src/store/mutation/index.ts
    @ ./src/store/index.ts
    @ ./src/main.ts
    @ multi ./build/dev-client ./src/main.ts
    ```

    **原因：**

    根据错误提示可以知道，`babel-loader@8` 依赖 `Babel 7.x` (也就是 `@babel/core`)。

    **如何解决：**

    卸载 `babel-core`

    ```bash
    npm un babel-core
    ```

    安装 `@babel/core`

    ```bash
    npm i -D @babel/core
    ```

5.  ### Cannot read property 'bindings' of null

    **报错日志：**

    ```log
    Module build failed (from ./node_modules/babel-loader/lib/index.js):

    TypeError: /Users/yourProjectPath/src/page/image-tag/bus.js: Cannot read property 'bindings' of null
    at Scope.moveBindingTo (/Users/yourProjectPath/node_modules/@babel/traverse/lib/scope/index.js:926:13)
    at BlockScoping.updateScopeInfo (/Users/yourProjectPath/node_modules/babel-plugin-transform-es2015-block-scoping/lib/index.js:364:17)
    at BlockScoping.run (/Users/yourProjectPath/node_modules/babel-plugin-transform-es2015-block-scoping/lib/index.js:330:12)
    at PluginPass.BlockStatementSwitchStatementProgram (/Users/yourProjectPath/node_modules/babel-plugin-transform-es2015-block-scoping/lib/index.js:70:24)
    at newFn (/Users/yourProjectPath/node_modules/@babel/traverse/lib/visitors.js:179:21)
    at NodePath.\_call (/Users/yourProjectPath/node_modules/@babel/traverse/lib/path/context.js:55:20)
    at NodePath.call (/Users/yourProjectPath/node_modules/@babel/traverse/lib/path/context.js:42:17)
    at NodePath.visit (/Users/yourProjectPath/node_modules/@babel/traverse/lib/path/context.js:90:31)
    at TraversalContext.visitQueue (/Users/yourProjectPath/node_modules/@babel/traverse/lib/context.js:112:16)
    at TraversalContext.visitSingle (/Users/yourProjectPath/node_modules/@babel/traverse/lib/context.js:84:19)
    at TraversalContext.visit (/Users/yourProjectPath/node_modules/@babel/traverse/lib/context.js:140:19)
    at Function.traverse.node (/Users/yourProjectPath/node_modules/@babel/traverse/lib/index.js:84:17)
    at traverse (/Users/yourProjectPath/node_modules/@babel/traverse/lib/index.js:66:12)
    at transformFile (/Users/yourProjectPath/node_modules/@babel/core/lib/transformation/index.js:107:29)
    at transformFile.next (<anonymous>)
    at run (/Users/yourProjectPath/node_modules/@babel/core/lib/transformation/index.js:35:12)
    ```

    **原因：**

    由于已经将 `Babel` 升级到 `7.x`，但是 `.babelrc` 仍在引用适用于 `Babel 6.x` 的 `babel-preset-env`

    **如何解决：**

    卸载 `babel-preset-env`

    ```bash
    npm un babel-preset-env
    ```

    安装 `@babel/preset-env`

    ```bash
    npm i -D @babel/preset-env
    ```

    从 `.babelrc` 中移除 `babel-preset-env`，改用 `@babel/preset-env`

    ```diff
    - { "presets": ["env"] }
    + { "presets": ["@babel/preset-env"] }
    ```

6.  ### this.setDynamic is not a function

    **报错日志：**

    ```log
    ERROR in ./src/lib/image-drop.js
    Module build failed (from ./node_modules/babel-loader/lib/index.js):
    TypeError: /Users/yourProjectPath/src/lib/image-drop.js: this.setDynamic is not a function
        at PluginPass.pre (/Users/yourProjectPath/node_modules/babel-plugin-transform-runtime/lib/index.js:31:12)
        at transformFile (/Users/yourProjectPath/node_modules/@babel/core/lib/transformation/index.js:96:27)
        at transformFile.next (<anonymous>)
        at run (/Users/yourProjectPath/node_modules/@babel/core/lib/transformation/index.js:35:12)
        at run.next (<anonymous>)
        at Function.transform (/Users/yourProjectPath/node_modules/@babel/core/lib/transform.js:27:41)
        at transform.next (<anonymous>)
        at step (/Users/yourProjectPath/node_modules/gensync/index.js:254:32)
        at /Users/yourProjectPath/node_modules/gensync/index.js:266:13
        at async.call.result.err.err (/Users/yourProjectPath/node_modules/gensync/index.js:216:11)
        at /Users/yourProjectPath/node_modules/gensync/index.js:184:28
        at /Users/yourProjectPath/node_modules/@babel/core/lib/gensync-utils/async.js:72:7
        at /Users/yourProjectPath/node_modules/gensync/index.js:108:33
        at step (/Users/yourProjectPath/node_modules/gensync/index.js:280:14)
        at /Users/yourProjectPath/node_modules/gensync/index.js:266:13
        at async.call.result.err.err (/Users/yourProjectPath/node_modules/gensync/index.js:216:11)
    ```

    **原因：**

    由于已经将 `Babel` 升级到 `7.x`，但是 `.babelrc` 仍在引用适用于 `Babel 6.x` 的 `babel-plugin-transform-runtime`

    **如何解决：**

    卸载 `babel-plugin-transform-runtime`

    ```bash
    npm un babel-plugin-transform-runtime
    ```

    安装 `@babel/plugin-transform-runtime`

    ```bash
    npm i -D @babel/plugin-transform-runtime
    ```

    从 `.babelrc` 中移除 `babel-preset-env`，改用 `@babel/plugin-transform-runtime`

    ```diff
    - { "plugins": ["babel-plugin-transform-runtime"] }
    + { "plugins": ["@babel/plugin-transform-runtime"] }
    ```

7.  ### ValidationError: Invalid options object. CSS Loader has been initialized using an options object that does not match the API schema.

    **报错日志：**

    ```log
    ERROR in ./node_modules/element-ui/lib/theme-chalk/index.css (./node_modules/css-loader/dist/cjs.js??ref--11-1!./node_modules/element-ui/lib/theme-chalk/index.css)
    Module build failed (from ./node_modules/css-loader/dist/cjs.js):
    ValidationError: Invalid options object. CSS Loader has been initialized using an options object that does not match the API schema.
    - options has an unknown property 'minimize'. These properties are valid:
    object { url?, import?, modules?, sourceMap?, importLoaders?, localsConvention?, onlyLocals? }
        at validate (/Users/yourProjectPath/node_modules/css-loader/node_modules/schema-utils/dist/validate.js:50:11)
        at Object.loader (/Users/yourProjectPath/node_modules/css-loader/dist/index.js:34:28)
    @ ./node_modules/element-ui/lib/theme-chalk/index.css 4:14-81 14:3-18:5 15:22-89
    @ ./src/main.ts
    @ multi ./build/dev-client ./src/main.ts
    ```

    **原因：**

    `minimize` 配置项被移除导致

    **如何解决：**

    删除 `css-loader` 的 `minimize` 选项

8.  ### The 'mode' option has not been set

    **报错日志：**

    ```log
    WARNING in configuration
    The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
    You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/
    ```

    **原因：**

    `webpack4` 开始，提供了 `mode` 选项用来启用 `webpack` 提供的内置选项，没有设置时，默认值为 `production`

    **如何解决：**

    开发环境设置：`development`，生成环境设置：`production`

至此已经可以跑通开发环境打包了

## 跑通生成环境打包

**运行过程中遇到的问题：**

1.  ### webpack.optimize.CommonsChunkPlugin has been removed, please use config.optimization.splitChunks instead.

    **报错日志：**

    ```log
    /Users/yourProjectPath/node_modules/webpack/lib/webpack.js:189
    		throw new RemovedPluginError(errorMessage);
    		^

    Error: webpack.optimize.CommonsChunkPlugin has been removed, please use config.optimization.splitChunks instead.
        at Object.get [as CommonsChunkPlugin] (/Users/yourProjectPath/node_modules/webpack/lib/webpack.js:189:10)
        at Object.<anonymous> (/Users/yourProjectPath/build/webpack.prod.conf.js:85:26)
        at Module._compile (module.js:643:30)
        at Object.Module._extensions..js (module.js:654:10)
        at Module.load (module.js:556:32)
        at tryModuleLoad (module.js:499:12)
        at Function.Module._load (module.js:491:3)
        at Module.require (module.js:587:17)
        at require (internal/module.js:11:18)
        at Object.<anonymous> (/Users/yourProjectPath/build/build.js:12:23)
        at Module._compile (module.js:643:30)
        at Object.Module._extensions..js (module.js:654:10)
        at Module.load (module.js:556:32)
        at tryModuleLoad (module.js:499:12)
        at Function.Module._load (module.js:491:3)
        at Function.Module.runMain (module.js:684:10)
    ```

    **原因：**

    > https://webpack.js.org/migrate/4/#commonschunkplugin

    根据错误提示信息可以知道，`webpack4` 开始 `CommonsChunkPlugin` 被删除

    **如何解决：**

    将 `CommonsChunkPlugin` 相关的配置删除，改用 `optimization.splitChunks` 配置项

    > The default configuration was chosen to fit web performance best practices, but the optimal strategy for your project might differ. If you're changing the configuration, you should measure the impact of your changes to ensure there's a real benefit.
    > https://webpack.js.org/plugins/split-chunks-plugin/

    根据插件文档的描述，这里我们没有额外增加配置，使用的是 `optimization.splitChunks` 配置项的默认值。

    ```js
    // webpack.config.js
    module.exports = {
      // ...
      optimization: {
        splitChunks: {
          chunks: "async",
          minSize: 30000,
          maxSize: 0,
          minChunks: 1,
          maxAsyncRequests: 5,
          maxInitialRequests: 3,
          automaticNameDelimiter: "~",
          automaticNameMaxLength: 30,
          name: true,
          cacheGroups: {
            vendors: {
              test: /[\\/]node_modules[\\/]/,
              priority: -10,
            },
            default: {
              minChunks: 2,
              priority: -20,
              reuseExistingChunk: true,
            },
          },
        },
      },
    };
    ```

2.  ### Error: Chunk.entrypoints: Use Chunks.groupsIterable and filter by instanceof Entrypoint instead

    **报错日志：**

    ```log
    /Users/yourProjectPath/node_modules/webpack/lib/Chunk.js:866
    	throw new Error(
    	^

    Error: Chunk.entrypoints: Use Chunks.groupsIterable and filter by instanceof Entrypoint instead
        at Chunk.get (/Users/yourProjectPath/node_modules/webpack/lib/Chunk.js:866:9)
        at /Users/yourProjectPath/node_modules/extract-text-webpack-plugin/dist/index.js:176:48
        at Array.forEach (<anonymous>)
        at /Users/yourProjectPath/node_modules/extract-text-webpack-plugin/dist/index.js:171:18
        at AsyncSeriesHook.eval [as callAsync] (eval at create (/Users/yourProjectPath/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:7:1)
        at AsyncSeriesHook.lazyCompileHook (/Users/yourProjectPath/node_modules/tapable/lib/Hook.js:154:20)
        at Compilation.seal (/Users/yourProjectPath/node_modules/webpack/lib/Compilation.js:1342:27)
        at compilation.finish.err (/Users/yourProjectPath/node_modules/webpack/lib/Compiler.js:675:18)
        at hooks.finishModules.callAsync.err (/Users/yourProjectPath/node_modules/webpack/lib/Compilation.js:1261:4)
        at AsyncSeriesHook.eval [as callAsync] (eval at create (/Users/yourProjectPath/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:24:1)
        at AsyncSeriesHook.lazyCompileHook (/Users/yourProjectPath/node_modules/tapable/lib/Hook.js:154:20)
        at Compilation.finish (/Users/yourProjectPath/node_modules/webpack/lib/Compilation.js:1253:28)
        at hooks.make.callAsync.err (/Users/yourProjectPath/node_modules/webpack/lib/Compiler.js:672:17)
        at _done (eval at create (/Users/yourProjectPath/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:9:1)
        at _err1 (eval at create (/Users/yourProjectPath/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:32:22)
        at _addModuleChain (/Users/yourProjectPath/node_modules/webpack/lib/Compilation.js:1185:12)
    ```

    **原因：**

    [extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) 在 `webpack4` 的时候被废弃，改用[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)

    **如何解决：**

    卸载 `extract-text-webpack-plugin`

    ```bash
    npm un extract-text-webpack-plugin
    ```

    安装 `mini-css-extract-plugin`

    ```bash
    npm i -D mini-css-extract-plugin
    ```

    将 `extract-text-webpack-plugin` 相关的配置删除，改用 `mini-css-extract-plugin`

    ```diff
    // utils.js
    -const ExtractTextPlugin = require('extract-text-webpack-plugin');
    +const MiniCssExtractPlugin = require('mini-css-extract-plugin');

    if (options.extract) {
    -  return ExtractTextPlugin.extract({
    -    use: loaders,
    -    fallback: 'vue-style-loader'
    -  });
    +  return [
    +    {
    +      loader: MiniCssExtractPlugin.loader,
    +      options: {
    +        hmr: process.env.NODE_ENV === 'development'
    +      }
    +    }
    +  ].concat(loaders);
    } else {
      return ['vue-style-loader'].concat(loaders);
    }
    ```

    ```diff
    // webpack.prod.conf.js
    -const ExtractTextPlugin = require('extract-text-webpack-plugin');
    +const MiniCssExtractPlugin = require('mini-css-extract-plugin');

    module.exports = {
      // ...
      plugins: [
    -   new ExtractTextPlugin({
    +   new MiniCssExtractPlugin({
          filename: utils.assetsPath('css/[name].[contenthash].css'),
        }),
      ]
    }
    ```

    如果不是使用 `vue-cli 2.x` 搭建的项目，可能需要这样修改：

    详情见：https://webpack.js.org/plugins/mini-css-extract-plugin/

    ```js
    // webpack.prod.conf.js
    // ***
    const MiniCssExtractPlugin = require("mini-css-extract-plugin");

    module.exports = {
      module: {
        rules: [
          // ***
          {
            test: /\.css$/,
            use: [
              {
                loader: MiniCssExtractPlugin.loader,
                options: {
                  hmr: process.env.NODE_ENV === "development",
                },
              },
              "css-loader",
            ],
          },
        ],
      },
      plugins: [
        // ***
        new MiniCssExtractPlugin({
          filename: "[name].[contenthash].css",
        }),
      ],
    };
    ```

3.  ### DefaultsError: `warnings` is not a supported option

    **报错日志：**

    ```log
    ERROR in static/js/170.b2935281265dd9ec23b7.js from UglifyJs

    DefaultsError: `warnings` is not a supported option
    at DefaultsError.get (eval at <anonymous> (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/node_modules/uglify-js/tools/node.js:18:1), <anonymous>:71:23)
    at Function.buildError (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/index.js:105:54)
    at results.forEach (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/index.js:258:52)
    at Array.forEach (<anonymous>)
    at taskRunner.run (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/index.js:233:17)
    at step (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:85:9)
    at done (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:96:30)
    at boundWorkers (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:101:13)
    at TaskRunner.boundWorkers (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:70:11)
    at enqueue (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:91:14)
    at tasks.forEach (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:111:9)
    at Array.forEach (<anonymous>)
    at TaskRunner.run (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/TaskRunner.js:89:11)
    at UglifyJsPlugin.optimizeFn (/Users/yourProjectPath/node_modules/uglifyjs-webpack-plugin/dist/index.js:227:18)
    at AsyncSeriesHook.eval [as callAsync] (eval at create (/Users/yourProjectPath/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:31:1)
    at AsyncSeriesHook.lazyCompileHook (/Users/yourProjectPath/node_modules/tapable/lib/Hook.js:154:20)
    ```

    **原因：**

    由于 `UglifyJs` 升级导致的，`uglifyjs-webpack-plugin` 依赖于 `UglifyJs`

    详情见：https://github.com/mishoo/UglifyJS2/issues/3394

    **如何解决：**

    将 `uglifyOptions.compress` 选项删除

    ```diff
    // webpack.prod.conf.js
    new UglifyJsPlugin({
      uglifyOptions: {
    -   compress: {
          warnings: false,
          drop_console: true,
          drop_debugger: true
    -   }
      }
    }),
    ```

4.  ### Unexpected token: keyword «const»

    **报错日志：**

    ```log
    ERROR in static/js/app.2c7ff4a31971e22c9397.js from UglifyJs
    Unexpected token: keyword «const» [static/js/app.2c7ff4a31971e22c9397.js:3865,0]
    ```

    **原因：**

    [`uglifyjs-webpack-plugin` 不支持 `es6` 语法](https://github.com/webpack-contrib/uglifyjs-webpack-plugin/issues/389)

    **如何解决：**

    卸载 `uglifyjs-webpack-plugin`

    ```bash
    npm un uglifyjs-webpack-plugin
    ```

    将 `uglifyjs-webpack-plugin` 相关的配置删除，不用额外增加配置，因为 `production` 模式下，`webpack` 默认使用 `terser-webpack-plugin` 来最小化 `JavaScript` 文件

## 总结

至此经历了一系列的各种问题之后，开发和生产环境打包都已经可以跑通了。这篇文章记录了的两个 `webpack 3.x` 项目升级到 `webpack 4.x` 时遇到的问题、原因和解决方法，希望对你们有帮助。

升级后生产环境打包速度有所提升，但是文件大小并不理想，下一篇我会讲如何对 `webpack` 的打包进行优化。

## 参考链接

- [vue-loader 如何从 v14 迁移 至 v15][1]
- [webpack loader 执行顺序][97]

[1]: https://vue-loader.vuejs.org/zh/migrating.html
[96]: https://github.com/chimurai/http-proxy-middleware#tldr
[97]: https://stackoverflow.com/questions/32234329/what-is-the-loader-order-for-webpack
