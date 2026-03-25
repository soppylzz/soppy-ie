---
title: 前端笔记：重拾Typescript<sup>(1)</sup>
date: 2025-12-18 15:32:42
tags:
banner: https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20251218160423519.png
poster:
  headline: 前端笔记：重拾Typescript
---

{% note color:blue 感想 研究生开题的事情暂时告一段落，我终于重新回到博客，开始记录一些前端相关的内容。写下这些文字时，心里其实有些复杂，却又说不太清楚。<br/><br/>我第一次接触前端，大概是在 2023 年的某个时候。回头看这几年的经历，本以为会有很多话想说，真正落笔时，却只剩下一种模糊的回望。那时用 <code>TS</code> 搭建项目，更多是在“用”，而不是在“理解”。我很少追问它们为何如此工作，更像是借助所谓的 <code>Vibe Coding</code> 和现成工具，拼凑出一些看似可用、却并不牢靠的东西。项目跑起来了，但心里始终明白，它们并不结实。<br/><br/>临近找工作的节点，我反而生出一种重新来过的冲动。借助 <code>AI</code>，也翻看网络上的博客，我想把过去那些仓促搭起的“豆腐渣工程”一点点拆掉，再慢慢重建。这个过程谈不上轻松，但至少是诚实的。<br/><br/>好在研究生阶段的生活，为我留出了这样的时间和余地。此刻的我，更像是一个带着当下记忆的穿越者，试图回到 2023 年的那个夏天。不是为了推翻过去的自己，而是想补上当初没来得及在意的细节，安放一个代码洁癖式的、偏执又温和的极客遗憾。 %}

## 前言

在开始学习 TypeScript 的具体语法之前，更有必要先明确一个问题：**TypeScript 在真实项目中究竟扮演了什么角色**。如果仅将它理解为“JavaScript 的超集”，学习过程很容易停留在语法层面，而忽略其工程价值。

从使用方式上看，TypeScript 并不会改变 JavaScript 的运行时行为。我们依然在编写 JavaScript，只是代码在进入运行环境之前，会先经过一次静态分析与编译过程：

```
TypeScript → tsc → JavaScript → running
```

因此，TypeScript 并不是运行时框架，而是一套服务于开发阶段的工具体系。它同时承担了静态分析、代码约束以及工程辅助等职责。

如果用一句话概括，可以将 TypeScript 理解为：

> **JavaScript 的高级 Linter 与类型封装工具。**

**🔹 像 Linter 一样提前发现问题**

传统 Linter（如 ESLint）主要关注<u>语法与代码风格</u>，而 TypeScript 关注的是更深层次的结构问题，例如类型不匹配、参数误用以及潜在的空值风险。这些问题能够在代码尚未运行之前被发现，而不是依赖运行时报错或开发者经验。

从作用上看，这与 C、Go 等静态类型语言中的编译器类似，但 TypeScript 的约束强度介于传统静态语言与动态语言之间，在工程实践中形成了一种折中的平衡。

**🔹 用类型描述代码意图**

TypeScript 允许开发者显式描述数据结构与函数使用方式：

```typescript
function getUser(id: number): User { ... }
```

类型信息并不仅是为了通过编译，更重要的是表达代码的使用意图：函数需要什么参数、返回什么结果，以及应当如何被正确调用。TypeScript 的价值并不在于限制写法，而在于降低理解成本，使代码对使用者更加友好。

**🔹 TypeScript 的典型应用场景**

并非所有项目都适合引入 TypeScript。当业务简单、数据结构扁平、项目周期较短且协作成本较低时，类型定义与维护的成本可能会超过其带来的收益。

但在以下两类场景中，TypeScript 的优势会被显著放大。

* **工具库或公共组件的开发**

  当代码的主要使用者是其他开发者时，理想状态是无需频繁查阅文档，仅通过 IDE 的补全和类型提示即可理解 API 的使用方式。在这种情况下，类型本身就是最准确的文档。相比 JSDoc，TypeScript 提供了更完整且可组合的类型表达能力。

