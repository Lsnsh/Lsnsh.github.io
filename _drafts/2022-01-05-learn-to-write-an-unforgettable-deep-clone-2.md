---
title: 写一个印象深刻的深拷贝-2
title_en: learn-to-write-an-unforgettable-deep-clone-2
date: 2022-01-05 14:43:44
tags:
  - JavaScript
---

## 背景

深拷贝，经常在各种场合能听到或者看到，有很简洁也有很复杂的。看完总觉得收获了不少，以为下次自己也能写出来，可实际面试中被问到时，基本已经忘光了。

此前在面试中受挫过，为了准备面试和重建知识体系，决定重学“前端八股文”（网友戏称，泛指前端基础知识），并以文章形式记录下来。

在学习写深拷贝时，直接参考的 [lodash.cloneDeep][11] 的源码，源码很完整但是不适合由浅入深，容易让人摸不着头脑，于是在掘金上找到了深拷贝最高赞的[《如何写出一个惊艳面试官的深拷贝?》][10]。

本篇文章主要学习 `ConardLi` 的 [《如何写出一个惊艳面试官的深拷贝?》][10] 的和 [lodash.cloneDeep][11] 源码，一步一步写出一个让自己印象深刻的深拷贝。

<!-- more -->

## 什么是深拷贝

浅拷贝（[Object.assign()][12]）会复制源对象上的所有值，而引用类型的值在源对象上存储的是它的内存地址或者叫指针，因此浅拷贝后，源对象和目标对象上的引用类型，其实还是同一个。

```js
let objects = {
  string: "lsnsh",
  object: { a: 1, b: 2 },
};

let actual = Object.assign({}, objects);
actual.string = "hello-" + actual.string;
actual.object.a = 3;

console.log(actual.string, objects.string);
// hello-lsnsh lsnsh
console.log(actual.object.a, objects.object.a);
// 3 3
```

深拷贝（cloneDeep）是在浅拷贝的基础上，处理了引用类型的数据，使得：

> 源对象与目标对象之间互相独立，其中任何一个对象的改动，都不会对另外一个对象造成影响。

```js
import cloneDeep from "../***.js";

let objects = {
  string: "lsnsh",
  object: { a: 1, b: 2 },
};

let actual = cloneDeep(objects);
actual.string = "hello-" + actual.string;
actual.object.a = 3;

console.log(actual.string, objects.string);
// hello-lsnsh lsnsh
console.log(actual.object.a, objects.object.a);
// 3 1
```

## 为什么要深拷贝

- 数据作为模板的场景（比如：编辑多个副本）
  - 基于源数据，生成副本，编辑时不希望影响源数据
- 数据需要还原的场景（比如：表单撤销编辑）
  - 基于源数据，生成副本，编辑时不希望影响源数据
  - 撤销编辑时，基于源数据，生成副本，重置表单数据
- 等等...

## 如何实现深拷贝

深拷贝的核心思想是浅拷贝 + 递归调用，下面我们将从一个个测试用例入手，逐步完善深拷贝的代码。

### 一、对象版本

- 对象版本：仅支持拷贝基本类型和包含基本类型的对象
- 不支持：拷贝数组、包含数组的对象等其他非基本类型的数据、以及循环引用

#### 测试用例

遍历测试用例的数据，调用深拷贝方法，逐一比较源对象和目标对象是否相等。

