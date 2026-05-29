---
title: 面试合集：进阶知识
date: 2026-05-14 11:47:31
tags:
---

## 底层知识

### 1. [Node.js 中的事件循环和浏览器中的事件循环有什么异同？](https://www.mianshiya.com/question/1843904484648972289)

> Node处理服务端操作：I/O、网络
>
> Web处理：交互、DOM、网络

#### ⭕️ 浏览器事件循环X

https://juejin.cn/post/7520429827620519988

1. 执行同步代码（调用栈清空为止，只<u>执行一次</u>）；
2. 清空当前所有「微任务」队列，**一次性全部执行完**全部微任务；
   * `Promise.then/catch/finally`、`async/await`语法糖；
   * `queueMicrotask()`；
   * `MutationObserver`；
3. <span style="color: red;">**浏览器执行渲染流程**</span>；
4. 取出一个「宏任务」执行，执行完后回到第 2 步；
   * `setTimeout`、`setInterval`
   * `AJAX`（异步）

#### 🌟 浏览器渲染管线

1. DOM树：渲染与事件循环是同步进行的，在解析DOM树时就会实时解析非`async`、`defer`的`<script>`标签；
2. CSSOM树：CSS会阻塞渲染。浏览器必须等待CSSOM构建完成才能进行后续步骤，以避免出现“无样式内容闪烁”（FOUC）。
3. Render树：合并上树，这棵树只包含需要在屏幕上显示的**可见节点**及其计算后的样式；
4. 布局(Layout/Reflow)
5. 绘制(Paint)

## 工程化

### 1. [ semver 规范中，~1.2.3 与 ^1.2.3 的版本号范围分别是什么？](https://www.mianshiya.com/bank/1831174767836995585/question/1831172726862131202)

> `~1.2.3`：`>=1.2.3 && <1.3.0`；
>
> `^1.2.3`：`>=1.2.3 && <2.0.0`；