* **业务复杂的大型项目**

  随着系统规模增长，问题往往不再是代码是否可运行，而是数据是否被正确理解和使用。接口字段被误解、重构遗漏边界条件等问题，本质上源于开发者的主观假设与真实数据之间的偏差。TypeScript 的类型系统无法消除所有错误，但可以显著缩小这类偏差出现的空间，从而降低复杂业务中的维护成本。

## 编译流程

计算机是一门实践型的学科。在系统学习 TypeScript 之前，不妨先配置一个实验环境，通过动手实践来加深理解。在实验环境中，我们主要探索两种 TypeScript 编译方案，既能方便查看原始 JS 输出，又能体验现代构建工具的高效处理：

* 纯`tsc`编译
* 使用`esbuild` + 类型检查的组合编译

这里我们先不妨执行如下命令安装后续会用到的工具：

```bash
npm i -D typescript esbuild ts-morph rimraf
```

### 1.  npx

在 Node.js 生态中，我们通常使用`npm`来安装和管理依赖。随着前端工具链不断扩展，CLI 工具的数量迅速增加，`npx`（**N**ode **P**ackage e**X**ecutor）作为`npm 5.2.0`（2017 年）引入的内置命令行工具，提供了一种更轻量的方式来**执行 *npm* 包，而无需全局安装**。

它解决的核心问题可以概括为三点：

* 临时执行 *npm* 包，不污染全局环境
* 优先使用项目本地已安装的版本
* 非常适合一次性命令（脚手架、构建工具等）

通俗地说，如果你只是“想用一下”某个 *CLI*，而不希望它长期存在于全局或项目依赖中，`npx`是一个几乎零成本的选择。需要补充的是：从`npm 7`（2020 年） 开始，官方逐步将推荐方式迁移到`npm exec`。`npx`更多被视为一种**历史产物 + 兼容入口**，目前仍然可用，但不再是首选写法。

#### 🧐 为什么需要`npx`

假设我们要使用`create-react-app`创建一个新项目，传统做法为：

```
npm install -g create-react-app
create-react-app my-app
```

这种方式的问题其实很明显：

1. 全局安装会长期占用磁盘空间
2. 版本容易过时，需要手动更新
3. 不同项目依赖不同版本时，冲突难以管理

这时我们便可以使用 *npx* 解决上述问题：

```bash
npx create-react-app my-app
```

这里的变化不只是“少打一条命令”。`npx`会在执行时临时下载所需版本，执行完成后不会在全局环境中留下任何痕迹。这种 **“即用即走”** 的模式，正是现代前端 CLI 的理想形态。

#### 👻 执行本地或远程`npm`包

对于一次性执行远程`npm`包（例如用于初始化项目的 CLI 工具），`npx`的优势已经非常直观，这里不再展开。更值得深入理解的是：**`npx`是如何调用本地`node_modules`中的命令行程序的**。

我们通过`npm`安装的很多包，其实都会附带可执行命令。这些命令默认会被放到`node_modules/.bin`目录中。下面是一个实际示例：

```bash
❯ ls -l ./node_modules/.bin
   rwxr-xr-x    6   soppylzz   staff    192 B     Thu Dec 18 19:43:39 2025  ./
   rwxr-xr-x   37   soppylzz   staff      1 KiB   Thu Dec 18 20:15:21 2025  ../
   rwxr-xr-x    1   soppylzz   staff     22 B     Thu Dec 18 19:43:39 2025  esbuild  ⇒ ../esbuild/bin/esbuild
   rwxr-xr-x    1   soppylzz   staff     26 B     Thu Dec 18 19:43:39 2025  rimraf  ⇒ ../rimraf/dist/esm/bin.mjs
   rwxr-xr-x    1   soppylzz   staff     21 B     Thu Dec 18 19:43:39 2025  tsc  ⇒ ../typescript/bin/tsc
   rwxr-xr-x    1   soppylzz   staff     26 B     Thu Dec 18 19:43:39 2025  tsserver  ⇒ ../typescript/bin/tsserver
```

