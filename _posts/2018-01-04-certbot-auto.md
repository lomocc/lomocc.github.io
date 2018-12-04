---
title: 使用 Let’s Encrypt 申请域名证书
tag: devops
---
# 使用 Let’s Encrypt 申请域名证书
[![Let’s Encrypt](https://letsencrypt.org/images/letsencrypt-logo-horizontal.svg)](https://certbot.eff.org/)

## 域名解析
为 `example.com` 域名添加 `A 记录`，指向 `服务器 IP`
## 获取 `certbot` 工具
```
$ wget https://dl.eff.org/certbot-auto
$ chmod a+x certbot-auto
```
## 申请证书

一般我使用 `--standalone` 方式，会创建一个 `web 服务器` 监听 `80` 端口来自动实现 `Let’s Encrypt` 远程验证，（如果 80 端口被占用，需要先关闭占用程序，如： `nginx -s stop`）
```
$ ./certbot-auto certonly --email example@email.com -d example.com --standalone
```
  如果已经有 `web 服务器` 监听 `80` 端口，可以使用 `--webroot` 方式，使用 `-w` 指定服务器根目录（程序会在根目录中生成 `/.well-known/acme-challenge/` 来实现验证）
```
$ ./certbot-auto certonly --email example@email.com -d example.com --webroot -w /usr/share/nginx/html
```

## 配置 `nginx` 
```
server {
    listen 443;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
}
```
最后重启服务器就能看到绿色的锁了。
