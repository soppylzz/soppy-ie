---
title: 前端笔记：CSS Object类型标注库
date: 2025-12-23 12:31:44
tags:
---

> 以下是我过往开发`CssInJs`工具库时整理的笔记，已通过 GPT 优化排版与表述

在 ***CSS-in-JS*** 场景中，样式不再是字符串，而是 *JavaScript* / *TypeScript* 对象。这一转变带来的直接问题是：**如何为 *CSS Object* 提供可靠、可维护的类型约束**。

`csstype`正是一个可以解决这一问题的基础类型库，几乎所有主流的 *CSS-in-JS* 方案都在不同程度上依赖它。本文将简单介绍其使用方法。

## 基础使用

### 1. 安装与依赖策略

`csstype`仅提供类型定义，不包含任何运行时代码，因此在不同项目中可以采用不同的安装策略：

```shell
npm install csstype -D     # 仅业务项目使用，不需要将 d.ts 打包
pnpm add csstype           # 通用组件库/工具库，直接作为依赖
```

### 2. 最小使用示例

`csstype`的核心能力是为 *CSS Object* 提供完整的属性与取值约束：

```typescript
import type * as CSS from 'csstype';

const style: CSS.Properties = {
  color: 'crimson',
};
```

在这个例子中，`CSS.Properties`已经覆盖了绝大多数日常 *CSS* 使用场景，并能提供完整的 *IDE* 自动补全与类型校验。

## 类型体系

`csstype`的设计并非只有一个简单的`Properties`接口，而是围绕 ***CSS* 属性来源、书写形式、取值能力** 构建了一套较为完整的类型标注范式。

### 1. Style 类型

从属性来源的角度，`csstype`将 *CSS* 属性划分为五类：

- **All**：完整集合，包含标准属性、厂商前缀属性、已废弃属性以及 *SVG* 专用属性
- **Standard**：当前规范中的标准 *CSS* 属性（包含 Longhand / Shorthand）
- **Vendor**：带有浏览器厂商前缀的属性
- **Obsolete**：已废弃或移除的属性
- **Svg**：仅在 SVG 场景中生效的属性

需要注意的是，`Standard`、`Vendor`等并不是独立体系，而是对 **All 的不同子集拆分**。

在类型层面，这种拆分是通过统一的 ***Variations*** 模板生成：

| All             | Standard                | Vendor                | Obsolete                | Svg                |
| --------------- | ----------------------- | --------------------- | ----------------------- | ------------------ |
| `${Variations}` | `Standard${Variations}` | `Vendor${Variations}` | `Obsolete${Variations}` | `Svg${Variations}` |

换句话说，`CSS.Properties`、`CSS.StandardProperties`、`CSS.VendorProperties`在结构上是同构的，只是属性集合不同。

### 2. At-rule 类型

除了普通的样式属性，*CSS* 还包含大量`@`规则，例如`@media`、`@keyframes`等。
`csstype`对这些规则的处理方式非常克制：**仅对 *key* 本身进行类型约束，而不介入其内部结构**。

源码中可以清楚地看到这一点：

```typescript
export type AtRules =
  | '@charset'
  | '@counter-style'
  | '@document'
  | '@font-face'
  | '@font-feature-values'
  | '@font-palette-values'
  | '@import'
  | '@keyframes'     // CSS-in-JS 中通常需要额外处理
  | '@layer'
  | '@media'
  | '@namespace'
  | '@page'
  | '@property'
  | '@scope'
  | '@scroll-timeline'
  | '@starting-style'
  | '@supports'
  | '@viewport';
```

这也解释了为什么在 *CSS-in-JS* 框架中，`@keyframes`往往需要单独的 *API* 或 *DSL* 来建模，而不是直接复用`csstype`。

### 3. Pseudo 类型

在 *CSS Object* 场景下，伪类和伪元素本质上是 **嵌套的 key**，而不是普通属性。`csstype`同样只负责 *key* 的合法性校验。

类型体系分为三层：

- **Pseudos**：所有伪类与伪元素的联合类型
- **AdvancedPseudos**：函数式伪类（如`:not()`、`:nth-child()`）
- **SimplePseudos**：不需要参数的简单伪类（如`:hover`、`:focus`）

