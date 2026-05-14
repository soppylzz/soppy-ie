---
title: 面试合集：基础知识
date: 2026-05-14 11:43:22
tags:
---

## JavaScript

### 1. [JavaScript 有哪些数据类型？它们的区别是什么？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886882163236865)

> Primitive：`String`、`Numberic`（IEEE754 + BigInt）、`Symbol`、`Boolean`、`undefined`（未赋值）、`null`（空引用） (栈)
>
> Reference：`Object`（引用堆存储，对象栈存储）

#### 🌟 Null遗留问题：

Javascript的类型标识符的二进制表示如下：

- 000：对象
- 010：浮点数
- 100：字符串
- 110：布尔
- 1：整数

而空指针`null`的二进制表示为`00000000`，为了向后兼容选择了保留；

```ts
function isNull(val: unknown) val is null {
return typeof val === "object" && !val
}
```

#### ☁️ **Undefined问题：**

`undefined`在早期Javascript中并非<u>保留字</u>，可以被重新赋值；推荐使用`void 0`来获取其原始值；

这里的`void`运算符用于**执行后面的表达式**，并强制返回`undefined`原始值；在一些响应式系统中可以使用其在符合`Eslint`规范下**强制触发依赖收集**；

#### 🤗 数值精度问题：

IEEE754是为了统一计算机浮点表示而设计的工业标准：

* 32bit： $(-1)^s\times 2^{e-127} \times (1.f)_2$；这里的符号对应为 $s$ - 符号位(1)，$e$ - 指数位(8), $f$ - 尾数位(23)；
* 64bit：$(-1)^s\times 2^{e-1023} \times (1.f)_2$；符号对应同时，$s$ (1)，$e$ (11)，$f$ (52)；

### 2. [如何判断 JavaScript 变量是数组？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886883568328705)

> 原型链：`obj.__proto__ === Array.prototype`（等价`Object.getPrototypeOf(obj)`）、`Array.prototype.isPrototypeOf(obj)`、`Object.prototype.toString.call(obj) === "[Object Array]"`（通用判别）；
>
> 构造函数：
>
> 1. `instanceOf`：其本质是构造函数自身挂载的静态方法`Constructor[Symbol.hasInstance]`，沿着实例的原型链<u>**逐级向上**</u>比对，判断该实例是否归属当前构造函数；
> 2. 我们也可以直接比较构造函数`obj.constructor === Array`；
>
> `Array.isArray`：靠引擎内部真实类型标记 `[[Class]]`，最终解决办法；

基于原形链与构造函数的判别方案存在跨iframe失效

```html
<!DOCTYPE html>
<html>
<body>
<iframe src="iframe.html" id="frame"></iframe>
<script>
  document.getElementById('frame').onload = function () {
    // must be same origin
    const iframeWin = document.getElementById('frame').contentWindow;
    const iframeArr = iframeWin.arr;
    console.log(iframeArr);

    // false
    console.log(iframeArr instanceof Array);
    console.log(iframeArr.__proto__ === Array.prototype)
    console.log(Array.prototype.isPrototypeOf(iframeArr))

    // true
    console.log(Array.isArray(iframeArr));
    console.log(Object.prototype.toString.call(iframeArr));
  }
</script>
</body>
</html>
```

### 3. [为什么 JavaScript 中 0.1 + 0.2 !== 0.3，如何让其相等？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886884738539521)

> `Number`类型采用IEEE754存储，存在二进制截取问题，解决方案有：`Number.EPSILON`、`Number.prototype.toFixed(sum): string`、`Number.toPrecision(sum)`；

### 4. [isNaN 和 Number.isNaN 函数有什么区别？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886885430599682)

> `isNaN(val)`执行类型对`val`的类型转换，而`Number.isNan`则不执行；

这里也提一下其他类型到`Number`的类型转换：

```js
const res = [
  "",               // 0
  true,             // 1
  null,             // 0
  "0x123",          // 291
  {},               // NaN
  "string",         // NaN
  undefined,        // NaN
  /* Symbol(), can't convert */
]
res.forEach(raw => { console.log(Number(raw)) })
```

### 5. [== 操作符的强制类型转换规则是什么？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886885631926274)

> `==`类型**转换规则**如下：
>
> 1. 比较`undefined`、`null`，仅等于自身与对方，`symbol`只与`symbol`比较；
>
> 2. 其他原始类型比较遵循：`bool` ← `number` ← `string`的强转规则；
>
> 3. 对于引用类型参考：
>
>    ```js
>    const a = new Proxy({}, {
>      get(target, key) {
>        console.log(key)
>        return target[key]
>      }
>    })
>       
>    void (a == "")
>       
>    // Symbol(Symbol.toPrimitive)
>    // valueOf
>    // toString -> Symbol(Symbol.toStringTag)
>    ```
>
> <p style="color: crimson">这里的类型转换是共用的，适用于所有「<b>隐式类型转换</b>」；</p>

