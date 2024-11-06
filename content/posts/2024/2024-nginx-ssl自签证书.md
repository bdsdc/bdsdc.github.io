---
title: "运维|SSL证书自动续签"
date: 2024-11-06
lastmod: 2024-11-06
author: 'bdsdc'
linkToMarkdown: true
categories: ['运维']
tags: ['nginx']
---

## 前期准备
- 系统： centos7
- SSL证书：certbot

## 安装
安装certbot 和 nginx 软件
```
yum install -y certbot  python3-certbot-nginx certbox-nginx nginx

```
## 生成证书
首先我们certbot支持多种模式生成证书， 我们这里用跟nginx搭配的方式生成证书

### nginx 配置 
需要先把nginx代理域名的 80端口配置正确，我们先以xxx.domain.com域名作为例子

```
server {
  listen 80
  server_name xxx.domain.com;
  access_log         /data/nginx/logs/access.log;
  client_header_timeout 1200s;
  client_body_timeout 1200s;
  client_max_body_size 500m;

  location / {
    proxy_set_header Host $server_name;	  
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
    proxy_pass http://127.0.0.1:40033;
  
  }
}

```


### certbot生成证书
```
certbot --nginx -d xxx.domain.com  #会自动识别nginx配置位置，帮你配置好nginx ssl
certbot certonly --nginx -d xxx.domain.com  #只会生成ssl证书，后面自己配置到nginx中
```
我们这里用第一种方式，自动帮忙配置，生成的配置看下
```
server {
  listen 443 ssl;
  server_name xxx.domain.com;
  access_log         /data/nginx/logs/domain-access.log;
  ssl_certificate /etc/letsencrypt/live/xxx.domain.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/xxx.domain.com/privkey.pem; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  client_header_timeout 1200s;
  client_body_timeout 1200s;
  client_max_body_size 500m;

  location / {
    proxy_set_header Host $server_name;	  
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
    proxy_pass http://127.0.0.1:40033;
  
  }

}
server {
    if ($host = xxx.domain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;
  server_name xxx.domain.com;
  return 404; # managed by Certbot

}

```
测试下nginx配置 
```
[root@bdser ~]# nginx -c /etc/nginx/nginx.conf -t 
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```

## 证书续签测试
```
#测试证书续签是否正常，如果正常执行，后面可以把正式续签命令加入定时任务
certbot renew --dry-run   
```
## 定时任务执行
上面测试如果没有问题，通过定时任务，每个月定时续签证书，可能在生成的时候，nginx会卡一下 
```
#每个月的第一天进行续签，--quiet：代表静默模式，不输出终端信息
0 0 1 * * /usr/bin/certbot renew --quiet 
```

