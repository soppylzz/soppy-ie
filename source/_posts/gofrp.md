---
title: 实践笔记：FRP服务搭建
date: 2025-06-27 14:27:40
tags:
  - record
  - network
categories:
  - 记录
---

## 前言

最近有公网IP服务器的需求，便想着搭建一个FRP服务器将流量转发到本地，节约服务器的计算资源（省钱）😓。关于FRP服务器搭建的细节大家可以参考：[gofrp.org](https://gofrp.org/zh-cn/docs/examples/)，这里我只简单介绍一下我的搭建过程。

## 搭建

我的设备环境如下：

| 设备           | 系统                   | 配置      | frp版本         |
| -------------- | ---------------------- | --------- | --------------- |
| **MC Server**  | `macOS`                | screct    | **frp_v0.62.1** |
| **SSH Server** | `ubuntu 24.10 desktop` | screct    |                 |
| **FRP Server** | `ubuntu 24.04 server`  | 4核 3Mbps |                 |

需要注意的是，经过实测3Mbps带宽不能很好的支持MC Server多人联机，一人游玩时也会出现间歇性卡顿，如果需要搭建一个能正常游玩的MC服务器，请选择带宽更高的服务器做代理服务器，或者直接用云服务器搭建一个MC服务器。在开始配置前，我们需要下载合适的FRP版本：[frp - release](https://github.com/fatedier/frp/releases)。

### 1. FRP

#### 🔑 TLS协议加密

我们可以通过配置TLS加密全局流量，这样我们便不需要对单个代理进行加密。参考官网教程创建`my-openssl.cnf`：

```ini
[ ca ]
default_ca = CA_default
[ CA_default ]
x509_extensions = usr_cert
[ req ]
default_bits        = 2048
default_md          = sha256
default_keyfile     = privkey.pem
distinguished_name  = req_distinguished_name
attributes          = req_attributes
x509_extensions     = v3_ca
string_mask         = utf8only
[ req_distinguished_name ]
[ req_attributes ]
[ usr_cert ]
basicConstraints       = CA:FALSE
nsComment              = "OpenSSL Generated Certificate"
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
[ v3_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = CA:true
```

创建生成`generate.sh`脚本：

```shell
# generate ca
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=example.ca.com" -days 5000 -out ca.crt

# generate server
openssl genrsa -out server.key 2048
openssl req -new -sha256 -key server.key \
	-subj "/C=XX/ST=DEFAULT/L=DEFAULT/O=DEFAULT/CN=server.com" \
       	-reqexts SAN \
	-config <(cat my-openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:localhost,IP:[SERVER_IP],DNS:[SERVER_DOMAIN]")) \
	-out server.csr

openssl x509 -req -days 365 -sha256 \
	-in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
	-extfile <(printf "subjectAltName=DNS:localhost,IP:[SERVER_IP],DNS:[SERVER_DOMAIN]") \
	-out server.crt

# generate client
openssl genrsa -out client.key 2048
openssl req -new -sha256 -key client.key \
	-subj "/C=XX/ST=DEFAULT/L=DEFAULT/O=DEFAULT/CN=client.com" \
       	-reqexts SAN \
	-config <(cat my-openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:client.com,DNS:example.client.com")) \
	-out client.csr

openssl x509 -req -days 365 -sha256 \
	-in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
	-extfile <(printf "subjectAltName=DNS:client.com,DNS:example.client.com") \
	-out client.crt
```

我们需要修改的有：`[SERVER_IP]`、`[SERVER_DOMAIN]`。由于我们可能有<u>多个FRPC客户端</u>，这里就不设置具体的`client.com`；至于开启哪种验证方式，如何进行配置详情可参考：[gofrp.org - tls](https://gofrp.org/zh-cn/docs/features/common/network/network-tls/)；

#### 🤔 传输协议选择

FRP支持多种协议的代理，这里引用一下原文的内容：

| 协议       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| **TCP**    | 提供纯粹的 TCP 端口映射，使服务端能够根据不同的端口将请求路由到不同的内网服务 |
| UDP        | 提供纯粹的 UDP 端口映射，与 TCP 代理类似，但用于 UDP 流量    |
| HTTP       | 专为 HTTP 应用设计，支持修改 Host Header 和增加鉴权等额外功能 |
| HTTPS      | 类似于 HTTP 代理，但专门用于处理 HTTPS 流量                  |
| **STCP**   | 提供安全的 TCP 内网代理，要求在 **<span style='color: cyan'>被访问者</span>** 和 **<span style='color: cyan'>访问者</span>** 的机器上都部署 `frpc`，不需要在服务端暴露端口 |
| SUDP       | 提供安全的 UDP 内网代理，与 STCP 类似，需要在 **<span style='color: cyan'>被访问者</span>** 和 **<span style='color: cyan'>访问者</span>** 的机器上都部署 `frpc`，不需要在服务端暴露端口 |
| XTCP       | 点对点内网穿透代理，与 STCP 类似，但流量不需要经过服务器中转 |
| **TCPMUX** | 支持服务端 TCP 端口的多路复用，允许通过同一端口访问不同的内网服务 |

需要注意的是：选择协议时，需要参考代理服务使用的协议。例如：Java版MC服务器默认使用TCP协议开放25565端口，而基岩版MC服务器则默认UDP协议，开放19132端口；SSH也是TCP协议，开放22端口。所以我选择代理**TCP协议**。

#### 🍎 TCP

当我们选择代理传统TCP协议时，需要给每个FRPC的服务指定一个FRPS的开放端口用于代理，参考配置如下：

```toml
# frpc.toml
[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000
```

这里的`remotePort`便是指定的FRPS服务器用于代理的接口。

#### 🍐 STCP

即使我们开启了TLS协议加密，也无法管理访问端的流量。如果我们需要访问端可信的话，便可使用STCP协议。STCP即`scret TCP`。此协议要求 **访问者** | `visitor` 也使用FRPC访问代理服务，参考配置如下：

```toml
# client frpc.toml
serverAddr = "x.x.x.x"
serverPort = 7000

[[proxies]]
name = "secret_ssh"
type = "stcp"
localIP = "127.0.0.1"
localPort = 22
# 只有与此处设置的 secretKey 一致的用户才能访问此服务
secretKey = "abcdefg"

# visitor frpc.toml
serverAddr = "x.x.x.x"
serverPort = 7000

[[visitors]]
name = "secret_ssh_visitor"
type = "stcp"
# 要访问的 stcp 代理的名字
serverName = "secret_ssh"
secretKey = "abcdefg"
# 绑定本地端口以访问 SSH 服务
bindAddr = "127.0.0.1"
bindPort = 6000
```

综上，实现STCP代理，我们只需在**客户端**代理服务中多设置`secretKey`，在**访问端**设置`serverName`、`scretKey`、`bindAddr`、`bindPort`即可。

#### 🍊 TCPMUX

此协议支持我们使用FRPS服务端的**一个端口**接收**多个代理客户端**的请求，实现端口的复用。由于我们是通过同一个公网端口访问的代理服务器，我们需要使用二级域名来区别不同的代理服务。参考配置如下：

```toml
# frps.toml
bindPort = 7000
tcpmuxHTTPConnectPort = 1337
# subdomainHost = "test.com"

# frpc.toml
serverAddr = "x.x.x.x"
serverPort = 7000

[[proxies]]
name = "proxy1"
type = "tcpmux"
multiplexer = "httpconnect"
customDomains = ["test1"]
localPort = 80
# or
# subdomain = "web"

[[proxies]]
name = "proxy2"
type = "tcpmux"
multiplexer = "httpconnect"
customDomains = ["test2"]
localPort = 8080
```

这里的`tcpmuxHTTPConnectPort`便是我们指定的HTTP CONNECT复用端口。

在使用TCPMUX复用端口时，我们必须指定代理配置中的`multiplexer`和域名，FRP支持我们使用`subdomain`+`subdomainHost`或`customDomains`两种方式指定域名。其中当两者同时使用时，**`customDomains`中不能是属于`subdomainHost`的子域名或者泛域名**。

简单介绍了如何使用FRP的TCPMUX功能，但目前我实在是用不上该功能。因为该功能目前（`v0.62.1`）仅支持HTTP CONNECT的复用器。

但这不妨碍我们了解HTTP CONNECT为何物。该协议使用场景是：客户端先通过HTTP CONNECT协议与服务端建立**TCP隧道连接**，之后两端所有的的HTTP通信就可以走原始TCP协议；我们在访问HTTPS网站时，就是使用了该协议，该协议的典型使用场景如下：

1. 目标服务器`example.com:443`建立TCP连接

   ```http
   CONNECT example.com:443 HTTP/1.1
   Host: example.com:443
   ```

2. 返回`HTTP/1.1 200 Connection Established`

3. 客户端和目标服务器之间的通信开始走**TLS加密的TCP流量**

由于这协议本质上是基于TCP协议，我们可以使用相应的代理工具，代理相关基于TCP协议的服务，如SSH；此外，可以看到该协议实际上是HTTP协议的子集，这可以解释为什么我们需要配置域名信息（**服务端使用HTTP请求头的`HOST`内容来区分同一IP端口的不同HTTP请求（HTTP/1.1）**）。

#### ⛑️ FRP权限管理

由于云服务器仅作为流量中转，并不需要多用户配置。我在云服务器上仅做了`systemd`设置，并没有为FRPS服务创建用户进行权限管理。但我的一台需要穿透的设备是需要多用户访问的Linux服务器，需要对FRPC进行权限管理；以下是详细步骤：

1. 将**FRPC服务**安装到系统级目录下（`/usr/local/frp`），并给`frpc.toml`设置全局可读、仅管理员可写权限；

   ```bash
   sudo mkdir -p /usr/local/frp
   sudo tar -zxvf /path/to/frp.tar.gz -C /usr/local/frp
   
   sudo chmod 644 /usr/local/frp/frpc.toml	# 用户可读，管理员可写
   ```

2. 为**FRPC服务**创建专用系统用户，避免使用root或普通用户运行服务，降低权限风险；

   ```bash
   sudo useradd -r -s /sbin/nologin frpc 	# 创建无登录权限的系统用户
   sudo chown -R frpc:frpc /usr/local/frp	# 归属frpc组
   ```

3. 将配置拆分成为**基础模块**和**用户自定义模块**，基础模块配置FRPS服务器信息，自定义模块配置各用户想要代理的服务；

   * 基础配置（`/usr/local/frp/frpc.toml`）：

     ```toml
     serverAddr = "xxx.xxx.xxx.xxx"
     serverPort = 7000
     
     includes = ["/etc/frp/conf.d/*.toml"]	# 动态加载子配置
     ```

   * 创建`/etc/frp/conf.d/`目录，允许用户创建自己的代理请求

     ```bash
     sudo mkdir /etc/frp/conf.d
     sudo chmod 775 /etc/frp/conf.d
     
     sudo groudadd frp_users
     sudo setfacl -dm g:frp_users:rwx /etc/frp/conf.d
     sudo usermod -aG frp_users [username]
     
     # 测试命令:
     groups 			# 查看当前用户所属组
     groups username		# 查看指定用户所属组
     id username		# 显示用户UID、主组GID及所有附加组
     ```

4. 设置`systemd`服务（略）；

### 2. 域名访问

完成上述配置后，即可通过 ***IP:Port*** 访问FRPS代理的内网服务。但如果希望使用域名访问代理服务，则需区分以下两种情况。

#### ⚙️ 域名访问TCP

我的需求只需要访问**SSH**访问与**MC Server**服务即可，访问这两种服务有着不同的解决方法；

当我们执行SSH命令（`ssh <domain_name>`）时，实际上是先通过DNS解析了服务器IP地址，然后再执行`ssh <server_ip>`，一般来说SSH是默认连接`<server_ip>:22`端口，但我们也可以指定端口连接：

* **命令行**：直接使用命令行连接`ssh <server_ip> -p <port>`

* **预配置**：在`~/.ssh/config`中设置端口

  ```
  Host <your_alias>
  	HostName <domain_or_ip>
  	User <username>
  	Port <port>
  	IdentityFile /path/to/file
  	IdentitiesOnly yes		# 仅使用指定密钥
  	ConnectTimeout 10           	# 连接超时
  	ServerAliveInterval 60 		# 应用层保活（可选）
  	TCPKeepAlive yes           	# 传输层保活（可选）
  ```

  （SSH保活需要客户端/服务器端同时进行配置，在这里就不过介绍，详情请参考：）

在MC中<u>仅</u>使用服务器域名（不添加端口信息）连接服务器实际上是用了**DNS服务记录**（SRV，Service Record）功能。在DNS服务提供商中使用`_service._proto`设置即可；

#### 🌐 域名访问HTTP

当我们使用HTTP协议访问FRP代理的内网服务时，整个过程可以分为两个阶段：

1. **客户端到FRPS服务器的通信**：在此阶段中，客户端通过HTTP/HTTPS访问FRPS服务器。是否启用HTTPS加密取决于FRPS服务器上使用的Web服务器软件（例如：Nginx）；我们需要配置FRP服务之外的SSL/TLS证书来启用HTTPS，从而确保客户端与FRPS服务器之间的通信安全。
2. **FRP内部通信**：此阶段是FRPS与FRPC之间的通信。需要给FRP服务配置额外的TLS证书（一般自己生成），来实现FRP服务间的传输层加密，具体配置方法可参考FRP官方文档：[HTTP & HTTPS](https://gofrp.org/zh-cn/docs/features/http-https/)。

接下来，我们将重点介绍第一阶段的通信过程，并简要介绍如何使用Nginx配置反向代理。

### 3. Nignx反向代理

Nignx是一个轻量级的Web服务器，反向代理服务器，电子邮件代理服务器。这里我只简单介绍一下Nginx的基础用法，以及如何使用Nignx代理HTTP/HTTPS和SSL TCP；关于Nignx的详细用法可以参考这篇中文教程：[Nignx中文手册](https://nginx.mosong.cc/guide/)。

Nignx支持编译安装和包管理器安装，由于我没有反向代理的性能要求，我就直接使用包管理器安装：

```bash
sudo apt update
sudo apt install nginx
```

Nignx配置的主要内容在于配置文件，其命令行功能不多，这里就简单介绍一下它的常用命令行功能：

| 命令                                                   |                    选项                     | 说明                                                         |
| ------------------------------------------------------ | :-----------------------------------------: | ------------------------------------------------------------ |
| <span style="white-space: nowrap">`nginx -v/-V`</span> |                      -                      | 参看nginx的版本信息，其中`-v`是输出简要信息，而`-V`则会额外输出Nignx编译时的参数信息。我们可以通过这些信息参看当前Nignx的**功能覆盖**情况，以及**配置文件**路径。 |
| `nginx -t/-T`                                          |                      -                      | 我们常用`nignx -t`测试Nignx配置文件语法是否合法，此命令会简要输出语法检查，而`-T`一般不常用。<br/>需要注意的是，有些配置文件**需要`su`权限访问**，所以一般情况我们需要使用`sudo nginx -t`执行该命令； |
| `nignx -s`                                             | `reload`<br/>`reopen`<br/>`stop`<br/>`quit` | 启动服务不是Nignx的命令，属于系统服务；其他选项使用方式如下：<br/>`reload`：使用此命令重新加载配置（常用）<br/>`reopen`：重启Nginx服务进程<br/>实践使用中，我们一般使用`systemd`统一管理服务，一般不会用这些命令； |
| `nignx -h`                                             |                      -                      | Nignx还有其他不常用的命令行命令，使用此命令获取使用帮助      |

Nginx的配置结构是模块化的，主要分为：

* 全局模块
* `events`块（连接处理）
* `http`块（Web服务器）
  * `server`块/`location`块
  * `upstream`块
* `stream`块（TCP/UDP代理）

下面我们详细了解一下这些板块。

#### 🍟 全局块

全局块主要配置 ***主进程*** 相关参数，例如：工作进程数、用户权限、日志路径、PID文件、是否允许 **崩溃转储** | `core dump`、配置文件包含路径；以下是我的Nginx的默认配置内容：

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;
```

以下是详细解释：

| 配置项                 | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `user`                 | 指定worker子进程运行的**用户**或**用户组**。通常设置为权限较小的用户（如`nginx`或`www-data`），以提高安全性。 |
| `worker_processes`     | 指定工作进程数量（`worker process`），即Nginx处理请求的核心数量。设置为`auto`表示自动匹配CPU核心数。 |
| `pid`                  | 指定Nginx**主进程**的PID文件路径                             |
| `error_log`            | 设置错误日志文件的路径（和级别）。<br/>日志级别包括：`debug`、`info`、`notice`、`warn`、`error`、`crit`。 |
| **`include`**          | 导入其他配置文件。                                           |
| `worker_rlimit_nofile` | 指定单个worker进程可以打开的最大文件描述符数。影响连接数上限，必须和系统`ulimit -n`配合设置。（**<span style='color: cyan'>高并发相关配置</span>**） |

#### 🌭 Events块

`events`块主要配置 ***连接处理方式***，其配置项解析如下：

| 配置项               | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `worker_connections` | 设置每个worker最大可同时处理的连接数。<br/>Nginx的最大连接数大约为：$\text{worker processes} \times \text{worker connections}$ ；该值受系统级别的`ulimit -n`限制，对于高并发服务，可调大此值。（**<span style='color: cyan'>高并发相关配置</span>**） |
| `use`                | 设置事件模型：<br/>- `epoll`：多路复用，支持高效大量并发（适用于`Linux`）<br/>- `kquene`：高性能事件驱动机制（适用于`BSD`、`macOS`）<br/>- `select`、`poll` |
| `multi_accept`       | 启用后，worker进程每次从内核事件队列中**尽可能多地接受新连接**；<br/>否则每次只接收一个连接，性能较低。 |
| `accept_mutex`       | 开启后，可以避免多个worker同时争抢**新连接**；<br/>默认为`on`，一般不进行显式配置； |
| `accept_mutex_delay` | 设置`accept_mutex`等待时间                                   |

#### 🍔 Http块

`http`块是Nginx配置中最为重要的部分，它负责处理HTTP协议相关内容。

我们可以参考Nginx默认配置逐步学习`http`块的配置方法。执行`nginx -V`后，我们可以看到`--conf-path=/etc/nginx/nginx.conf`的编译选项，这就是当钱Nginx所有配置文件的入口。其默认内容如下：

```nginx
http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

 	# Dropping SSLv3, ref: POODLE
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain ...

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

接下来我们逐步了解这些`http`块的默认配置和其他配置。

1. **MIME类型与默认类型**：我们需要载入MIME类型映射，告诉浏览器如何处理不同文件扩展名，如果无法识别MIME类型，则浏览器会使用默认类型识别。

   ```nginx
   include /etc/nginx/mime.types;
   default_type application/octet-stream;
   ```

2. **日志配置**：我们可以在`http`块中指定日志保存路径，这里只简单介绍一下相关基础用法。

   ```nginx
   access_log /var/log/nginx/access.log;
   error_log /var/log/nginx/error.log;
   ```

   我们可以在输出路径后加上日志级别，也可以通过`log_format`配置项给某一类型（以`main`为例）的日志设置格式：

   ```nginx
   log_format main '$remote_addr - $remote_user [$time_local] "$request" '
   		'$status $body_bytes_sent "$http_referer" '
   		'"$http_user_agent" "$http_x_forwarded_for"';
   ```

3. **文件传输优化**：这部分也是Nginx的默认配置，这里只简单介绍一下相关配置项的说明。

   * `sendfile on;`：使用零拷贝（无需内核态切换）传输文件，提高性能。
   * `tcp_nopush on;`：配合`sendfile`，减少TCP包数量。

4. **Gzip压缩**：当客户端支持Gzip（请求头包含`Accept-Encoding: gzip`），Nginx可以对响应体进行压缩，再发送给客户端，客户端解压后显示内容。这可以有效减少贷款小号，加速网页加载速度，其相关配置型说明如下：

   | 指令                | 说明                                      | 推荐设置                                                     |
   | ------------------- | ----------------------------------------- | ------------------------------------------------------------ |
   | `gzip`              | 是否启用Gzip                              | `on`                                                         |
   | `gzip_vary`         | 是否在响应中添加`Vary: Accept-Encoding`头 | `on`（推荐）                                                 |
   | `gzip_comp_level`   | 压缩级别，1~9，级别越高压缩越小但耗CPU    | `5`（推荐）                                                  |
   | `gzip_buffers`      | 用于压缩的缓冲区大小                      | `16 8k`（表示 16 个 8KB 缓冲区）                             |
   | `gzip_http_version` | 客户端的最小HTTP版本                      | `1.1`（推荐）                                                |
   | `gzip_types`        | 指定要压缩的MIME类型                      | `text/html`, `application/json`, `text/css`, `application/javascript` 等 |
   | `gzip_min_length`   | 最小响应长度（字节），小于此值不压缩      | `1024`                                                       |
   | `gzip_disable`      | 禁用特定UA的压缩（通常是IE6）             | `"msie6"`                                                    |
   | `gzip_proxied`      | 控制代理请求是否压缩                      | `any`、`expired`、`no-cache`等                               |

   Nginx默认配置文件中相关配置如下：

   ```nginx
   ##
   # Gzip Settings
   ##
   
   gzip on;
   
   # gzip_vary on;
   # gzip_proxied any;
   # gzip_comp_level 6;
   # gzip_buffers 16 8k;
   # gzip_http_version 1.1;
   # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
   ```

除了上述配置项之外，我们还可以对`http`块，执行`upstream`和`server`的相关配置。

#### 🥪 Upstream块

Nginx中的`upstream`模块是其作为 ***反向代理和负载均衡器*** 的核心部分，用于定义后端服务器组（即<u>上游</u>服务器群），并通过配置进行请求分发。我当前的需求仅为简单代理TCP与HTTP，对负载均衡并没有要求，所以这里就简单介绍一下`upstream`模块的基础用法。一个典型的`upstream`配置如下：

```nginx
upstream backend {
    server 192.168.0.1;
    server 192.168.0.2;
}
...
location / {
    proxy_pass http://backend;
}
```

其核心用法是：在`upstream`中配置服务器集群地址，然后在`proxy_pass`中调用注册的服务。其中`upstream`支持如下负载均衡策略：

* **轮询**（默认）：每个请求轮流分配给上游服务器

* **权重轮询**：通过配置不同服务器的权重，使得分配的请求量成对应比例关系

  ```nginx
  upstream backend {
      server 192.168.0.1 weight=3;
      server 192.168.0.2 weight=1;
  }
  ```

  （第一个服务器会接受3倍于第二个的请求量）

* **IP Hash**：此方法会保证客户端与服务端的会话一致性，我们需要在`upstream`的开头加上`ip_hash;`

  需要注意的是⚠️：此模式不能和`backup`、`down`一起使用。

* Nginx还支持使用第三方模块提供的负载均衡策略，例如：**最少连接** | `least_conn` 等；

`upstream`的`server`配置项支持如下参数：

| 参数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| `weight=n`          | 权重，默认1                                                  |
| `max_fails=n`       | 最大失败次数（默认1）                                        |
| `fail_timeout=time` | 达到失败次数后暂停多久（默认10s）                            |
| `backup`            | 标记为**备份服务器**（主服务器全挂后才用）                   |
| `down`              | 标记该服务器为**临时不可用**                                 |
| `resolve`           | 启用DNS动态解析（需搭配`resolver`）<br/>(**<span style='color: cyan'>适用于服务注册发现</span>**) |

#### 🚀 Server与Location块

`server`配置块是`http`配置的核心所在。其常见配置如下：

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/nginx/ssl/example.crt;
    ssl_certificate_key /etc/nginx/ssl/example.key;

    root /var/www/example;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;

    access_log /var/log/nginx/example.access.log;
    error_log /var/log/nginx/example.error.log;
}
```

我们逐步介绍它的配置内容：

* **`listen`**：指定服务器监听的IP和端口。

  ```nginx
  listen 80;			# 监听 IPv4 的 80 端口
  listen [::]:80;			# 监听 IPv6 的 80 端口
  listen 443 ssl http2;		# 监听 443 端口，并启用 SSL 和 HTTP/2
  ```

  在指定完端口后，我们还可以添加参数开启相应功能：

  * `default_server`：设置为默认server
  * `ssl`：开启SSL（通常用于443）
  * `http2`：开启HTTP/2（需配合 SSL）

* **`server_name`**：给当前服务器指定与之匹配的域名（HTTP请求头的`HOST`）。我们既可以输入完整的域名，也可以使用正则或占位符。

  ```nginx
  server_name example.com www.example.com;
  server_name ~^www\d+\.example\.com$;  	# 正则匹配
  server_name _;  			# 匹配所有未匹配的请求（catch-all）
  ```

* `return`：返回状态码或重定向。

* `error_page`：自定义错误提示页。

* `ssl_certificate`、`ssl_certificate_key`：配置用于HTTPS的SSL证书路径。

* `access_log`、`error_log`：类似全局块中的日志配置，记录该虚拟服务器的日志信息（注意路径不要与全局块中重复）。

  ```nginx
  access_log /var/log/nginx/example.access.log;
  error_log /var/log/nginx/example.error.log;
  ```

* **<span style='color: pink'>`root`、`index`与`location`</span>**：

  其中`root`用于设置请求的根目录，当我们使用`http://[server_name]/path/to/file`时，实际上是将HTTP请求转换为`[root]/path/to/file`获取服务器的静态文件。

  而`index`则用于指定访问`http://[server_name]`时，默认返回的页面文件。

  `location`用来声明路由。我们可以通过配置`location`与`rewrite`实现API的统一管理：

  ```nginx
  server {
    # ...
    
    location / {
      # root index
      # try_files $uri $uri/ /index.html;
      try_files $uri $uri/ =404;
    }
    
    location /node/ {
      proxy_pass http://localhost:3000/;
      rewrite ^/node(/.*)$ $1 break;
      include proxy_params;
    }
  }
  ```

  在`location`中，我们使用 **`try_files`** 获取`root`文件夹下的页面文件，当没有找到指定的文件时，我们可以选择返回固定页面，或者返回 *404 Not Find* ，让Nginx统一处理错误请求。

  此外，这里使用 **`rewrite`** 去掉了访问服务的URL前缀，使得我们可以使用`/node/api`访问服务原本提供的`/api`接口。这里的 **`proxy_pass`** 是实现服务代理的核心配置，表示将修改后的URL请求转发给在3000端口开启的服务。在转发HTTP请求时，我们还可以添加如下配置使得代理的服务能知道原始请求的IP和协议：

  ```nginx
  proxy_http_version 1.1;			# 指定HTTP协议版本
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  ```

  当我们使用HTTP请求非页面的静态文件时，Nginx会自动处理，但是我们也可以通过设置缓存头加快响应时间：

  ```nginx
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
      root /var/www/myweb;
      expires 7d;
      add_header Cache-Control "public";
  }
  ```

#### 📟 Stream块

Nginx的`stream`块是独立于`http`配置块的，我们需要到`/etc/nginx/nginx.conf`中修改主配置文件。`stream`配置与`http`配置同级，那么`stream`块大概需要单独配置`log`与`upstream`等设置。这里就不过多讲解该配置块的配置细节，直接给出一个参考配置：

```nginx
stream {
  log_format  tcp_log  '$remote_addr [$time_local] '
			'$protocol $status $bytes_sent $bytes_received '
			'$session_time';
  
  access_log  /var/log/nginx/stream-access.log  tcp_log;
  error_log   /var/log/nginx/stream-error.log   info;
  
  upstream backend_mysql {
    # 采用默认负载均衡
    server 10.0.0.11:3306 max_fails=3 fail_timeout=30s;
    server 10.0.0.12:3306 max_fails=3 fail_timeout=30s;
  }
  
  server {
    listen 3306 ssl reuseport backlog=1024;
    # 推荐：开启 PROXY Protocol，以便将客户端真实 IP 传给后端
    # 后端须支持并配置 proxy_protocol
    proxy_protocol on;
    
    ## TLS 配置 —— 强制 TLS1.2+，使用现代 ciphersuite
    ssl_certificate           /etc/nginx/ssl/mysql.example.com.crt;
    ssl_certificate_key       /etc/nginx/ssl/mysql.example.com.key;
    ssl_protocols             TLSv1.2 TLSv1.3;
    ssl_ciphers               'TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:!aNULL:!eNULL:!RC4:!3DES';
    ssl_prefer_server_ciphers on;
    ssl_session_cache         shared:SSL:10m;
    ssl_session_timeout       10m;   
    ssl_dhparam               /etc/nginx/ssl/dhparam.pem;        # 2048-bit DH 参数文件
    
    # 可选：要求客户端证书（双向 TLS）
    # ssl_verify_client on;
    # ssl_client_certificate /etc/nginx/ssl/ca.crt;

    # 转发到上游
    proxy_pass backend_mysql;

    # 超时及缓冲设置
    proxy_connect_timeout   5s;
    proxy_timeout           300s;
    # 禁用 buffering，降低延迟
    proxy_buffering off;
  }
}
```

上述内容为GPT生成的适用于`Mysql`的TCP代理配置，但如果我们希望使用Nginx代理SSH，则可以不加上SSL/TLS证书加密，SSH协议本身在建立连接时就包含了密钥交换、身份验证和数据加密等功能。以下是用于代理SSH协议的Nginx配置：

```nginx
stream {
  log_format  tcp_log  '$remote_addr [$time_local] '
			'$protocol $status $bytes_sent $bytes_received '
			'$session_time';
  access_log  /var/log/nginx/ssh-access.log  tcp_log;
  error_log   /var/log/nginx/ssh-error.log   info;
  
  upstream ssh_backend {
    server 192.168.10.10:22 max_fails=3 fail_timeout=30s;
  }
  
  server {
    listen [port];
    proxy_pass ssh_backend;
    
    # 保留客户端真实 IP
    # 需要对服务器ssh进行额外配置
    proxy_protocol on;
    
    proxy_connect_timeout 5s;
    proxy_timeout 1h;
    
    # 限制连接速率（防止暴力破解）
    limit_conn_zone $binary_remote_addr zone=ssh_conn:10m;
    limit_conn ssh_conn 10;
  }
}
```

## 借物表

| 参考资料                                                     | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **<span style='white-space: nowrap'>[gofrp中文文档](https://gofrp.org/zh-cn/docs/)</span>** | 深入了解gofrp功能、配置项及使用细节的核心手册                |
| **[nginx中文手册](https://nginx.mosong.cc/guide/)**          | 掌握Nginx配置、模块功能与最佳实践的权威参考                  |
| **[chatgpt](https://chatgpt.com/)**                          | 本文部分内容参考GPT                                          |
|                                                              | 如发现文中有未标明的引用信息，请与我联系。<br/>在核实后，我将在第一时间补充相关引用信息或进行修正。 |



