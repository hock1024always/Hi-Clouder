# nginx

## What：什么是nginx？

Nginx是一个高性能的 **开源 Web 服务器**，同时也可作为 **反向代理****服务器、****负载均衡****器** 和 **HTTP 缓存**。它由俄罗斯工程师 Igor Sysoev 开发，最初发布于 2004 年，现已成为全球最流行的 Web 服务器之一（与 Apache 齐名）。

## Why：为什么要使用nginx？

Nginx 因其 **高性能、低资源占用** 和 **高并发处理能力** 而广受欢迎，主要适用于以下场景：

1. **作为 Web 服务器（静态内容服务）**

Nginx 能高效处理静态文件（HTML、CSS、JS、图片等），比传统服务器（如 Apache）更快，适合静态网站或前后端分离架构。 **示例配置：**

```Nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;  # 静态文件目录
}
```

1. **反向代理****（****Reverse Proxy****）**

Nginx 可以接收客户端请求，并转发到后端服务器（如 Node.js、Python、Java 应用），隐藏真实服务器 IP，提升安全性。 **示例配置：**

```Nginx
server {
    listen 80;
    server_name api.example.com;
    location / {
        proxy_pass http://localhost:3000;  # 转发到本地的 Node.js 服务
    }
}
```

1. **负载均衡****（Load Balancing）**

当流量较大时，Nginx 可以将请求分发到多个后端服务器，避免单点过载，提高系统可用性。 **示例配置（****轮询****策略）：**

```nginx
upstream backend {
    server 192.168.1.100:8080;
    server 192.168.1.101:8080;
}
server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

1. **HTTP 缓存与加速**

Nginx 可以缓存静态资源甚至动态内容，减少后端服务器压力，加快用户访问速度。 **示例配置：**

```Nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m inactive=60m;
server {
    location / {
        proxy_cache my_cache;
        proxy_pass http://backend;
    }
}
```

1. **SSL****/****TLS** **终止（****HTTPS** **支持）**

Nginx 可以统一管理 HTTPS 证书，减轻后端服务器的 SSL 解密负担。 **示例配置：**

```Nginx
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    location / {
        proxy_pass http://backend;
    }
}
```

1. **高并发与低资源占用**

Nginx 采用 **事件驱动（Event-Driven）架构**，单机可轻松支持数万并发连接，而内存占用极低（相比 Apache 更节省资源）。

## How：如何使用nginx

这里不详细阐述，可以跟着下面的文章进行实践学习

[nginx学习，看这一篇就够了：下载、安装。使用：正向代理、反向代理、负载均衡。常用命令和配置文件,很全-CSDN博客](https://blog.csdn.net/qq_40036754/article/details/102463099)

[nginx学习：配置文件详解，负载均衡三种算法学习，上接nginx实操篇_负载均衡如何处理文件-CSDN博客](https://blog.csdn.net/qq_40036754/article/details/127775066)

其他文档： [Nginx安装 | Nginx 中文官方文档](https://wizardforcel.gitbooks.io/nginx-doc/content/Text/1.3_install.html)

[Ubuntu系统下Nginx安装_ubuntu安装nginx-CSDN博客](https://blog.csdn.net/zxm528/article/details/130826570)