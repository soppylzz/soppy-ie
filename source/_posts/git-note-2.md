---
title: Git笔记：分支与调试
date: 2025-05-20 22:09:53
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

## 分支命令

几乎所有的版本控制系统都以某种形式支持分支。 使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。下面我们详细了解一下Git常用的分支命令。

### 1. 基础​​

#### 🌿 `git branch`

`branch`既可以执行对分支的**管理**操作，也可以执行**可视化**分支的操作：

使用`git branch <branch_name>`可以对分支进行管理，Git会在`HEAD`所在的提交节点新建一个新的分支。`branch`还支持如下分支管理方式：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <span style='white-space: nowrap'>`-f <branch> <target>`</span> | **强制移动**一个分支（必须显示指出）指向另一个提交           |
| `-d <branch>`                                                | **安全删除**已合并到当前分支的分支                           |
| `-D <branch>`                                                | **强制删除**分支，无论是否合并                               |
| `-m <branch>`                                                | **安全**重命名分支及其`reflog`，如果目标分支名已存在，<br/>则会**报错并拒绝操作** |
| `-M <branch>`                                                | **强制**重命名分支及其`reflog`，如果目标分支名已存在，<br/>则会<span style='color: pink'>**覆盖原有分支**</span> |
| <span style='color: cyan'>**`--track <local> <remote>`**</span> | 创建一个新的分支，并为其指定其跟踪的远程分支<br/>当不使用`--track`指定创建分支对应的`<remote>`分支时，系统会**自动指定**`origin/<local>`为远程分支 |
| **`--no-track <local>`**                                     | 创建分支时**不跟踪远程分支**                                 |

Git有一套独特的远程仓库同步方法，当我们使用`push`或`clone`与远程仓库交互时，Git的本地分支`local`会与其对应的远程分支`origin/local`交互。我们在创建本地分支时，使用`--track`命令可以指定与其对应的远程分支，以便我们个性化的与远程仓库交互。

当我们使用`-D`模式时，需要注意待删除分支不能是**工作分支**（`worktree`的使用方法详见后续章节），否则会报如下错误：

```bash
error: cannot delete branch '<branch>' used by worktree
				at 'D:/Code/Javascript/Learn/git-learn'
```

