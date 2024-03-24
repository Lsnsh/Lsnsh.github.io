---
title: Gulp v3 升级到 v4 时遇到的问题
title_en: problems-encountered-when-upgrading-gulp-v3-to-v4
date: 2022-11-04 15:49:21
tags:
  - JavaScript
  - Gulp
  - Log
  - NodeJS
---

- Background
- Question
- Reason
- Solution
- Summary

---

## Background

- Node.js
- NPM
- Gulp

### Node.js

> Node.js® is an open-source, cross-platform JavaScript runtime environment.
>  
> [https://nodejs.org/en/](https://nodejs.org/en/)


![nodejs-1.png](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/nodejs-1.png)

---


**Dependencies:**

- Libraries 
   - V8
   - libuv
   - llhttp
   - c-ares
   - OpenSSL
   - zlib
- Tools 
   - npm
   - gyp
   - gtest

**Standard library:**

- path, fs
- net, http
- process, child_process, cluster
- ...

### NPM

> npm is the package manager for Node.js. It was created in 2009 as an open source project to help JavaScript developers easily share packaged modules of code.
>  
> [https://www.npmjs.com/about](https://www.npmjs.com/about)


![npm-1.png](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/npm-1.png)

---


**Some package:**

- ...
- ant-design, element-ui, material-ui
- react, vue, angular
- babel, typescript, eslint, postcss
- **gulp**, webpack, vite

---

- aws-sdk, axios, lodash
- express, koa, nestjs, egg, midway
- mongodb, redis, mysql
- ...

### Gulp

> **A toolkit to automate & enhance your workflow**
>  
> Leverage gulp and the flexibility of JavaScript to automate slow, repetitive workflows and compose them into efficient build pipelines.
>  
> [https://gulpjs.com/](https://gulpjs.com/)


![gulp.png](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/gulp.png)

- task
- composable(pipe, series, parallel)
- plugin

## Question

### 1. Frist gulp test execution failed

```
➜  **-***-****** git:(dev) gulp test
ReferenceError: primordials is not defined
    at fs.js:36:5
    at req_ (/Users/*****/work/**-***-******/node_modules/natives/index.js:143:24)
    at Object.req [as require] (/Users/*****/work/**-***-******/node_modules/natives/index.js:55:10)
    at Object.<anonymous> (/Users/*****/work/**-***-******/node_modules/vinyl-fs/node_modules/graceful-fs/fs.js:1:37)
    ...
```
#### Reason

> Gulp 3 is no longer supported.
>  
> [https://github.com/gulpjs/gulp/issues/2324](https://github.com/gulpjs/gulp/issues/2324)


- Gulp v3.9.1 release time: 2019-04-22
- Node v12.x initial release time: 2019-04-23

Because Gulp v3.x not support NodeJS v12 and above.

If switch to Node v10, can work normally.

![](https://github.***.io/storage/user/308/files/0944b900-1eeb-11ed-932f-f52af3ed6810#id=XEbXC&originHeight=819&originWidth=740&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

#### Solution

Upgrade to Gulp v4:

```
npm i -D gulp@latest
```

```diff
-   "gulp": "^3.9.1",
+   "gulp": "^4.0.2",
```

#### Result

```
➜  **-***-****** git:(***) ✗ gulp test
AssertionError [ERR_ASSERTION]: Task function must be specified
    at Gulp.set [as _setTask] (/Users/*****/work/**-***-******/node_modules/undertaker/lib/set-task.js:10:3)
    at Gulp.task (/Users/*****/work/**-***-******/node_modules/undertaker/lib/task.js:13:8)
    at Object.<anonymous> (/Users/*****/work/**-***-******/tasks/deploy.js:4:6)
    at Module._compile (internal/modules/cjs/loader.js:1085:14)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1114:10)
    at Module.load (internal/modules/cjs/loader.js:950:32)
    at Function.Module._load (internal/modules/cjs/loader.js:790:12)
    at Module.require (internal/modules/cjs/loader.js:974:19)
    at require (internal/modules/cjs/helpers.js:101:18)
    at requireDir (/Users/*****/work/**-***-******/node_modules/require-dir/index.js:128:33)
```

### 2. Second gulp test execution failed

```
➜  **-***-****** git:(***) ✗ gulp test
AssertionError [ERR_ASSERTION]: Task function must be specified
    at Gulp.set [as _setTask] (/Users/*****/work/**-***-******/node_modules/undertaker/lib/set-task.js:10:3)
    at Gulp.task (/Users/*****/work/**-***-******/node_modules/undertaker/lib/task.js:13:8)
    at Object.<anonymous> (/Users/*****/work/**-***-******/tasks/deploy.js:4:6)
    at Module._compile (internal/modules/cjs/loader.js:1085:14)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1114:10)
    at Module.load (internal/modules/cjs/loader.js:950:32)
    at Function.Module._load (internal/modules/cjs/loader.js:790:12)
    at Module.require (internal/modules/cjs/loader.js:974:19)
    at require (internal/modules/cjs/helpers.js:101:18)
    at requireDir (/Users/*****/work/**-***-******/node_modules/require-dir/index.js:128:33)
```

#### Reason

**deploy.js:4:6**

```javascript
gulp.task("deploy", ["update_endpoint"], function () {
  // ...
});
```

Because `gulp.task()` second params need function, task arrays no longer supported.

And if want to support task serialization, need use `gulp.series()`.

![gulp-api-1.png](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/gulp-api-1.png)

#### Solution

Use new syntax/api refactor task code:

Case 1:

```diff
- gulp.task('deploy', ['update_endpoint'], function () {
- ...
+ ...
+ gulp.task("deploy", gulp.series("update_endpoint", upload));
```

Case 2:

```diff
- gulp.task('docs', function() {
- ...
+ ...
+ gulp.task("docs", function(cb) {
+  cb();
```

Other changes.

#### Result

After Change, `gulp test` can be exec on node v12 above:

![](https://github.***.io/storage/user/308/files/52950880-1eeb-11ed-8886-3ee28a61a98b#id=gjPny&originHeight=496&originWidth=618&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## Test

**Task struct:**

```
➜  **-***-****** git:(dev) gulp -T
Tasks for ~/work/**-***-******/gulpfile.js

├── build
├── update_endpoint
├─┬ deploy
│ └─┬ <series>
│   ├── update_endpoint
│   └── upload
├─┬ local-deploy
│ └─┬ <series>
│   ├── localUpdateEndpoint
│   └── upload
├── docs
├─┬ init
│ └─┬ <series>
│   ├── <anonymous>
│   ├── <anonymous>
│   └── copyCustom
├── copy-source
├─┬ package
│ └─┬ <series>
│   ├── copy-source
│   └── zipRelease
└── test
```

### Teamcity

-  A build log on the first day:
https://teamcity.***.io/viewLog.html?buildId=791019&buildTypeId=HliPortal_DevRc_GdHliPortal&tab=buildLog
![](https://github.***.io/storage/user/308/files/42c8f480-1eea-11ed-813b-620577919026#id=V79Sj&originHeight=478&originWidth=538&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  A build log on the next day:
https://teamcity.***.io/viewLog.html?buildId=791394&buildTypeId=HliPortal_DevRc_GdHliPortal&tab=buildLog
![](https://github.***.io/storage/user/308/files/9982fe00-1eeb-11ed-8845-093f21ae7b65#id=kKECI&originHeight=494&originWidth=587&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 

### Local

<!-- TODO: use local images -->

#### NodeJS v14

In a clean git clone, use NodeJS v14(v14.20.0):

- **gulp init**
![](https://github.***.io/storage/user/308/files/0908e580-1f0c-11ed-815a-56805d6875a7#id=VpLj8&originHeight=349&originWidth=885&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
- **gulp test**
![](https://github.***.io/storage/user/308/files/93514980-1f0c-11ed-8038-55f00084a0ab#id=ELqkg&originHeight=350&originWidth=884&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
- **gulp build**
![](https://github.***.io/storage/user/308/files/9c421b00-1f0c-11ed-90ad-3e7c58f1c33a#id=Z0L2j&originHeight=349&originWidth=885&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
- **gulp docs**
![](https://github.***.io/storage/user/308/files/a19f6580-1f0c-11ed-80b6-db67bdcbce56#id=Ywry3&originHeight=351&originWidth=886&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
- **gulp local-deploy**
![](https://github.***.io/storage/user/308/files/ae23be00-1f0c-11ed-920d-be64e7d63576#id=QseUn&originHeight=351&originWidth=886&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
   - [chore(gulp): local-deploy define config variable](https://github.***.io/_********/**-***-******/pull/953/commits/49cc7e61d2bfe51d35ebe1e4c9a1c9e7b2172a85)
![](https://github.***.io/storage/user/308/files/c5fb4200-1f0c-11ed-9f39-d1d0df260150#id=XhC0e&originHeight=348&originWidth=885&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
- **gulp package**
![](https://github.***.io/storage/user/308/files/cdbae680-1f0c-11ed-8939-7bd4535903e5#id=CPUo5&originHeight=381&originWidth=885&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
- **gulp --tasks**
![](https://github.***.io/storage/user/308/files/d57a8b00-1f0c-11ed-990c-692fe59bc720#id=rJPhB&originHeight=349&originWidth=886&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

#### NodeJS v12

In a clean git clone, use NodeJS v12(v12.22.12):

-  **gulp init**
![](https://github.***.io/storage/user/308/files/b5ed5d80-1f20-11ed-9c84-5c277f5b10eb#id=ZkucU&originHeight=344&originWidth=892&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
![](https://github.***.io/storage/user/308/files/77f03980-1f20-11ed-9869-c0927e995fae#id=uOJyP&originHeight=350&originWidth=891&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  **gulp test**
![](https://github.***.io/storage/user/308/files/7a529380-1f20-11ed-8093-457721414c07#id=tkSWn&originHeight=347&originWidth=891&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  **gulp build**
![](https://github.***.io/storage/user/308/files/7de61a80-1f20-11ed-810b-1c8374cbfd43#id=npCEQ&originHeight=348&originWidth=888&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  **gulp docs**
![](https://github.***.io/storage/user/308/files/80e10b00-1f20-11ed-8db0-477adefbc99e#id=gPhYk&originHeight=351&originWidth=892&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  **gulp local-deploy**
![](https://github.***.io/storage/user/308/files/83dbfb80-1f20-11ed-9469-f61306d63e7c#id=gSyme&originHeight=348&originWidth=889&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  **gulp package**
![](https://github.***.io/storage/user/308/files/88081900-1f20-11ed-8007-4ca405529876#id=C5sV2&originHeight=380&originWidth=890&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 
-  **gulp --tasks**
![](https://github.***.io/storage/user/308/files/8b030980-1f20-11ed-86af-ac6cd7fcf312#id=zy7Fn&originHeight=347&originWidth=892&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=) 

## Thrid Question

gulp ** task exec failed, and missing `dateformat` module.

TODO: What caused ? **package lock file ?**

### Reason

`gulp-***@0.1.21` use `gulp@3.9.1` is out of date.

```
➜  **-***-****** git:(dev) npm ls gulp
**-***-******@1.2.0 /Users/*****/work/**-***-******
├── gulp@4.0.2
└─┬ gulp-***@0.1.21
  └── gulp@3.9.1
```

The [dateformat](https://www.npmjs.com/package/dateformat) used in the project is a child dependency from `gulp-***@0.1.21`
the dependencies are as follows:

```bash
➜  **-***-****** git:(dev) npm ls dateformat
**-***-******@1.2.0 /Users/*****/work/**-***-******
└─┬ gulp-***@0.1.21
  └─┬ gulp@3.9.1
    └─┬ gulp-util@3.0.8
      └── dateformat@2.2.0
```

![](https://github.***.io/storage/user/308/files/4ad1da00-23ab-11ed-969b-26da2f963ca3#id=Spjzf&originHeight=697&originWidth=1343&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### Solution

Upgrade `gulp-***` to v0.2.1:

```bash
npm i -D gulp-***@latest
npm i -D dateformat@latest
```

```diff
+    "dateformat": "^5.0.3",
...
-    "gulp-***": "^0.1.21",
+    "gulp-***": "^0.2.1",
```

### Result

After change, the dependencies are as follows:

```
➜  **-***-****** git:(feature/upgrade-gulp-***-and-clear-unused-dependencies) npm ls gulp
**-***-******@1.2.0 /Users/*****/work/**-***-******
├── gulp@4.0.2
└─┬ gulp-***@0.2.1
  └── gulp@4.0.2  deduped

➜  **-***-****** git:(feature/upgrade-gulp-***-and-clear-unused-dependencies) npm ls dateformat
**-***-******@1.2.0 /Users/*****/work/**-***-******
└── dateformat@5.0.3
```

`gulp package` works normally:

![](https://github.***.io/storage/user/308/files/3db5ea80-23ad-11ed-8d57-f896f74ed37f#id=rUXOU&originHeight=282&originWidth=827&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## Summary

- Gulp ?
- Node.js ?
- NPM ?

---

### Gulp ?

**Gulp v3 not support Node v12 above:**

> And to clarify even further, the `natives` module would only come from a VERY outdated version of `graceful-fs` that is not included in gulp 4 so it's going to be a different issue within your project (like an incorrect lockfile).
>  
> [https://github.com/gulpjs/gulp/issues/2324#issuecomment-488646731](https://github.com/gulpjs/gulp/issues/2324#issuecomment-488646731)


![](https://github.***.io/storage/user/308/files/101cfd00-1ee6-11ed-9458-c18cfe019769#id=Wqj2h&originHeight=807&originWidth=845&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- In Gulp v4, `vinyl-fs` from ^0.3.0 upgrade to ^3.0.0
- while upgrading the primary dependency, `graceful-fs` as a sub dependency, is also upgraded from ^3.0.0 to ^4.0.0.

```
npm WARN deprecated graceful-fs@1.2.3: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
```

![gulp-package-2.png](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/gulp-package-2.png)

### Node.js ?

> - Current - Should incorporate most of the non-major (non-breaking) changes that land on nodejs/node main branch.
> - Active LTS - New features, bug fixes, and updates that have been audited by the LTS team and have been determined to be appropriate and stable for the release line.
> - Maintenance - Critical bug fixes and security updates. New features may be added at the discretion of the LTS team - typically only in cases where the new feature supports migration to later release lines.
> 
 
> [https://github.com/nodejs/release#release-schedule](https://github.com/nodejs/release#release-schedule)


- The versions before Node v14 are outdated
- Node v14 end-of-line is 2023-04-30
- Node v18 enter LTS status
![schedule.svg](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/schedule.svg)
- Node package security
- More feature and better performance

### NPM ?

Released with NodeJS.

![npm-2.png](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/npm-2.png)

- More feature and better performance 
   - package lock file
   - overrides

Other NodeJS package manager:

- Yarn
- PNPM

#### PNPM

> pnpm stands for performant npm. [@rstacruz ](/rstacruz ) came up with the name. 
>  
> [https://pnpm.io/faq](https://pnpm.io/faq)


![pnpm.jpeg](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/pnpm.jpeg)

![pnpm-node-modules-structure.jpeg](/assets/images/2022-11-04-problems-encountered-when-upgrading-gulp-v3-to-v4/pnpm-node-modules-structure.jpeg)

When installing dependencies with npm or Yarn Classic, all packages are hoisted to the root of the modules directory. As a result, source code has access to dependencies that are not added as dependencies to the project.

By default, pnpm uses symlinks to add only the direct dependencies of the project into the root of the modules directory.

## References

- [https://nodejs.org/en/](https://nodejs.org/en/)
- [https://www.npmjs.com/about](https://www.npmjs.com/about)
- [https://gulpjs.com/](https://gulpjs.com/)
- [https://github.com/gulpjs/gulp/issues/2324](https://github.com/gulpjs/gulp/issues/2324)
- [https://github.***.io/_********/**-***-******/pull/972](https://github.***.io/_********/**-***-******/pull/972)
- [https://github.***.io/_********/**-***-******/pull/953](https://github.***.io/_********/**-***-******/pull/953)
- [https://artifactory.***.io/artifactory/webapp/#/packages/npm/gulp-***/?state=eyJxdWVyeSI6eyJucG1OYW1lIjoiZ3VscC1obGkifX0%3D](https://artifactory.***.io/artifactory/webapp/#/packages/npm/gulp-***/?state=eyJxdWVyeSI6eyJucG1OYW1lIjoiZ3VscC1obGkifX0%3D)
- [https://github.com/nodejs/release#release-schedule](https://github.com/nodejs/release#release-schedule)
- [https://pnpm.io](https://pnpm.io/faq)
- ...
