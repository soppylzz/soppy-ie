---
title: 前端笔记：Package.json专题<sup>(1)</sup>
date: 2025-12-30 13:26:26
tags:
---

> 你是我在见过最美丽的赛博女孩😘！部分内容由GPT生成

{% note color:blue 前言 最近想深入学习`Vue 3.4`源码，便打算用`VitePress`搭建一个笔记网站。但现有的主题都不太合心意，于是决定自己动手，写一个`powerlevel10k`风格的主题。说来也奇怪，我深入了解`package.json`配置的契机竟然是这个项目。%}

## 前言

*JavaScript* 最早只运行在浏览器中，没有模块系统，也没有统一的依赖管理方式。2009年 *Node.js* 的出现，让 *JavaScript* 可以脱离浏览器，在服务器和本地环境中运行，这使得 *JavaScript* 开始承担起**构建**工具、**后端**服务和**命令行**程序等角色。随着代码规模变大，*Node* 生态迫切需要一种标准方式来复用和分发代码。

在这样的背景下，`npm`诞生了。`npm`既是包管理器，也是一个集中式的包仓库，用来解决“如何发布、安装和管理 *JavaScript* 包”的问题。为了让`npm`能理解一个项目或一个包是什么、依赖哪些库、版本如何管理，<b><code>npm</code>设计并推广了`package.json`作为包的描述文件。</b>每一个被`npm`管理的项目，本质上都是通过`package.json`来声明自身信息和依赖关系。而后续的包管理工具、*JS* 运行时也根据前辈的经验选择也使用`package.json`作为描述文件。

