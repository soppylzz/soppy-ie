---
title: Git笔记：背景与基础
date: 2025-05-07 15:19:42
tags:
  - git
  - tools
  - tutorial
categories:
  - 笔记 
---

> 本文参考资料为**Git**官网的官方文档 [Git - Reference](https://git-scm.com/docs) ；
>
> 本文内容存在时效性，请在参考本文学习**Git**时注意甄别；

## 前言

站在某种角度观察，**人**和**程序**都是一条 *忒修斯之船* 。人在一生中几乎代谢更新掉全身的细胞，但记忆作为连续性的载体，使人始终被视为同一个「个体」。程序也是如此，尽管代码在其生命周期中经历多次重构与替换，但我们仍可通过**Git**窥见它最初的样貌。在以往的学习与实践中，我也使用**Git**和远程仓库管理代码，但由于缺乏规范，提交记录常常杂乱无序，像是随手拍下的人生片段，往往未能捕捉到真正关键的时刻。借由这篇博客，我希望系统梳理自己的**Git**学习过程，并尝试为未来的每一个程序版本，留下更具意义的时间切面。

## 背景

在学习一个程序或系统时，**建立对其功能和设计理念的整体认识，往往能为后续理解各个细节提供清晰的框架**。如此一来，我们后续学到的每一部分，都能在脑海中找到合适的位置，自然也更容易记忆与应用。因此，在正式进入**Git**的具体命令之前，不妨先一同了解它背后的设计思路。

当前版本控制系统主要存在两种**存储流派**：

* **基于差异**：这类系统为每个版本**存储**其与之前版本的差异，当我们想要获取某一个版本的文件源码时，只需要将其之前的变化差异累加起来就可以得到；这类系统的实例有SVN；
* <span style='color: cyan'>**快照流**</span>：这类系统不同于上述系统，每次记录版本时都为每个文件生成快照，并保存快照索引；特别的是，如果某一文件在版本更新时没有发生变化，这类系统便不会为其生成快照，Git就是属于这种类型；

除了存储模型上的创新，**Git的另一个核心特性是其分布式架构**。在Git中，开发者本地仓库拥有与远程仓库几乎完全一致的提交历史，包括所有的代码快照、提交记录、分支信息等。这种结构使得我们即使在脱离网络的情况下，也能完成绝大部分操作；只有在需要与远程协作（如`push`或`pull`）时才需要联网。

Git不仅在效率上做得出色，它还具备非常强的数据完整性保障机制。Git会对每一个对象使用`SHA-1`算法进行计算哈希索引，这确保了Git会发现我们对文件的所有修改。

### 1. Git工作阶段

在使用Git管理项目时，每个被管理的文件都会处于某种状态，表示它当前在Git生命周期中的位置。文件有以下四种工作状态：

| 状态                                 | 所在区域                                                     | IDE可视化                                                    | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| *`untracked`*                        | <span style='white-space: nowrap'>工作区`Working Directory`</span> | <span style='color: crimson'>**`file.txt`**</span>           | 文件数据未被Git跟踪，且未被`.gitignore`屏蔽跟踪，<span style='color: cyan'>**待`git add`跟踪暂存**</span> |
| **`committed`**<br/>**`unmodified`** | 工作区<br/>版本库`Commit History`                            | **`file.txt`**                                               | 文件数据已经保存到Git**本地版本库**，且当前工作区也存在该文件数据 |
| **`modified`**                       | 工作区                                                       | <span style='color: steelblue'>**`file.txt`**</span>         | 文件数据已被修改，<span style='color: cyan'>**待`git add`暂存**</span> |
| **`staged`**                         | 暂存区<br/>`Staging Area/Index`                              | <span style='color: green;white-space: nowrap;'>**`file.txt`**[添加]</span><br/><span style='color: steelblue;white-space: nowrap;'>**`file.txt`**[更新]</span><br/><span style='color: pink;white-space: nowrap;'>**`file.txt`**[删除]</span> | 已对文件当前版本进行标记，已包含到下次提交快照中；           |


![Git 下文件生命周期图。](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/lifecycle-1747301807005.png)

<center>figure 1.1. 文件的状态变化周期.</center>

需要注意的是，这里的<span style='color: pink;white-space: nowrap;'>**`file.txt`**[删除]</span>是在暂存区里面标记为**删除**，但工作区文件任存在时，文件在`IDE`中显示的可视化状态。此外，还需注意Git中 <u>*文件工作状态*</u> 与 *<u>Git工作区</u>* 的区别，一个是**文件状态**，一种是**文件存放区**；

![areas](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/areas-1747135748002-1747302236852.png)

<center>figure 1.2. 工作目录、暂存区域以及Git仓库.</center>

### 2. Git配置

我们可以通过`git config`命令来查看或设置Git的各种行为配置，Git支持多级别的配置系统，每个级别的配置会覆盖其下级配置，从而实现灵活控制：

| 级别                                                         | <span style='white-space: nowrap'>优先级</span> | 存储                                                         | 说明                                     |
| ------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| **系统 \| `--system`**                                       | +                                               | <span style='white-space: nowrap'>Win：`C:\ProgramData\Git\config`（<span style='color: pink'>`Git 2.x` 以后引入</span>）</span><br/>Unix：`/etc/gitconfig` | 存储系统上所有用户及他们仓库的通用配置； |
| <span style='white-space: nowrap'>**用户 \| `--global`**</span> | ++                                              | Win：`C:\Users\$USER\.gitconfig`<br/>Unix：`~/.gitconfig`or`~/.config/git/config` | 存储当前用户仓库的通用配置；             |
| **本地 \|`--local`**                                         | +++                                             | All：`<repo>/.git/config`                                    | 存储当前仓库的配置；                     |

在实际开发中，我们常用以下Git配置命令来调整使用体验和工作流程：

####  ⚙ 用户与网络

安装完Git之后，要做的第一件事就是设置我们的用户名和邮件地址。 这一点很重要，因为每一个Git提交都会使用这些信息，它们会写入到我们的每一次提交中；

```bash
git config --global user.name "xxxx"
git config --global user.email "xxx@example.com"
```

Git配置代理可以让Git请求通过指定的`HTTP`/`HTTPS`或`SOCKS`代理服务器转发；关于上述通信协议的区别以及其在代理软件中的使用在这里就不过多介绍，**这值得单独在一篇文章中讲解**；在配置时需要注意的一点是，**Git for Windows**对`SOCKS5`的支持有限，在Windows上Git推荐使用`HTTP`/`HTTPS`；

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

#### 🔍️ 配置管理

当我们想查看当前项目下有哪些Git配置时，可以使用如下命令：
```bash
git config --list 			# 列出当前所有配置
git config --list --show-origin 	# 列出当前配置及其来源
git config user.name 			# 查看单条配置
```

当某项配置不再需要或设置错误时，我们可以使用`--unset`参数来删除它。需要注意的是，当我们使用`--unset`删除配置时，其后面必须跟一个配置**key**，如果没有则会报错，即我们不能使用`--unset`重置所有设置：

```bash
git config --global <config_name>
git config --local --unset core.autocrlf	# 成功执行
git config --local --unset 			# 报错
```

Git配置都是保存在配置文本文件中，除了直接编辑对应级别的配置文件，我们还可以使用如下命令编辑配置文件：

```bash
git config --<level> --edit
git config --local --edit	# 编辑项目本地Git配置
```

#### 🗨️ 配置别名

使用Git命令行管理项目时，频繁输入完整的命令既繁琐又耗时。为提升效率，Git支持配置命令别名，让我们通过更简洁的方式执行常用操作。我们可以为**Git命令**设置缩写形式，也可以使用`!`前缀配置执行任意**Shell**命令的快捷方式（更推荐使用`terminal`自带的别名工具）：

| 类型      | 命令                                                  |
| --------- | ----------------------------------------------------- |
| **SHELL** | `git config --global alias.<cmd_name> '!<shell_cmd>'` |
| **GIT**   | `git config --global alias.<cmd_name> '<git_cmd>'`    |

下面是设置Git命令缩写和Shell命令别名的示例：

```bash
git config --local alias.co 'checkout'		# 执行checkout
git config --local alias.vs '!code'		# 打开vscode

# 使用
git co -b new_branch
git vs
```

#### ☁ 凭证管理

如果不进行任何设置使用Git远端命令控制远端仓库，我们需要频繁输入账户信息。Git为我们提供了 **凭证管理工具** | `credential.helper` 以解决这个问题。Git官方推荐的`credential.helper`配置即相关软件更新频繁，Git中文文档可能来比较更新，前往Git英文文档查询是较优选择。在我写博客的时候，Git有如下适用于不同版本的凭证管理助手：

| 平台        | 工具                                                         | 说明                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Windows** | `git-credential-wincred`                                     | 凭证存储在Windows凭据管理器中。随**Git for Windows**提供     |
| **Linux**   | <span style='white-space: nowrap'>`git-credential-libsecret`</span><br/>`cache` | 凭证存储在Linux密钥服务中，如`GNOME`密钥圈或`KDE`钱包。一般由**Linux**发行版提供；<br/>Linux也支持短暂（15分钟）存储用户凭证在**cache**中； |
| **MacOS**   | <span style='white-space: nowrap'>`git-credential-osxkeychain`</span> | 凭证存储在macOS密钥链中。随**macOS Git**提供                 |

不同平台查看已保存的凭证的方式各不相同，以下是各个平台的查看方式：

* <span style='color: cyan'>**Windows**</span>：打开“控制面板 → 凭据管理器”；
* **Linux (libsecret)**：`seahorse`或`secret-tool`；
* **MacOS**：打开“钥匙串访问”；

除了设置上述版本工具，我们还可以进行如下`credential.helper`设置，我们可以使用`git config`管理凭证工具的配置，在这里就不过多介绍Git命令；

| 工具        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| **`""`**    | 每次都输入账号密码                                           |
| **`cache`** | 仅限**Linux**/**macOS**，默认缓存15分钟                      |
| **`store`** | 保存在明文文件中（`~/.git-credentials`）（<span style='color: pink'>**不安全**</span>） |

值得一提的是，Git的凭证管理适用于**使用`HTTP`协议**管理（`push`、`fetch`等）远程仓库；当我们想要 <u>*使用`SSH`协议*</u> 管理远程仓库时，则需要配置远程仓库的`SSH keys`；

#### 📓 文本编辑器

Git可以使用`core.editor`配置项来控制默认编辑器。需要注意的是：**Linux**下Git**默认**使用`vim`作为编辑器；**Windows**下**默认**使用Git Bash自带`vim`；以下是**Windows**下配置`notepad`、`vscode`作为默认编辑器的代码：

```bash
git config --global core.editor "<editor_run_command>"		# 通用配置命令

git config --local core.editor "notepad"			# 配置notepad
git config --local core.editor "code --wait --new-window"	# 配置vscode

git commit --allow-empty 					# 提交空信息用于调试
```

配置VSCode时，`code`就是VSCode的启动命令，而`--wait`参数会让Git等待VS Code编辑器关闭。如果我们想每次`git commit`时都`vscode`都创建一个新的窗口，可以添加`--new-window`指令；

#### 🌳 合并/对比工具

Git默认提供简单的命令行合并/对比行为，Git可以通过配置来使用自定义的 `mergetool` | **合并工具**  和 `difftool` | **对比工具** 。这里我只介绍**Merge**/**Diff**工具的配置，其具体使用方法会在后面单独介绍。

| 类型        | 用途                                         |
| ----------- | -------------------------------------------- |
| `difftool`  | 显示两个版本（如 commit、branch）的**差异**  |
| `mergetool` | 在出现**冲突**时帮助我们对比和合并冲突的文件 |

以下是**Windows**下配置`vscode`作为合并/对比工具：

```bash
# 通用配置命令
git config --global diff.tool <tool_name>	# 可省略
git config --global merge.tool <tool_name>	# 可省略

git config --global difftool.<tool_name>.cmd 'tool_run_command'
git config --global mergetool.<tool_name>.cmd 'tool_run_command'

# 配置vscode作为合并/对比工具
git config --local diff.tool vscode
git config --local merge.tool vscode

git config --local difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
git config --local mergetool.vscode.cmd 'code --wait $MERGED'

# 关闭每次都询问工具选择
git config --local difftool.prompt false
git config --local mergetool.prompt false
```

需要注意的是，因为我们在配置时使用了`$`符号，在`Window CMD`中配置<u>相关工具</u>的`cmd`时，使用双引号`“”`会导致配置失败。

#### 🪟 Windows配置

在Windows上使用Git时，由于与类Unix系统在文件路径、换行符、文件名大小写敏感等方面存在差异，因此Git提供了一些**针对 Windows 系统的专属或高度相关的配置项**，用来提高跨平台兼容性和开发体验。

* **`core.autocrlf`**：

    在Windows中，文本文件的换行符默认是`CRLF`（`\r\n`），而类Unix系统中，是 `LF`（`\n`）。为了避免跨平台协作中出现换行符混乱，我们需要使用`core.autocrlf`控制Git**在提交前和检出后如何处理换行符**。以下是各个系统的推荐：

    | 平台        | 配置    | 说明                                     |
    | ----------- | ------- | ---------------------------------------- |
    | **Windows** | `true`  | 提交时将CRLF转为LF，**检出**时再转为CRLF |
    | **Linux**   | `input` | 提交时将CRLF转为LF，**检出**时不做处理   |

    除了控制Git提交时**换行符**的转换，我们还可以使用`core.safecrlf`来控制待`add`代码的**换行符**检查策略：

    | 设置              | 说明                                                |
    | ----------------- | --------------------------------------------------- |
    | **`true` [默认]** | 拒绝提交混合换行符的文件                            |
    | **`false`**       | 不检查换行符混用问题                                |
    | **`warn`**        | 发出警告但<span style='color: pink'>允许提交</span> |

* **`core.ignorecase`**：

    Windows文件系统`NTFS`默认对文件名大小写**不敏感**，但Git是大小写敏感的。

    | 设置值                   | 说明                                                         |
    | ------------------------ | ------------------------------------------------------------ |
    | **`true` [Windows默认]** | 忽略文件名大小写<br/>如果我们使用此设置，在修改文件名大小写时**不会触发Git跟踪** |
    | **`false`**              | 严格区分文件名大小写（推荐在跨平台项目中手动设为`false`）    |

#### 🤟 我的配置

下面介绍以下我是如何配置Git的。我在Windows环境下使用**Git for Windows**。安装时选择集成**Git Bash**，这使得我们可以在`git config alias`中直接执行Shell命令。配置策略上，我仅在Git配置的全局级别（`--global`）中设置 <u>*网络代理*</u> 与 <u>*用户信息*</u> 的设置，项目特定配置则保留在本地级别（`--local`）。以下是我的Git全局配置命令：

```bash
git config --global user.name "xxxx"
git config --global user.email "xxx@example.com"

git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

git config --global alias.pi '!f(){ sh "$HOME/.soppyrc/git-custom-ini.sh"; }; f'
git config --global alias.ei '!code $HOME/.soppyrc/git-custom-ini.sh'
```

需要注意的是，由于Git Bash的环境特性，`$HOME`会被自动映射到`C:\Users\<username>\`目录，因此当我们在使用`git pi`执行`git-custom-ini.sh`时，命令能够正常执行。

此外，**Windows**与**Git Bash**的**路径格式转换**存在诸多问题，后续会在介绍SHELL的文章中详细讲解，在这里就不过多介绍了。这里只简单介绍我遇到的问题：

| <span style='white-space: nowrap'>问题</span>     | `$HOME`错误转换问题                                          |
| ------------------------------------------------- | ------------------------------------------------------------ |
| **<span style='white-space: nowrap'>描述</span>** | 当我们使用 ***双引号*** 添加`pi`命令（`!$HOME/..`）时，`$HOME`会被Git Bash自动转换为Windows路径格式，`gitconfig`中**结果为`!C:\\Users\\lzz/.soppyrc/git-custom-ini.sh`，这路径不会Git Bash被正常解析**。 |

解决方法：

* **直接编辑/更换单引号**：直接编辑`gitconfig`文件，或者更换<u>单引号</u>设置`pi`解决问题；

    ```bash
    [alias]
    	pi = !$HOME/.soppyrc/git-custom-ini.sh
    ```

* **使用SHELL函数**：使用如下命令设置别名；

  ``````bash
  git config --global alias.pi '!f(){ sh "$HOME/.soppyrc/\
  						git-custom-ini.sh"; }; f'
  ``````

| <span style='white-space: nowrap'>问题</span>     | 单双引号选择问题                                             |
| ------------------------------------------------- | ------------------------------------------------------------ |
| **<span style='white-space: nowrap'>描述</span>** | 当我们使用 ***双引号*** 添加`ei`命令（`code ..`）时，`$HOME`会自动解析为Windows路径格式，**格式同上**。运行`ei`命令会打开`C:/Userlzz/..`文件，**这是因为VSCode错误解析该路径**。 |

解决方法：

* **更改路径**：给路径添加 *单引号* ，`gitconfig`中路径被添加上单引号，此时`ei`命令能够成功运行，但是`gitconfig`中地配置仍为：`ei = !code 'C:\\Users\\lzz/.soppyrc/git-custom-ini.sh'`；

* **更换单引号**：使用单引号添加`ei`命令，命令运行结果与`gitconfig`中配置结果均符合预期；

为了使上述配置地别名命令能够正常运行，我们需要在`$HOME/.soppyrc`文件夹下创建`sh`脚本，以下是该脚本的具体内容：

```shell
#!/bin/bash
echo "Setting Git aliases and tools..."

git config --local alias.cre "config --local core.autocrlf true"
git config --local alias.lge "log --oneline --graph --all --decorate"
git config --local alias.cae "commit --amend --no-edit"

git config --local core.editor "code --wait --new-window"

git config --local diff.tool vscode
git config --local merge.tool vscode
git config --local mergetool.vscode.cmd 'code --wait $MERGED'
git config --local difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

git config --local alias.st status
git config --local alias.co checkout
git config --local alias.sw switch
git config --local alias.br branch
git config --local alias.ci commit
git config --local alias.lg log
git config --local alias.df diff
git config --local alias.ad add

# Windows Only
git config --local core.autocrlf true
git config --local core.ignorecase false

echo "Done and remember to configure your GPG, SSH."
```

## 基础命令

在学习Git命令的过程中，我们经常需要查阅某个命令的用法。Git提供了多种方式来获取帮助信息：

* **`--help`**：打开该命令的完整文档，例如：`git <command> --help`；

* **`-h`**：在终端中查看简略帮助信息，例如：`git -h`、`git revert -h`；

    值得一提的是对于多层命令（例如`git stash <sub_cmd>`）而言，`-h`会递归输出该命令的**所有子命令**的帮助信息，所以不一定需要对每个子命令都使用`-h`输出帮助信息。

在本章节中，我们将深入学习Git的常用与进阶命令。与其单纯地阅读命令解释，不如通过实际操作来加深理解。因此，我们参考如下指令创建一个**Git Mock**项目，在真实的仓库环境中练习每个命令的使用方式与效果：

```bash
mkdir git-learn
cd git-learn

code .gitignore	# echo ".idea">test.txt

git init
git config --local core.editor "code --wait --new-window"
git config --local alias.lg 'log --oneline --graph --all --decorate'
git config --local alias.st 'status'

code test.txt	# echo "first commit">test.txt

git commit -a
git branch -M main

git tag -a first -m 'Tag first commit'	# 标记初始节点
# 使用`git checkout first`将HEAD移动到Git开始节点
```

### 1. 基础

在Git发布**20周年**的一次访谈中，**Linus**提到，他日常使用的Git命令其实只有几种。事实上，对于大多数人来说，只需掌握几条常用命令，就足以应对团队协作或参与开源项目的需求。想要深入学习Git命令，我们不妨先来了解这些常用命令。

#### 🧭 Git引用与运算符

`commit`是Git中保存代码快照的对象，我们使用Git **引用** | `ref` 来操作每个具体的`commit`对象，常见的引用类型有：

* **`HEAD`**：指向当前操作的**默认**`commit`，即“当前所在的位置”；

    在Git中，绝大多数操作都会围绕`HEAD`所指向的`commit`进行。如果在一些需要 `commit`的命令中不指定引用，Git默认会使用`HEAD`作为目标。

    | 状态       | 说明                                                         |
    | ---------- | ------------------------------------------------------------ |
    | `attached` | `HEAD`指向一个**分支**，此时<u>当前分支</u>始终会跟随最新的提交 |
    | `detached` | `HEAD`指向一个具体的`commit`，只有`HEAD`指向更新的提交       |

* **`branch`**：指向**对应分支**的最后一次`commit`，是一种可移动的引用；

* **`tag`**：指向某一个**固定**`commit`，当我使用`git checkout`切换`HEAD`到`tag`上时，等同于切换到`hash id`上；

* ***`hash id`***：指向**固定ID**的`commit`，`hash id`是对`commit`对象的直接引用；

我们可以使用`^`、`~`运算符找到一个引用的相对引用，以下是详细用法：

| <span style='white-space: nowrap'>符号</span> | 说明                                                       | 示例                                                         |
| --------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| `~`                                           | 表示从当前提交沿着**第一父提交的路径**向上追溯若干代父提交 | -  `HEAD~1`：表示`HEAD`的父`commit`<br/>- `HEAD~2`：表示`HEAD`的祖父`commit` |
| `^`                                           | 表示当前提交的**第几个父节点**，常用于合并提交             | <span style='white-space: nowrap'>- `HEAD^`：通常等同于 `HEAD~1`，指向第一个父`commit`</span><br/>- `HEAD^2`：表示第二个父`commit` |

需要注意，**只有当Git执行`octopus merge`（<u>目前</u>没有冲突处理步骤，只允许无冲突合并）时**，合并节点才会有两个以上父节点。此时，父节点对应的<u>`<parent_number>`遵从使用`merge`合并时的输入顺序</u>。

为了体会`<parent_number>`的命名顺序，我们不妨运行如下**HEAD MOCK**,我们按照时间**先后顺序**在Git的`first`创建待合并的分支：`OM_1`、`OM_2`、`OM_3`、`OM_4`。然后执行`(HEAD -> OM_1)git merge OM_4 OM_2 OM_3`，在合并后的`HEAD -> OM_1`上使用`HAED^<parent_number>`，有如下结果

| 命令                       | 结果   |
| -------------------------- | ------ |
| `git log --oneline HEAD^2` | `OM_4` |
| `git log --oneline HEAD^3` | `OM_2` |
| `git log --oneline HEAD^4` | `OM_3` |

需要注意的是，<span style='color: pink'>**在`detached`状态下的`HEAD`的相关查询命令的结果也是如此**</span>。

#### 👶 `git add` & `commit`

`add`和`commit`是Git最常用的命令。尽管我们可能在之前已经初步了解了他的用法，这里我们还是简要了解以下其常用参数选项：

| `git add`                                                 | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| **`.`**                                                   | 当前目录下所有更改（新增+修改，不包括删除）                  |
| **`-A`/`--all`**                                          | 所有更改（包括新增、修改、删除）都加入暂存区                 |
| **`-u`/`--update`**                                       | 只添加已跟踪文件的更改（含删除），不包括新文件               |
| `-p`/`--patch`                                            | **交互式选择文件部分<span style='color: cyan'>变更</span>**加入暂存区，注意该选项**不能添加文件追踪** |
| `-n`/`--intent-to-add`                                    | 标记为将添加的新文件，但内容还未加入                         |
| `-v`                                                      | 显示详细的添加过程                                           |
| <span style='color: cyan'>**`-i`/`--interactive`**</span> | 打开一个交互式命令界面（比`-p`更复杂）选择文件变更加入       |

| `git commit`                              | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **`-m "msg"`**                            | 添加提交说明                                                 |
| **<span style='color: cyan'>`-a`</span>** | 自动添加**已跟踪文件**的更改，不需要`git add`，常用`-am`更新`Modified File` |
| **`--amend`**                             | 修改**最近一次**提交（可改内容和说明）                       |
| **`--no-edit`**                           | 和`--amend`一起使用时，不修改说明                            |
| `--allow-empty`                           | 允许提交一个**无变更**的空提交                               |
| `--dry-run`                               | 模拟提交过程，不实际提交                                     |
| **`--verbose`/`-v`**                      | **提交时显示 diff 内容**                                     |
| `--date=...`                              | 自定义提交时间                                               |
| `--author="..."`                          | 自定义提交作者信息                                           |
| `--reset-author`                          | 重置作者为当前配置的Git用户                                  |
| **`--signoff`/`-S`**                      | 添加GPG签名（**常用于开源项目**）                            |

#### 🚫 `.gitignore`

`.gitignore`文件是Git项目中的一个特殊文件，用于告诉Git**哪些文件或文件夹不应被跟踪，即不纳入版本控制**。它是版本控制过程中非常重要的一个工具，尤其是在避免将临时文件、构建输出、敏感信息（如密码配置）等纳入仓库时。

当我们在**Windows CMD**中使用`echo`命令创建该文件时，请注意编码格式。这是因为当我们使用`.gitignore`时，需要考虑<span style='color: pink'>**编码格式一致性**</span>。Git默认文本编码是`UTF-8`，我们可以通过在仓库根目录创建`.gitattributes`，并写入如下内容。这样，Git知道工作区`.gitignore`是`GBK`编码，提交时会把它转换成`UTF-8`存入Git本地版本库：

```shell
.gitignore text working-tree-encoding=GBK
```

我们可以按照**路径匹配模板**与**文件匹配模板**的分类分别学习匹配字符串：

| 路径模板 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| `temp`   | 匹配所有名为`temp`的文件与文件夹，需要注意这里的`temp`文件没有`ext` |
| `temp/`  | 匹配所有`temp`文件夹                                         |
| `/<xxx>` | 匹配**根目录**下的文件或文件夹，<u>这里匹配模板用`/`开头表示根目录</u> |
| `**/`    | 匹配**任意层级**目录（<span style='color: pink'>`Git 2.x` 以后引入</span>） |

| 文件模板         | 说明                                      |
| :--------------- | ----------------------------------------- |
| `*.log`          | 匹配所有`.log`结尾的文件                  |
| `*`              | 匹配零个或多个任意字符                    |
| `?`              | 匹配一个任意字符                          |
| `[...]`          | 匹配括号内的任意一个字符                  |
| `!important.txt` | **不忽略**`important.txt`，即使之前被忽略 |

需要注意的是，**`.gitignore`只能影响未被Git跟踪`track`的文件**。如果某一个文件已被跟踪，即使我们后来把它加入 `.gitignore`，它仍然会被跟踪。我们需要使用[`git rm`命令](#🗑%EF%B8%8F-git-rm)取消文件跟踪。此外，`.gitignore`文件还支持在项目子目录中创建，它会覆盖或补充上层目录的规则。

我们可以使用如下命令调试`.gitignore`，查看某个文件是否被忽略：

```bash
git check-ignore -v <filepath>
```

### 2. 撤销

在[1. Git工作阶段](#1-Git工作阶段)中，我们已经知道了Git中工作区与文件的联系。当我们使用Git管理我们的项目版本时，主要是在对三种存储区中的文件进行各种操作。我们可以使用`git status`查看当前项目的状态，查看的文件类型有：

1. **`staged`**：文件已更改，且放入暂存区；
2. **`not staged`**：文件已更改，但未放入暂存区；
3. **`untracked`**：文件未被Git追踪；

我们可以使用`git add`**添加`not staged`文件到暂存区**或者**添加并追踪`untracked`文件**，然后使用`git commit`来提交暂存区的更改到Git本地版本库，这是Git提交的**基本流程**：

```bash
git switch main

code test.txt		# echo "staged add">>test.txt
git add ./test.txt 	# 添加test.txt文件到暂存区
code test.txt 		# echo "not staged add">>test.txt
code new.txt		# echo "untracked file">new.txt

git status		# git status会显示项目当前的三种存储区的文件
git commit		
git status		# 只会commit已经存入暂存区的文件

git commit -a 		# finish experiment
```

在理想情况下，我们每次都能准确地`add`或`commit`正确的修改内容。但现实中错误的操作不可避免：也许我们添加了不该添加的文件，或提交了有问题的代码。遇到这种情况，不必慌张，Git提供了`git restore`、`git reset` 、`git revert`和`git rm`工具来帮我们解决问题。

#### 🌱 `git restore`

`git restore`是Git在`2.23`版本中引入的新命令，是对旧命令的**语义拆分**。该命令用于撤销文件的更改，类似于<kbd>ctrl</kbd>+<kbd>z</kbd>。常用选项如下：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <span style='white-space: nowrap'>`--source=<commit>`</span> | 指定还原的来源，**默认是暂存区`INDEx`**，注意我们实际上并不能使用`INDEX`访问暂存区 |
| **`--staged`**                                               | 将目标文件从<span style='color: cyan'>暂存区还原</span>，不影响工作区。该选项相当于取消`git add`的效果：<br/>- 既可以取消**已跟踪文件**的暂存操作<br/>- 也可以取消**未跟踪文件**的跟踪、暂存操作 |
| **`--worktree`**                                             | 将目标文件<span style='color: cyan'>还原指定版本到工作区</span>，即取消工作区修改，不影响暂存区。<br/>**默认的`git restore`实际上**就是`git restore --source=INDEX --worktree` |
| `--ours`                                                     | 在合并冲突中使用，选择“我们”的版本（当前分支），用于解决冲突时还原文件内容 |
| `--theirs`                                                   | 在合并冲突中使用，选择“对方”的版本（合并进来的分支），同样用于冲突解决 |

当我们使用`merge`合并两个有冲突的分支时，Git会自动修改冲突文件（此时，冲突文件中会出现冲突标记`<<<<<<<`、`=======`、`>>>>>>>`）。此时，冲突文件为`not staged`状态，我们需要执行：修改冲突 → `add`到暂存区 → `commit`提交，才能完成`merge`流程。而`git restore`为我们提供了**快速合并**的方式，`--ours`就是以当前分支的结果替换掉`<<<<<<<...>>>>>>>`的内容。

```bash
# On branch restore, restore-dev
git checkout restore
git checkout -b restore-dev

code test.txt	# echo "dev add" >> test.txt
git commit -a

git checkout main
git merge dev

# Result
# Auto-merging test.txt
# CONFLICT (content): Merge conflict in test.txt
# Automatic merge failed; fix conflicts and then commit the result.
# 
# Conflict
# <<<<<<< HEAD
# staged add
# not staged add
# =======
# dev add
# >>>>>>> restore-dev
```

下面我们分别执行`--ours`与`--theirs`的合并方式，结果如下：

```bash
git restore --ours .\test.txt\
# staged add
# not staged add

git restore --theirs .\test.txt\
# dev add
```

#### ⚡️ `git revert`

`git revert`是Git中用于撤销某次提交的命令，它的特点是**不会修改项目的历史记录**，而是通过创建一个**新的提交**来“反向更改”原有提交的改动。常用选项如下：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `--edit`<br/>`--no-edit`                                     | 控制**是否打开editor修改**Git自动生成的提交信息（默认`--edit`） |
| `--commit`<br>`--no-commit`/`-n`                             | 控制还原后**是否提交**（默认`-n`）<br/>- `--commit`：在提交信息（自动/手动）确定后，Git会自动提交撤销记录；<br/>- `--no-commit`：在提交信息确定后，Git不会提交撤销，这方便我们针对撤销结果**继续更改**，或者执行新的撤销 |
| **`-m <parent-number>`**                                     | 当回退一次合并提交时，指定**主干父提交**的编号，告诉Git应该以哪个父分支为基准进行回退 |
| <span style='white-space: nowrap'>`--continue`/`--abort`</span><br/> `--skip`/`--quit` | 用于在回退多个提交或遇到冲突时控制流程                       |
| `--signoff`/`-s`                                             | 在`revert`的提交信息中添加`Signed-off-by`签名                |
| `-S[<keyid>]`                                                | 如果我们配置了GPG密钥，该命令会对回退提交进行签名，用于增强提交的身份验证 |

光看`options`列表也许不能帮助我们很好的了解其功能，下面我们结合例子讲解`git revert`的用法。

* **撤消多个提交**：`revert`支持一次性撤销多个`commit`

    当我们执行`git revert A B C`时，Git实际上执行了如下指令：

    ```bash
    git revert A
    git revert B
    git revert C
    ```

    在使用`revert`撤销多个提交时，有如下需要注意的情况：

    * 如果撤销序列正常，且我们添加了`-n`选项，Git会依次进行撤销，并在最后将是否提交撤销的决策权交予我们；

    * 当我们提供了乱序的`commit id`时，Git会处理<u>每一次撤销引起的冲突</u>。如果这时我们添加了`-n`选项，Git仍会执行每一次冲突处理，**需要处理前一次冲突提交**后，才能执行下一次冲突处理，这时`-n`没有效果；
    * 我们将撤销多个提交的`revert`当成多次执行`revert`命令就能很好的理清Git处理的逻辑；

* **处理冲突**：当我们使用`revert`时也会遇到**冲突**，以下是`revert`的**Conflict Mock**：

    ```bash
    git checkout revert
    
    code revert.txt 	# echo "revert 1" > revert.txt
    git commit -a
    ...
    code revert.txt 	# "revert 2" -> "revert 3"
    git commit -a
    
    git log --oneline --graph --decorate
    # Result
    # * 03323ac (HEAD -> revert) revert 3
    # * 375b19a revert 2
    # * 790281c revert 1
    # * 1aa32af first commit
    ```

    如果我们想撤销`HEAD`提交不会发生冲突，但当我们想撤销`HEAD^`就会发生冲突。

    * `f8319d7(HEAD)`：内容变更为`revert 2` → `revert 3`
    * `b5b25a8(HEAD^)`：内容变更为`revert 1` → `revert 2`，更改后内容为<u>`revert 2`</u>；

    此时，Git**无法直接撤销`HEAD^`提交**，Git会进入类似于`merge`中处理冲突的状态并自动修改冲突文件，其结果如下：

    ```txt
    <<<<<<< HEAD
    revert 3
    =======
    revert 1
    >>>>>>> parent of b5b25a8 (revert 2)
    ```

    这时候，Git猜测我们的想法是撤销掉`HEAD^`的修改结果，故Git会列出基于`HEAD^`的**向前撤销**和**向后撤销**两种方式，然后我们再执行如下命令即可完成`revert`操作：

    ```bash
    git add .
    git revert --continue	# 此时，会弹出编辑器界面用于commit
    ```

* **撤销合并**：我们在执行合并节点的撤销时，需要添加`-m <parent_number>`选项**指定撤回的父节点**，以下是`revert`的**Merge Mock**：

    ```bash
    # 准备阶段
    git checkout revert-merge
    code revert.txt
    git commit -a
    
    git checkout revert
    git merge revert-merge
    
    git commit -a
    
    # 撤销合并节点
    git revert -n -m 1 HEAD
    git revert -n -m 2 HEAD
    ```


#### 🔥 `git reset`

`git reset`也是Git中用于撤销的命令。与生成新提交的`git revert`不同，它直接将`HEAD`退回到指定的提交对象上，因此该命令**不会新增提交**。根据不同的使用场景，我们可以选择不同选项来决定该命令是否影响**暂存区**与**工作区**。

| 选项                                                | 说明                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| `--soft`                                            | 撤销提交但**工作区不变**，工作区内容自动`add`到**暂存区**    |
| **`--mixed`[默认]**                                 | 撤销提交但**工作区不变**，工作区内容未被`add`                |
| `--hard`                                            | 完全回退到指定提交记录，**之后的提交改动都会丢失**<br/>可以搭配`git reflog`查看历史并恢复 |

`reset`的撤销选项还有`--keep`与`--merge`。但经过测试，我发现上述功能就能很好的覆盖大部分`reset`任务，故就不再过多讲解其他的选项。

需要注意的是，对于`--soft`与`--mixed`模式，其撤回造成的结果中可能存在`staged`/`not staged`文件。此时，我们既可以使用`restore`清空暂存区/工作区的修改，也可以使用`git reset --hard HEAD`直接撤回到该提交节点的保存状态，两者可以达成一样的效果；

#### 🗑️ `git rm`

上述命令已经可以执行大部分撤销任务，如果要更加自由的执行撤销任务或者完成其他特殊任务，可以考虑使用`git rm`命令。`git rm`是Git中用于**从暂存区和工作目录中删除文件**的命令。常用选项如下：

| 选项                                                        | 说明                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| **`--cached`**                                              | 只从暂存区中删除，不删除工作目录                             |
| `-r`/`-f`                                                   | `-r`用于递归删除目录，`-f`用于强制删除文件（例如删除修改过的`not staged`文件）<br/>两者常一起使用：`-rf` |
| <span style='white-space: nowrap'>`--ignore-unmatch`</span> | 尝试删除**不存在**文件时，使用该命令不会报错                 |
| `--dry-run`                                                 | **模拟删除**，显示会删除哪些文件                             |

`git rm`常在我们更新`.gitignore`时使用。`.gitignore`**只对未被 Git 跟踪的文件有效**。如果我们已经将某个文件提交到了Git中，即使后来加进`.gitignore`，它仍会被跟踪。这时我们需要使用`--cached`选项，在暂存区中删除以跟踪文件：

```bash
# Change
git rm -rf --cached .
git add .

git commit -m 'chore(.gitignore): add file: xxx to ignore'

# Check
git check-ignore xxx
git checkout HEAD^
git check-ignore xxx
```

需要注意使用`git rm -rf`删除文件时，Git会自动将删除操作的更改`add`到暂存区。这时候我们想要<u>还原删除文件</u>可以使用`git restore --source=HEAD .`从最近提交还原。需要注意的是，这里不能使用`git restore .`因为该命令默认是从暂存区中还原文件，而暂存区已经被`git rm`更改了，故会报如下错误：

```bash
error: pathspec '.' did not match any file(s) known to git
```

