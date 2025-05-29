---
title: Python笔记：Conda常用命令
date: 2024-10-9 20:25:57
tags:
  - conda
  - python
  - tutorial
categories:
  - 笔记
---

> conda 是个复杂的 Python 版本管理隔离工具，这里我只是简单介绍一些 Conda 的常用命令，详细请参考：[命令 — conda 24.7.1 文档](https://docs.conda.org.cn/projects/conda/en/stable/commands/index.html) 。

## 管理conda

### 1. 帮助指南

#### 🛠️ 获取帮助
当我们不确定某个`conda`命令的具体用法时，可以通过添加`-h`或`--help`参数来查看该命令的使用说明：

```cmd
conda [order_name] -h
conda [order_name] --help
conda create --help		# Example: show <create> usage
```

#### 🔍 信息查询
当我们想要了解**Conda**本身或环境的基本信息时，可以使用`conda info`命令。该命令也支持添加参数以获取更准确的输出：

```cmd
conda info		# Example: show .condarc
```

`conda info`常用查询参数：

| 查询参数     | 功能                        |
| ------------ | --------------------------- |
| *`--envs`*   | 列出当前用户已知的Conda环境 |
| *`--system`* | 显示系统级环境变量信息      |

此外，**Conda**也支持查询所有Python环境的相关信息（<span style='color:gray'>如果以管理员身份运行，**Conda**会查询全部用户的所有Python环境</span>）：

```cmd
conda search --envs
```

### 2. 配置概述

**Conda**有两种管理配置信息的方式，一种是通过`conda config`指令管理，另一种是通过编辑`.condarc`文件管理。值得一提的是`.xxxrc`在计算机领域很常见，`rc`是`Resource Configuration`的缩写，通常用作<span style='color: cyan'>存储应用程序</span>或<span style='color: cyan'>命令行工具</span>的用户偏好设置和配置选项，例如：`.npmrc`、`.nvmrc`。

#### ⚙ 使用`conda config`命令配置
查看**Conda**环境配置的方法有两种：一种是`--show`，该命令只会查询字段的**当前配置** ；另一种是`--get`，该命令可以查看对该字段进行变更的<span style='color: cyan'>**指令记录** </span>。

```shell
# Show all config
conda config --show

# Show [field_name] config
conda config --show [field_name]	
conda config --get [field_name]

# Example: check channels config
conda config --show channels
conda condig --get channels
```

查看**Conda**环境配置字段的**含义**以及**当前配置**：

```shell
conda config --describe [field_name]

# Example: show channels describe
conda config --describe channels
```

管理**Conda**环境配置数据，需要注意的有：`--add`与`--prepend`实现的效果是一样的；`--add`等操作是用于<span style='color: cyan'>**数组类型**</span>，而`--set`是用于<span style='color: cyan'>**值类型**</span>。

```shell
conda config --append <Key> [Value]

# Next two order equals to each other
conda config --prepend <Key> [Value]
conda config --add <Key> [Value]

const config --set <Key> [Value]

# Useful for '--add' and '--set'
conda config --remove <Key> [Value]
conda config --remove-key <Key> [Value]

# Example: add mirror-source to channels
conda config --add channels [source_url]

# # Result:
# # channels:
# # 	- [source_url]


# Example: add conda-forge to custom_channels
conda config --set custom_channels.conda-forge [source_url]

# # Result:
# # custom_channels:
# # 	conda-forge: [source_url]
```

#### 📝 编辑`.condarc`文件
当我们使用`conda info`查看当前**Conda**的信息时，可以看见以下两个信息：

|    user config file    | populated config files |
| :--------------------: | :--------------------: |
| C:\Users\lzz\\.condarc | C:\Users\lzz\\.condarc |

其中`populated config files`是指当前**Conda**运行实例**已加载的Conda配置信息**我们可以编辑**windows**的环境变量，来实现加载项目目录中`.condarc`文件的目的。

```shell
set CONDARC=%CD%\.condarc		# Add condarc path
set CONDARC=				# Remove condarc path
```

我们也可以根据**Conda**提供的`.condarc`查找策略来设置`.condarc`文件路径：

```python
if on_win:
SEARCH_PATH = (
  "C:/ProgramData/conda/.condarc",
  "C:/ProgramData/conda/condarc",
  "C:/ProgramData/conda/condarc.d",
)
else:
SEARCH_PATH = (
  "/etc/conda/.condarc",
  "/etc/conda/condarc",
  "/etc/conda/condarc.d/",
  "/var/lib/conda/.condarc",
  "/var/lib/conda/condarc",
  "/var/lib/conda/condarc.d/",
)

SEARCH_PATH += (
  "$CONDA_ROOT/.condarc",
  "$CONDA_ROOT/condarc",
  "$CONDA_ROOT/condarc.d/",
  "$XDG_CONFIG_HOME/conda/.condarc",
  "$XDG_CONFIG_HOME/conda/condarc",
  "$XDG_CONFIG_HOME/conda/condarc.d/",
  "~/.config/conda/.condarc",
  "~/.config/conda/condarc",
  "~/.config/conda/condarc.d/",
  "~/.conda/.condarc",
  "~/.conda/condarc",
  "~/.conda/condarc.d/",
  "~/.condarc",
  "$CONDA_PREFIX/.condarc",
  "$CONDA_PREFIX/condarc",
  "$CONDA_PREFIX/condarc.d/",
  "$CONDARC",
)
```

当有多个`populated config files`存在时，也就是**Conda**读入了多个`.condarc`文件时，**Conda**遵循以下优先策略：

* 当配置值是数组或者列表时按优先级合并，若为原始值时保留到优先级高的配置；
* 各`.condarc`优先级由低到高依次为：解析器的`.condarc` → $CONDARC 指定的`.condarc` → `config order`参数 → <span style='color:  silver'><u>环境变量</u></span>；

当我们使用`conda config`编辑`.condarc`文件时，可以通过`--system`、`--env`、`--file [file_name]`来指定配置输出位置：

| 参数                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| *`--system`*                                                 | [**默认选项**] 输出到`User Config File`（`C:\Users\lzz\.condarc`）中 |
| *`--env`*                                                    | 输出到当前活动环境下的`.condarc`中，如果没有则同`--system`   |
| *<span style='white-space: nowrap'>`--file [filename]`</span>* | 输出到指定`.condarc`文件中                                   |

### 3. 版本与更新

我们可以使用如下指令<u>查看</u>，<u>更新</u>**Conda**版本。

```cmd
# Check conda version
conda -V
conda --version

# Update conda
conda update conda	

# Update Anaconda
conda update Anaconda	
```

### 4. 通道配置

**Conda**官方的库提供的Python包较少，有时需要到第三方社区维护的**Conda**通道去找寻相关的Python包，例如：<u>conda-forge</u> 、<u>bioconda</u> 。故我们需要配置 *default_channels* 与 *custom_channels* 为**Conda**提供多个下载通道。

值得一提的是，在使用**Conda**管理的Python 项目也可以使用`pip`安装所需的包，但是这种`conda install`与`pip install`混用的情况很容易导致库的依赖关系混乱。

🔗 **设置官方通道及其镜像通道**

我们打开`.condarc`后会发现**Conda**自动帮我们加入了`channels: -default`，而且官方通道的地址不是在`.condarc`中配置的，故我们只需要配置官方通道的镜像源即可。

我们可以直接在 *channels* 里面添加官方通道的镜像源：

```shell
conda config --add channels [channel_url]
```

也可以将官方通道的镜像源添加到 *default_channels* 里面，以便与<u>conda-forge</u>等第三方社区通道区分开来：

```shell
conda config --add default_channels [channel_url]
```

#### 🧱 添加社区通道
使用第三方社区通道（如 <u>conda-forge</u>），我们需要配置`channels: -[community_name]`：

```shell
conda config --add channels [community_name]

# Example: add conda-forge community channel
conda config --add channels conda-forge
```

对于第三方社区通道的地址，我们通常不用配置镜像源直接配置其地址即可。需要注意的是使用命令行工具添加配置的过程较为特殊，这里建议直接修改`.condarc`即可。可别忘了`--add`于`--set`的区别。

```shell
conda config --set custom_channels.[community_name] [channel_url]

# Example: add conda-forge upon custom_channels
conda config --set custom_channels.conda-forge 
					http://mirrors.aliyun.com/anaconda/cloud
```

指定第三方社区通道下载包：

```shell
conda install [package_name] --channel [community_name]
conda install [package_name] -c [community_name]

# Example: use conda-forge install GDAL
conda install GDAL --channel conda-forge
```

#### 🎯 优先级问题
关于 *channels* 的优先级问题，我们可以使用`--describe`去查看**Conda**官方的介绍。*channels* 中越靠前优先级越高，`--prepend`、`--append`来控制 *channel* 的插入顺序。当**Conda**要下载东西的时候便会从优先级高的通道<span style='color: cyan'>依次查找</span>到优先级低的通道。 

我们可以通过改变`channel_priority`配置来改变**Conda**下载策略：

* *flexible*：只要在 ***所有通道*** 中找到了对应库便下载，不会引起 *unsatisfiable error*。在下载没有指定版本的包时，包版本高的优先。
* *strict*：如果 ***高优先级通道*** 没有找到对应库便会报错。在下载没有指定版本的包时，通道优先级高的优先。

```shell
conda config --set channel_priority strict
conda config --set channel_priority flexible
```

#### ⚙️ 其他通道配置

* *show_channel_urls*：控制着**Conda**是否在执行操作（如搜索、安装或更新包）时显示通道的`URL`。
* *channel_alias*：通道别名，**Conda**会先在 *channels* 与 *custom_channels* 中进行匹配，如果没找到相对应的完整`URL`，则会根据 *channel_alias* 进行补全，其默认值如下：

    ```shell
    conda config --describe channel_alias
    
    # # Result:
    # # channel_alias (str)
    # #   The prepended url location to associate with channel names.
    # #   
    # # channel_alias: https://conda.anaconda.org
    
    conda install GDAL -c conda-forge
    
    # # Result:
    # # Equals toCondainstall GDAL -c 
    # #	https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
    ```

#### 📄 `.condarc`示例配置

```yaml
channels:
  - defaults
  - conda-forge

show_channel_urls: true

# Conda-forge recommended
channel_priority: strict

default_channels:
  - http://mirrors.aliyun.com/anaconda/pkgs/main
  - http://mirrors.aliyun.com/anaconda/pkgs/r
  - http://mirrors.aliyun.com/anaconda/pkgs/msys2

custom_channels:
  conda-forge: http://mirrors.aliyun.com/anaconda/cloud
  msys2: http://mirrors.aliyun.com/anaconda/cloud
  bioconda: http://mirrors.aliyun.com/anaconda/cloud
  menpo: http://mirrors.aliyun.com/anaconda/cloud
  pytorch: http://mirrors.aliyun.com/anaconda/cloud
  simpleitk: http://mirrors.aliyun.com/anaconda/cloud
```

### 5. 缓存相关

#### 🗑️ `conda clean`命令

| 参数                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| *`-i`、<br/>`--index-cache`*                                 | 删除索引缓存                                                 |
| *`-t`、`--tarballs`*                                         | 删除缓存的软件包压缩包                                       |
| *`-a`、`--all`*                                              | 删除索引缓存、锁定文件、未使用的缓存软件包、压缩包和日志文件 |
| *`-p`、<br/>`--packages`*                                    | 从可写软件包缓存中删除**未使用**的软件包（<span style='color:cyan'>注意！此模式不会检查使用符号链接返回软件包缓存安装的软件包</span>） |
| *`-f`、<br/><span style='white-space: nowrap'>`--force-pkgs-dirs`</span>* | 删除**所有可写**的软件包缓存，**此选项未包含在`--all`中**（<span style='color:cyan'>注意！此模式将破坏使用符号链接返回软件包缓存安装软件包的环境</span>） |

#### 📂 设置包缓存文件夹
我们可以使用`conda config`为 *pkg_dirs* 添加包缓存目录。

```shell
conda config --add pkgs_dirs [package_cache_path]
```

我们分别使用`config info`与`--describe`来查看该配置。可以发现**Conda**自带包缓存的文件夹。

```shell
conda info

# # Result:
# # package cache : D:\Software\Anaconda\pkgs
# #                 C:\Users\lzz\.conda\pkgs
# # 				C:\Users\lzz\AppData\Local\conda\conda\pkgs


conda config --describe pkgs_dirs

# # Result:
# # pkgs_dirs (sequence: primitive)
# # pkgs_dirs: []
```

我们可以使用`conda clean`清空 *pkgs_dirs* 里面的包缓存。

```shell
conda clean --f
conda clean --force-pkgs-dirs
```

## 管理环境

### 1. 查询环境

我们可以通过`env`或者`info`指令查看 *conda environments* 。

```shell
# Next three order equals ot each other
conda env list
conda info -e
conda info --envs

# # Result:
# # conda environments:
# # base                     D:\Software\Anaconda
# # geo                      D:\Software\Anaconda\envs\geo
```

### 2. 环境管理

#### 🆕 创建环境
我们可以`create`指令创建一个全新的Python虚拟环境：

```shell
conda create [env_name]
conda create [env_name] [python] [packages]

# Example: create env with specifying python and packages
conda create geo python=3.8 matplotlib plot
```

也可以根据`environment.yaml`来还原一个环境：

```shell
conda env create --file [env_name].yml

# Example: create env by env.yaml
conda env create --file environment.yml
```

#### 🗑️ 删除环境
`remove`指令可以用于删除环境，也可以用于删除指定环境中的包：

```shell
# Delete specified env and all packages in it
conda remove --name [env_name] --all

# Delete specified package in env
conda remove --name [env_name] [package_names]
```

#### 💾 导出环境配置
`env`指令用于环境的导入导出，注意`env.yml`文件中已有虚拟环境名称：

```shell
# Get env.yml
conda env export --name [env_name] > [env_name].yml

# Create env by env.yml
conda env create --file [env_name].yml

# Example: export and create by env.yml
conda env export --name geo > env.yml
conda env create --file env.yml
```

#### 🚀 使用环境

```shell
conda activate [env_name]

# Next two order equals to each other
conda activate
conda deactivate
```

### 3. 包管理

#### 📄 查询包
**Conda**查询包有两个指令，一个是`list`，其用于查询已安装的包；

```shell
# For all environments
conda list

conda list --name [env_name]

# # Result: 
# # packages in environment at D:\Software\Anaconda\envs\geo:
# # Name                    Version                   Build  Channel
# # ca-certificates           2024.7.2             haa95532_0
# # libffi                    3.4.4                hd77b12b_1


# Special attributes
conda list --explicit --name [env_name]
conda list --show-channel-urls --name [env_name]
```

`list`还有以下参数可选，用于查看包安装的详细信息：

| 参数                    | 说明                       |
| ----------------------- | -------------------------- |
| *`--explicit`*          | 在输出时显示包的确切版本号 |
| *`--show-channel-urls`* | 在输出时显示包的来源`URL`  |

另一个是`search`，其用于在 *channels* 中查询指定包。请注意`conda search`还有个查询电脑上Python环境的功能。

```shell
conda search [package_name]

# Fuzzy search
conda search [package_name]*

conda search [package_name] --channel [channel_name]

# Special
conda search [package_name] --info

# # Result:
# # gdal 3.6.2 py39hf6e6a5b_2
# # -------------------------
# # file name   : gdal-3.6.2-py39hf6e6a5b_2.conda
# # name        : gdal
# # version     : 3.6.2
# # build       : py39hf6e6a5b_2
# # build number: 2
# # size        : 1.8 MB
# # url         : [url]
# # dependencies: [dependencies]
```

`search`还有以下参数可选，用于更好的查询包：

| 参数          | 说明                   |
| ------------- | ---------------------- |
| *`--channel`* | 指定查询的通道         |
| *`--limit`*   | 限定查询结果显示个数   |
| *`--info`*    | 显示查询结果的详细信息 |

#### 📥 安装包
我们使用`conda install`安装包，我们可以在安装的时候指定包的版本、查找的<u>*channel*</u> 。有些包的下载过程中需要我们在命令行输入`Y`，`--yes`可以帮我们自动输入。

```shell
conda install [package_name]

conda install [package_name] --name [env_name]

# Specified attributes
conda install [package_name]=[version]
conda install [package_name] --channel [channel_name]

# Auto check yes
conda install [package_name] --yes
```

这里还需要补充在使用时需要注意的几个参数：

| 参数                                       | 说明                           |
| ------------------------------------------ | ------------------------------ |
| *`--force-reinstall`*                      | 确保请求安装的包卸载并重新安装 |
| *`--freeze-installed`、`--no-update-deps`* | 不要更新或更改已安装的依赖项   |

还需要注意的一点是：**Conda**只支持在官方通道和第三方社区通道安装包，如果要安装`.wheel`格式的包，请使用`pip`安装，这时候需要注意之前提到的环境污染问题。

#### 🗑️ 卸载包
我们使用`conda uninstall`卸载包，需要注意`conda uninstall`默认<span style='color:cyan'>同时删除包的依赖</span>，<u>如果使用`--force`，则不会删除其依赖</u>。

```shell
conda uninstall [package_name]

conda uninstall [package_name] --name [env_name]

# Auto check yes
conda uninstall [package_name] --yes

# Maybe go into broken
conda uninstall [package_name] --force
conda uninstall [package_name] --force-remove
```

我们可以使用`conda uninstall --help`查看**Conda**对`--force`的描述。不要轻易使用`--force`，<u>这会使我们的环境变得破碎不稳定</u>。

```shell
# # --force-remove, --force
# # 	Forces removal of a package without removing packages that depend on it. 
# # 	Using this option will usually leave your environment in a broken and 
# #     						inconsistent state.
```

#### ♻️ 更新包
我们使用如下指令更新指定包到最新版本，如果要更新到指定版本，请使用功能更强大的`conda install`。

```shell
conda update [package_name] --name [env_name]
```