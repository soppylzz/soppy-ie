---
title: 面试合集：基础知识<sup>(1)</sup>
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

```js
const arr = [10, 20, 30];

for (const k in arr) {
  console.log(k);      // '0' '1' '2'
  console.log(arr[k]); // 10 20 30
}

for (const v of arr) {
  console.log(v); // 10 20 30
}
```

1. isEmpty(obj: object)

2. add...
3. 浏览器API

## CSS

### 1. [有哪些 CSS 选择器？请分别介绍](https://www.mianshiya.com/bank/1810644318667538433/question/1810655443832311810)

> | 选择器类型        | 示例写法                                                     | 权重格式 |
> | :---------------- | :----------------------------------------------------------- | :------- |
> | 通用选择器        | `*`                                                          | (0,0,0)  |
> | 标签 / 元素选择器 | `div`                                                        | (0,0,1)  |
> | 伪元素选择器      | `::before`、`::after`                                        | (0,0,1)  |
> | 类选择器          | `.cls`                                                       | (0,1,0)  |
> | 属性选择器        | `[type="text"]`、`[class*="wrap"]`（支持`$` / `*` / `^`通配符） | (0,1,0)  |
> | 伪类选择器        | `:hover`                                                     | (0,1,0)  |
> | ID 选择器         | `#app`                                                       | (1,0,0)  |
>
> 这里说明一下：伪元素支持`:`（CSS2）与`::`（CSS3）两种书写语法；

此外还有如下特殊规则：

1. 行内样式：
   * 写法：`<div style="color: red"></div>`；
   * 权重：权重高于普通选择器，低于`!important`；
2. `!important`：权重高于所以非`!important`样式；
3. `@layer`层级优先级：
   * 可通过显式声明`@layer import, base`自定义层级先后；也可通过 HTML 引入顺序**隐式**决定层级优先级。
   * 所有`@layer`内部样式，优先级全部低于无层级的普通 CSS 样式

### 2. [ 利用 CSS 隐藏元素的方法有哪些？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655446143373314)

> | 隐藏方式                              | 核心描述                                               |
> | :------------------------------------ | :----------------------------------------------------- |
> | `display: none;`                      | 脱离文档流，不占位、不渲染、不可交互，读屏器忽略       |
> | `visibility: hidden;`                 | 仅视觉隐藏，保留页面占位，不可点击                     |
> | `opacity: 0;`                         | 完全透明，保留占位和布局，仍可点击触发事件             |
> | `position: absolute; left:-9999px;`   | 绝对定位移出视口，不占常规文档流，多用于无障碍隐藏文本 |
> | `clip-path: inset(100%);`             | 裁剪视觉区域实现隐藏，保留占位，无交互                 |
> | `width:0; height:0; overflow:hidden;` | 压缩尺寸 + 溢出隐藏，不占用页面空间，兼容旧浏览器      |
>
> 在选择隐藏方法时，我们需要考虑元素是否脱离 **文档流** | `normal flow` 的问题；

### 3. [说说你对 line-height 的理解及其赋值方式](https://www.mianshiya.com/bank/1810644318667538433/question/1810655447552659458)

> `line-height`可以设置**数字值**、**长度值**、**关键字** （`normal`，浏览器根据默认算法来计算行高）；
>
> 这里主要说明一下`1`与`1em`的区别，`em`在浏览器中会被替换为等价的`px`，子类元素继承的不是**原始表达式**，而是**计算值**；
>
> ```html
> <!DOCTYPE html>
> <html lang="en">
> <head>
>   <meta charset="UTF-8">
>   <title>Title</title>
> </head>
> <style>
>   .line { line-height: 1em; font-size: 20px; };
>   .line span { font-size: 10px };
> </style>
> <body
> <div class="line">LINE<span>HEIGHT</span><!-- still line-height: 20px; --></div>
> </body>
> </html>
> ```

### 4. [CSS 中可继承与不可继承属性有哪些？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655445124157441)

> 可继承属性初始化用`inherit`；不可继承属性初始化用`initial`；`unset`都适用；
>
> 当我们想回退浏览器默认样式时（非`W3C`默认样式）可以使用`revert`；

#### 1️⃣ 可继承属性

* 字体相关：`font-*`等；
* 文本相关：`color`、`letter-spacing`、`line-height`、`direction`、`text-align`、`text-transform`、`visibility `、`white-space`、`word-spacing`、`text-indent`、`text-shadow`、`quotes`等；
* 其他：`cursor`、`opacity`（特殊继承机制，可部分继承）；

#### 2️⃣ 不可继承属性

* 盒模型：`margin`、`padding`、`border-*`、`width`、`height`等；
* 布局相关：`display`、`position`、`inset`、`z-index`、`float`、`overflow`、`clear`等；
* 其他：`background-*`、`clip`、`list-style`、`content`、`outline`等；

### 5. [ CSS 优化和提高性能的方法有哪些？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655447636545537)

> 1. **合并和最小化 CSS 文件**：减少文件加载时间；
> 2. **减少高级选择器使用**：减少浏览器解析；
> 3. **使用 CSS Sprite**：减少图片加载次数；
> 4. **动画调优**：`will-change`优化`transform`动画，避免复杂动画；
> 5. **引入位置**：在 HTML 的`head`中提前引入样式文件；

