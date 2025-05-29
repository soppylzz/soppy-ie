---
title: Git笔记：远程与实践
date: 2025-05-23 16:35:53
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

## 远程命令

我们在前面的章节中已经了解了常用的**Git本地命令**，但是Git作为**分布式**版本控制系统是如何保证各个分布式节点的一致性呢？答案就是 **远程仓库** | `remote repository` 。 下面我们将从**Git通用远程命令**与**Github**两个方面学习Git与远程仓库的交互。

### 1. 配置

我们可以使用`git remote`管理当前项目的远程仓库链接：

* **仓库可视化**：我们可以使用如下命令可视化远程仓库配置：

    * `git remote [-v/--verbose]`：

        查看所有远程仓库**简称**，这里的`-v`/`--verbose`选项是`remote`及其子命令的可视化选项，其只能紧随`remote`出现；

    * `git remote [-v/--verbose] show <name>`：

        查看一个具体仓库的详细信息，可查询的信息包括：`url`、`HEAD`、`remote branches`等。该命令默认启用`-v`选项，无需显式添加。

        此外，`git remote show`在执行时会默认**实时访问**远程仓库以获取最新信息，我们可以使用`-n`（提示解释为：do not query remotes）跳过对远程仓库的访问，以提升执行效率。

* **`url`管理**：我们可以使用如下命令管理远程仓库的`url`：

    * `git remote set-url <name> <newurl> [<oldurl>]`：用于修改某个已存在远程仓库的`url`，**常用于协议变更**；

        这里的`<oldurl>`是可选项，当远程仓库存在多个`url`时，可以指定要替换的旧`url`；我们也可以**使用`--push`选项**，单独修改`push`的`url`，不影响`fetch`的`url`；

    * `git remote get-url <name>`：用于查询远程仓库的`url`，这里也可以使用`--push`选项指定查询对象；

* **仓库管理**：

    我们可以**使用`git remote add`给项目添加远程仓库**，该命令常用选项如下：

    | 选项                  | 说明                                                         |
    | --------------------- | ------------------------------------------------------------ |
    | `-f`/`--fetch`        | 在添加远程仓库后，立即从远程仓库拉取数据（相当于随后执行一次 `git fetch <name>`） |
    | `--tags`              | 配合 `--fetch` 使用时，仅拉取远程的 tag 信息，不包括分支     |
    | `-t/--track <branch>` | **限制拉取**的分支，仅跟踪指定的远程分支（可多次指定）。<u>默认情况下，Git会拉取所有远程分支</u> |

    我们可以**使用`git remote rename <old> <new>`给远程仓库的重命名**；**使用`git remote remove <name>`移除某个远程仓库**，或者使用`git remote remove`的缩写`git remote rm`；

* **`fetch`设置**：

    Git本地仓库有一个指向当前 **检出** | `checkout` 的引用：`HEAD`。同样Git远程仓库也有一个类似的引用`<branch>/HEAD`用于表示**远程仓库的默认分支**。当远程仓库的默认分支发生变化时，我们可以使用`git remote set-head <branch>`命令更改远程仓库默认分支的**本地追踪引用**：

    | `set-head`选项  | 说明                                  |
    | --------------- | ------------------------------------- |
    | `<branch>`      | 指定具体分支名称，例如：`origin/main` |
    | `-a`/`--auto`   | 自动从远程仓库读取并设置其默认分支    |
    | `-d`/`--delete` | 删除本地的`remote/HEAD`引用           |

    需要说明的是，<span style='color: gray'>`<remote>/HEAD`的具体使用场景我目前**尚未深入了解**</span>，因此此处暂不展开，后续将在学习与实践中进一步补充。

    默认情况下，`git fetch`会拉取远程仓库的**所有分支引用**；我们可以使用`git remote set-branches <remote>`**限制fetch指定分支**。该命令常用选项如下：

    | `set-branch`选项 | 说明                                                 |
    | ---------------- | ---------------------------------------------------- |
    | `<branch>`       | 设置fetch的 <u>*远程分支名称*</u> ，可以同时设置多个 |
    | `--add`          | 在已有的基础上追加分支，不覆盖原有设置               |

    **这些配置我们也可以通过编辑`.git/config`的内容进行更改。**