可以看到，`esbuild`、`rimraf`、`tsc` 等第三方包都在`.bin`目录下暴露了对应的命令，而这些文件本身只是 **软连接** | `symbolic link` 。真正的实现代码位于各自的包目录中，例如`node_modules/typescript/bin/tsc`。

当我们执行这些命令时，本质上就是在运行软连接所指向的目标文件。

**🔹 `shebang`与可执行文件**

这些被链接的文件，大多是带有 ***shebang*** 的可执行脚本。例如：

```bash
#!/usr/bin/env node
require('../lib/tsc.js')
```

这行声明告诉系统：使用当前环境中的 Node.js 来执行该文件。因此即使它本质上是一个文本脚本，也可以像普通命令一样直接运行。当然，也存在例外情况。比如`esbuild`，本身是用 Go 编写并编译好的二进制文件。

**🔹 补充**

当我们执行`npx tsc`或`npm exec tsc`时，*npm* 实际上会在当前进程中**临时修改`PATH`**，将`node_modules/.bin`加入到最前面，从而确保解析到的是项目本地版本的命令。

这也解释了一个常见现象：在现代 *IDE* 的集成终端中，很多时候**即使不使用`npx`，也能直接运行本地 CLI**。这是因为 IDE 已经帮你在`tty`会话中预先设置好了`PATH`。通过查看 `echo $PATH`，通常就能看到项目的`node_modules/.bin`目录已经被包含在内。

### 2. tsc编译

#### 🤖 typescript