这里补充一下`==`、`===`与`Object.is(v1, v2)`：`==`会在类型不一致时进行类型转换预处理，`===`与`Object.is`则不会；后两者在`-/+0`、`NaN`存在些许区别；

### 6. [JavaScript 中 || 和 && 操作符的返回值是什么？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886886579838977)

> `||`，返回第一个真值项或末项；`&&`，返回第一个假值项或末项；`??`，返回第一个非`null`/`undefined`项或末项；

注意！这里并没有执行强转，因为假值项都是些静态值：`false`、`0`、`null`、`undefined`、`NaN`；（可以用`Proxy`测试`object`）

### 7. [什么是 JavaScript 中的包装类型？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886887007657985)

> 包装类型是JavaScript中的一种机制，它允许原始值临时拥有对象的属性和方法：
>
> 1. `String`
> 2. `Number`
> 3. `Boolean`

当我们使用其对引用类型进行显示类型转换：

```js
const a = new Proxy({}, {
  get(target, key) {
    console.log(key)
    return target[key]
  }
})

String(a)
// Symbol(Symbol.toPrimitive)
// toString
// Symbol(Symbol.toStringTag)
Number(a)
// Symbol(Symbol.toPrimitive)
// valueOf
// toString
// Symbol(Symbol.toStringTag)
Boolean(a)
// [empty]
```

### 8. [JavaScript 中 Map 和 Object 的区别是什么？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886888165285890)

> **键值相关**：
>
> `Object`包含原形链的默认键，且键只支持`string｜symbol`，使用除此之外的类型时，会对键值进行`String(k)`类型转换；而`Map`则只有显示注入的键值；
>
> 键值顺序也存在不同，`Map`的键值顺序与插入顺序有关，而`Object`在ES规范下有如下排序规则：
>
> 1️⃣ 所有数字索引键（$0 \sim 2^{32}-2$） → 2️⃣ 普通字符串键（插入顺序） → 3️⃣ Symbol键（插入顺序）

### 9. [ JavaScript 脚本异步加载如何实现？各有什么区别？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886888882511874)

> 1. `async`：异步加载，立即执行，适合独立性较高的脚本；
> 2. `defer`：在HTML解析后加载，保证执行脚本在文档中出现顺序执行；
> 3. 动态创建`<script/>`；
> 4. 使用模块化工具加载；

### 10. [什么是 JavaScript 的类数组对象？如何转化为数组？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886888970592257)

> `const obj = { 0: 0, 1: 1, length: 2 };`
>
> 1. `Array.from(obj)`
> 2. `[...obj]`：需要内部使用`yield`实现迭代器`Symbol.iterator`；

Javascript函数的`arguments`对象是`ArrayLike`对象，原因如下：

1. 历史原因：其在早起被引入，此时没有实现真正的数组；
2. 性能原因：如果用数组实现，会带来性能开销；

```js
function argumentTest(...args) {
  console.log(Object.prototype.toString.call(args));
  console.log(Object.prototype.toString.call(arguments));
}

argumentTest(1, 2, 3)
// [object Array]
// [object Arguments]
```

### 11. [什么是 AJAX？如何实现一个 AJAX 请求？](https://www.mianshiya.com/bank/1810644471159848962/question/1810886889784287233)

> AJAX（Asynchronous JavaScript And XML）不是一个 API 或方法，而是一种**思想和技术**，指在<u>**不刷新页面的情况下**</u>使用 JavaScript 与服务器进行异步通信。而`XHR`与`fetch`是两种实现方案；

#### 👴 XMLHttpRequest

```js
const xhr = new XMLHttpRequest();

xhr.open('GET', 'https://api.example.com/data', true); // true for async
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
        console.log(JSON.parse(xhr.responseText));
    }
};
xhr.send();
```

使用场景：

* 对 **上传进度 / 下载进度** 有需求（目前`Promise`不支持）；
* 需要对请求的每一个状态（readyState）做精细控制。

#### 👦 Fetch

```js
fetch('https://api.example.com/data')
    .then(response => {
        if (!response.ok) {
            throw new Error('internet error');
        }
        return response.json();
    })
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.error('error', error);
    });
```

两者对比：

| 特性              | XHR                                          | Fetch<sup>new</sup>                |
| ----------------- | -------------------------------------------- | ---------------------------------- |
| API风格           | 回掉函数                                     | Promise                            |
| 监控**请求**进度  | ✅                                            | ❌                                  |
| 监控**响应**进度  | ✅                                            | ✅                                  |
| Service Worker    | ❌                                            | ✅                                  |
| Cookie<u>控制</u> | ❌（默认携带，`withCredentials`控制跨域携带） | ✅（`credentials`控制`cookie`携带） |
| 自定义Referrer    | ❌                                            | ✅                                  |
| 流式处理          | ❌                                            | ✅（`ReadableStream`）              |
| 请求取消          | ✅                                            | ✅（`AbortController`）             |

1. XSS、CSRF
2. mouseenter/mouseover
3. for in, for of, map, forEach, substring, substr（DEPRECATED）

4. isEmpty(obj: object)

5. add...
6. 浏览器API



## CSS