* **`git remote prune <name>`**：

    该命令用于**清理**本地已经被删除的**远程分支**。简单来说，如果某个远程分支已被他人删除，而我们本地还保留了它的追踪分支（如 `remotes/origin/feature-x`），那么可以通过`git remote prune <remote>`命令清除这些 *<u>无效引用</u>* 。此外，我们也可以使用`--dry-run`来模拟执行该命令，**仅输出将要被删除的引用**，不实际删除。

### 2. 操作

#### ⬆️ `git push`

`git push`是Git中用于将**本地仓库的提交上传到远程仓库**的命令，是远程协作开发中的核心命令之一。我们可以以如下形式使用该命令：

* <span style='color: crimson'>**`git push`**</span>：

    将<u>当前分支</u>推送到远程仓库的 *<u>特定分支</u>* ；我们通过配置`push.default`参数，来控制`git push`默认推送行为：

    | 模式                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | **`simple`**✅（Git 2.x）                                     | 仅推送当前分支到与其名称相同的远程分支，并**要求它已设置为`upstream`**。若无跟踪关系会报错。更安全，防止意外推送错误分支。 |
    | <span style='white-space: nowrap'>**`matching`**⬇️（Git 1.x）</span> | 推送所有本地分支到同名的远程分支（如果远程存在）。适用于多人共享所有分支的仓库，但风险高。 |
    | `current`                                                    | 只推送当前分支到远程的**同名分支**，不管有没有设置`upstream`。适合个人项目。 |
    | `upstream`<br/>`tracking`                                    | 推送当前分支到它的`upstream`分支（即我们用`-u`设置过的远程分支）。 |
    | `nothing`                                                    | 默认不推送任何分支，需要我们<u>每次手动指定</u>。            |

* **`git push <remote> <lo_branch>`**：推送`<lo_branch>`推送到远程仓库。

* **`git push <remote> <re_branch>:<lo_branch>`**：

    这是 `git push`的完整使用方式。除了支持定制化的推送操作，该命令还可用于**删除远程分支**：`git push <remote> :<lo_branch>`等价于`git push <remote> --delete <lo_branch>`。

    <span style='color: gray'>注意：此处删除的是 `<lo_branch>` 所追踪的远程分支。其绑定的远程分支取决于是否在创建本地分支时，通过 `git branch <branch> --track <re_branch>` 显式指定了追踪目标。</span>

`git push`有很多常用的选项，用于控制推送的行为。以下是其常见选项：

| 选项                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **<span style='white-space:nowrap'>`-u`/`--set-upstream`</span>** | 将本地分支与远程分支建立跟踪关系，便于以后直接使用`git push`、`git pull`和`git fetch` |
| `--all`                                                      | 推送所有本地分支（不推荐在多人协作下使用）                   |
| **`--tags`**                                                 | 推送所有本地的Git标签`tag`到远程                             |
| <span style='color: crimson'>**`-f`/`--force`**</span>       | **强制推送**，覆盖远程分支的历史；<br/>当我们修改了本地提交历史，远程与本地存在**历史冲突**，需要使用此选项推送到远程仓库 |
| **`--force-with-lease`**                                     | 更安全的强推，避免覆盖他人提交（**推荐**）                   |
| `--delete`                                                   | 删除<u>远程分支</u>或<u>标签</u>                             |
| `--dry-run`                                                  | 模拟推送，不实际更改远程仓库                                 |
| `-v`/`--verbose`                                             | 输出详细信息                                                 |
| `--mirror`                                                   | 推送所有`refs`，常用于仓库备份或迁移                         |

#### ⬇️ 仓库拉取

我们可以使用`git fetch`命令从远程仓库获取最新的数据（分支、标签、提交等），这些数据只会被拉取到本地的远程分支上，<span style='color: steelblue'>**需要我们手动合并到本地分支上**</span>。我们可以以如下命令形式使用该命令：

* **`git fetch <remote>`**：使用该命令获取`<remote>`仓库的 <u>*所有远程分支*</u>、<u>*标签*</u>、<u>*引用*</u> 等内容；
* **`git fetch <remote> <lo_branch>`**：只拉取`<lo_branch>`追踪的远程分支的更新内容；
* <span style='color: gray'>`git fetch --all`</span>：如果我们添加了多个远程仓库，可以使用该命令拉取所有仓库的数据；

`git fetch`的常用选项如下：