源码示意如下：

```typescript
export type Pseudos = AdvancedPseudos | SimplePseudos;
```

在实际工程中，如果我们希望对伪类对象本身进行类型约束，可以这样进行类型标注：

```typescript
import type * as CSS from 'csstype';

export type CSSPseudosObject = {
  [K in CSS.Pseudos]?: CSS.Properties;
};

const pseudos: CSSPseudosObject = {
  ':hover': {
    color: 'crimson',
  },
};
```

### 4. Variations

`csstype`中最容易被忽略、但在工程中非常关键的概念是 ***Variations***。它描述了两个维度的差异：

- 属性名的书写形式
- 属性值是否支持 **回退** | `fallback`

具体来说：

- **Default**：使用 camelCase（JS 风格）
- **Hyphen**：使用 kebab-case（CSS 原生风格）
- **Fallback**：属性值允许是字符串或字符串数组
- **HyphenFallback**：同时支持 kebab-case 与回退值

例如：

```typescript
// camelCase
const styles1: CSS.Properties = {
  backgroundColor: 'crimson',
};

// kebab-case
const styles2: CSS.PropertiesHyphen = {
  'background-color': 'crimson',
};

```

回退值在 *CSS* 中本身就是合法且常见的写法：

```css
background-image:
  url(image1.png),
  url(image2.png),
  linear-gradient(#fff, #ccc);
```

在 *TypeScript* 中可以通过`PropertiesFallback`表达：

```typescript
const styles: CSS.PropertiesFallback = {
  backgroundImage: [
    'url(image1.png)',
    'url(image2.png)',
    'linear-gradient(#fff, #ccc)',
  ],
};
```

四种 *Variation* 对应关系如下：

| Style 类型                     | Key 类型               | Value 类型   |
| ------------------------------ | ---------------------- | ------------ |
| `CSS.Properties`               | camelCase（`Default`） | Non-fallback |
| `CSS.PropertiesHyphen`         | kebab-case（`Hyphen`） | Non-fallback |
| `CSS.PropertiesFallback`       | camelCase              | Fallback     |
| `CSS.PropertiesHyphenFallback` | kebab-case             | Fallback     |

### 5. Generics

`csstype`中几乎所有核心接口都暴露了两个泛型参数：

```typescript
CSS.Properties<TLength = string | 0, TTime = string>
```

它们分别用于约束：

- **长度类属性**（如`width`、`height`）
- **时间类属性**（如`transitionDuration`）

例如：

```typescript
import type * as CSS from 'csstype';

const styles: CSS.Properties<string | number, number> = {
  width: 100,
  height: '100px',
  transitionDuration: 1000,
};
```

这一设计为强类型样式系统（*Design Token* / *Typed Theme*）提供了良好的扩展点。

## 工程实践中的常见模式

### 1. 使用模块增强扩展 Properties

在真实项目中，`csstype`的默认定义往往无法完全覆盖业务需求，例如：

- 私有实验属性
- CSS 自定义属性（*CSS Variables*）
- 主题命名空间变量

这时可以通过 ***TypeScript* 模块增强** 进行扩展：

```typescript
import type * as CSS from 'csstype';

declare module 'csstype' {
  interface Properties {
    WebkitRocketLauncher?: string;

    '--theme-color'?: 'black' | 'white';

    [index: `--theme-${string}`]: any;
    [index: `--${string}`]: any;

    [index: string]: any;
  }
}
```

这种方式的优点是 **无侵入、可渐进增强**，非常适合业务项目。

### 2. 重新定义类型而非强行增强泛型接口

需要特别注意的是：`csstype`的核心接口大量使用了 **泛型参数**，而 *TypeScript* 对「带泛型接口的模块增强」支持非常有限。在实际工程中，如果我们需要：

- 修改泛型约束
- 引入更严格的值类型
- 深度定制 *CSS Object* 结构

通常更合理的做法是：**基于`CSS.Properties`重新封装一层业务级类型**，而不是试图通过`declare module 'csstype'`去「<u>改写</u>」原始定义。这也是多数成熟 *CSS-in-JS* 框架的实际选择。