### 6. [display:inline-block 什么时候会显示间隙？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655447972089857)

> `inline-block`元素虽然呈现为块级元素，但在排版行为上仍保留了一部分行内元素的特性。行内元素之间的空格字符（换行、空格）会被渲染为一个普通空格，导致视觉上出现间隙；
>
> 解决方案：
>
> 1. 移除空格、或者用 HTML 注释代替换行；
> 2. 设置负边距修正；
> 3. 设置 Container 的`font-size: 0px;`；

### 7. [怎么实现元素的水平垂直居中？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655448987111426)

> 1. Flex布局下设置`align-items`、`justify-content`；
> 2. Grid布局下设置`place-items`；
> 3. 绝对定位下设置`left:50; top:50%; transform: translate(-50%, -50%);`；
> 4. 设置`display: table;`与`display: table-cell;`；

### 8. [为什么需要清除浮动？清除浮动的方式有哪些？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655449364598786)

> **浮动工作原理**：
>
> 1. 浮动元素**脱离标准文档流**，不再占用原位置空间，普通块级元素会自动填补其空位；
> 2. 浮动元素沿水平方向左右排布，直至触碰父容器边界或其他已浮动元素；
> 3. 浮动不改变块级元素布局位置，**仅环绕影响文本、行内元素**的排列流向；
>
> **浮动造成的问题**：
>
> 1. 当父容器中只有浮动子元素会**高度塌陷**；
> 2. 非浮动元素跟随显示：如果一个元素没有设置浮动，而前面有浮动元素，则该元素会尝试绕过浮动元素显示，这可能导致意料之外的布局错位；
> 3. 如果第一个元素未浮动，而后续元素浮动，可能引起布局混乱，通常建议统一浮动方向或全部浮动。
>
> **解决方法**：
>
> 1. 在所有浮动元素之后插入一个空`<div>`并设置`clear:both;`；
> 2. 给父容器设置`overflow:hidden`或`overflow:auto`，触发 *块级格式化上下文*（`BFC`），自动计算高度；
> 3. 通过`::after`伪元素创建一个隐藏的块级元素，并设置`clear:both`，同时用`zoom:1`触发 IE6/7 的 hasLayout。
> 4. Flex/Grid布局代替浮动；

#### 🚩 BFC

`BFC`全称块级格式化上下文（`Block Formatting Context`），是一个独立的渲染区域，使其内部子元素不受外部元素影响；有助于解决 ***浮动***、***边距重叠*** 等问题；创建方法有：

1. `float:left | right`；
2. `position: absolute | fixed`；
3. `display: inline-block | table-cell`等；
4. 设置`overflow`；

### 9. [说说网页元素的层叠顺序](https://www.mianshiya.com/bank/1810644318667538433/question/1810655449746280449)

> 层叠上下文；

### 10. [CSS 的 position 有哪些属性值，区别是什么？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655449830166530)

> 包含块；
>
> `display`、`float`、`position`关系；
>
> `absolute`-`relative`
>
> `absolute`-`fixed`：包含块，transform元素；

### 11. [ flex-shrink 和 flex-grow 的默认值是多少？作用是什么？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655451117817858)

> shrink默认1，空间不足所需；grow默认0，当为1时，空间充足放大；
>
> bias初始大小（auto，size）；
>
> flex: grow shrink bias;
>
> flex: 1 => 1 1 0% （0用于自动平分）；
>
> flex: 0 | none => 0 0 auto
>
> Flex: auto => 1 1 auto

### 12. [字体设置解读](https://www.mianshiya.com/bank/1810644318667538433/question/1810655451302367234)

> font-family: system-ui,-apple-system,BlinkMacSystemFont,segoe ui,Roboto, Helvetica,Arial, sans-serif 

### 13. [ 如何检测 CSS 动画的 FPS 值？](https://www.mianshiya.com/bank/1810644318667538433/question/1810655451495305217)

> rAF、PerformanceOberserver

## HTML

### 1. [HTML 中，b 与 strong 标签的区别是什么？](https://www.mianshiya.com/bank/1810644197003362306/question/1810654508913569793)

> b-strong，i-em，是否带有语义信息；可以自定义区别（一般带有语义信息的表示特别强调）；

### 2. [ DOCTYPE（文档类型）的作用是什么？](https://www.mianshiya.com/bank/1810644197003362306/question/1810654507286179841)

> 文档解析类型；h5为`<!DOCTYPE html>`

### 3. [HTML 的 script 标签中 defer 和 async 有什么区别？](https://www.mianshiya.com/bank/1810644197003362306/question/1810654507466534913)

> Async：下载后立即执行；
>
> Defer：等HTML全部解析后执行；

### 4. [ 常用的 HTML meta 标签有哪些？](https://www.mianshiya.com/bank/1810644197003362306/question/1810654507630112770)

> Keywords、Description：网页显示
>
> Og(open graph)

### 5. [HTML5 相比于 HTML 有哪些更新？](https://www.mianshiya.com/bank/1810644197003362306/question/1810654507818856449)

> 语义话标签；媒体标签；
>
> 表单增强；WS；SVG直接嵌入；

### 6. [ HTML5 的离线储存怎么使用？它的工作原理是什么？](https://www.mianshiya.com/bank/1810644197003362306/question/1810654508355727362)