| 选项               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `--all`            | 拉取所有远程仓库                                             |
| **`--prune`**      | 删除<u>本地</u>已经**失效的远程跟踪分支**（远程仓库中已被删除的分支），<br/>`git fetch --prune <remote>`等价于`git remote prune <remote>` |
| `--dry-run`        | 模拟拉取操作，不会更改本地远程仓库                           |
| `--tags`           | 抓取所有的`tag`，即使这些`tag`没有和分支关联                 |
| **`-f`/`--force`** | 强制更新远程跟踪分支，即使会导致非`fast-forward`更新，对应`git push -f` |
| **`--depth=<n>`**  | 进行浅克隆，只抓取最新的`<n>`层提交                          |
| **`--unshallow`**  | 如果仓库是浅克隆的，这个选项会补全历史，变成完整克隆         |

当我们使用`git fetch`拉取完远程仓库数据后，还需要手动执行`git merge`合并本地仓库内容与远程仓库拉取下来的内容。该流程提供了更细致的合并控制，但操作上相对繁琐耗时。此时，可使用`git pull`命令一次性完成**拉取与合并**，该命令适用于更简单、自动化的场景。

同`git push`类似，<span style='color: steelblue'>`git pull`也可以用`git config`设置其**默认处理方式**</span>。可供设置的配置项有：`pull.rebase`与`pull.ff`。

其中，`pull.rebase`配置项的可选值如下：

| 可选值        | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| `false` ✅     | 使用`merge`方式合并远程更改<br/>普通团队协作，保留合并历史   |
| `true`        | 使用`rebase`将本地更改移到远程之后<br/>保持提交历史线性（如 **GitHub Fork** 场景） |
| **`merges`**  | 使用`rebase`，并**保留原有`merge commit`**（Git ≥ 2.18）<br/>需要保持分支合并信息 |
| `interactive` | 启用交互式`rebase`（Git ≥ 2.26）<br/>需要手动编辑每条提交    |

`pull.ff`配置项的可选值如下：

| 可选值   | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| `true` ✅ | 允许`fast-forward`合并<br/>无本地提交的场景，保持历史简洁    |
| `false`  | 禁止`fast-forward`，强制生成`merge commit`<br/>明确需要保留合并历史，如 PR 合并记录 |
| `only`   | **只**允许`fast-forward`，否则中止`pull`<br/><u>强制保持线性历史</u>，避免任何`merge commit` |

`git pull`的常用选项如下：

| 选项                                           | 类型     | 说明                                                         |
| ---------------------------------------------- | -------- | ------------------------------------------------------------ |
| **`--rebase`**                                 | 变基     | **Git默认使用`git merge`的方式**合并远程仓库与本地仓库，<br/>该选项使用**变基**而不是合并来整合远程更改 |
| `--no-rebase`                                  | 变基     | 显式要求使用合并方式，即使配置中默认使用`rebase`             |
| **`--ff-only`**                                | 合并策略 | 仅允许 **快进合并** \| `fast forward` ，不允许需要合并的场景 |
| `--no-ff`                                      | 合并策略 | 即使可以<u>快进合并</u>，也强制生成一个`merge commit`，便于保留分支合并记录 |
| `--no-commit`                                  | 合并策略 | 合并后不自动创建提交，我们**可以手动检查合并结果**、修改后再提交 |
| **`--squash`**                                 | 合并策略 | 把拉取的更改<span style='color: steelblue'>**压缩成一个提交并合并**</span>进来，不保留每个单独的提交记录<br/>这只是合并更改，<u>需要我们手动提交</u> |
| <span style='color: gray'>`--depth <n>`</span> | 拉取控制 | 只获取最近的`n`个提交记录                                    |
| `--tags`                                       | 拉取控制 | **<u>*同时*</u> 拉取标签**                                   |
| `--all`                                        | 拉取控制 | 拉取并合并所有远程仓库                                       |
| `--autostash`                                  | 本地保护 | 在`pull`时，自动`stash`本地更改                              |
| <span style='color: gray'>`--no-edit`</span>   | 编辑控制 | 合并过程中不启动编辑器，自动使用默认的合并信息               |

Git远程仓库还有一个命令：`git clone <url>`，该命令等价于：

```shell
git init
git remote add origin <repo>
git fetch origin
git checkout -b <default-branch> origin/<default-branch>
```

`git clone`的具体使用方法将在后续有时间时补充，欢迎关注更新。

## Github实践

在使用GitHub协作时，良好的账户配置和开发流程可以大幅提升效率和安全性。下面我们详细了解一下**SSH Key、GPG Key、2FA**等安全配置，以及**Pull Request**和**GitHub Actions**的使用方法，以帮助我们更好的使用Github。