```js test/1.test.js
import cloneDeep from "../1.js";

function test() {
  let objects = {
    boolean: false,
    string: "lsnsh",
    number: 99,
    array: [1, 2, { value: 3 }],
    object: { a: 1, b: 2, c: { value: 3 } },
  };

  for (let prop in objects) {
    let object = objects[prop],
      actual = cloneDeep(object);

    console.log(
      `should clone ${prop}(`,
      actual,
      "vs",
      object,
      "):",
      actual === object
    );
  }
}

test();
```

#### 代码

先判断源对象类型，基本类型的数据，直接返回；

对象类型的数据，遍历后递归调用深拷贝，完成对每一项的深拷贝。

```js js/1.js
export default function cloneDeep(data) {
  // 基本类型，直接返回
  if (typeof data !== "object") {
    return data;
  }

  let ret = {};

  // 遍历对象，递归拷贝子项
  for (let p in data) {
    ret[p] = cloneDeep(data[p]);
  }

  return ret;
}
```

#### 运行测试用例

![](/assets/images/写一个印象深刻的深拷贝/1.test.png)

从测试用例运行结果来看：

- 基本类型的数据，全等（`===`）的比较结果正确（`true`）
- 对象数据，全等的比较结果也正确（`false`），对象的每一项也都相等
- 数组数据，由于目标对象初始化为空对象（`let ret = {}`），导致遍历拷贝数组成员后，目标对象被赋值成类数组对象。

### 二、数组版本

- 数组版本：支持拷贝基本类型、对象和数组
- 不支持：其他非基本类型的数据、以及循环引用

#### 测试用例

增加 `null`、`undefined` 和循环引用的 `case`。

```diff test/2.test.js
function test() {
  let objects = {
    boolean: false,
    string: "lsnsh",
    number: 99,
-    array: [1, 2, { value: 3 }],
+    array: [1, 2, { value: 3 }, null],
-    object: { a: 1, b: 2, c: { value: 3 } },
+    object: { a: 1, b: 2, c: { value: 3 }, d: null },
+    null: null,
+    undefined: undefined,
  };

+  objects["circular references"] = objects;

  // ...
}

test();
```

#### 代码

使用 `typeof` 获取 `null` 会得到 `'object'` 的返回值，因此单独拿出来判断；

使用 `Array.isArray()` 方法检查是否是数组，给目标对象初始化正确的值。

```diff js/2.js
export default function cloneDeep(data) {
-  // 基本类型，直接返回
+  // null 和基本类型，直接返回
-  if (typeof data !== "object") {
+  if (data === null || typeof data !== "object") {
    return data;
  }

-  let ret = {};
+  // 对象和数组，初始化不同的值
+  let ret = Array.isArray(data) ? [] : {};
  
  // ...
}
```

#### 运行测试用例

![](/assets/images/写一个印象深刻的深拷贝/2.test.png)

从测试用例运行结果来看：

- 目标对象初始化后，数组数据能正常拷贝
- 新增的 `null` 和 `undefined` 也正常拷贝
- 存在循环引用的 `objects` 对象数据，使得递归没法终止，导致代码运行报错并抛出错误信息：`RangeError: Maximum call stack size exceeded`，这是我们下一步要解决的问题

#### 执行过程

![](/assets/images/写一个印象深刻的深拷贝/2.process.png)

到这里深拷贝方法的大体流程就已经出来了：

- 输入数据，检查类型
- 如果是不需要特殊处理的数据/部分基本类型的数据（流程图中简化了），直接返回
- 否则根据类型的不同，对目标类型做初始化
- 然后递归调用的拷贝数据，直到递归终止，完成深拷贝


补充：只是数组阶段的流程总结，特殊情况的特殊处理（循环引用等），特殊类型的特殊处理，有些类型在初始化后即可结束（基本类型的包装类型的实例：`new Boolean(false)`），在后续阶段中我们会遇到。

### 三、循环引用版本

- 循环引用版本：支持拷贝基本类型、对象和数组，以及存在循环引用的情况
- 不支持：基本类型的包装类、函数、Map、Set 等非基本类型

#### 测试用例

循环引用的 `case`，在上一阶段中就已经存在了。

```js test/3.test.js
// ...
function test() {
  // ...
  objects["circular references"] = objects;
  // ...
}

test();
```

#### 代码

> Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现

使用 Map 结构保存深拷贝时，源对象的值和目标对象的值；

通过 `map.has(data)` 检查源对象是否存在，即是否存在循环引用的情况，如果存在，直接返回保存过的目标对象。

```diff js/3.js
- export default function cloneDeep(data) {
+ export default function cloneDeep(data, map = new Map()) {
  // ...
  let ret = Array.isArray(data) ? [] : {};
+  
+  if (map.has(data)) {
+    return map.get(data);
+  }
+  map.set(data, ret);

  for (let p in data) {
-    ret[p] = cloneDeep(data[p]);
+    ret[p] = cloneDeep(data[p], map);
  }

  return ret;
}
```

#### 运行测试用例

![](/assets/images/写一个印象深刻的深拷贝/3.test.png)

从测试用例运行结果来看：

- 代码运行没有报错，存在循环引用的对象，拷贝后在控制台也正常输出了

### 四、Map 和 Set 版本

- Map 和 Set 版本：支持拷贝基本类型、对象、数组、Map 和 Set，以及存在循环引用的情况
- 不支持：基本类型的包装类、函数等非基本类型

#### 测试用例

增加 `Map` 和 `Set` 类型的数据

```diff test/4.test.js
function test() {
  let objects = {
    boolean: false,
    string: "lsnsh",
    number: 99,
    array: [1, 2, { value: 3 }, null],
    object: { a: 1, b: 2, c: { value: 3 }, d: null },
+    map: new Map([
+      ["name", "张三"],
+      ["age", 18],
+    ]),
+    set: new Set([1, 2, "three"]),
    null: null,
    undefined: undefined,
  };
  // ...
}

test();
```

#### 代码

- 定义数组、对象、Map 和 Set 的类型标签常量
- getTag(): 使用对象原型上的 `toString()`  方法能获取到类型标签
- isObject(): 将 `function` 归为对象类型，排除 `null` 类型，因为 `typeof null` 会得到 'object'
- initCloneByTag(): 根据类型标签，初始化不同的值
- 然后根据类型标签的不同，执行各自不同的拷贝操作

```diff
+ const arrayTag = "[object Array]";
+ const objectTag = "[object Object]";
+ const mapTag = "[object Map]";
+ const setTag = "[object Set]";

+ function isObject(value) {
+   const type = typeof value;
+   return value !== null && (type === "object" || type === "function");
+ }

+ function getTag(value) {
+   return Object.prototype.toString.call(value);
+ }

+ function initCloneByTag(value, tag) {
+   switch (tag) {
+     case arrayTag:
+       return [];
+     case objectTag:
+       return {};
+     case mapTag:
+       return new Map();
+     case setTag:
+       return new Set();
+   }
+ }
+
- export default function cloneDeep(data, map = new Map()) {
+ function cloneDeep(data, map = new Map()) {
-  // null 和基本类型，直接返回
+  // 克隆 null 和基本类型，直接返回
-  if (data === null || typeof data !== "object") {
+  if (!isObject(data)) {
    return data;
  }

+  const tag = getTag(data);
-  // 对象和数组，初始化不同的值
+  // 根据类型，初始化不同的值
-  let ret = Array.isArray(data) ? [] : {};
+  let ret = initCloneByTag(data, tag);

  // 记录拷贝过的值，处理循环引用
  if (map.has(data)) {
    return map.get(data);
  }
  map.set(data, ret);

+  // 克隆 Map
+  if (tag === mapTag) {
+    data.forEach((value, key) => {
+      ret.set(key, cloneDeep(value, map));
+    });
+  }
+  // 克隆 Set
+  if (tag === setTag) {
+    data.forEach((value, key) => {
+      ret.add(cloneDeep(value, map));
+    });
+  }
+  // 克隆对象和数组
  for (let p in data) {
    ret[p] = cloneDeep(data[p], map);
  }

  return ret;
}

+ export default cloneDeep;
```

#### 运行测试用例

![](/assets/images/写一个印象深刻的深拷贝/4.test.png)

从测试用例运行结果来看：

- 代码运行没有报错，`Map` 和 `Set`，拷贝后在控制台也正常输出了

### 五、基本类型的包装类型版本

- 基本类型的包装类型版本：支持拷贝基本类型及其包装、对象、数组、Map 和 Set，以及存在循环引用的情况
- 不支持：Symbol、正则、函数、日期等非基本类型版本

#### 测试用例

基本类型除了字面量的形式外，还有以基本类型的包装类的实例存在的形式，增加基本类型的包装类（Boolean、String、Number）实例的值。

```diff test/5.test.js
function test() {
  let objects = {
    // ...
+    "boolean object": new Boolean(true),
+    "string object": new String("hello"),
+    "number object": new Number(1023),
    null: null,
    undefined: undefined,
  };
  // ...
}

test();
```

#### 代码

- 定义基本类型（Boolean、String、Number）的类型标签常量
- initCloneByTag(): 根据新增的类型标签，使用基本类型的构造函数，初始化基本类型包装类的实例

```diff
+const booleanTag = "[object Boolean]";
+const stringTag = "[object String]";
+const numberTag = "[object Number]";
// ...
const setTag = "[object Set]";

// ...

function initCloneByTag(value, tag) {
+  let valueConstructor = value.constructor;
  switch (tag) {
+    case booleanTag:
+      return new valueConstructor(+value);
+    case stringTag:
+    case numberTag:
+      return new valueConstructor(value);
    case arrayTag:
      return [];
    case objectTag:
      return {};
    case mapTag:
      return new Map();
    case setTag:
      return new Set();
  }
}

function cloneDeep(data, map = new Map()) {
  // ...
}

export default cloneDeep;
```

#### 运行测试用例

![](/assets/images/写一个印象深刻的深拷贝/5.1.test.png)

从测试用例运行结果来看：

- 布尔类型的实例能正常拷贝
- 拷贝字符串的实例对象时，运行报错并抛出：`TypeError: Cannot assign to read only property '0' of object '[object String]'`
  - 大概意思是：字符串对象的属性可以遍历访问，每个字符可以以数组下标的形式访问，但是是只读的，不能进行二次赋值

#### 代码改进

通过打断点调试 [lodash.cloneDeep][11] 的源码，发现 lodash 在拷贝对象属性时，做了一些[过滤处理][16]，这里我们做了一些简化：

- hasOwnProperty(): 使用对象原型上的 hasOwnProperty 方法，检查变量自身是否有指定的属性（key）
- assignValue(): 变量上指定的属性不存在或值不相等时，进行赋值

```diff
// ...

+function hasOwnProperty(object, key) {
+  return Object.prototype.hasOwnProperty.call(object, key);
+}

+function assignValue(object, key, value) {
+  const objectValue = object[key];
+  if (!(hasOwnProperty(object, key) && objectValue === value)) {
+    object[key] = value;
+  }
+}

function initCloneByTag(value, tag) {
  // ...
}

function cloneDeep(data, map = new Map()) {
  // ...

  // 克隆对象和数组
  for (let p in data) {
-    ret[p] = cloneDeep(data[p], map);
+    // 属性值不存在或值不相等时，才会进行赋值
+    assignValue(ret, p, cloneDeep(data[p], map));
  }

  return ret;
}

export default cloneDeep;
```


#### 运行测试用例

![](/assets/images/写一个印象深刻的深拷贝/5.2.test.png)

从测试用例运行结果来看：

- 代码运行没有报错，基本类型的包装类型的实例都拷贝成功了

### 六、性能优化版本

- 性能优化版本

#### 测试用例

- performanceTest(): 新增性能测试方法
  - 初始化 array object 和 like array object
  - 执行 cloneDeep 并记录执行耗时
    - 根据参数 isWhile 决定是使用 while，还是 for 遍历
  - 返回执行耗时
- 执行 total 次 performanceTest，打印每次执行耗时、以及计算的平均耗时

```js
function test() { // ... }

// test();

function performanceTest(isWhile) {
  let objects = {
    "array object": [],
    "like array object": {},
  };

  // console.time("init");
  let i = 1 * 1024 * 1024;
  while (i > 0) {
    objects["array object"][i] = i;
    objects["like array object"][i] = i;
    i--;
  }
  // console.timeEnd("init");

  let begin, end, duration;
  if (isWhile) {
    begin = globalThis.performance.now();
    cloneDeep(objects, undefined, isWhile);
    end = globalThis.performance.now();
  } else {
    begin = globalThis.performance.now();
    cloneDeep(objects, undefined, isWhile);
    end = globalThis.performance.now();
  }
  duration = (end - begin).toFixed(3);
  console.log(isWhile ? "while" : "for", "duration:", duration + "ms");

  return +duration;
}

let i = 0,
  total = 10;

let whileTable = {
  record: [],
  whileAvg: 0,
};

for (i = 0; i < total; i++) {
  whileTable.record.push(performanceTest(true));
}
whileTable.whileAvg = whileTable.record.reduce((a, b) => a + b, 0) / total;

console.table(whileTable);

// ---

// let forTable = {
//   record: [],
//   forAvg: 0,
// };

// for (i = 0; i < total; i++) {
//   forTable.record.push(performanceTest(false));
// }
// forTable.forAvg = forTable.record.reduce((a, b) => a + b, 0) / total;

// console.table(forTable);
```

#### 代码

- 新增测试需要的参数 isWhile 和 if else 的代码

```diff
-function cloneDeep(data, map = new Map()) {
+function cloneDeep(data, map = new Map(), isWhile = true) {
  // ...

  // 克隆对象和数组
+  // TODO: 性能测试需要，实际 while/for 只会二选一，保留其中一种写法
+  if (isWhile) {
+    let i = 0,
+      keys = Object.keys(data);
+    while (i < keys.length) {
+      let key = keys[i];
+      // 属性值不存在或值不相等时，才会进行赋值
+      assignValue(ret, key, cloneDeep(data[key], map, isWhile));
+      i++;
+    }
+  } else {
    for (let p in data) {
      // 属性值不存在或值不相等时，才会进行赋值
      assignValue(ret, p, cloneDeep(data[p], map, isWhile));
    }
+  }

  return ret;
}

export default cloneDeep;
```

#### 运行测试用例

- 分别在首次打开终端（Terminal Session A、B）后，运行测试用例
  - Session A: while#1
  - Session B: for#1

![](/assets/images/写一个印象深刻的深拷贝/6.1.test.png)
![](/assets/images/写一个印象深刻的深拷贝/6.2.test.png)

- 另外四组运行测试用例的情况统计：
  - Session C: for#2 -> while#3 -> for#4 -> while#5
  - Session D: while#2 -> for#3 -> while#4 -> for#5

| 组别\各阶段耗时（ms） | 0       | 1       | 2       | 3       | 4       | 5       | 6       | 7       | 8       | 9       | 平均值    |
| --------------------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | --------- |
| for#2(Session C)      | 698.748 | 623.037 | 682.723 | 700.934 | 699.449 | 615.233 | 696.697 | 688.337 | 627.851 | 693.115 | 672.61239 |
| while#2(Session D)    | 678.141 | 589.085 | 655.147 | 660.993 | 585.023 | 645.599 | 670.573 | 583.643 | 646.959 | 660.828 | 637.5991  |
| for#3(Session D)      | 834.881 | 671.708 | 653.65  | 684.887 | 621.239 | 661.587 | 690.679 | 622.909 | 674.385 | 682.122 | 679.8047  |
| while#3(Session C)    | 902.724 | 589.732 | 636.617 | 665.078 | 651.595 | 592.791 | 678.123 | 661.07  | 589.01  | 669.497 | 663.6237  |
| for#4(Session C)      | 706.396 | 629.787 | 672.822 | 687.058 | 701.863 | 624.88  | 691.147 | 810.148 | 683.244 | 693.831 | 690.1176  |
| while#4(Session D)    | 677.284 | 601.243 | 637.241 | 664.218 | 598.322 | 654.407 | 679.73  | 604.676 | 642.763 | 669.523 | 642.9407  |
| for#5(Session D)      | 695.383 | 615.485 | 674.902 | 684.288 | 609.677 | 668.501 | 688.831 | 610.795 | 675.724 | 681.93  | 660.5516  |
| while#5(Session C)    | 676.316 | 590.415 | 659.954 | 665.958 | 765.763 | 588.631 | 671.326 | 656.047 | 588.927 | 668.64  | 653.19769 |

- 根据各组别的平均耗时，以 for 为基准，计算 while 的提升效率百分比：
  - #1: 4.351%
  - #2: 5.206%
  - #3: 2.38%
  - #4: 6.836%
  - #5: 1.113%
  - 平均: 3.9772%


从测试用例运行结果来看：

- 目前 case 的比较结果 while 和 for in 相差不大，while 平均能快 4% 左右
- 初始化 `objects["array object"]` 和 `objects["like array object"]` 大概耗时 200ms 左右，而 #3 第 1 次执行耗时较多（834.881 vs 902.724），#4 和 #5 两组的平均耗时相差较大，可能和垃圾回收机制有关

### 七、内存优化版本

> WeakMap，它的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

- 内存优化版本

#### 测试用例

```js test/7.test.js
// test
```

#### 代码

```diff
- function cloneDeep(data, map = new Map()) {
+ function cloneDeep(data, map = new WeakMap()) {
```

#### 运行测试用例

<!-- ![](/assets/images/写一个印象深刻的深拷贝/7.test.png) -->

从测试用例运行结果来看：

改动前后的内存消耗有明显变化

### 六、其余类型

到目前为止我们已经处理了对象、数组、Map、Set以及基本类型的字面量，其他类型的处理，分为两类：

- 可拷贝的类型
- 不可拷贝的类型

如果是可拷贝的类型，获取到具体类型的 `Tag` 时，针对性的处理即可，而不可拷贝类型，可以选择返回空对象或者其他值来表示不可拷贝。

#### 可拷贝的类型

- [object Arguments]
- [object Array]
- [object Boolean]
- [object Date]
- [object Map]
- [object Number]
- [object Object]
- [object Promise]
- [object RegExp]
- [object Set]
- [object String]
- [object Symbol]
- 
- [object ArrayBuffer]
- [object DataView]
- [object Float32Array]
- [object Float64Array]
- [object Int8Array]
- [object Int16Array]
- [object Int32Array]
- [object Uint8Array]
- [object Uint8ClampedArray]
- [object Uint16Array]
- [object Uint32Array]

#### 不可拷贝的类型

- [object Function]
- [object GeneratorFunction]
- [object Error]
- [object WeakMap]

## 原生拷贝方法

### 浅拷贝（Object.assign()/Array.Prototype.slice()）

`JavaScript` 语言内置了浅拷贝对象和数组的方法：

- `Object.assign(target, ...sources)`
  - 将源对象上所有可枚举属性的值，遍历的复制到目标对象上
- `Array.Prototype.slice([begin[, end]])`
  - 方法返回新的数组对象，根据 `begin` 和 `end` 决定如何浅拷贝，默认复制每一项，不会改变原数组

### 深拷贝（structuredClone()）

## 参考资料

- [如何写出一个惊艳面试官的深拷贝? - 掘金][10]
- [lodash.cloneDeep - GitHub][11]
- [如何从JS Console中的console.timeEnd（）获取输出？ - Thinbug][8]
- [Performance.now() - MDN][9]
- [Object.assign() - MDN][12]
- [深拷贝 - 百度百科][13]

## 总结

> 纸上得来终觉浅，绝知此事要躬行。——《冬夜读书示子聿》

[0]: https://github.com/lsnsh/code-world/src/JS/cloneDeep/
[1]: https://github.com/lsnsh/code-world/src/JS/cloneDeep/1.js
[1.test]: https://github.com/lsnsh/code-world/src/JS/cloneDeep/test/1.test.js
[8]: https://www.thinbug.com/q/12492979
[9]: https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/now
[10]: https://juejin.cn/post/6844903929705136141
[11]: https://github.com/lodash/lodash/blob/4.5.0-npm-packages/lodash.clonedeep
[12]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
[13]: https://baike.baidu.com/item/%E6%B7%B1%E6%8B%B7%E8%B4%9D/22785317
[14]: https://developer.mozilla.org/en-US/docs/Web/API/structuredClone
[15]: https://es6.ruanyifeng.com/#docs/set-map#WeakMap
[16]: https://github.com/lodash/lodash/blob/0ceb6d1dad4c0a89faf29958a55d706e60978843/lodash.clonedeep/index.js#L788
