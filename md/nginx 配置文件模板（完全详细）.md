> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/619165119)

以下是一个完整的 nginx 配置文件模板，包含了所有常见的配置，并且每个部分都有详细的注释说明：

```
# 全局参数
user nginx;              # Nginx进程运行用户
worker_processes auto;   # Nginx工作进程数，通常设置为CPU核数
error_log /var/log/nginx/error.log warn;    # 错误日志路径和日志级别
pid /run/nginx.pid;      # 进程PID保存路径

# 定义事件模块
events {
    worker_connections 1024;    # 每个工作进程最大并发连接数
    use epoll;                  # 使用epoll网络模型，提高性能
    multi_accept on;            # 开启支持多个连接同时建立
}

# 定义HTTP服务器模块
http {
    # 缓存文件目录
    client_body_temp_path /var/cache/nginx/client_temp;
    proxy_temp_path /var/cache/nginx/proxy_temp;
    fastcgi_temp_path /var/cache/nginx/fastcgi_temp;

    # 定义日志格式，main是默认的日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"';

    # 默认访问日志保存路径和格式
    access_log /var/log/nginx/access.log main;

    # 定义MIME类型
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 代理参数
    proxy_connect_timeout 6s;       # 连接超时时间
    proxy_send_timeout 10s;         # 发送超时时间
    proxy_read_timeout 10s;         # 接收超时时间
    proxy_buffer_size 16k;          # 缓冲区大小
    proxy_buffers 4 32k;            # 缓冲区个数和大小
    proxy_busy_buffers_size 64k;    # 忙碌缓冲区大小
    proxy_temp_file_write_size 64k; # 代理临时文件写入大小

    # 启用压缩，可以提高网站访问速度
    gzip on;
    gzip_min_length 1k;                    # 最小压缩文件大小
    gzip_types text/plain text/css application/json application/javascript application/xml;

    # 定义HTTP服务器
    server {
        listen 80;              # 监听端口

        server_name example.com;    # 域名

        # 重定向到HTTPS，强制使用HTTPS访问
        if ($scheme != "https") {
            return 301 https://$server_name$request_uri;
        }

        # HTTPS服务器配置
        ssl_certificate      /etc/nginx/ssl/server.crt;    # SSL证书路径
        ssl_certificate_key  /etc/nginx/ssl/server.key;    # SSL私钥路径

        # SSL会话缓存参数
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

        # 配置代理路径
        location / {
            proxy_pass http://localhost:8080;        # 转发请求的目标地址
            proxy_set_header Host $host;             # 设置请求头中的Host字段
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                            # 设置HTTP头中的X-Forwarded-For字段，表示客户端真实IP，多个IP用逗号隔开
            proxy_set_header X-Real-IP $remote_addr; # 设置请求头中的X-Real-IP字段，表示客户端真实IP
        }

        # 配置静态文件访问路径
        location /static/ {
            alias /path/to/static/files/;   # 静态文件的目录
            expires 7d;                     # 静态文件缓存时间
            add_header Pragma public;       # 添加HTTP响应头
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }

        # 配置错误页面
        error_page 404 /404.html;           # 404错误页
        location = /404.html {
            internal;                       # 不接受外部访问
            root /usr/share/nginx/html;     # 404错误页文件所在目录
        }

        # 配置重定向
        location /old/ {
            rewrite ^/old/([^/]+) /new/$1 permanent;   # 将/old/xxx路径重定向为/new/xxx，返回301状态码
        }
    }

    # 其他服务配置
    # server {
    #     ...
    # }

    # 配置TCP负载均衡
    upstream backends {
        server backend1.example.com:8080 weight=5;  # 后端服务器地址和权重
        server backend2.example.com:8080;
        server backend3.example.com:8080 backup;   # 备用服务器
        keepalive 16;                               # 连接池大小
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backends;             # 负载均衡转发请求的目标地址
            proxy_set_header Host $host;            # 设置请求头中的Host字段
            proxy_set_header X-Real-IP $remote_addr; # 设置请求头中的X-Real-IP字段，表示客户端真实IP
        }
    }
}
```

这个配置文件包含了全局参数、事件模块、HTTP 服务器、服务定义、静态文件访问路径、错误页面、重定向、TCP 负载均衡等各种配置，并且每个部分都有详细的注释。您可以根据具体需求进行修改和优化。