### 1. 配置

#### 🔐 SSH Keys

Git支持通过 ***HTTP*** / ***HTTPS*** 或 ***SSH*** 协议与远程仓库通信，两者在身份验证和安全机制上有显著不同：

| 特性           | HTTP/HTTPS                                                   | SSH                           |
| -------------- | ------------------------------------------------------------ | ----------------------------- |
| 🧩 **协议样例** | `https://github.com/<your_path>`                             | `git@github.com/<your_path>`  |
| 🔒 **验证方式** | 用户名+密码/`Token`                                          | 公钥+私钥（无需每次输入）     |
| 📅 **适合场景** | 临时操作、`CI`/`CD` 脚本                                     | 常规开发、本地部署、长期使用  |
| ⭐ **安全性**   | 中（依赖 `Token` 或密码保护）                                | 高（加密通道+非对称密钥认证） |
| 🎯 **易用性**   | 每次 `push`/`pull` 需认证<br/>（可以使用 `credential.helper` 缓存） | 登录一次后自动认证            |

关于`HTTP`/`HTTPS`的配置细节我们已经在[Git笔记：背景与基础](https://soppylzz.github,io//soppy-ie/2025/05/07/git-note-1/)中详细介绍了，这里我们着重学习下 ***SSH*** 的相关配置；

我们打开 ***Terminal*** 执行如下命令，生成一对 *SSH* 密钥：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

其中：

| 选项                                                      | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| **`-C`**                                                  | 注释，一般填写我们的邮箱地址                                 |
| <span style='white-space: nowrap'>**`-t ed25519`**</span> | 使用**Ed25519**算法，我们也可以使用`-t rsa`指定RSA算法（`ed25519`比`rsa`更现代且安全） |
| **`-f`**                                                  | 自定义**公钥**/**私钥**保存路径<br/>例如：`ssh-keygen -f /your/custom/path/your_key_name`。需要注意，`ssh-keygen`不会自动创建目录，我们需要事先`mkdir -p`； |

在我们执行完密钥生成命令后，系统会提示我们输入保存路径：

* **macOS**上公钥一般会存储在`~/.ssh/id_<algorithm>.pub`；
* **Windows**则一般存储在`C:/Users/<username>/.ssh/id_<algorithm>.pub`；

接着我们将 **公钥** | `pub_key` 复制并添加到[Github: Add new SSH key](https://github.com/settings/ssh/new)上，在配置完后我们可以使用如下命令验证 *SSH* 是否配置成功：

```bash
ssh -T git@github.com
```

***SSH*** 的具体用法会在另一篇文章中介绍，这里暂不展开。这里只简单介绍一下当我们使用 *SSH* 协议连接Github时，**本地密钥**是如何获取的，当没有**显式指定**要使用的私钥（例如没有配置 `~/.ssh/config` 文件）， *SSH* 的默认行为如下， *SSH* 会尝试用这些私钥依次连接目标主机，直到成功或全部失败。

1. **尝试使用 `ssh-agent` 中已经加载的密钥**（如果有）。
2. 如果没有 `ssh-agent`，则尝试按如下顺序查找这些默认文件：
    * `~/.ssh/id_rsa`
    * `~/.ssh/id_ecdsa`
    * `~/.ssh/id_ed25519`
    * <span style='color: gray'>`~/.ssh/id_dsa`（已过时）</span>
3. `SSH`**默认只会尝试标准名称的密钥**（如`id_rsa`、`id_ed25519`），除非我们做了明确配置。

我们可以通过配置`~/.ssh/config`文件，告诉 ***SSH*** 连接不同`server`时使用哪个密钥：

```yaml
# GitHub 配置
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github_rsa
  IdentitiesOnly yes

# GitLab 配置
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/gitlab_ed25519
  IdentitiesOnly yes
```

其中：

* `Host`：匹配你在`git remote`中使用的主机名；
* `IdentityFile`：指定使用的私钥路径；
* `IdentitiesOnly yes`：告诉 *SSH* 只使用指定的密钥，而不尝试默认密钥或`ssh-agent`中的其他密钥。

我们也可以使用`ssh agent`管理私钥并简化 *SSH* 身份验证，在这里就不过多介绍，详见：[终端笔记：SSH]()。

#### 🗝️ GPG Keys

**Git+GPG**是一个用于保障Git提交安全性的重要机制。通过将GPG（GNU Privacy Guard）与Git配合使用，我们可以对Git提交进行加密签名，证明这些提交确实是我们本人所作，并且未被篡改。

Git原生支持使用**私钥**对`commit`与`tag`进行GPG签名。签名后，Github可以通过**公钥**验证该提交是否来自指定开发者。GPG的下载方式可以参考如下表格：

| 平台        | 下载                                                         |
| ----------- | ------------------------------------------------------------ |
| **Windows** | 下载[GnuPG](https://www.gnupg.org/download/index.html)并安装，或者**使用`git bash`自带的GPG** |
| **macOS**   | `brew install gnupg`                                         |
| **Linux**   | `sudo apt install gnupg`                                     |

对于Github的GPG设置我们可以参考一下步骤进行配置：

1. **生成自己的GPG密钥对**：

    允许`gpg --full-generate-key`，**交互式**的输入个人信息、密钥生成参数等，生成自己的GPG密钥对。需要注意的是，为了使Github能够顺利的验证提交信息，我们需要**填写与Github注册邮箱一致的邮箱**。最终`gpg`会输出如下文本：

    ```bash
    gpg: /c/Users/lzz/.gnupg/trustdb.gpg: trustdb created
    gpg: directory '/c/Users/lzz/.gnupg/openpgp-revocs.d' created
    gpg: revocation certificate stored as '/c/Users/lzz/.gnupg/openpgp- \
    revocs.d/<pub_fp>.rev'
    public and secret key created and signed.
    
    pub   rsa4096 2025-05-27 [SC] [expires: 2035-05-25]
          <pub_fp>
    uid                      soppylzz (Github GPG) <numbers@qq.com>
    sub   rsa4096 2025-05-27 [E] [expires: 2035-05-25]
    ```

    其中，`pub`是**公钥的部分摘要**，或者叫密钥对的 *<u>**完整指纹**</u>* ；`uid`是生成时输入的个人信息。除了在生成GPG密钥对时查看`pub`，我们也可以使用`gpg --list-keys`列出本地存储的所有GPG密钥信息。

2. **导出公钥至Github**：

    使用`gpg --armor --export <pub_fp>`导出完整的公钥，然后将完整的公钥复制到[Github: Add new GPG key](https://github.com/settings/gpg/new)；

3. **给Git配置GPG密钥**：

    我们可以在Git中执行如下命令，给Git添加GPG签署支持：

    ```bash
    # 告诉 Git 使用该 GPG Key
    git config --global user.signingkey <pub_fp>
    ```

    需要注意，如果我们使用的是GPG 2.x，可能需要显示指定GPG程序路径：`git config --global gpg.program $(which gpg)`。

4. **GPG签署Git提交记录**：

    我们可以在`commit`时加上`-S`参数，给这次提交加上GPG签名；我们也可以执行如下命令，让Git自动帮我们进行签名：

    ```bash
    # 默认签名所有提交
    git config --global commit.gpgsign true
    
    # 取消默认签名
    # git config --global --unset commit.gpgsign
    ```

    但不论是否需要手动加上`-S`，`commit`时皆会弹出对话框，需要输入该密钥的密码，以确保是密钥拥有者本人操作。最后，提交被`push`到远端后，便会出现<span style="
      display: inline-flex;
      color: #2ea44f;
      font-size: 12px;
      font-weight: 500;
      padding: 0px 8px;
      margin: 0px 4px;
      border: 1px solid #2ea44f;
      border-radius: 20px;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif;
      vertical-align: middle;
    ">
      <span>Verified</span>
    </span>的认证标记。

5. **导入信任GPG密钥**：

    **当我们想要验证别人的GPG签名**，就需要从GitHub上导入他们的**GPG公钥**，Github公钥获取地址为`https://github.com/<username>.gpg`，如果我们想要导入某一个用户的GPG公钥，可以参考如下命令：

    ```bash
    curl -s https://github.com/<username>.gpg | gpg --import
    ```

    当我们直接在Github网页端操作仓库时，Github也会为签署它的GPG签名，其公钥可以通过[webflow-gpg](https://github.com/web-flow.gpg)获取。导入完成后，我们会看到类似输出：

    ```bash
    $ curl -s https://github.com/web-flow.gpg | gpg --import
    gpg: key 4AEE18F83AFDEB23: public key "GitHub (web-flow commit signing) \
    <noreply@github.com>" imported
    gpg: key B5690EEEBB952194: public key "GitHub <noreply@github.com>" imported
    gpg: Total number processed: 2
    gpg:               imported: 2
    ```

6. **验证并管理签名**：

    我们可以在执行`git log`时，加上`--show-signature`来验证某一个签名提交。如果验证成功，Git 会显示如下内容：

    ```bash
    gpg: Signature made ... using RSA key ID ABCDEF1234567890
    gpg: Good signature from "The Octocat <octocat@github.com>"
    ```

    我们也可以在输入`gpg --edit-key <pub_fp>`后，再次输入`trust`，来编辑已有GPG签名的信任状态。当然处理更改信任状态，我们还可以**输入`help`来查看其他编辑功能**。

#### 📻 GPG常用命令

如果我们相对**Git+GPG**进行其他操作，可以参考如下命令列表：

1. **密钥生成与管理**：

    以下是`gpg`与密钥创建和管理相关的命令：

    | 选项                          | 说明                                                        |
    | ----------------------------- | ----------------------------------------------------------- |
    | **`gpg --full-generate-key`** | 生成一对GPG密钥（含完整交互式选项），推荐使用该方式创建密钥 |
    | `--list-keys`                 | 列出本地所有GPG公钥                                         |
    | `--list-secret-keys`          | 列出本地所有GPG私钥                                         |
    | `--fingerprint`               | 查看GPG密钥的指纹（fingerprint），用于精确识别              |
    | `--edit-key <keyid>`          | 进入密钥编辑模式，可更改密码、信任等级、添加子密钥等        |
    | `--delete-key <keyid>`        | 删除某个GPG公钥                                             |
    | `--delete-secret-key <keyid>` | 删除某个GPG私钥                                             |

2. **密钥导入与导出**：

    对于GPG，密钥有如下管理方式：

    * **密钥导出**可用于备份、分发；
    * **密钥导入**用于添加他人的公钥或恢复自己的密钥；
    * **撤销证书**则用于失效或撤销密钥的场景，确保安全性；

    以下是这些管理方式的常用命令：

    | 选项                                                         | 说明                                          |
    | ------------------------------------------------------------ | --------------------------------------------- |
    | `--export -a <keyid>`                                        | 导出指定密钥的公钥，`-a`表示ASCII文本方式导出 |
    | `--export-secret-keys -a <keyid>`                            | 导出指定密钥的私钥，需妥善保存                |
    | `--import <file>`                                            | 从文件中导入GPG密钥                           |
    | <span style='white-space: nowrap'>`curl -s https://github.com/<user>.gpg \|gpg --import`</span> | 从GitHub导入指定用户的公钥                    |
    | `--gen-revoke <keyid>`                                       | 为密钥生成撤销证书，用于失效、泄露等场景      |
    | `--import revoke.asc`                                        | 导入撤销证书，使密钥失效                      |

3. **加密、解密、签名与验证**：

    在实际使用中，我们经常需要加密文件进行传输，或对文件进行签名防止篡改。以下命令涵盖GPG的核心功能：**加密**、**解密**、**签名与验证**：

    | 选项                     | 说明                                   |
    | ------------------------ | -------------------------------------- |
    | `-e -r <recipient>`      | 加密文件，`-r`指定收件人（公钥持有者） |
    | `-d file.gpg`            | 解密文件，需有对应私钥                 |
    | `-s file`                | 对文件进行签名，生成带签名的`.gpg`文件 |
    | `-b file`                | 为文件生成分离式签名`.sig`文件         |
    | `--verify file.sig file` | 验证签名文件的真实性                   |

4. **其他实用命令**：

    除了核心功能，GPG还有一些辅助命令，如清除缓存、信任模型设置等，便于在自动化脚本、密钥服务器同步等场景下使用：

    | 选项                       | 说明                                     |
    | -------------------------- | ---------------------------------------- |
    | `--armor`/`-a`             | 使用ASCII文本格式输出                    |
    | `gpgconf --kill gpg-agent` | 杀掉当前的**GPG代理进程**，清除缓存等    |
    | `--trust-model always`     | 强制信任导入的公钥（适用于自动化脚本）   |
    | `--refresh-keys`           | 从公钥服务器刷新密钥（例如获取撤销状态） |
    | `--card-status`            | 查看智能卡/硬件密钥的状态（如`YubiKey`） |
