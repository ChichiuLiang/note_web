> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6997589728459489317)

**这是我参与 8 月更文挑战的第 18 天，活动详情查看：[8 月更文挑战](https://juejin.cn/post/6987962113788493831 "https://juejin.cn/post/6987962113788493831")**

> 本期带来 Nginx 系列的最后一篇，讲解 Nginx location 如何进行路由转发，如何通过配置 location 实现自己想要的路由转发效果。

> 更多文章在我的 Github 及个人公众号【全栈道路】上，欢迎观赏[【前端知识点】](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fprogrammer-zhang%2Ffront-end "https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fprogrammer-zhang%2Ffront-end")，如有受益，不要钱，小手点个 Star。

阅读本文您将收获
--------

*   nginx location 配置
*   nginx 跨域原理

Nginx location 配置
-----------------

### Nginx location 语法规则

*   `location [=|~|~*|^~] /uri/ { … }`
    *   = 开头表示精确匹配
    *   ^~ 开头表示 uri 以某个常规字符串开头，理解为匹配 url 路径即可，匹配符合后，不再向下匹配
    *   ~ 开头表示区分大小写的正则匹配
    *   ~* 开头表示不区分大小写的正则匹配
    *   !~ 和 !~* 分别为区分大小写不匹配及不区分大小写不匹配 的正则
    *   / 通用匹配，任何请求都会匹配到

### 实际使用建议

*   必选规则
    
    *   直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理
    *   这里是直接转发给后端应用服务器了，也可以是一个静态首页
    
    ```
    location = / {
        proxy_pass http://xxxx.com/index
    }
    ```
    
*   必选规则
    
    *   处理静态文件请求，这是 nginx 作为 http 服务器的强项
    *   有两种配置模式，目录匹配或后缀匹配, 任选其一或搭配使用
    
    ```
    location ^~ /static/ {
        root /webroot/static/;
    }
    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        root /webroot/res/;
    }
    ```
    
*   通用规则
    
    *   用来转发动态请求到后端应用服务器, 可以在前端页面增加路由转发，增加一个共同的前缀
    *   非静态文件请求就默认是动态请求，自己根据实际把握
    *   毕竟目前的一些框架的流行，带. php,.jsp 后缀的情况很少了
    
    ```
    location /api/ {
        proxy_pass http://xxxx.com/api/abc
    }
    ```
    

### Rewrite 语法规则

*   last – 基本上都用这个 Flag，表示完成 Rewrite
*   break – 中止 Rewirte，不在继续匹配
    *   last 和 break 的区别
        *   last 一般写在 server 和 if 中，而 break 一般使用在 location 中
        *   last 不终止重写后的 url 匹配，即新的 url 会再从 server 走一遍匹配流程，而 break 终止重写后的匹配
        *   break 和 last 都能组织继续执行后面的 rewrite 指令
*   redirect – 返回临时重定向的 HTTP 状态 302，地址栏显示跳转后的地址
*   permanent – 返回永久重定向的 HTTP 状态 301，地址栏显示跳转后的地址
    *   redirect 和 permanent
        *   临时重定向：对旧网址没有影响，但新网址不会有排名
        *   永久重定向：新网址完全继承旧网址，旧网址的排名等完全清零

### 常见全局变量

*   `$args`： 这个变量等于请求行中的参数，同 $query_string
*   `$content_length`： 请求头中的 Content-length 字段。
*   `$content_type`： 请求头中的 Content-Type 字段。
*   `$document_root`： 当前请求在 root 指令中指定的值。
*   `$host`： 请求主机头字段，否则为服务器名称。
*   `$http_user_agent`： 客户端 agent 信息
*   `$http_cookie`： 客户端 cookie 信息
*   `$limit_rate`： 这个变量可以限制连接速率。
*   `$request_method`： 客户端请求的动作，通常为 GET 或 POST。
*   `$remote_addr`： 客户端的 IP 地址。
*   `$remote_port`： 客户端的端口。
*   `$remote_user`： 已经经过 Auth Basic Module 验证的用户名。
*   `$request_filename`： 当前请求的文件路径，由 root 或 alias 指令与 URI 请求生成。
*   `$scheme`： HTTP 方法（如 http，https）。
*   `$server_protocol`： 请求使用的协议，通常是 HTTP/1.0 或 HTTP/1.1。
*   `$server_addr`： 服务器地址，在完成一次系统调用后可以确定这个值。
*   `$server_name`： 服务器名称。
*   `$server_port`： 请求到达服务器的端口号。
*   `$request_uri`： 包含请求参数的原始 URI，不包含主机名，如：”/abc/index.html?token=1a”
*   `$uri`： 不带请求参数的当前 URI，$uri 不包含主机名，如”/foo/bar.html”。
*   `$document_uri`： 与 $uri 相同。

### 一个关于重定向的参数处理问题

*   问题描述：Nginx 在进行 rewrite 后都会自动添加上旧地址中的参数部分，而这对于重定向到的新地址来说可能是多余
*   解决方案：
    *   转发规则添加 `?` 后缀
*   问题实例：
    *   把 `http://xxx.com/test.html?params=xxx` 重定向到 `http://xxx.com/new.html`
    *   若按照默认的写法：`rewrite ^/test.html(.*) /new permanent;`
    *   重定向后的结果是：`http://xxx.com/new.html?params=xxx`
    *   如果改写成：`rewrite ^/test.html(.*) /new.html? permanent;`
    *   那结果就是：`http://xxx.com/new.html`
*   指定参数 Rewirte
    *   `rewrite ^/test.html /new.html permanent;` // 重写向带参数的地址
    *   `rewrite ^/test.html /new.html? permanent;` // 重定向后不带参数
    *   `rewrite ^/test.html /new.html?id=$arg_id? permanent;` // 重定向后带指定的参数

### 消除 `proxy_pass` 转发 `/`的恐怖阴影

*   在配置 Nginx 过程中路由转发的一个小小 `/` 可能会给你带来很多麻烦，虽然只是一个小小的斜杠，但是转发的结果千差万别
*   两种情况
    
    *   `proxy_pass` 不带`/`
    
    ```
    location /alpha/ {
        proxy_pass  http://192.168.xxx.xxx:80;
    }
    http://domain/alpha/ --> http://192.168.xxx.xxx:80/alpha/
    http://domain/alpha/beta/abc --> http://192.168.xxx.xxx:80/alpha/beta/abc
    ```
    
    *   `proxy_pass` 带`/`
    
    ```
    location /alpha/ {
        proxy_pass  http://192.168.xxx.xxx:80/;
    }
    http://domain/alpha/ --> http://192.168.xxx.xxx:80/
    http://domain/alpha/beta/abc --> http://192.168.xxx.xxx:80/beta/abc
    ```
    

Nginx 能够跨域的原理
-------------

### 需要了解的几个知识点

*   跨域：一个域的 javascript 脚本和另外一个域的内容进行交互在浏览器端是不被允许的。
*   同源：两个页面具有相同的协议（protocol），主机（host）和端口号（port），只要有其一不同都属于不同源
*   同源策略：是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响
*   跨域影响
    *   无法读取非同源网页的 Cookie、LocalStorage 和 IndexedDB
    *   无法接触非同源网页的 DOM
    *   无法向非同源地址发送 AJAX 请求

### 举例说明跨域

<table><thead><tr><th>当前页面 url</th><th>被请求页面 url</th><th>是否跨域</th><th>原因</th></tr></thead><tbody><tr><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%2F" target="_blank" title="http://www.test.com/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com/</a></td><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%2Findex.html" target="_blank" title="http://www.test.com/index.html" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com/index.html</a></td><td>否</td><td>同源（协议、域名、端口号相同）</td></tr><tr><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%2F" target="_blank" title="http://www.test.com/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com/</a></td><td><a href="https://link.juejin.cn?target=https%3A%2F%2Fwww.test.com%2Findex.html" target="_blank" title="https://www.test.com/index.html" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com/index.html</a></td><td>跨域</td><td>协议不同（http/https）</td></tr><tr><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%2F" target="_blank" title="http://www.test.com/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com/</a></td><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.baidu.com%2F" target="_blank" title="http://www.baidu.com/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.baidu.com/</a></td><td>跨域</td><td>主域名不同（test/baidu）</td></tr><tr><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%2F" target="_blank" title="http://www.test.com/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com/</a></td><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fblog.test.com%2F" target="_blank" title="http://blog.test.com/" ref="nofollow noopener noreferrer" one-link-mark="yes">blog.test.com/</a></td><td>跨域</td><td>子域名不同（www/blog）</td></tr><tr><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%3A8080%2F" target="_blank" title="http://www.test.com:8080/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com:8080/</a></td><td><a href="https://link.juejin.cn?target=http%3A%2F%2Fwww.test.com%3A7001%2F" target="_blank" title="http://www.test.com:7001/" ref="nofollow noopener noreferrer" one-link-mark="yes">www.test.com:7001/</a></td><td>跨域</td><td>端口号不同（8080/7001）</td></tr></tbody></table>

### 跨域原理

*   同源策略仅存在于客户端，服务器端不存在同源策略

写在最后
----

如果你觉得这篇文章对你有益，烦请点赞以及分享给更多需要的人！

##### 欢迎关注【全栈道路】及微信公众号【全栈道路】，获取更多好文及免费书籍！

##### 有需要【百度】&【字节跳动】&【京东】&【猿辅导】内推的也请留言哦，你将享受 VIP 级极速内推服务~

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eeb06ecf8de04649853ede815e17b21a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/761230e18b6940f7a36ae73c303a787e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.jpg?format=original)

