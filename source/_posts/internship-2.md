---
title: 面试合集：基础知识<sup>(2)</sup>
date: 2026-05-14 11:45:26
tags:
---

## ES6

### 1. [ES6 箭头函数和普通函数有什么区别？](https://www.mianshiya.com/bank/1810644716832817154/question/1810656913092780034)

> `this`问题；不能new；
>
> 普通function只能new的方法 => （`this instanceof ConstructorFn`）| （`new.target`），对应ES6的Class
>
> obj.func中的this表示obj，func中的this表示构造实例对象；obj.arrow中的this表示obj所在上下文中的this

### 2. [ ES Module 与 CommonJS 模块方案有什么异同？](https://www.mianshiya.com/bank/1810644716832817154/question/1810656914632089602)

> ESM：同步/异步加载，动态导入；静态导出，支持tree shaking；
>
> CJS：同步加载；动态导出





函数中`this`指向：

1. `method()`：全局作用域
2. `obj.method()`：`obj`
3. `new method()`：创建的对象，因此判别`new`调用的方法有：`this instanceof method`、`!new.target`两种方案；
4. `call`、`apply`、`bind`：`call`、`apply`指向提供的`this`参数，`bind`实现原理一样，不过采用了闭包来实现返回函数的`this`指向修改；

```ts
function bind(ctx) {
	// new call
  function() {
    new 原始函数
  }
  // normal call
  function() {
    原始函数.call()
  }
}
```

this指向根据使用位置确定，而不是调用位置，这是因为他是基于函数闭包实现，而闭包属于**词法作用域**，在**编译时**确定；



type interface区别：

1. extends｜&
2. 对象｜全部
3. Union支持（"a" | "b"），interface 实现一般要用keyof Inter
4. 声明合并问题，interface支持拓展；



闭包是 JavaScript 中一个核心概念，指的是**函数能够记住并访问其<u>词法作用域</u>，即使该函数在其词法作用域之外执行**。

更通俗地说：当一个内部函数被保存到外部，它依然可以访问当初定义它时的外部函数中的变量，这些变量并没有随外部函数执行完毕而销毁。

