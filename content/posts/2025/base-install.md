---
title: "运维| Docker基础环境搭建"
date: 2025-05-19
lastmod: 2025-05-19
author: 'bdsdc'
linkToMarkdown: true
categories: ['Docker']
tags: ['Docker']
---

## 环境清单

- 系统： Debian12
- 配置： 至少4c4g40G 或者 2c4g
- Docker版本：28.1.1
- Docker支持多平台构建

## host 解析
我们通过配置域名的方式，来访问我们安装的服务 ，比如我这里跑服务IP：192.168.31.236
```shell
# hosts配置
192.168.31.236 harbor.bdser.cc 
```

## 安装docker 

首先我们先配置好apt 源，我们这里切换为阿里云源
```shell
#0. 替换成国内源
root@base:/data/nginx/nginx_data# cat /etc/apt/sources.list
deb https://mirrors.aliyun.com/debian/ bookworm main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian/ bookworm main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian-security/ bookworm-security main
deb-src https://mirrors.aliyun.com/debian-security/ bookworm-security main
deb https://mirrors.aliyun.com/debian/ bookworm-updates main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian/ bookworm-updates main non-free non-free-firmware contrib
deb https://mirrors.aliyun.com/debian/ bookworm-backports main non-free non-free-firmware contrib
deb-src https://mirrors.aliyun.com/debian/ bookworm-backports main non-free non-free-firmware contrib
#1. 进行更新
apt-get update
apt-get install  ca-certificates  gnupg lsb-release bash-completion

#2. 添加Docker官方GPG密钥（使用阿里云镜像）
wget -q0 - http://mirrors.aliyun.com/docker-ce/linux/debian/gpg I sudo apt-key add -

#3. 设置Docker 的APT 仓库（使用阿里云镜像)
echo "deb [trusted=yes] http://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_releasecs) stable" > /etc/apt/sources.list.d/docker-ce.list

#4. 安装 Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## 配置docker
配置docker启动参数配置文件 /etc/docker/daemon.json 

```shell
{
   "registry-mirrors": [
	"https://docker.1panel.dev",
	"https://docker.fxxk.dedyn.io",
	"https://docker.xn--6oq72ry9d5zx.cn",
	"https://docker.llkk.cc",
	"https://a.ussh.net",
	"https://docker.zhai.cm"
    ],
    "log-driver": "json-file",
    "log-opts":   {
    	"max-size": "200m",
    	"max-file": "3"
       },
	"live-restore": true
}
```
## 启动docker
```shell 
systemctl daemon-reload
systemctl start docker
```
## nginx 环境部署
我们这里准备用docker方式部署nginx，然后本机器中，其他用Docker部署的服务，都通过这个nginx代理出来 
由于nginx配置文件需要长期保存，所以我们需要把配置通过volumes方式持久本机保存

```
apt-get install git 
```

###  新建nginx环境目录
```
mkdir -p /data/nginx/nginx_data/conf
mkdir -p /data/nginx/nginx_data/logs
mkdir -P /data/nginx/nginx_data/conf.d    #用于存放各个服务的代理配置
```
### nginx 主配置
```shell
user root;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

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

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log  /var/log/nginx/error.log;
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

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```
### nginx 默认default.conf配置
```shell
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	server_name _;

	location / {
		root /usr/share/nginx/html;
		index index.html index.htm;
	
	}
	location /healthz {
		access_log off;
		return 200 "healthy";
		add_header Content-Type text/plain;
	}


}

```
### 使用Docker compose部署nginx
```shell
services:
  nginx:
    container_name: nginx-proxy 
    image: nginx:1.27-alpine
    restart: always
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - /data/nginx/nginx_data/conf/nginx.conf:/etc/nginx/nginx.conf:ro 
      - /data/nginx/nginx_data/conf.d:/etc/nginx/conf.d:ro 
      - /data/nginx/nginx_data/logs:/var/logs/nginx:rw 

    healthcheck:
      test: wget-quiet-tries=1 --spider http://localhost/healthz || exit 1 
      interval: 30s
      timeout: 5s
      retries: 3

```
### 启动nginx容器
```shell
docker compose up -d 
```