往期好文
----

[微信 JS API 支付的实现](https://juejin.cn/post/6993980017227563021#heading-18 "https://juejin.cn/post/6993980017227563021#heading-18")

[创建个性化的 Github 个人主页](https://juejin.cn/post/6991424007069237262 "https://juejin.cn/post/6991424007069237262")

[面试官问你`<img>`是什么元素时你怎么回答](https://juejin.cn/post/6901104219995635725 "https://juejin.cn/post/6901104219995635725")

[特殊的 JS 浮点数的存储与计算](https://juejin.cn/post/6922727457212612621 "https://juejin.cn/post/6922727457212612621")

[[万字长文] 百度和好未来面试经含答案 | 掘金技术征文](https://juejin.cn/post/6844904116267778055 "https://juejin.cn/post/6844904116267778055")

[前端实用正则表达式 & 小技巧，一股脑全丢给你🏆 掘金技术征文 | 双节特别篇](https://juejin.cn/post/6879418373365563399 "https://juejin.cn/post/6879418373365563399")

[冷门的 HTML tabindex 详解](https://juejin.cn/post/6888924266008412167 "https://juejin.cn/post/6888924266008412167")

[几行代码教你解决微信生成海报及二维码](https://juejin.cn/post/6885952653075939335 "https://juejin.cn/post/6885952653075939335")

[Vue3.0 响应式数据原理：ES6 Proxy](https://juejin.cn/post/6878097147649064974 "https://juejin.cn/post/6878097147649064974")

[[前端面试] 前端缓存问题看这篇，让面试官爱上你](https://juejin.cn/post/6844904078615511054 "https://juejin.cn/post/6844904078615511054")

[如何优雅地画一条细线](https://juejin.cn/post/6992184390210027527 "https://juejin.cn/post/6992184390210027527")

[[三分钟小文] 前端性能优化 - HTML、CSS、JS 部分](https://juejin.cn/post/6844904065688666119 "https://juejin.cn/post/6844904065688666119")

[[三分钟小文] 前端性能优化 - 页面加载速度优化](https://juejin.cn/post/6844904065688682510 "https://juejin.cn/post/6844904065688682510")

[[三分钟小文] 前端性能优化 - 网络传输层优化](https://juejin.cn/post/6844904065692860424 "https://juejin.cn/post/6844904065692860424")