[TypeScript](https://github.com/microsoft/TypeScript) 是微软发布并维护的开源项目，本质上是一个用 **JavaScript / TypeScript** 实现的 TypeScript 编译器（早期版本使用 *JavaScript* 完成初始构建，随后逐步迁移为使用 *TypeScript* 来编译自身）。我们可以通过`tsc`（**T**ype**S**cript **C**ompiler）直接将 TypeScript 源码编译为标准的 JavaScript。例如：

```bash
npx tsc
```

默认情况下，`tsc`的编译输出不会进行压缩或混淆。这使得生成的 JavaScript 代码具有良好的可读性，能够直观地体现 **TypeScript 是 JavaScript 的超集**，以及编译前后代码之间的对应关系。这一特性在学习 TypeScript 的类型系统、语法特性以及调试编译结果时尤为有价值。

在实际项目或实验性场景中，`tsc`通常会结合`tsconfig.json`一起使用，用于配置 **编译目标** | `target` 、**模块系统** | `module` 以及 **严格模式** | `strict` 等编译选项，从而更精细地控制编译行为。

#### 🐞 编译配置项

TypeScript 官方提供的编译器入口是`tsc`命令。它本质上是对 TypeScript Compiler API 的一层 CLI 封装。在使用`tsc`进行编译时，所有可用的配置项都可以通过命令行查看。最直接的方式是执行：

```bash
tsc --help 			# 查看所有配置项
tsc --help --all 	# 查看完整的配置列表
```

需要注意的是，这里展示的命令行参数并不只是“CLI 专属能力”，它们中的绝大多数，最终都会参与 TypeScript 的编译流程配置，只是以不同的方式被传入编译器。

在理解了`tsc`的基本使用方式之后，下面选取`tsc --help`中**工程实践里最常用、对编译流程影响最大的配置项**，用表格的形式进行整理，便于在阅读编译流程时快速对照理解。

> 表格中带有「sth<sup>\*</sup>」标记的配置项，表示它们**不仅可以通过命令行传入，也可以在项目配置中声明**。

| 分类                                                   | 配置项                                                       | 说明与使用场景                                               |
| ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **项目入口**                                           | `tsc`                                                        | 编译当前工作目录下的项目，默认查找并读取 `tsconfig.json`，是最常见的项目编译方式 |
|                                                        | `--project`, `-p`<sup>\*</sup>                               | 指定某个目录或配置文件作为编译入口，常用于 *monorepo* 或多项目结构 |
|                                                        | `tsc file.ts`                                                | 忽略项目配置，使用默认编译器选项编译指定文件，适合快速验证或临时编译 |
| **构建模式**                                           | `--build`, `-b`<sup>\*</sup>                                 | 用于构建 *composite* 项目及其依赖关系，支持增量编译，常见于大型工程 |
|                                                        | `--watch`, `-w`                                              | 监听文件变化并自动重新编译，主要用于本地开发                 |
|                                                        | **`--noEmit`<sup>\*</sup>**                                  | 只进行类型检查而不输出文件，常用于 *CI*、*lint* 或类型校验阶段 |
| **输出控制**                                           | `--outDir`<sup>\*</sup>                                      | 指定所有编译输出文件的目录，是最常用的输出路径控制方式       |
|                                                        | `--outFile`<sup>\*</sup>                                     | 将所有输出合并为一个文件，通常用于 *UMD* 或非模块化场景      |
|                                                        | `--removeComments`<sup>\*</sup>                              | 在输出结果中移除注释，常用于产物优化                         |
| **声明文件**                                           | **`--declaration`, `-d`<sup>\*</sup>**                       | 生成`file.d.ts`声明文件，**是库开发的基础**                  |
|                                                        | `--declarationMap`<sup>\*</sup>                              | 为<u>声明文件</u>生成 *sourcemap*，方便调试类型定义          |
|                                                        | <span style='white-space: nowrap'>`--emitDeclarationOnly`<sup>\*</sup></span> | 只输出声明文件而不生成 JS，常见于类型包或 API 抽取           |
| <span style='white-space: nowrap'>**SourceMap**</span> | `--sourceMap`<sup>\*</sup>                                   | 为<u>生成的 JavaScript 文件</u>创建 *sourcemap*，用于调试    |
| **目标环境**                                           | **`--target`, `-t`<sup>\*</sup>**                            | 指定输出 JavaScript 的语言版本，影响语法降级与内置能力       |
|                                                        | **`--module`, `-m`<sup>\*</sup>**                            | 指定生成的模块系统，如`CommonJS`、`ESM`、`NodeNext`等        |
|                                                        | **`--lib`<sup>\*</sup>**                                     | 指定<span style='color: crimson'>**运行时环境**</span>的类型声明集合，如`DOM`、`ES`、`WebWorker`等 |
| **JS 支持**                                            | `--allowJs`<sup>\*</sup>                                     | 允许 JavaScript 文件参与编译，常用于 *JS* 向 *TS* 的渐进迁移 |
|                                                        | `--checkJs`<sup>\*</sup>                                     | 对 JavaScript 文件进行类型检查，增强 *JS* 的类型安全         |
| **JSX**                                                | `--jsx`<sup>\*</sup>                                         | 指定 *JSX* 的编译方式，主要用于`React`等前端框架             |
| **类型检查**                                           | **`--strict`<sup>\*</sup>**                                  | 开启一组严格的类型检查规则，强烈建议在工程中启用             |
| **类型注入**                                           | **`--types`<sup>\*</sup>**                                   | **显式指定需要注入的类型包**，常用于 *Node*、测试环境        |
| **模块兼容**                                           | `--esModuleInterop`<sup>\*</sup>                             | 改善`CommonJS`与`ESModule`的互操作体验，*Node* 项目中常见    |

### 2. 代码编译

此外，TypeScript 编译器本身也可以被当作一个普通的库来使用。其入口就是`typescript`这个`npm`包：

```typescript
const ts = require("typescript")
```

当我们在代码中直接使用`typescript`时，实际上是在**手动组织一整套 TypeScript 的编译流程**：

1. 收集待编译的源文件
2. 解析源文件，生成 *抽象语法树*（`AST`）
3. 构建 *编译上下文*（`Program`）
4. 进行符号绑定与类型检查
5. 生成 *诊断信息*（`Diagnostics`）
6. 执行 *代码生成*（`Emit`）

#### 1️⃣ 创建`Program`

我们可以通过如下代码创建一个`Program`实例。`Program`表示一次完整的 TypeScript 编译上下文，它以入口文件为起点，负责加载并解析所有相关源码，构建 **抽象语法树** | `AST` 、符号表以及类型系统，并在此基础上完成类型检查与代码生成等工作。其基本函数签名如下：

```typescript
ts.createProgram(
  rootNames: readonly string[],
  options: ts.CompilerOptions,
  host?: ts.CompilerHost,
  oldProgram?: ts.Program,
  configFileParsingDiagnostics?: readonly ts.Diagnostic[]
): ts.Program
```

各参数含义说明如下： 

* `rootNames`：TypeScript 编译的**入口文件列表**（如`index.ts`）。编译器会从这些文件出发，递归解析所有通过`import`/`export`引用的依赖模块。
* `options`：编译选项配置，等价于`tsconfig.json`中的`compilerOptions`字段，用于控制目标语法版本、模块系统、严格模式等编译行为。
* `host`：[可选] 自定义文件系统和文件读取行为，常用于内存编译或工具链开发，例如`.vue`文件可通过自定义 Host 转换为`SourceFile`。
* `oldProgram`：[可选] 支持增量编译，复用已有语法和类型信息。
* `configFileParsingDiagnostics`：[可选] 用于上报`tsconfig.json`解析阶段产生的诊断信息，通常无需手动传入。

创建`Program`后，整个项目的 *AST*、符号表以及类型系统就已经建立，为后续类型检查、诊断生成和代码转换提供了完整上下文。

值得注意的是，`Program`实例一旦构建完成，其内部的符号表和 *TypeChecker* 会缓存当前 *AST* 的类型信息，如果之后对 *AST* 做修改，*TypeChecker* 并不会自动重新计算类型，因此 *Transform* 的顺序和诊断生成的顺序需要特别关注。

#### 2️⃣ 语法分析

通过`program.getSourceFiles()`可以获取编译上下文中所有被解析的源文件，每个对象都是`SourceFile`实例，对应源码的 *AST*：

```typescript
const sourceFiles = program.getSourceFiles();
sourceFiles.forEach(file: SourceFile => {
  if (!sourceFile.isDeclarationFile) {
    console.log("analysis: ${sourceFile.fileName}")
    
    function visit(node: ts.Node) {
      console.log("node type: ${ts.SyntaxKind[node.kind]}")
      ts.forEachChild(node, visit)
    }
    visit(file)
  }
});
```

遍历 *AST* 可以获取函数、变量、类、模块等结构信息，支持静态分析、定制化检查或代码重构操作。对于项目中包含`.vue`、`.svelte`等文件，通过自定义`CompilerHost`将其转换为 TypeScript `SourceFile` 后，整个文件也可以纳入分析流程，实现多文件类型统一处理。

#### 3️⃣ 类型检查

类型检查是 TypeScript 静态分析的核心环节。通过`program.getTypeChecker()`获取`TypeChecker`，可以对 *AST* 节点执行类型推断和符号查询：

```typescript
const checker = program.getTypeChecker();

sourceFiles.forEach(file: SourceFile => {
  function visit(node: ts.Node) {
    if (ts.isFunctionDeclaration(node) && node.name) {
      const type = checker.getTypeAtLocation(node);
      const returnType = type.getCallSignatures()[0]?.getReturnType();
      console.log(`Function: ${node.name.text}, Return type: ${checker.typeToString(returnType)}`);
    }
    ts.forEachChild(node, visit);
  }
  visit(file)
})
```

这里需要明确一个概念：***TypeChecker* 本身不会生成错误或诊断**，它只是提供了类型和符号信息的查询接口。真正的类型检测和错误报告是在`program.getSemanticDiagnostics()`等方法中完成的。

#### 4️⃣ 诊断信息

在编译过程中，TypeScript 会生成语法、语义和全局错误诊断：

```typescript
const syntacticDiagnostics = program.getSyntacticDiagnostics();
const semanticDiagnostics = program.getSemanticDiagnostics();
const globalDiagnostics = program.getGlobalDiagnostics();

const allDiagnostics = [
  ...syntacticDiagnostics,
  ...semanticDiagnostics,
  ...globalDiagnostics,
];

allDiagnostics.forEach(d => {
  const message = ts.flattenDiagnosticMessageText(d.messageText, '\n');
  const fileName = d.file?.fileName ?? '<global>';
  const { line, character } = d.file ? d.file.getLineAndCharacterOfPosition(d.start ?? 0) : { line: 0, character: 0 };
  console.log(`${fileName} (${line + 1},${character + 1}): ${message}`);
});
```

诊断信息提供语法或类型错误的位置和详细信息，是 *IDE*、*CI* 校验以及类型安全保障的重要基础。

#### 5️⃣ 结果生成

最终的编译结果可通过`program.emit()`输出，并支持在生成前对 *AST* 进行自定义转换：

```typescript
program.emit(
    targetSourceFile?: ts.SourceFile,
    writeFile?: ts.WriteFileCallback,
    cancellationToken?: ts.CancellationToken,
    emitOnlyDts?: boolean,
    customTransformers?: ts.CustomTransformers
): ts.EmitResult
```

各参数含义说明如下：

- `targetSourceFile`：[可选] 如果指定某个`SourceFile`，则仅对该文件进行 *emit*；否则会对整个 *Program* 的所有源文件执行 *emit*。

- `writeFile`：[可选] 自定义文件写入回调，用于控制输出内容或附加额外信息。回调签名如下：

  ```typescript
  (fileName: string, 
   data: string, 
   writeByteOrderMark: boolean, 
   onError?: (message: string) => void, 
   sourceFiles?: readonly ts.SourceFile[]) => void
  ```

  通过该回调，我们可以获取编译后的代码（`data`）、输出文件名（`fileName`）以及对应的源文件列表（`sourceFiles`），非常适合做自定义构建或代码分析工具。

- `cancellationToken`：[可选] 用于在长时间编译过程中提供取消机制，一般工具链或 *IDE* 会使用它来支持中断。

- `emitOnlyDts`：[可选] 如果设为 `true`，则只生成类型声明文件（`.d.ts`），不会输出 *JS* 代码。

- `customTransformers`：[可选] 用于在 *emit* 阶段对 *AST* 做自定义转换，允许在 *TypeScript* 自身转换前 (`before`) 或后 (`after`) 注入新的节点。例如可以添加注释、日志或修改代码结构。

`emit` 方法会返回一个 `ts.EmitResult` 对象，其中包含：

- `emitSkipped`：是否跳过了 *emit*（例如遇到错误时）。
- `emittedFiles`：实际输出的文件路径列表。
- `diagnostics`：*emit* 阶段产生的诊断信息。

> 🔹 需要注意的是，`Program`内的类型信息在 emit 前已经缓存，如果在自定义 transformer 中修改 AST，TypeChecker 不会自动更新类型，因此在做类型敏感的代码转换时，需要小心顺序问题。

通过`program.emit`，我们不仅可以生成最终的 JavaScript 代码，还能灵活插入自定义逻辑或收集源文件信息，为构建工具、代码分析或插件开发提供了强大的基础。

## 常用编译配置

在实际项目中，`tsconfig.json`是 TypeScript 的核心配置文件，它决定了整个项目的编译行为和类型检查规则。为了方便理解，我们将配置项分为 **顶层配置** 与 **`compilerOptions`配置** 两类：

### 1. 顶层配置项

顶层常用配置如下：

| 配置项            | 类型                                             | 说明与使用场景                                               |
| ----------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `extends`         | string                                           | 继承其他配置文件，例如`{"extends": "./base.json"}`，可用于复用公共配置或多项目管理。 |
| `files`           | string[]                                         | 显式指定要编译的文件列表，通常只在单文件或特殊场景使用。若使用 `include`，一般不需要配置此项。 |
| `include`         | string[]                                         | 指定要包含的**文件**或**目录**，可使用 glob 语法，例如`{"include": ["src/**/*"]}`。 |
| `exclude`         | string[]                                         | 指定要排除的文件或目录，如`["node_modules"]`，可减少编译量并提升性能。 |
| `references`      | array                                            | 用于项目引用（*monorepo* 多 *tsproj* 的依赖关系），**通常与`{"composite": true}`配合使用**。 |
| `compileOnSave`   | <span style='white-space: nowrap'>boolean</span> | 在支持的 *IDE* 中，保存文件时自动触发编译。                  |
| `typeAcquisition` | object                                           | 针对 JavaScript 项目自动获取类型声明，用于混合 *JS/TS* 项目。 |
| **`ts-node`**     | object                                           | **ts-node 特定配置**，如果使用 ts-node 运行 TS 代码，可在这里指定参数。 |

### 2. `compilerOptions`配置项

`compilerOptions`常用配置如下：

| 配置项                          | 类型                                             | 说明与使用场景                                               |
| ------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `target`                        | string                                           | 指定输出 JavaScript 的版本，例如`"es6"`，影响语法降级与内置对象支持。 |
| `module`                        | string                                           | 指定**模块系统**，如`"commonjs"`、`"esnext"`，影响模块解析和打包。 |
| `lib`                           | string[]                                         | 指定**编译时引入的库**，如`"es6"`、`"dom"`、`"webworker"`，决定运行时可用 *API* 类型。 |
| `allowJs`                       | boolean                                          | 允许编译 *JS* 文件，适合逐步迁移项目。                       |
| `checkJs`                       | boolean                                          | 对 *JS* 文件进行类型检查，增强 *JS* 的类型安全。             |
| `jsx`                           | string                                           | 指定 *JSX* 编译方式，如`"react"`、`"react-jsx"`（React 17+）。 |
| `declaration`                   | boolean                                          | 生成`.d.ts`类型声明文件，是**库开发**的基础。                |
| `declarationMap`                | boolean                                          | 生成声明文件对应的 *sourcemap*，便于调试类型定义。           |
| `sourceMap`                     | boolean                                          | 为生成的 *JS* 文件创建 *sourcemap*，用于调试。               |
| `outDir`                        | string                                           | 指定编译输出目录，如`"dist"`。                               |
| `rootDir`                       | string                                           | 指定项目源代码根目录，用于统一输出目录结构。                 |
| `strict`                        | boolean                                          | 开启严格模式，包括`noImplicitAny`、`strictNullChecks`等，建议在工程中启用。 |
| **`noImplicitAny`**             | boolean                                          | **禁止隐式 *any* 类型，防止类型不明确。**                    |
| `strictNullChecks`              | boolean                                          | 严格检查`null`/`undefined`，避免潜在空值错误。               |
| `moduleResolution`              | string                                           | 模块解析策略，如`"node"`或 `"classic"`，影响模块查找行为。   |
| `esModuleInterop`               | boolean                                          | 允许默认导入 *CommonJS* 模块，改善 *Node* 项目兼容性。       |
| `allowSynthetic-DefaultImports` | boolean                                          | 允许从没有默认导出的模块默认导入，通常与`esModuleInterop`配合使用。 |
| `resolveJsonModule`             | boolean                                          | 支持直接导入 *JSON* 文件，例如`"import data from './data.json'"`。 |
| `isolatedModules`               | boolean                                          | 每个文件单独编译，<u>常与 *Babel/ts-loader* 配合使用</u>。   |
| `removeComments`                | boolean                                          | 编译输出时移除注释，减小体积。                               |
| `skipLibCheck`                  | boolean                                          | 跳过**库**文件类型检查，提高大项目编译速度。                 |
| `paths`                         | object                                           | 配置模块路径别名，通常与`baseUrl`配合使用，便于管理复杂目录结构。 |
| `baseUrl`                       | string                                           | 模块解析基准目录，常与`paths`配合使用。                      |
| `incremental`                   | boolean                                          | 启用增量编译，提高大型项目的编译效率。                       |
| `composite`                     | <span style='white-space: nowrap'>boolean</span> | 用于项目引用，要求开启`declaration`和`outDir`，常见于 *monorepo*。 |

## 借物表

| 参考资料                                                     | 说明       |
| ------------------------------------------------------------ | ---------- |
| [Vue3.0 前的 TypeScript 最佳入门实践](https://juejin.cn/post/6844903865255477261?searchId=20251218152826607FD058167D5B74A7B2#heading-34) | 掘金文章   |
| [ChatGPT](https://chatgpt.com/)                              | AI生成内容 |
