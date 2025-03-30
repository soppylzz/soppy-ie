# 贡献指南

感谢你对本项目的关注！这是一个基于**Hexo**的静态博客网站，我们欢迎任何形式的贡献，包括但不限于：**内容创作**、**页面优化**、**bug 修复**、**功能改进**、**构建流程优化**等。请本项目的自动化工作流实现 ***自动创建分支*** 、 ***部署*** 和 ***发布*** 等流程。

> 我的电脑环境如下：
>
> * **`node`**：`v18.20.4`
> * **`npm`**：`v10.8.2`
> * **`nvm`**：`v1.1.12`

## 🧾 如何贡献

### 1. Fork 项目

点击页面右上角的 “Fork” 按钮，将项目复制到你的仓库。

### 2. 创建分支

在 **<your_fork_repo> / Actions / 🆕 New branch** 页面点击 “workflow_dispatch” 根据指示创建 **工作分支** ，以下是分支选择建议：

* **`new/*`**：添加新的文章（科研，工程）；
* **`old/*`**：修改旧的文章（内容修正，排版更改等）；
* **`site/*`**：更改网站的非文章内容（构建脚本，网站主题，Actions脚本等）；

在远端创建分支后，我们需要将其拉取到本地，然后再**对应位置**创建一个 **上游** | `upstream` 为该远端分支的新分支：

```bash
git fetch
git checkout <your_remote_branch>
git branch <your_local_branch> -u <your_remote_branch>
# after work
git push origin <your_local_branch>
```

### 3. 提交改动

提交前请确保代码格式统一，保证网站能够在多端正常显示。

提交时，请使用 [Commitizen](https://github.com/commitizen/cz-cli) 实现规范化`commit`命令的提交消息；

```bash
npm run commit	# "script": {"commit": "cz"}
```

如果遇到其他使用需要`git`命令提交的情况，请遵从 `.release-it.json` 中的提交类型与 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/v1.0.0) 格式。

### 4. Pull Request

推送到你自己的仓库后，在GitHub上提交Pull Request：

* 标题清晰描述更改内容
* 填写`PR`描述，说明变更的目的、影响及测试方式
* 关联相关`Issue`（如果有）

## 🧹 内容更新建议

使用`hexo new post <article_name>`创建博客文章，或者在`/source/_posts`目录下创建`.md`文件；文件有以下要点需要注意：

* 文件命名尽量使用英文小写、短横线分隔，例如：`my-new-post.md`

* 每篇文章**必须**包含`YAML`头信息，如：

    ```yaml
    title: Python笔记：Conda常用命令
    date: 2024-10-9 20:25:57
    tags:
      - conda
      - python
      - tutorial
    categories:
      - 笔记
    ```

## 🙌 欢迎加入！

未来网站会进行 **重构** | `refactor` 。届时，欢迎大家给我们的网站添砖加瓦。