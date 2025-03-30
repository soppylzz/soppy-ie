---
title: 实践笔记：Hexo博客搭建与任务流探索
date: 2025-03-30 15:16:30
tags:
  - hexo
  - typora
categories:
  - 工具
---

## 前言

### 1. 感想

在正式开始前，先讲讲我打算搭建一个博客的动机吧。我最初的打算时在我写完`vladinena`组件库与`milize`样式工具库后，写一个静态博客框架来构建我的个人博客；但是研究生的学习与研究任务实在是太重，光是`slidev-theme-cqupt`主题包的编写就花了我研一上的空闲时间，所以我想还是得先拿一个工具把自己再学习与研究过程中的所思所想都记录下来，以便我日后再次接触时快速上手。

研一上我都是直接在本地写`markdown`记录研究思考，但是等我想往回找之前的笔记时，搜索工作就成了大问题，无法有效检索的笔记简直就是废纸，纯浪费时间。我希望从这第一篇笔记开始记录我日后的实践与感想，希望在<u>未来的某一时刻</u>我看到这段话时能感叹：研究生三年时间真没白费！

我写这篇笔记仅用于记录探索`Hexo` + `Typora`的心得与收获，内容可能存在认知偏差或信息滞后，如果你在参考本文搭建你自己的博客时出现问题，欢迎与我联系。

### 2. 前置

以下是我的环境配置：

* OS：`windows 11`；
* IDE：`webstorm 2024`；
* Env：`nvm@1.1.12` + `node@18.20.4` + `npm@10.8.2`；

根据我之前使用`github actions`的经验，这次我选择`ci\cd` + `hexo`的方式搭建与维护个人博客。经过调研我最终选择参考如下网站构建我的**博客写作任务流**：

* [hexo-theme-cactus:cactus:](https://github.com/probberechts/hexo-theme-cactus)[简洁的设计保质期更长]；
* [typora+图床详解](https://zhuanlan.zhihu.com/p/346410333)；



## 建站

关于`Hexo`如何建站，`hexo-theme-cactus`以及介绍得很详细了，我就不过多赘述；这里只介绍一下`hexo-cli`、`cactus`个性化与`deploy`的一些处理方法：

### 1. 个性化

常见的`hexo`主题的基础`font-size`都是设置为`14px`，这对于我的电脑屏幕（1080p）太小了，我需要更改它的样式设置；`hexo`字体大小设置其实是运用了`rem`与`px`的关系（详情参考[掘金 | rem与px](https://juejin.cn/search?query=rem与px&fromSeo=0&fromHistory=0&fromSuggest=0)），简单的说`rem`是基于父级的`px`计算得来，与之相关的<u>大屏适配</u>与`px2rem`是一个常见面试问题；

`cactus`没有为所有的样式细节提供`_config.yml`的API，这需要去`themes/cactus/source/css`中去修改`stylus`编写的样式代码，更改细节我就不提了，如下是我更改了的板块：

* 字体文件：更改文件`add`、`_fonts.styl`；
* `Markdown`渲染样式：更改文件`style.styl`；
* 其他样式：`_partial`、`_variables.styl`；

### 2. 部署

传统的`hexo`博客部署需要在本地有一个完整的`hexo-site`项目，这不便于我随时随地更新我的博客内容，因此我选择使用`github actions`来构建并部署网站。

`hexo`进行`cicd`部署的方式有两种：

1. 使用`hexo-cli`的`deploy`部署：这需要在`repository`中配置`DEPLOY_KEY`与`DEPLOY_PUB`；如果你想通过此方法部署你的博客，可以参考[GitHub Actions 来自动部署 Hexo](https://zhuanlan.zhihu.com/p/170563000)。
2. 使用`hexo-cli`的`generate` + `github`官方脚本`deploy-pages@v4`部署：**这无需进行额外配置**；

我在使用`slidev-theme-cqupt`时，发现当`github`执行`actions`时没有`npm`缓存，将会花费很长时间在`npm install`上，因此在编写`soppy-ie/.github/workflows/deploy.yml`时，我参考`github docs`添加了相应的**缓存脚本**：
```yaml
      - name: cache dependencies
        id: cache-dependencies-deploy
        # npm cache files are stored in `~/.npm` on Linux/macOS
        uses: actions/cache@v4
        env:
          cache-name: cache-node-modules
        with:
          path: |
            ~/.npm
          # add field 'deploy' to insulate different workflows
          key: ${{ runner.os }}-deploy-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-deploy-${{ env.cache-name }}-
            ${{ runner.os }}-deploy-
            ${{ runner.os }}-

      - if: ${{ steps.cache-dependencies-deploy.outputs.cache-hit != 'true' }}
        name: list the state of node modules
        continue-on-error: true
        # npm list for debugging
        run: |
          echo 'deploy-modules-list'
          npm list

      - name: install dependencies
        run: | 
          npm install
          npm install hexo-cli -g
```

以下是我参考的文档的链接：

* [缓存依赖项以加快工作流程 - GitHub 文档](https://docs.github.com/zh/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows)；
* [actions/cache: Cache dependencies and build outputs in GitHub Actions](https://github.com/actions/cache)；

此外，我还使用了在这几年常用的`cicd`任务流工具来优化代码提交，版本更新等流程，后续我会开一篇新笔记讲讲这些任务流工具及其原理[ :) 请原谅我立了个FLAG🚩]；



## 图床

关于如何搭建`typora` + `picgo` + `tencent cloud cos`图床任务流网上的教程已经十分详尽了，这里我只简单介绍搭建图床时我认为重要的点。

### 1. PicList与COS

我需要在图床软件中管理腾讯云对象存储`cos`中的图片数据，因此选择采用`PicGo`的改进版本`PicList`来管理图床；当我们配置图床时有如下信息需要我们手动填写，我在这里列出它们的获取链接：

* `SecretId`：[访问密钥](https://console.cloud.tencent.com/cam/capi)；

* `SecretKey`、`APPID`：同上；

    ![appid_secret](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/appid_secret-1743317823925.png)

* `Bucket`：[存储桶列表 - 对象存储](https://console.cloud.tencent.com/cos/bucket)；

* 存储区域：同上；

    ![bucket](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/bucket-1743317833294.png)

`PicList`除了支持<u>云端同步管理</u>之外，还支持<u>高级重命名</u>与<u>对象存储管理</u>，这里我就不过多介绍了，详情参考它的官网：[PicList](https://piclist.cn/)；

### 2. Typora

最后，我只需要在`typora`中完成如下设置即可：

![typora_settings](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/typora_settings-1743318387774.png)



## 结尾

愿此程山海，以破茧之姿逐光而行；

**音乐分享🎸**：[森七菜「深海」](https://www.bilibili.com/video/BV1264y1x7Gd/?spm_id_from=333.337.search-card.all.click&vd_source=65d8b9fdb891b3e70379f1ed4c961c27)；
