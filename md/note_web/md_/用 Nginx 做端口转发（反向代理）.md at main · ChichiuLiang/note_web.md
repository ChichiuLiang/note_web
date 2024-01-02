> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [github.com](https://github.com/ChichiuLiang/note_web/tree/main/md)

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/108740468)

有时我们会使用一些 java 或 node 应用，但又不想让他们直接监听 80 端口，这时就需要用到端口转发

本文中，我们介绍 Nginx 如何做端口转发，还有各种转发规则

[](#将域名转发到本地端口)将域名转发到本地端口
-------------------------

首先介绍最常用的，将域名转发到本地另一个端口上

```
server{
  listen 80;
  server_name  tomcat.shaochenfeng.com;
  index  index.php index.html index.htm;

  location / {
    proxy_pass  http://127.0.0.1:8080; # 转发规则
    proxy_set_header Host $proxy_host; # 修改转发请求头，让8080端口的应用可以受到真实的请求
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

这样访问 [http://tomcat.shaochenfeng.com](http://tomcat.shaochenfeng.com) 时就会转发到本地的 8080 端口

[](#将域名转发到另一个域名)将域名转发到另一个域名
---------------------------

```
server{
  listen 80;
  server_name  baidu.shaochenfeng.com;
  index  index.php index.html index.htm;

  location / {
    proxy_pass  http://www.baidu.com;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

这样访问 [http://baidu.shaochenfeng.com](http://baidu.shaochenfeng.com) 时就会转发到 [http://www.baidu.com](http://www.baidu.com)

[](#本地一个端口转发到另一个端口或另一个域名)本地一个端口转发到另一个端口或另一个域名
---------------------------------------------

```
server{
  listen 80;
  server_name 127.0.0.1; # 公网ip
  index  index.php index.html index.htm;

  location / {
    proxy_pass  http://127.0.0.1:8080; # 或 http://www.baidu.com
    proxy_set_header Host $proxy_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

这样访问 [http://127.0.0.1](http://127.0.0.1) 时就会转发到本地的 8080 端口或 [http://www.baidu.com](http://www.baidu.com)

[](#加--与不加-)加 / 与不加 /
---------------------

在配置 proxy_pass 代理转发时，如果后面的 url 加 /，表示绝对根路径；如果没有 /，表示相对路径

例如

1.  加 /

```
server_name shaochenfeng.com
location /data/ {
    proxy_pass http://127.0.0.1/;
}
```

访问 [http://shaochenfeng.com/data/index.html](http://shaochenfeng.com/data/index.html) 会转发到 [http://127.0.0.1/index.html](http://127.0.0.1/index.html)

1.  不加 /

```
server_name shaochenfeng.com
location /data/ {
    proxy_pass http://127.0.0.1;
}
```

访问 [http://shaochenfeng.com/data/index.html](http://shaochenfeng.com/data/index.html) 会转发到 [http://127.0.0.1/data/index.html](http://127.0.0.1/data/index.html)

欢迎查看更多运维技术文章

[邵晨峰的官网](https://www.shaochenfeng.com)