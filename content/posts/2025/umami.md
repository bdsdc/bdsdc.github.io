---
title: "运维| 本地部署umami网站统计"
date: 2025-02-10
lastmod: 2025-02-10
author: 'bdsdc'
linkToMarkdown: true
categories: ['运维']
tags: ['web']
---

# 本地部署umami网站统计

## 前言
在数字化时代，网站数据对于每一位站长、博主或企业来说，就如同航海中的灯塔，指引着我们前行的方向。我们渴望了解每天有多少双眼睛在浏览我们精心打造的内容，访客来自何方，他们又对哪些页面情有独钟。而传统的网站统计工具，如 Google Analytics，虽然功能强大，但也存在着数据隐私、使用复杂等问题。现在，有一个全新的选择摆在我们面前 ——Umami，一个基于 Nodejs 开发的轻量级网站浏览统计系统。它不仅能让你轻松搭建属于自己的网站统计后台，还在诸多方面展现出超越传统工具的优势，为你带来更直观、更便捷、更安全的网站数据洞察体验。接下来，就让我们一同深入探索 Umami 的奇妙世界。


## umami介绍
- 支持本地部署
- 需要数据库支持
开源地址：https://github.com/umami-software/umami/blob/master/docker-compose.yml
网站地址：https://umami.is/
因为我们是要统计网站分析，所以这个网站是要外网可以访问的，现在一般网站都是https，所以umami也是要支持https
这样就可以避免很多不必要的错误和影响。能让我们搭建部署以及使用更顺利，我的个人理解哈~ 

## 使用方案
- 第一种： 通过免费服务 Vercel 和 Supabase 这种方式来实现
- 第二种： 在本地服务器搭建 本地服务器 + cloudflare tunnel代理 
- 第三种： 在云服务器上搭建（我使用这个方案，但是会有一些不同）

## 部署umami
- 通过1panel部署postgre数据库
- 通过WSL-ubuntu用docker方式部署 umami
- 通过云主机nginx 代理ubuntu的umami服务

### 安装postgre数据库
打开1panel，找到商店，搜索postgre数据库 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202502101619421.png)
因为postgre用非root用户启动，会报权限问题，这里我们通过创建docker volume卷，把容器中目录挂载到volume上 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202502101623913.png)

也可以提前命令行创建卷,然后docker-compose最下面的volumes就不需要配置了
```shell
docker volume create postgre-data
```
### 创建umami数据库 
通过1panel创建数据库 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202502101626131.png)

### 安装umami
我们修改一下docker-compose配置，umami会自动链接数据库，进行初始化库,这里我们官方的数据库项删掉，因为我们上面已经安装过

端口我改成8001 
```shell
---
version: '3'
services:
  umami:
    image: ghcr.io/umami-software/umami:postgresql-latest
    ports:
      - "8001:3000"
    environment:
      DATABASE_URL: postgresql://umami:umami123@192.168.31.239:5432/umami  # 用户名:密码@host:port/dbname 
      DATABASE_TYPE: postgresql
      APP_SECRET: replace-me-with-a-random-string
    init: true
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl http://localhost:3000/api/heartbeat"]
      interval: 5s
      timeout: 5s
      retries: 5
```
### 云主机上nginx代理
首先我们现在阿里云上添加解析，前提拥有一个域名，通过certbot配置https证书
```shell
certbot --nginx -d umami.xxx.cn 

server {
  server_name umami.xxx.cn;
  access_log         /data/nginx/logs/umami-xxx-cn-access.log;
  client_header_timeout 1200s;
  client_body_timeout 1200s;
  client_max_body_size 500m;

  location / {
    proxy_pass http://100.64.0.2:8001;
  
  }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/umami.xxx.cn/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/umami.xxx.cn/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = umami.xxx.cn) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;
  server_name umami.xxx.cn;
    return 404; # managed by Certbot


}
```
### 访问
默认用户名：admin 
默认密码：umami
浏览器访问 http://127.0.0.1:8001
通过域名访问 http://umami.xxx.cn 

## 网站设置
通过umami网页，选择设置，添加网站 ，自定义名字 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202502101640133.png)
## 博客接入
由于我这里用的mkdocs，按照官网文档要求，需要在overside下面，新建一个main.html,然后把统计分析代码放在<head> ... </head>中

地址：https://squidfunk.github.io/mkdocs-material/customization/#overriding-partials

```shell
root@DESKTOP-CK75KU2:/mnt/d/mkdocs/docs/overrides# pwd
/mnt/d/mkdocs/docs/overrides
root@DESKTOP-CK75KU2:/mnt/d/mkdocs/docs/overrides# cat main.html   | head 
{#-
  This file was automatically generated - do not edit
-#}
{% import "partials/language.html" as lang with context %}
<!doctype html>
<html lang="{{ lang.t('language') }}" class="no-js">
  <head>  
     <script defer src="https://umami.xxx.cn/script.js" data-website-id="5bc2395b-97cd-470f-a8d4-61ba3e352af7"></script> 	   
    {% block site_meta %}
      <meta charset="utf-8">

```
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202502101645822.png)

## 效果
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202502101648951.png)

## 完结
欢迎沟通使用