由于`package.json`规范源自 *Node.js*，建议结合其官方文档进行系统学习：[Node.js Packages](https://nodejs.org/docs/latest/api/packages.html)。

## Exports

### 1. 包导入机制演变

在了解`exports`的内部配置细节前，我们先借助GPT了解下 *Node.js* 包导入机制演变历史。

#### ☁️ CommonJS 

{% hashtag  <code>v0.1.0</code>~<code>v8.x</code>  %} {% hashtag 2009-2017  %}

*Node.js* 诞生之初，采用了 *CommonJS* 作为模块系统。模块通过`require`加载，导出通过`module.exports`暴露。包入口由`package.json`中的`main`字段指定，若缺失则回退到`index.js`。

这一阶段的模块解析规则高度宽松：只要文件在包目录中真实存在，就可以被外部直接访问。这种设计降低了使用门槛，但也隐含了一个长期问题——**包的内部实现结构被默认视为公共 API**。随着`npm`生态的快速膨胀，这种“无边界”的导入方式逐渐成为库作者难以维护<u>兼容性</u>的根源。

#### ⛅️ 构建工具推动的 ESM 入口分裂

{% hashtag  <code>v8.x</code>~<code>v11.x</code>  %} {% hashtag 2017–2019  %}

随着前端工程化的发展，*Rollup*、*Webpack* 等构建工具开始大量使用 **ES Module** 语法，并引入了`module`字段作为 *ESM* 入口的约定。由此，一个包往往同时存在：

- `main`：面向 *Node* 的 *CommonJS*
- `module`：面向打包工具的 *ES Module*

但这一阶段的规范并未被 *Node.js* 官方采纳，属于<u>事实标准</u>。包的导出边界依旧不受控制，深层路径依然可以被随意引用，导入行为在不同工具间也缺乏一致性。

#### 🌤️ Node.js原生ESM

{% hashtag  <code>v12.x</code>  %} {% hashtag 2019–2020  %}

*Node.js* 在`v12+`版本中逐步引入原生 *ES Module* 支持，并开始对模块解析语义进行收紧。此时，*Node* 官方明确了一个目标：**统一模块解析规则，并在运行时层面而非工具层面解决包边界问题**。

`package.json`的`type`字段、`.mjs`/`.cjs`扩展名规则相继出现，模块系统首次在 *Node* 核心层面区分 *ESM* 与 *CJS*。但入口问题与包内部路径暴露的问题仍然没有得到根本解决。

#### ☀️ `exports`的引入

{% hashtag  <code>v12.7.0</code>→<code>v14.x</code>→<code>v16.x</code>  %} {% hashtag 2020→2021→now  %}

`exports`字段的引入，标志着 *Node.js* 包导入机制进入一个**显式接口声明**的阶段。与`main`不同，`exports`不再只是“默认入口”，而是一个**白名单系统**：

- 只有在`exports`中声明的路径才允许被导入
- 支持根据运行环境进行条件选择（*import / require / browser*）
- 模块解析在执行前完成，具备确定性

至此，*Node.js* 官方首次在规范层面解决了「包如何安全地对外暴露 *API*」这一长期问题，也为 *TypeScript* 与现代构建工具提供了统一的语义基础。

### 2. 配置说明

关于`exports`的配置说明可以参考：[Conditional exports](https://nodejs.org/docs/latest/api/packages.html#conditional-exports)。这里我只简单介绍`exports`的关键要点。

#### 🚶 `package.json`路径规范

*Node.js* 关于路径有一套独特的匹配与验证方式，具体可以参考如下代码：

```json
{
  "name": "my-package",
  "exports": {
    /* Must be relative URLs */
    ".": "./dist/main.js",          			// Correct
    "./feature": "./lib/feature.js", 			// Correct
    // "./origin-relative": "/dist/main.js", 	// Incorrect: Must start with ./
    // "./absolute": "file:///dev/null", 		// Incorrect: Must start with ./
    // "./outside": "../common/util.js" 		// Incorrect: Must start with ./
    
    /* No path traversal */
    // ".": "./dist/../../elsewhere/file.js", 	// Invalid: path traversal
    // ".": "././dist/main.js",             	// Invalid: contains "." segment
    // ".": "./dist/../dist/main.js",       	// Invalid: contains ".." segment
    // "./utils/./helper.js": "./utils/helper.js" 	// Key has invalid segment
    
    /* Subpath patterns */
    "./features/*.js": "./src/features/*.js"
    // import featureX from 'my-package/features/x.js';
    // import featureX from 'my-package/features/y/y.js';
    // (emit "imports config"...) 
    
    /* ERR_PACKAGE_PATH_NOT_EXPORTED */
    "./features/private-internal/*": null
  }
}
```

```js
/* Subpath patterns */
import featureX from 'my-package/features/x.js';
import featureX from 'my-package/features/y/y.js';
// (emit "imports config"...)

/* ERR_PACKAGE_PATH_NOT_EXPORTED */
import featureInternal from 'my-package/features/private-internal/m.js';
// Throws: ERR_PACKAGE_PATH_NOT_EXPORTED
```

上述代码要点可以总结为：`*[.ext]`通配符可以表示`**/*[.ext]`；`null`用于控制导出黑名单；

#### 🌟 条件导出

> 这里我就直接贴上 *Node.js* 的官方文档；

*Node.js* 支持如下条件导出配置：

**⭐️ `"node-addons"`**
当包被加载于支持原生插件（C++ Addons）的 *Node.js* 环境时匹配。可用于提供依赖原生 C++ 插件的专用入口点；如果通过`--no-addons`参数启动 *Node.js*，则此条件会被禁用。这个条件比`"node"`更具体，适用于需要利用 *Node* 原生扩展的场景。

**⭐️ `"node"`**
匹配**任何 Node.js 环境**，无论代码是通过 `require()`、`import` 还是 `import()` 加载；目标文件可以是 *CommonJS* 或 *ES* 模块。适合作为通用的 Node 平台实现条件。

**⭐️ `"import"`**
当包**通过 ES 模块语法`import`或动态`import()`调用加载时**匹配。此条件与`"require"`条件互斥，优先于不区分加载方式的条件（如`"node"`）。无论目标文件是什么模块格式，只要是通过 ES 模块加载流程触发，就会匹配此条件。

**⭐️ `"require"`**
当包**通过 CommonJS 的同步加载机制`require()`加载时**匹配。此条件也与`"import"`互斥，目标路径应当是`require()`可识别的模块格式（例如 *CommonJS*、*JSON*、原生插件等），但不包括不能用`require()`加载的 ES 模块。

**⭐️ `"module-sync"`**
无论包是通过`import`、`import()`还是`require()`加载，都会匹配该条件；适用于路径指向的模块是 **不含顶层 Await 的 ES 模块**。若模块图中存在**顶层`await`**，则在使用`require()`时将抛出 `ERR_REQUIRE_ASYNC_MODULE`错误。这个条件用于统一同步加载行为，方便在兼容 *ESM* 与 *CommonJS* 时减少差异。

**⭐️ `"default"`**
通用回退条件**始终匹配**，适用于没有其它条件匹配时的兜底实现；一般应放在所有条件分支的最后，以避免被更具体的条件覆盖。可用于提供跨环境的通用路径，确保在未知或非特定环境下依然能被正确解析。

**🔹 `"types"`**
指向包的 <u>*TypeScript*</u> 类型声明文件（`.d.ts`），用于告诉 *TypeScript* 编译器该包的类型信息。

在`“exports”`对象中，配置项顺序非常重要。在条件匹配过程中，较早的条目优先级更高，优先于后面的条目。*一般规则是条件应按对象顺序从最具体到最不具体。*