不添加操作目标`<branch>`的`git branch`命令可以对分支进行可视化。使用该命令会列出所有<u>本地分支</u>，并将当前分支用`*`标记。此外，`branch`还支持如下可视化方式：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-a`/`--all`                                                 | 显示所有分支，包括**本地**和**远程分支**                     |
| `-r`/`--remotes`                                             | 仅显示**远程分支**                                           |
| <span style='color: cyan'>`-v`/`--verbose`</span>            | 显示**本地**分支的详细信息，包括：`<branch_name>`、`<latest_commit_id>`和`<latest_commit_msg>` |
| <span style='color: cyan'>`-vv`</span>                       | 显示**本地**和**远程**分支的详细信息。相较于`-v`选项，该选项可以显示本地分支追踪的远程分支 |
| `--merged`/`--no-merged`                                     | 控制是否列出已合并到**当前分支**的分支                       |
| `-l`/`--list <pattern>`                                      | 列出满足**匹配模式**的分支<br/>例如：执行`git branch -l "dev-*"`，Git会列出符合`dev-*`匹配模式的**分支** |
| `--contains <commit>`<br/><span style='white-space: nowrap'>`--no-contains <commit>`</span> | 控制是否列出包含**指定提交**的分支                           |

#### 📦️ `git stash`

当我们在修改当前分支代码时，若需要临时切换分支处理其他任务，可以使用`git stash`将**工作区**与**暂存区**的改动保存到Git的**储藏堆栈**中，以避免未完成的修改被提交。`git stash`的常用命令如下：

* **`git stash push`**

    该命令是缓存<u>未提交记录</u>的主要命令，当我们使用`git stash`相当于使用该命令。该命令常用选项如下：

    | 选项                       | 说明                                                         |
    | -------------------------- | ------------------------------------------------------------ |
    | `-m`/`--message`           | 保存时添加说明信息，便于在还原缓存时识别                     |
    | `-u`/`--include-untracked` | 保存**未跟踪**的文件（不包括被`.gitignore`忽略掉的文件）     |
    | `-k`/`--keep-index`        | 保存时保留**暂存区**内容，`git add`添加到暂存区的数据不会被还原，仅缓存**工作目录的更改**。<span style='color: cyan'>这里的`index`就是暂存区的另一种表示</span> |
    | `-a`/`--all`               | 保存所有文件变动以及**所有忽略/未跟踪**文件                  |
    | `-p`/`--patch`             | **交互式选择**要保存的内容（类似`git add -p`）               |

    我们使用`git stash list`可视化暂存堆栈可以看到形如`stash@{index}`的堆栈索引，**栈顶元素是最近一次存储（`stash@{0}`）**；

* **`git stash show <stash>`**：

    显示指定缓存的**文件变动摘要**，`<stash>`默认为`stash@{0}`，常用选项如下：

    | 选项                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | `-p`/`--patch`                                               | 展示具体的代码行差异（类似`git diff`）                       |
    | `-a`/`--all`                                                 | 显示所有文件的代码行差异                                     |
    | <span style='white-space: nowrap'>`-u`/`--include-untracked`</span> | 存储时使用了`-u`或`-a`选项保存了**未跟踪文件**，此选项会显示这些文件的差异 |

* **`git stash apply`**：

    该命令用于将**保存的文件变动**恢复到当前工作目录，但不同于`git stash pop`，该命令**不会删除存储条目**，常用选项如下：

    | 选项          | 说明                             |
    | ------------- | -------------------------------- |
    | **`--index`** | 恢复存储时，保留**暂存区**的状态 |
    | `--quiet`     | 静默模式，不显示操作信息         |

* **`git stash pop`**：

    该命令用于**恢复并删除**暂存栈顶的**最新保存的文件变动**，常用选项如下：

    | 选项          | 说明                             |
    | ------------- | -------------------------------- |
    | **`--index`** | 恢复存储时，保留**暂存区**的状态 |
    | `stash@{n}`   | 指定恢复特定位置的`stash`        |

    需要注意的是，`git stash`应用缓存时也有可能出现**冲突**，如果出现冲突或者其他事件，都会打断`git stash pop`的自动删除`stash`任务，这时需要我们手动完成后续操作。

* **`git stash clean`**：用于删除**所有暂存**的修改记录。

* **`git stash list`**：列出当前已缓存的暂存提交节点列表；

* **`git stash drop <stash>`**：用于删除Git暂存堆中指定的临时保存的修改记录，`<stash>`默认为`stash@{0}`。

* **`git stash branch <branch> <stash>`**：该命令用于**基于暂存记录**，在当前`HEAD`处**创建新分支**，并自动应用该`stash`的修改，`<stash>`默认为`stash@{0}`；

#### 🎯 `git  switch` & `checkout`

Git的分支切换是日常开发中常用操作之一，**我们可以使用`git checkout`或者`git switch`完成分支切换操作**。

`git switch`该命令是**Git 2.23**引入的新命令，用于替代`git checkout`中**切换分支**的功能。相比于`git checkout`，`git switch`语义更清晰、功能也聚焦于**分支切换**，其常用选项如下：

| 选项                                            | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| **`-c`/`--create`**                             | 创建并切换到新分支                                           |
| **`-C`**                                        | 强制创建分支，若分支已存在则重建分支                         |
| `-f`/`--force`/`--discard-changes`              | 强制切换，**放弃当前改动**                                   |
| `--detach`                                      | 进入**detached HEAD**状态                                    |
| <span style='color: cyan'>**`--orphan`**</span> | 创建一个不含任何提交的新分支。**使用该选项时，不需要再使用`-c`** |

`git switch --orphan`用于创建一个 **孤立分支** | `orphan branch` 。这种分支的特点是：**没有父提交，历史完全独立**，<u>适合需要完全隔离历史记录的场景</u>，例如：使用`Github Pages`部署静态网页时，我们不想要`gh-pages`有其他构建文件，我们可以使用该选项创建`gh-pages`。

需要注意的是，当我们使用此命令创建一个新的**孤立分支**时，**未追踪/忽略**文件会一并移入新分支的工作目录。部分情况下，暂存区中的数据也会移入新的分支中：

* [✔︎] 暂存区中**只有未追踪文件**的变更记录，此时进入新的孤立分支也会有这些数据；

* [✖] 暂存区中<u>存在已最终文件</u>的变更记录，此时创建新分支的任务会被中止，并出现如下报错：

    ```bash
    error: Your local changes to the following files 
    			would be overwritten by checkout:
            ...
    Please commit your changes or stash them before you switch branches.
    Aborting
    ```

    当然我们可以使用`-f`进行分支强行切换。但是此后**未追踪文件会被直接删除**，不会移入新分支的工作目录。

**`git checkout`是一个相当全能的Git命令**，它可以用来切换分支、恢复文件、检出特定提交等。这个命令在Git的多个操作中扮演关键角色，我们就在本章详细学习一下`git checkout`的用法。其常用选项如下：

| 选项                                                      | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| `-b <branch>`                                             | 创建并切换分支<br/><span style='color: gray'>*等价命令：`git switch -c <branch>`*</span> |
| `-B <branch>`                                             | **强制创建分支**，放弃当前改动<br/>**重置分支**，如果分支已存在，则会重置到当前`HEAD` |
| `<target>`=<br/>`<branch>`/`<commit>`/`<tag>`             | 切换到指定的<u>本地分支/具体提交/标签</u>上，这里我们用`<target>`统一代指 |
| `--detach <target>`                                       | 进入**detached HEAD**状态，`<target>`默认为`HEAD`<br/><span style='color: gray'>*等价命令：`git switch -d <target>`*</span> |
| `--ours <file>`/`--theirs <file>`                         | 合并冲突时选择版本，**也可以用`--`指定处理文件**<br/><span style='color: gray'>*等价命令：`git restore --ours`/`--theirs`*</span> |
| `-f`/`--force`                                            | 强制切换分支，忽略未保存更改<br/><span style='color: gray'>*等价命令：`git switch -f <target>`*</span> |
| <span style='color: cyan'>**`<target> -- <file>`**</span> | 恢复特定`<target>`提交的文件<br/><span style='color: gray'>*等价命令：`git restore --source <target> -- <file>`*</span> |

#### 🏷️ `git tag`

`git tag`用于给某个**特定提交**打标签的一条命令。标签一般用于标记重要的版本点，比如发布版本，方便后续查找、**检出** | `checkout` 和管理。我们可以使用该命令**创建**和**管理**标签，以下是常用的<u>创建标签选项</u>：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`-a <tag> [<commit>]`/`--annotate`**                       | 创建一个 **附注标签** \| `annotated tag` 。我们常使用`git tag -a <tag> [<commit>]`创建标签。我们在创建标签时，可以指定`commit`作为打标目标，如果不指定则会默认当前`HEAD` |
| `-s <tag>`/`--sign`                                          | 创建**GPG签名标签**，需要预先配置`user.signingkey`           |
| <span style='white-space: nowrap'>`-u <pub_fp>`/`--local-user=<pub_fp>`</span> | 使用指定的GPG密钥ID（`pub_key`）创建**GPG签名标签**          |
| `-m <message>`                                               | 为标签添加说明信息，需配合`-a`/`-s`使用                      |

了解完Git标签如何创建后，我们再来学习一下如何管理标签。我们可以使用以下命令**列出标签**：

* **`-l [<pattern>]`/`--list`**：用于**列出标签**，直接运行`git tag`就等于执行了`git tag -l`，常用的通配符如下：

    | 通配符 | 说明         |
    | ------ | ------------ |
    | `*`    | 任意字符     |
    | `?`    | 单个任意字符 |
    | `[]`   | 字符集合     |

    例如，我们可以使用`git tag -l "v1.*"`来匹配所有以 `v1.` 开头的标签；

* **`--sort=<key>`**：用于**排序标签列表**，Git默认是按字典序排序（不是按时间），可以通过`--sort`指定排序规则，常见的排序字段有：

    | 排序字段        | 说明                                         |
    | --------------- | -------------------------------------------- |
    | `refname` ✅     | 标签名称，字典序排序                         |
    | *`creatordate`* | 标签创建日期                                 |
    | *`taggerdate`*  | 打标签者的时间                               |
    | `version`       | 按照**语义化版本**排序（比如`v1.9`<`v1.10`） |
    | **`-<key>`**    | 反向排序                                     |

* **`--contains <commit>`**：列出包含某提交的标签；

* **`--merged <commit>`**：列出已合并到某提交的标签；

* <span style='color: cyan'>**`--format=<format>`**</span>：自定义输出格式。我们可以自定义每个标签的输出格式，类似`git log --pretty=format`的用法，常用占位符如下：

    | 占位符                 | 说明                             |
    | ---------------------- | -------------------------------- |
    | `%(*objectname)`       | 对应提交的`SHA1`                 |
    | `%(refname)`           | 标签**全名**（`refs/tags/v1.0`） |
    | `%(refname:short)`     | 标签**短名**（`v1.0`）           |
    | `%(taggername)`        | 打**标签人**                     |
    | `%(creatordate:short)` | 创建时间（简短）                 |
    | `%(subject)`           | 标签说明第一行                   |

关于Git标签，还有以下可能会用到的命令：

* **删除标签**：使用`git push <remote> --delete <tag>`删除远程仓库中的标签，也可以使用`git tag -d <tag>`删除本地标签；
* **查看详情**：使用`git show <tag>`显示标签内容和对应的提交详情；
* **验证签名**：使用`-v <tag>`选项，验证指定标签的GPG签名；

#### 🌐 `git worktree`

`git worktree`命令用于**在同一个仓库`Repository`下管理多个工作目录`Working Directory`**。这对于需要同时处理多个分支的开发场景非常有用。

当我们在需要在两个分支上同时工作时，频繁切换分支会浪费我们的时间，这时候可以使用`git worktree`将<u>某一个分支 **检出** | `checkout`</u>到**新的工作目录**下，同时保留主仓库的原始工作目录，这样我们就可以在多个分支上同时工作。`git worktree`常用命令如下：

* **`git worktree add <path> <branch-or-commit>`**：

    该命令是我们**创建新工作目录**的主要命令。其中`<path>`用于指定目录路径；`<branch-or-commit>`可选，指定用于`checkout`的`branch`或者`commit`，其参数设置有如下几种情况：

    * **指定已存在分支**：Git会新建一个新目录并`checkout`在这个分支上；
    * **不指定分支**：Git会在新目录中根据`path`创建一个新的分支，并`checkout`这个分支；
    * **指定提交**：Git会在新目录下以**detached HEAD**状态`checkout`这个提交；

    `git worktree add`的常用选项如下：

    | 选项                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | `-b <new-branch>`                                            | 创建并`checkout`一个新分支（相当于 `git checkout -b`）       |
    | `--detach`                                                   | 以**detached HEAD**模式`checkout`一个提交                    |
    | `--force`                                                    | 强制覆盖已存在的**工作目录**`<path>`，需要谨慎使用           |
    | <span style='white-space: nowrap'>`--checkout`/`--no-checkout`</span> | 控制添加**工作目录**后，是否显式执行`checkout`，**默认是`--checkout`** |
    | **`--lock`**                                                 | 锁定<span style='color: cyan'>工作树</span>，防止其被删除或移动 |
    | `--orphan <branch>`                                          | 创建一个**孤立分支**并`checkout`（相当于`git checkout --orphan`） |

    两个工作目录间如何合并，可以执行`worktree`的**Merge Mock**实验：

    ```bash
    # Experiment-1
    # main
    git checkout first
    git switch -c work_main
    git worktree add ../sub_work
    git lg
    git checkout sub_work 
    # fatal: 'sub_work' is already used by worktree at 'D:/Code/Javascript/Learn/git-learn'
    
    # sub
    cd ../sub_work
    git commit --allow-empty
    cd ../learn-git
    git lg
    # 在sub_worktree中对分支的修改已经影响到main_worktree中的提交
    # 这说明工作目录不同分支间是同步提交的
    ```

    上述实验有如下总结：

    * 不同工作目录不能同时处于一个分支上；
    * 工作目录间不同分支的提交记录是同步的；

* **`git worktree move <old_path> <new_path>`**：

    该命令用于**移动工作目录（非主工作目录`main`）**，使用此命令移动工作目录**不会改变其绑定的分支或提交信息**；我们可以使用`mv`作为`move`的缩写命令。

    需要注意的是，`git worktree lock`会影响此命令的执行，当然我们可以使用`-f`/`--force`选项来<u>强制更改</u>工作目录位置。以下是要点总结：

    * ✅ 自动更新 Git 的内部路径记录；
    * ⚠️ 手动移动目录会导致 Git 记录失效；
    * ❌ 不支持主仓库工作目录移动；
    * ❗ `​<old_path>`必须为有效路径，`<new_path>`必须存在；

* **`git worktree remove <exist_path>`**：用于**删除工作目录（非主工作目录`main`）**，可以使用`rm`作为`remove`的缩写命令。如果待操作<u>工作目录</u>被锁定，需要解锁或者使用`-f`/`--force`强制执行；

* **`git worktree prune`**：

    该命令是管理`worktree`的重要工具，用于清理Git仓库中**已经失效的工作树记录**，以确保`.git/worktrees/`目录中的信息保持干净一致。其常用选项如下：

    | 选项                               | 说明                                                         |
    | ---------------------------------- | ------------------------------------------------------------ |
    | `-n`/`--[no-]dry-run`              | 控制是否执行`prune`命令                                      |
    | `-v`/`--[no-]verbose`              | 控制是否显示每个被清除的工作树信息                           |
    | **` --[no-]expire <expiry-date>`** | **用于清理“临时锁定”的工作树**，<span style='color: cyan'>只对锁定工作目录生效</span><br/>时间格式可为：`1.hour.ago`、`2.days.ago`、`now`、RFC时间等 |

* <span style='color: gray'>**`git worktree repair <path>`**</span>：该命令用于**修复`.git/worktrees/`元数据目录**，主要用于在某些意外情况下（如物理目录移动、系统崩溃、备份还原等）**重新同步Git对工作树的识别和绑定**；

* **`git worktree list`**：该命令用于**可视化所有的工作目录**。可以使用`-v`/`--verbose`显示更多信息；使用`--[no-]expire <expiry-date>`可以根据`prune`过期时间显示信息；

* **`git worktree lock <path>`**：该命令用于**将某个工作目录锁定**，防止它被误删或自动清理。我们可以使用`--reason <reason_msg>`说明锁定原因；

* **`git worktree unlock`**：该命令用于**将某个工作目录解锁**；

### 2. 编辑

对分支的编辑操作是Git分支操作的核心，

#### 🤝 `git merge`

`git merge`命令用于将多个分支的内容合并成一个新的提交。如果`HEAD`为**detached HEAD**状态，需要指定合并主分支、带合并分支；否则Git会默认主分支为`HEAD`指向的分支。其常用选项如下：

| 选项                 | 说明                                                       |
| -------------------- | ---------------------------------------------------------- |
| **`--ff`/`--no-ff`** | 控制是否进行<u>快速合并</u>，**默认为进行快速合并**        |
| **`--ff-only`**      | 只允许<u>快速合并</u>合并，不能快速合并时会报错            |
| **`--squash`**       | 合并多个提交为一个，但不会产生`merge commit`，需要手动提交 |
| `--commit`           | **默认行为**，合并成功后自动创建提交                       |
| `--no-commit`        | 合并后不自动提交，可用于预览更改或做修改                   |
| `--edit`             | 打开编辑器修改默认合并提交信息（默认开启）                 |
| `--abort`            | 取消当前冲突中的合并过程，回到合并前的状态                 |
| `--continue`         | 在解决冲突后继续合并                                       |

**快速合并** | `fast-forward` 适用与如下场景，当我在一个分支`sub`上开发，这时`main`没有任何新的提交，当我们使用`git merge --ff`合并时，Git 可以通过`--ff`将`main`分支快速指向`sub`分支的最新提交以达成快速地合并。`--ff`/`--no-ff`的实际效果如下：

* **`--ff`效果**：

    ```bash
    A---B---C (main)
             \
              D---E (feature)
    
    git merge feature --ff
    
    A---B---C---D---E (main)
    ```

* **`--no-ff`效果**：

    ```bash
    A---B---C (main)
             \
              D---E (feature)
    
    git merge feature --no-ff
    
    A---B---C---------M (main)
             \       /
              D-----E (feature)
    ```

当我们想要将**孤立分支合并到主分支**上时，如果直接使用`git merge`会出现 *“警告没有共同祖先”* 的报错，如果我们想要合并它们可以使用`--allow-unrelated-histories`选项执行强行合并。此外，`git merge`的多分支合并可以参考[Git笔记：背景与基础](https://soppylzz.github.io/soppy-ie/2025/05/07/git-note-1/#🧭-Git引用与运算符)；

当`git merge`合并时若发生冲突，Git会依次打开冲突文件（如果多个的话），我们可以通过`git mergetool`使用**冲突合并工具**处理冲突。

#### 📏 `git rebase`

`git rebase`也可用于分支的合并，它的合并原理是 **变基** | `rebase` 一个分支到另一个分支的基础之上，从而实现更整洁、线性的提交历史。此命令同`git merge`一样，需要区分不同`HEAD`状态下的参数输入：

* **`detached`**：使用`git rebase <base_branch> <attach_branch>`进行变基，先输入作为”基“的分支，再输入待变基的分支。例如：`git rebase rb1 rb2`就是将`rb2`的提交变基到`rb1`上；

* **`branch`**：使用`git rebase <base_branch>`进行变基，是将当前分支变基到输入分支上；

    ```bash
    git checkout first
    git branch rb1
    git branch rb2
    
    # rb1, rb2做如下操作
    git checkout rb1
    git commit --allow-empty
    
    # 下面两种变基方式等价
    git checkout rb1
    git rebase -i rb2
    
    git lg
    git checkout <rb2_commit_hash>
    git rebase -i rb2 rb1
    ```

需要注意的是，使用`git rebase`进行分支融合，**待变基分支的指向会随着命令执行而改变**。`git rebase`的常用选项如下：

| 选项                                                      | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| <span style='color: pink'>**`-i`/`--interactive`**</span> | 进入**交互式变基**模式，可对每个提交进行操作（如**修改、合并、重排**等） |
| `--continue`                                              | 解决变基冲突后继续执行变基操作                               |
| `--skip`                                                  | 跳过当前出错的提交，不再应用它                               |
| `--abort`                                                 | 取消变基，恢复到开始前的状态                                 |
| **`--keep-empty`**                                        | 默认变基会丢弃空提交，使用该选项**保留空提交**               |
| `--autostash`                                             | 在变基前自动暂存工作区的变更，并在变基后恢复                 |
| <span style='color: gray'>`--exec <cmd>`</span>           | 在每个`rebase`的提交之后执行一个命令（可用于测试）           |

#### 🍒  `git cherry-pick`

`git cherry-pick`用于将提交从一个分支<span style='color: cyan'>**复制到当前分支**</span>，<u>这不会改变原分支的指向</u>。其使用方法如下：

* `git cherry-pick <commit-hash>`：复制单个提交；
* **`git cherry-pick <start-commit>..<end-commit>`**：复制一组提交，**注意提交先后顺序**；
* `git cherry-pick <hash1> <hash2> <hash3>`：按照先后顺序依次复制提交；其等价为：

    ```bash
    git cherry-pick <hash1>
    git cherry-pick <hash2>
    git cherry-pick <hash3>
    ```

其常用选项如下：

| 选项          | 说明              |
| ------------- | ----------------- |
| `<commit>`    | 应用指定提交      |
| `-e <commit>` | 修改提交信息      |
| `-x <commit>` | 添加原提交信息    |
| `--abort`     | 放弃`cherry-pick` |
| `--continue`  | 解决冲突后继续    |

## 调试命令

### 1. 搜索

#### 🔭 `git grep`

`git grep`用于**在版本库中的文件里<u>搜索字符串</u>或<u>正则表达式</u>**。由于它只在Git跟踪的文件中查找，它的速度比普通`grep`命令快。我们可以按照如下方式使用此命令：

* `git grep -e <pattern>`：在项目中的 *文件/文件夹* 下查找匹配`<pattern>`的内容。需要注意，Git并不会匹配**文件夹名称**；
* `git grep -e <pattern> -- <folder-or-filer>`：使用`--`指定查找的范围；

`git grep`的常用选项如下：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <span style='color: gray'>`-e <pattern>`</span>              | 显式指定模式，支持正则表达式                                 |
| `-i`                                                         | 忽略大小写进行匹配，默认区分大小写                           |
| **`-w`**                                                     | **只匹配整个单词**，避免部分匹配                             |
| `-n`/`--line-number`                                         | 显示匹配的**行号**                                           |
| <span style='color: gray'>*`-c`/`--[no-]count`*</span>       | 显示匹配成功文件，与匹配行数                                 |
| <span style='color: gray'>*`-l`/`--[no-]files-with-matches`*</span> | 显示匹配成功文件，不显示行数                                 |
| `-L`/`--files-without-match`                                 | 列出不包含匹配内容的文件名                                   |
| *`-A <n>`*                                                   | 匹配行 **后** 额外显示 `n` 行上下文                          |
| *`-B <n>`*                                                   | 匹配行 **前** 额外显示 `n` 行上下文                          |
| *`-C <n>`*                                                   | 匹配行前后各显示 `n` 行上下文（等价于 `-A n -B n`）          |
| `--untracked`                                                | <span style='color: cyan'>**Git默认只搜索已跟踪的文件**</span>，使用此选项可以搜索未被跟踪的文件 |
| `--cached`                                                   | 只在**暂存区**中搜索                                         |
| `--no-index`                                                 | 使用此命令可以搜索非Git管理的仓库（即根目录下没有`.git`的仓库）中的文件 |

#### <span style='color: cyan'>🔍 `git bisect`</span>

`git bisect`用于**快速定位引入bug的提交**。它采用**二分查找算法**，通过标记「好」和「坏」的提交，自动帮你在提交历史中缩小范围，直到找出引入问题的那一个提交。

我们使用`git bisect start`开始二分查找错误提交，使用`git bisect bad`标记错误提交，使用`git bisect good <commit>`来标记**已知**没有的提交。然后，Git会自动`checkout`中间提交，**我们只需测试每次`checkout`的提交是否有问题**，并用`git bisect good/bad`告诉Git即可，当找出有问题的提交后，Git 会输出类似：

```bash
<commit_hash> is the first bad commit
```

#### 🧾 `git reflog`

`git reflog`用于查看本地 Git 仓库的 **引用日志** | `reference log` 。它记录了**所有分支引用（如 `HEAD`）变动的历史**，即使某些提交已经被`reset`、`rebase`、`checkout`、`commit --amend`等操作「隐藏」或「丢弃」，也能通过 `reflog` 找回。`git reflog`的常用命令如下：

* **`git reflog [show]`**：

    该命令用于显示本地仓库的**引用**（`HEAD`、`<branch>`）的历史记录。当我们使用`git reflog`时，实际上就是在使用该命令。其常用选项如下：

    | 选项                              | 说明                                                         |
    | --------------------------------- | ------------------------------------------------------------ |
    | `<target>`                        | 指定查看的引用，默认是`HEAD`，也可以用于**分支引用**的查看   |
    | `-n <N>`<br/> `--max-count=<N>`   | 限制显示最近的`<N>`条记录                                    |
    | `--pretty=<format>`               | 格式化输出，支持格式：`oneline`、`short`、`medium`、`full`、`raw`等 |
    | `--date=<format>`                 | 设置日期显示格式，支持：`relative`、`iso`、`local`、`default`、`raw` |
    | `--abbrev=<N>`/<br/>`--no-abbrev` | 设置提交哈希缩写长度，**默认是 Git 配置的 `core.abbrev`**，可以手动设置。<br/>也可以使用`--no-abbrev`关闭哈希缩写 |
    | `--all`                           | 查看所有分支和`HEAD`的`Reflog`                               |

* **`git reflog expire`**：

    该命令用于**清理引用日志**。当我们使用Git编辑提交树时，会有一些节点从树上消失，但Git会将这些消失的提交记录保存起来。这些提交记录Git默认保存90天，我们也可以使用`git reflog expire`根据指定策略，删除不再需要保留的`reflog`；

    注意！`git reflog expire`并不直接删除对象，而是让这些记录“过期”，**供后续`git gc`（垃圾回收）真正清理**。其常用选项如下：

    | 选项                          | 说明                                                         |
    | ----------------------------- | ------------------------------------------------------------ |
    | `--expire=<time>`             | 指定普通（非`HEAD`）`reflog`的过期时间（**默认90天**）       |
    | `--expire-unreachable=<time>` | 指定**不可达对象**的过期时间（**默认30天**）                 |
    | `--all`                       | 作用于所有引用                                               |
    | `--rewrite`                   | Git默认行为是**直接删除整个日志文件**，使用该选项可以保留**日志文件**中仍然有效的条目，只删掉过期的部分 |

* **`git reflog delete <reflog>`**：该命令用于**手动删除特定`reflog`记录**的命令；

### 2. 分析

#### 🧐 `git diff`

`git diff`是Git中最重要的几个命令之一，它为我们提供了**行级别**展示代码间 **差异** | `diff` 的方案。我们可以使用如下方式使用该命令：

* **`git diff`**：当我们使用该命令时，情况更为复杂。
    * `git diff`默认比较**暂存区**与**工作区**；
    * `git diff --cached`/`--staged`用于比较<u>暂存区与`HEAD`</u>的差距；
* **限制范围**：
    * 我们可以使用`-- <file-or-folder>`限定Git比较范围，范围既可以是一个单独文件，也可以是个文件夹；
    * 我们也可以使用`git diff <ref>:<file-or-folder> <ref>:<file-or-folder>`的形式更加自由的使用`diff`工具。这里的`<ref>`是指Git引用，`<file-or-folder>`同上；

其常用选项如下：

| 选项                                    | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| `-u`/`-p`                               | 生成**补丁**格式的差异输出，这是默认选项，不需要显示输入     |
| `--patch-with-stat`                     | 输出**补丁**并附带统计信息                                   |
| **`--stat`**                            | 显示简略**统计**（修改的文件及增减行数）                     |
| `--numstat`                             | 显示数字**统计**（增/减行数）                                |
| `--name-only`                           | 显示修改的**文件名**                                         |
| `--name-status`                         | 显示**文件名**、<u>变更状态</u>（`M` - 修改，`A` - 添加，`D` - 删除） |
| **`--color-words`**                     | 高亮显示<span style='color: pink'>单词级</span>差异，适合`Markdown`、`Latex`等文本内容 |
| **`-w`**                                | 忽略<u>所有</u>空格变化                                      |
| **`-b`**                                | 忽略<u>行末</u>空格                                          |
| <span style='color: gray'>*`-z`*</span> | 用`NUL`字符分隔输出（适合脚本解析）                          |
| *`--full-index`*                        | 显示完整的对象哈希值，默认显示短哈希                         |
| `--no-index <file> <file>`              | 直接比较两个文件间的差异，不需要考虑是否为Git项目，`git grep`中也有该选项 |

`git diff`有如下常用的`combo`：

* **`git diff <commit>^!`**：对比某个提交与其父提交的差异。`^!`是Git的语法糖，`<commit>^!`等价于`<commit>^ <commit>`；
* **`git diff <target>..<target>`**：比较两个提交间的差异，我们也可以使用`<target> <target>`来代替原输出，两种效果相同；
* <span style='color: cyan'>**`git diff <branch_1>...<branch_2>`**</span>：比较<u>`<branch_2>`</u>与<u>`<branch_1>`的最近**共同祖先**</u>的差异；

我们也可以使用`git difftool`调用第三方工具来查看`diff`，我们只需要将上述`git diff`改为`git difftool`即可。**注意，使用`git difftool`时，不需要添加`options`**。

#### 😠 `git blame`

`git blame`用于**逐行追溯文件修改历史**的命令。它能帮助开发者快速查看代码的每一行最后一次被修改的提交、作者和时间。我们可以使用`git blame <file>`来查看文件的提交记录，其常用选项如下：

| 选项                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `-e`/`--show-email`    | 显示作者的邮箱地址                                           |
| **`-L <start>,<end>`** | 限制文件查看范围，聚焦于某几行                               |
| **`-w`**               | 忽略空格修改                                                 |
| `-C`/`-M`              | 检测代码是否是从其他文件移动或复制而来，其中：<br/>- `-C`用于检查同一提交内的代码移动<br/>- `-M`用于检测跨提交的代码移动 |

#### 🐲 `git ls-files`

`git ls-files`是用于显示Git各个分区中文件信息的命令。它主要用于查看Git跟踪的文件状态，适合<u>脚本处理</u>或<u>精确查询文件</u>状态。当我们直接使用`git ls-files`时，Git会列出索引中所有**已跟踪的文件**，其常用选项如下：

| 选项                   | 说明                                                   |
| ---------------------- | ------------------------------------------------------ |
| `-c`/`--cached`        | 列出所有已被**跟踪**且**在暂存区**中的文件             |
| `-m`/`--modified`      | 列出已被**跟踪**，但工作目录中**有修改且未暂存**的文件 |
| `-d`/`--deleted`       | 列出索引中存在但工作目录中已删除                       |
| `-o`/`--others`        | 列出未被Git跟踪的文件（默认排除`.gitignore`中的文件）  |
| `-i`/`--ignored`       | 列出被`.gitignore`规则忽略的文件                       |
| `-s`/`--stage`         | 显示文件的模式、哈希值和冲突阶段（用于合并冲突时）     |
| `--recurse-submodules` | 若项目包含子模块，递归列出子模块中的文件               |
