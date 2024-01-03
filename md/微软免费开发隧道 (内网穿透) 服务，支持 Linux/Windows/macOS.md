> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IwLIugD0BN169ZnL5okiiw)

![](https://mmbiz.qpic.cn/mmbiz_jpg/bG6onXJ2q5UzoaqGytsSe45icveBgMwmcXFJgFjUF3f2NlNVu98XTnKMVpwqgG9EzjqVILzWVcIanmMDaW2B7pw/640?wx_fmt=jpeg&from=appmsg)

开发隧道允许开发人员跨 Internet 安全地共享本地 Web 服务。使你能够将本地开发环境与云服务连接，与同事共享正在进行的工作或帮助构建 Webhook。开发隧道适用于临时测试和开发，不适用于生产工作负荷。

博主上篇文章： 文章发布后，有大佬提醒微软一个服务：开发隧道，可不依赖 VSCODE 或者 Visual Studio 可以独立运行！

经过一波研究确实不错！整理分享一下！

本文介绍的软件提供的功能基本和上一篇文章中的功能一样！但是全程需要命令行操作，对于不太懂脚本的童鞋还是建议参考上一篇文章！

更多内容：内网穿透 / Github

软件下载
----

Windows x64：https://aka.ms/TunnelsCliDownload/win-x64

macOS (arm64)：https://aka.ms/TunnelsCliDownload/osx-arm64-zip

macOS (x64)：https://aka.ms/TunnelsCliDownload/osx-x64-zip

Linux x64：https://aka.ms/TunnelsCliDownload/linux-x64

使用教程
----

本文主要以 Linux 服务器为例！

### 软件授权

服务器上下载软件，重命名为 `devtunnel`方便后续使用！

`chmod 777 devtunnel`

### 登录账号

执行命令：`./devtunnel user login -g -d`    （如果电脑有浏览器优先调起浏览器实现登陆授权）

提示如下：

```
[root@dev~]# ./devtunnel user login -g -d
Browse to https://github.com/login/device and enter the code: 932E-F865
Logged in as malaohu using GitHub.
```

浏览器打开： https://github.com/login/device  按输出内容要求 CODE：xxxx-xxxx

![](https://mmbiz.qpic.cn/mmbiz_jpg/bG6onXJ2q5UzoaqGytsSe45icveBgMwmcyjxV04fvPA3N05icsCEto5yz6ppKEt5tacOaEb7EcHiaZuM4tc127x9A/640?wx_fmt=jpeg&from=appmsg)

同意授权，即可完成登陆！

![](https://mmbiz.qpic.cn/mmbiz_jpg/bG6onXJ2q5UzoaqGytsSe45icveBgMwmcU7T6R2FYSOZ6y7pRuPBwID9tMe074iacuMzee7eAnhGdDUU9QQNzupw/640?wx_fmt=jpeg&from=appmsg)

如登陆提示如下，是因为国内访问 Github 出现间歇性无法访问！

[root@dev~]# ./devtunnel user login -g -d  
Connection timed out (github.com:443)

多尝试几次即可，或者查看本博**系列文章：GitHub 国内加速** 

### 启用转发

转发端口：`./devtunnel host -p 8000`

提示如下：

```
[root@dev ~]# ./devtunnel host -p 8000
Hosting port: 8000
Connect via browser: https://601tvmcl.inc1.devtunnels.ms:8000, https://601tvmcl-8000.inc1.devtunnels.ms
Inspect network activity: https://601tvmcl-8000-inspect.inc1.devtunnels.ms

Ready to accept connections for tunnel: majestic-ocean-3x8q6r8
```

浏览器访问：https://601tvmcl.inc1.devtunnels.ms:8000  或者  https://601tvmcl-8000.inc1.devtunnels.ms 使用授权的 Github 账号登陆即可访问！

启动匿名访问：`./devtunnel host -p 8000 --allow-anonymous`

```
[root@dev ~]# ./devtunnel host -p 8000 --allow-anonymous
Hosting port: 8000
Connect via browser: https://sc42hdl7.inc1.devtunnels.ms:8000, https://sc42hdl7-8000.inc1.devtunnels.ms
Inspect network activity: https://sc42hdl7-8000-inspect.inc1.devtunnels.ms

Ready to accept connections for tunnel: sneaky-cat-mxlcl7z
```

任何人浏览器访问 `https://sc42hdl7.inc1.devtunnels.ms:8000  或者  https://sc42hdl7-8000.inc1.devtunnels.ms` 即可！

文末有更高级的用法说明，上面内容基本满足临时使用！

### 测试速度

博主测试通过转发下载，速度还行！大概 1M-2M 每秒的速度！

![](https://mmbiz.qpic.cn/mmbiz_jpg/bG6onXJ2q5UzoaqGytsSe45icveBgMwmcI0gbHHqhiaDLxjCVzl3Dtciar5LcH6AF1olesyFqNqianx1VZumiaibHd9w/640?wx_fmt=jpeg&from=appmsg)

官方说明网速限制为 20MB/s，但是内网穿透受本地服务器上行带宽的影响。一般家庭带宽大家关注的是下行带宽，上行带宽一般是缩水的。

使用限制
----

以下限制每个月重置！

<table><thead><tr><th>资源</th><th>限制</th></tr></thead><tbody><tr><td>带宽流量</td><td>每个用户 2 GB</td></tr><tr><td>隧道</td><td>每个用户 5 个</td></tr><tr><td>活跃连接数</td><td>每个端口 20 个</td></tr><tr><td>端口</td><td>每条隧道 10 个</td></tr><tr><td>HTTP 请求率</td><td>每个端口 1500 / 分钟</td></tr><tr><td>数据传输率</td><td>每个隧道高达 20 MB/s</td></tr><tr><td>最大 Web 转发 HTTP 请求正文大小</td><td>16MB</td></tr></tbody></table>

命令说明
----

开发隧道提供了用于创建和管理开发隧道的命令行接口（CLI）工具。 本文介绍各种 `devtunnel` CLI 命令的语法和参数。

`devtunnel` CLI 命令以预览版提供。 将来的版本中，命令名称和选项可能会更改。

### 全局选项

*   `-v, --verbose`：启用详细输出。
    
*   `-?, -h, --help`：显示帮助和使用情况信息。
    

### 管理用户凭据

开发隧道服务需要登录才能授权管理和访问开发隧道。 默认情况下，只有创建开发隧道的用户才能访问开发隧道，但该用户可能会向其他人授予访问权限。

登录后，登录令牌将缓存在系统安全密钥链中，并在过期前几天有效。 注销 CLI 会清除此缓存的令牌，但不会清除任何浏览器 Cookie。 如果浏览器用于通过开发隧道进行身份验证，则可能包括开发隧道访问令牌。

<table aria-label="表 1"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel user login</code></td><td>使用 Microsoft 或 GitHub 帐户登录。</td></tr><tr><td><code>devtunnel user logout</code></td><td>清除缓存的令牌</td></tr><tr><td><code>devtunnel user show</code></td><td>显示当前登录状态</td></tr></tbody></table>

提示

`devtunnel login` 以及 `devtunnel logout` 用于登录和注销的速记命令。

下面是有关使用这些命令的一些示例：

<table aria-label="表 2"><thead><tr><th>示例</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel user login</code></td><td>使用 Microsoft 组织（Microsoft Entra ID）或个人帐户登录</td></tr><tr><td><code>devtunnel user login -g</code></td><td>使用 GitHub 帐户登录</td></tr><tr><td><code>devtunnel user login -d</code></td><td>如果无法通过本地交互式浏览器登录，请使用设备代码登录的 GitHub 帐户<em>登录</em></td></tr><tr><td><code>devtunnel user login -g -d</code></td><td>如果无法通过本地交互式浏览器登录，请使用设备代码登录的 GitHub 帐户<em>登录</em></td></tr></tbody></table>

### 托管开发隧道

`devtunnel host` 是用于托管开发隧道的主命令。 应在运行要通过开发隧道访问的服务器的主机系统上运行该命令。

<table aria-label="表 3"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel host</code></td><td>托管开发隧道。 如果未指定开发隧道 ID，则会创建一个新的 <em>临时</em> 开发隧道，该隧道在连接关闭后将被删除。</td></tr></tbody></table>

下面是有关使用此命令的一些示例：

<table aria-label="表 4"><thead><tr><th>示例</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel host -p 3000</code></td><td>为在主机系统上侦听端口 3000 的服务器托管临时开发隧道。</td></tr><tr><td><code>devtunnel host -p 3000 --allow-anonymous</code></td><td>托管临时开发隧道并启用匿名客户端访问。</td></tr><tr><td><code>devtunnel host -p 3000 5000</code></td><td>为侦听端口 3000 和 5000 的本地服务器托管临时开发隧道。</td></tr><tr><td><code>devtunnel host -p 8443 --protocol https</code></td><td>为使用 HTTPS 协议的端口 8443 上的服务器托管临时开发隧道。</td></tr><tr><td><code>devtunnel host -p 8000 --expiration 2d</code></td><td>托管具有自定义过期时间的临时开发隧道。 最小值为 1 小时（1 小时），最大值为 30 天（30d）。</td></tr><tr><td><code>devtunnel host TUNNELID</code></td><td>托管以前已配置的现有开发隧道。</td></tr></tbody></table>

允许匿名访问开发隧道意味着 Internet 上的任何人都可以连接到本地服务器（如果他们可以猜测开发隧道 ID）。

按 **Control-C** 停止开发隧道主机进程，并通过开发隧道终止任何客户端连接。  
如果未提供现有开发隧道，进程自动创建的开发隧道将在进程退出时被删除。

### 连接开发隧道

**使用 Web 转发 UI：**

该 `devtunnel host` 命令显示类似于以下内容的输出：

```
Hosting port 3000 at https://l3rs99qw-3000.usw2.devtunnels.ms/
```

显示的 `https:` URI 对于开发隧道端口是唯一的：第一个组件是包含给定开发隧道 ID 和端口号的子域。

如果托管端口连接到 Web 服务器，则可以从任何位置直接在浏览器中打开该 URI。 如果访问开发隧道需要授权，则对 URI 的初始请求将重定向到登录页，并在用户获得授权后返回到站点。

如果托管端口连接到 Web 服务，则该 URI 可由 Web 服务客户端应用程序用作基 URI。 但是，如果开发隧道不允许匿名访问，则 Web 服务客户端通常不知道如何进行身份验证。 如果 Web 服务可以安全地公开，请考虑允许匿名访问。 否则，Web 服务客户端可能会添加具有开发隧道访问令牌的请求标头来授权连接。

**使用 CLI：**

CLI 可用于将客户端端口上的端口转发到开发隧道端口，而不是让客户端浏览器或应用程序直接连接到开发隧道中继 URI。 如果开发隧道不允许匿名访问，客户端可能还需要登录。

```
devtunnel connect TUNNELID
```

*   替换为 `TUNNELID` 主机上使用的同一开发隧道 ID。
    

成功的客户端输出类似于以下内容：

```
Connected to tunnel: l3rs99qwSSH: Forwarding from 127.0.0.1:3000 to host port 3000.SSH: Forwarding from [::1]:3000 to host port 3000.
```

现在，在客户端上可以使用 `localhost:3000` IPv4 或 IPv6 在主机上共享的服务器 3000。 （“SSH” 前缀是因为开发隧道服务基于用于端口转发的标准 SSH 协议生成。如果托管端口连接到 Web 服务器， `http://localhost:3000/` 则可以在浏览器中打开。 在这种情况下，无需进一步授权，因为客户端的 CLI 登录令牌用于在必要时授权连接。

### 高级：管理开发隧道

无需托管开发隧道即可创建开发隧道。 这对于高级开发隧道配置和管理非常有用，例如：

*   列出所有拥有开发隧道
    
*   添加和删除开发隧道的端口
    
*   管理开发隧道访问控制
    
*   将元数据添加到开发隧道，例如说明和标记
    

<table aria-label="表 5"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel create</code></td><td>创建持久性开发隧道</td></tr><tr><td><code>devtunnel list</code></td><td>列出开发隧道</td></tr><tr><td><code>devtunnel show</code></td><td>显示开发隧道详细信息</td></tr><tr><td><code>devtunnel update</code></td><td>更新开发隧道属性</td></tr><tr><td><code>devtunnel delete</code></td><td>删除开发隧道</td></tr><tr><td><code>devtunnel delete-all</code></td><td>删除所有开发隧道</td></tr></tbody></table>

下面是有关使用这些命令的一些示例：

<table aria-label="表 6"><thead><tr><th>示例</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel create -a</code></td><td>创建允许匿名访问的持久性开发隧道。</td></tr><tr><td><code>devtunnel create -d 'my tunnel description'</code></td><td>创建具有不可搜索说明的持久开发隧道。</td></tr><tr><td><code>devtunnel create --expiration 4h</code></td><td>创建具有自定义过期时间的持久开发隧道。 最小值为 1 小时（1 小时），最大值为 30 天（30d）。</td></tr><tr><td><code>devtunnel create myTunnelID</code></td><td>创建具有自定义隧道 ID 的持久开发隧道。</td></tr><tr><td><code>devtunnel create --tags my-web-app v1</code></td><td>创建持久开发隧道并应用可搜索标记。</td></tr><tr><td><code>devtunnel list --tags my-web-app</code></td><td>列出具有任何指定标记的开发隧道。</td></tr><tr><td><code>devtunnel list --all-tags my-web-app v1</code></td><td>列出具有所有指定标记的开发隧道。</td></tr><tr><td><code>devtunnel show</code></td><td>显示上次使用的开发隧道的详细信息。</td></tr><tr><td><code>devtunnel show TUNNELID</code></td><td>显示开发隧道的详细信息。</td></tr><tr><td><code>devtunnel update TUNNELID -d 'my new tunnel description'</code></td><td>更新开发隧道的说明。</td></tr><tr><td><code>devtunnel update TUNNELID --remove-tags</code></td><td>从开发隧道中删除所有标记。</td></tr><tr><td><code>devtunnel update TUNNELID --expiration 10d</code></td><td>使用新的自定义过期时间更新开发隧道。 最小值为 1 小时（1 小时），最大值为 30 天（30d）。</td></tr><tr><td><code>devtunnel delete TUNNELID</code></td><td>删除开发隧道。</td></tr><tr><td><code>devtunnel delete-all</code></td><td>删除所有开发隧道。</td></tr></tbody></table>

提示：大多数 CLI 命令隐式地对上次使用的开发隧道进行操作，但如有必要，可以选择指定开发隧道 ID。

### 高级：管理开发隧道端口

最初使用 `devtunnel create` 命令创建的开发隧道没有端口。 使用 `devtunnel port` 命令在托管之前添加端口：

<table aria-label="表 7"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel port create</code></td><td>创建开发隧道端口</td></tr><tr><td><code>devtunnel port list</code></td><td>列出开发隧道端口</td></tr><tr><td><code>devtunnel port show</code></td><td>显示开发隧道端口详细信息</td></tr><tr><td><code>devtunnel port update</code></td><td>更新开发隧道端口属性</td></tr><tr><td><code>devtunnel port delete</code></td><td>删除开发隧道端口</td></tr></tbody></table><table aria-label="表 8"><thead><tr><th>示例</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel port create -p 3000 --protocol http</code></td><td>添加具有指定协议的端口</td></tr><tr><td><code>devtunnel port list TUNNELID</code></td><td>列出当前端口</td></tr><tr><td><code>devtunnel port show TUNNELID -p 3000</code></td><td>显示端口 3000 的详细信息</td></tr><tr><td><code>devtunnel port update -p 3000 --description 'frontend port'</code></td><td>更新开发隧道端口说明</td></tr><tr><td><code>devtunnel port delete -p 3000</code></td><td>删除端口</td></tr></tbody></table>

创建端口时，如果自动检测无法正常工作，则可以选择指定协议。 当前选项为 “http”、“https” 或“auto”（默认值）。 如果托管端口为 HTTPS，则建议将端口协议设置为 “https”; 否则，“auto” 可能很好。

使用上述命令配置开发隧道后，开始托管它：

```
devtunnel host
```

### 高级：管理开发隧道访问

使用以下命令，可以颁发开发隧道访问令牌，以提供对开发隧道的其他客户端访问权限，而无需允许匿名访问。 访问控制项命令允许你在开发隧道和开发隧道端口上配置访问控制。

<table aria-label="表 9"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel token</code></td><td>颁发开发隧道访问令牌</td></tr><tr><td><code>devtunnel access create</code></td><td>创建访问控制项</td></tr><tr><td><code>devtunnel access list</code></td><td>列出访问控制条目</td></tr><tr><td><code>devtunnel access delete</code></td><td>删除访问控制项</td></tr><tr><td><code>devtunnel access reset</code></td><td>将访问控制条目重置为默认值</td></tr></tbody></table>

下面是有关使用这些命令的一些示例：

<table aria-label="表 10"><thead><tr><th>示例</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel token TUNNELID --scopes connect</code></td><td>获取可共享的开发隧道的 “连接” 访问令牌，以便暂时访问开发隧道。</td></tr><tr><td><code>devtunnel access create TUNNELID --anonymous</code></td><td>在开发隧道上启用匿名客户端访问。</td></tr><tr><td><code>devtunnel access create TUNNELID --anonymous --expiration 4h</code></td><td>使用自定义访问控制过期时间在开发隧道上启用匿名客户端访问。 最小值为 1 小时（1 小时），最大值为 30 天（30d）。</td></tr><tr><td><code>devtunnel access create TUNNELID --port 3000 --anonymous</code></td><td>在端口 3000 上启用匿名客户端访问。</td></tr><tr><td><code>devtunnel access create TUNNELID --tenant</code></td><td>在开发隧道上启用当前的 Microsoft Entra 租户访问。</td></tr><tr><td><code>devtunnel access create TUNNELID --org ORG</code></td><td>在开发隧道上按名称启用 GitHub 组织访问。</td></tr></tbody></table>

提示：GitHub 组织访问权限需要 将 Dev Tunnels GitHub 应用安装到组织。

### 补充命令

如果需要显式设置或取消设置上次使用开发隧道的此本地缓存，可以使用这些命令。

<table aria-label="表 11"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel set</code></td><td>设置默认开发隧道</td></tr><tr><td><code>devtunnel unset</code></td><td>清除默认开发隧道</td></tr></tbody></table>

### 诊断命令

<table aria-label="表 12"><thead><tr><th>命令</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel clusters</code></td><td>按位置列出可用的服务群集</td></tr><tr><td><code>devtunnel echo</code></td><td>在本地端口上运行诊断回显服务器</td></tr><tr><td><code>devtunnel ping</code></td><td>将诊断消息发送到远程回显服务器</td></tr></tbody></table><table aria-label="表 13"><thead><tr><th>示例</th><th>说明</th></tr></thead><tbody><tr><td><code>devtunnel clusters --ping</code></td><td>列出按度量延迟排序的可用服务群集。</td></tr><tr><td><code>devtunnel echo http --port 8080 --interface 127.0.0.1</code></td><td>在端口 8080 上启动本地 http 诊断服务器。</td></tr></tbody></table>

### 疑难解答

若要排查 CLI 问题 `devtunnel` ，以下提示可能很有用：

*   确保使用的是最新版本的 `devtunnel` CLI。 使用 `devtunnel --version`.. 检查当前安装的版本。
    
*   该 `--verbose` 选项输出调试消息，这些消息可以提供额外的诊断信息。
    

**提醒：更完整以及后续请点击：****阅读原文** **去发现****！**

![](https://mmbiz.qpic.cn/mmbiz_gif/bG6onXJ2q5X1gl7CTtCLIrM623xwsHT7HPzDILrFxKUzvpvW3pKZqUHxRvicT8qQbeCnglffRWGI3PAqCGyBib9g/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC6iavic0tIJIoZCwKvUYnFFiaibvE8yamPS1RojR1NmfucyIyYeOvaY7CcN5WW1hVg11mZCuwayLtnnBQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)