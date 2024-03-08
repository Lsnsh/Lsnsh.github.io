---
title: 使用 GoGoCode 将 Vue2 项目升级到 Vue3
title_en: use-gogocode-upgrade-vue2-project-to-vue3
date: 2021-10-28 14:53:39
tags:
  - Vue
  - GoGoCode
  - Log
  - Webpack
---

## 背景

在最近一期的「前端早早聊-大会直播」上，来自阿里妈妈的叶兮做了一期关于 `《如何把 Vue2 一键升级 Vue3 - GoGoCode》` 的分享，然后就接触到了 [GoGoCode](https://gogocode.io/zh)，看直播的时候就感觉很酷，于是迫不及待的就开始使用了

尝试转换的项目，是早期基于 `Vue-CLI 2.x` 搭建的，目录结构如下：

```bash
.
├── build
├── config
├── logs
├── node_modules
├── src
├── CHANGELOG.md
├── DEPLOY.md
├── README.md
├── package-lock.json
├── package.json
├── prettier.config.js
└── tsconfig.json
```

<!-- more -->

## 使用 GoGoCode 升级

![](/assets/images/使用GoGoCode将Vue2项目升级到Vue3/g-1.jpeg)

> [Vue2 到 Vue3 升级指南 - GoGoCode 官方文档](https://gogocode.io/zh/docs/vue/vue2-to-vue3)

### 1. 安装工具

```bash
npm install gogocode-cli -g
```

### 2. 迁移源文件

> 注意：-s 后面是原文件的目录/文件名，-o 后面是输出的目录/文件名，如果两者相同，转换插件会覆盖你的代码，在此操作前请做好备份。—— [迁移源文件 - GoGoCode 官方文档](https://gogocode.io/zh/docs/vue/vue2-to-vue3#%E8%BF%81%E7%A7%BB%E6%BA%90%E6%96%87%E4%BB%B6)

**tips**: 迁移转换前，修改原目录名称（备份原目录，转换迁移后，基于 `Git Diff` 方便查看文件变更）

1. 建议将原文件目录（比如 `src` 目录），修改为带 `-old` 后缀（`src-old`）；
2. 在 `.gitignore` 中忽略 `src-old` 目录；
3. 然后开始迁移源文件

```bash
gogocode -s ./src-old -t gogocode-plugin-vue -o ./src
```

### 3. 依赖升级

```bash
gogocode -s package.json -t gogocode-plugin-vue -o package.json
npm install
```

以上三步顺利做完之后， `GoGoCode` 帮我们把 `Vue2` 的代码转换成 `Vue3` 的代码了，并且升级 `Vue 全家桶` 相关的依赖。

此时你可以尝试运行项目，可能会报一大堆错误，别着急接着往下看。

### 4. 第三方组件库升级

- [Element Plus](https://github.com/element-plus/element-plus)
- [Ant-Design-Vue](https://github.com/vueComponent/ant-design-vue/)
- [Vant](https://github.com/youzan/vant)
- ...

如果你有使用第三方 `UI` 组件库，需要手动升级到对应组件库支持 `Vue3` 的版本。

### 5. 其他依赖和拓展升级

- [Devtools 扩展](https://v3.cn.vuejs.org/guide/migration/introduction.html#devtools-%E6%89%A9%E5%B1%95)
- [Volar - VSCode 拓展](<(https://v3.cn.vuejs.org/guide/migration/introduction.html#ide-%E6%94%AF%E6%8C%81)>)

  > 推荐使用 VSCode 和 `Vue` 官方拓展 [`Volar`](<(https://github.com/johnsoncodehk/volar)>)，它为 `Vue 3` 提供了全面的 `IDE` 支持

  你需要在 `VSCode` 中禁用（工作区） `Vetur`，避免两者冲突

- [@vue/babel-plugin-jsx](https://github.com/vuejs/jsx-next)
- [eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue)
- [@vue/test-utils](https://github.com/vuejs/vue-test-utils-next)
- [其他项目 - Vue 官方文档](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E5%85%B6%E4%BB%96%E9%A1%B9%E7%9B%AE)

## 检查破坏性变化

![](/assets/images/使用GoGoCode将Vue2项目升级到Vue3/g-3.jpeg)

`Vue` 全家桶和第三方 `UI` 组件库都有不同程度的破坏性变化：

1. [Vue2 到 Vue3 升级指南 - GoGoCode 官方文档](https://gogocode.io/zh/docs/vue/vue2-to-vue3)
2. [从 Vue2 迁移 - Vue 官方文档](https://v3.cn.vuejs.org/guide/migration/introduction.html)
3. [从 3.x 迁移到 4.0 - Vuex 官方文档](https://next.vuex.vuejs.org/zh/guide/migrating-to-4-0-from-3-x.html)
4. [从 Vue2 迁移 - Vue Router 官方文档](https://next.router.vuejs.org/zh/guide/migration/index.html)
5. [Breaking Change List - Element Plus](https://github.com/element-plus/element-plus/issues/162)
6. ...

基于 `GoGoCode` 使用 `gogocode-plugin-vue` 插件，已经帮我们自动转换了绝大部分 `Vue` 代码，可能会有少数情况的 `Vue` 代码转换没有覆盖到。

以及第三方 `UI` 组件库的破坏性变化（比如组件名称变更、属性变更等），就需要我们查阅对应的 `文档` 和 `更新日志`，手动同步更改，重新运行项目进行测试，根据出现的错误信息再去搜索、查阅，循环往复，直到没有报错，一切正常。

或者你有兴趣和时间的话，可以开发一个 [GoGoCode 插件](https://gogocode.io/zh/docs/specification/plugin) 插件供后人使用。

**tips:** 设置终端同时显示更大的行数，方便在控制台中查看所有报错信息

1. `iTerm2` - `Proferences` - `Profiles`（建议新建并切换到自定义的配置） - `Terminal` - `Scrollback Buffer` - 勾选 `Unlimited scrollback`
2. `VSCode` - 设置 `terminal.integrated.scrollback` - 等于一个很大的数字（比如 `9999999`）

## 遇到的问题

![](/assets/images/使用GoGoCode将Vue2项目升级到Vue3/g-2.jpg)

1. `Vue` 单文件组件中 `template pug` 语法，根节点必须顶格写
2. [@vue/compiler-sfc] ::v-deep usage as a combinator has been deprecated. Use :deep(\<inner-selector\>) instead.

   ```diff
   -.el-table /deep/ .warning-row {
   +.el-table:deep(.warning-row) {
   ```

3. VueCompilerError: <KeepAlive> expects exactly one child component.
   `Vue2` 只是在运行时检测是否只有子节点，`Vue3` 会在编译时检测，使用 `v-if` 和 `v-else-if` 确保只有一个子节点

   参考：https://forum.vuejs.org/t/vue-3-transitions-and-multiple-child-elements/105110

4. VueCompilerError: v-model value must be a valid JavaScript member expression

   `v-model` 的值不能包含中文，不能进行防御性编程判断（eg: v-model='oItem.stock_type && oItem.stock_type.key'）

   ```diff
   - v-model='oItem.stock_type && oItem.stock_type.key'
   + v-if='oItem.stock_type && oItem.stock_type.key'
   + v-model='oItem.stock_type.key'
   ```

5. Uncaught Error: Catch all routes ("\*") must now be defined using a param with a custom regexp.

   参考：[删除了 \*（星标或通配符）路由 - Vue Router](https://next.router.vuejs.org/zh/guide/migration/index.html#%E5%88%A0%E9%99%A4%E4%BA%86-%EF%BC%88%E6%98%9F%E6%A0%87%E6%88%96%E9%80%9A%E9%85%8D%E7%AC%A6%EF%BC%89%E8%B7%AF%E7%94%B1)

6. Uncaught TypeError: Cannot read properties of undefined (reading 'config')
   1. 使用 `app.config.globalProperties` 取代 `Vue.prototype`
   2. 如果不能直接获取到 `app`，定义 `install` 方法，从参数中获取 `app`，再完成挂载
7. Uncaught TypeError: Cannot set properties of undefined (setting '\$Lazyload')

   `vue-lazyload` 目前不支持 `Vue3`，需要等作者更新

   参考：https://github.com/hilongjw/vue-lazyload/issues/455

8. Feature flags **VUE_OPTIONS_API**, **VUE_PROD_DEVTOOLS** are not explicitly defined.

   参考：https://github.com/vuejs/vue-next/tree/master/packages/vue#bundler-build-feature-flags

9. ...

## 总结

![](/assets/images/使用GoGoCode将Vue2项目升级到Vue3/v-1.png)

不知不觉间 [Vue 3.0 正式版发布](https://github.com/vuejs/vue-next/releases/tag/v3.0.0)，距今已经过去一年多，每次推送看到 `Vue3` 相关的文章的时候，都感觉是在催促我赶紧用 `Vue3`，再不用就落伍了一样。

然而当我有想法要学习 `Vue3` 的时候，有时不知从哪里入手，学习效率不高（学习五分钟，休息两小时...），直接看官方文档和示例，仿写过文档上的示例，但没过多久就忘记了，又得重头再来。

归根结底还是光输入，但输出的太少，印象不够深刻，所以就有了这篇文章，坚持输出，巩固知识。

> 抢经济发育，我一顿操作猛如虎
>
> 关键团战，我唯唯诺诺，一看战绩 `0-5`！
>
> Skr Skr

这不点个赞 👍 再走？
