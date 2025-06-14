---
title: "运维| ubuntu搭建服务"
date: 2024-11-12
lastmod: 2024-11-12
author: 'bdsdc'
linkToMarkdown: true
categories: ['windows']
tags: ['tools']
---

## 前言
家庭信息化系统服务，要体现麻雀虽小五脏俱全，打造只我们自己小家的系统信息化
比如备份照片，存储资料，影视电影，文档分享，财务，理财，以及慢慢积累我们家庭的生活成长，为我们的生活留下岁月痕迹

## 面板推荐
我们需要一个强大的运维面板，能够支持web界面部署和查看 ，方便我们运维和管理
- 开源方案： 1Panel 是新一代的 Linux 服务器运维管理面板
- 部署方式： 在线方式本地部署

产品优势
- 高效管理：用户可以通过 Web 图形界面轻松管理 Linux 服务器，实现主机监控、文件管理、数据库管理、容器管理等功能；
- 快速建站：深度集成开源建站软件 WordPress 和 Halo，域名绑定、SSL 证书配置等操作一键搞定；
- 应用商店：精选上架各类高质量的开源工具和应用软件，协助用户轻松安装并升级；
- 安全可靠：基于容器管理并部署应用，实现最小的漏洞暴露面，同时提供防火墙和日志审计等功能；
- 一键备份：支持一键备份和恢复，用户可以将数据备份到各类云端存储介质，永不丢失。

官方提供脚本方式安装
官方文档地址： https://1panel.cn/docs/installation/online_installation/
我们选择在线方式本地部署

```
根据系统选择不同命令的执行方式
#执行脚本的要用bash执行，默认ubuntu是dash，脚本语法可能不兼容
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && bash quick_start.sh
```
这里要注意： **由于WSL用的是微软的ubuntu内核，脚本匹配会有问题 ，因为平台是x86，所以你可以把脚本变量指定架构x86,平台是amd64**



安装成功后，控制台会打印面板访问信息，可通过浏览器访问 1Panel：
http://目标服务器 IP 地址:目标端口/安全入口

```
如果使用的是云服务器，请至安全组开放目标端口。
ssh 登录 1Panel 服务器后，执行 1pctl user-info 命令可获取安全入口（entrance）
```
安装成功后，可使用 1pctl 命令行工具来维护 1Panel


## 服务方案推荐： 

### 网盘
我们选择cloudreve3.8版本，但是官方讨论4.0会有很大更新，期望4.0版本尽快上线
- 开源方案： cloudreve
- 部署方式： docker

创建目录
```
mkdir /mnt/d/cloudreve/config/ -pv
touch /mnt/d/cloudreve/config/conf.ini
touch /mnt/d/cloudreve/config/cloudreve.db
mkdir /mnt/d/cloudreve/{uploads,avatar}
```

docker 启动
```
docker run -itd \
--name=cloudreve \
--restart=always \
-p 5212:5212 \
--mount type=bind,source=/mnt/d/cloudreve/config/conf.ini,target=/cloudreve/conf.ini \
--mount type=bind,source=/mnt/d/cloudreve/config/cloudreve.db,target=/cloudreve/cloudreve.db \
-v /mnt/d/cloudreve/uploads:/cloudreve/uploads \
-v /mnt/d/cloudreve/avatar:/cloudreve/avatar \
cloudreve/cloudreve:3.8.3 

```
### 照片 
- 开源方案：photoprism 
- 部署方式： 通过1panel 应用商店
由于photoprism依赖数据库，这里我们提前安装mariadb，也是通过应用商店来安装，节省了很大部署时间

需要注意，可以在安装的时候，选择编辑docker-compose 文件，针对-v 存储目录挂载进行修改，修改成已经规划好的存储大目录

### qt下载
- 开源方案： qbittorrent
- 部署方式： 通过1panel 应用商店
这里我们直接通过1panel的应用商店直接安装就可以了，为运维部署提供了方便 

需要注意，可以在安装的时候，选择编辑docker-compose 文件，针对-v 存储目录挂载进行修改，修改成已经规划好的存储大目录

### 影音系统
- 开源方案：jellyfin
- 部署方式：通过1panel 应用商店

需要注意，可以在安装的时候，选择编辑docker-compose 文件，针对-v 存储目录挂载进行修改，修改成已经规划好的存储大目录
