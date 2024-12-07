---
title: "个人照片相册系统和备份完美解决方案"
date: 2024-12-06
lastmod: 2024-12-06
author: 'bdsdc'
linkToMarkdown: true
categories: ['homesystem']
tags: ['tools']
---


# 个人照片神级相册系统和备份完美解决方案

## 前言
随着时代进步和发展，照片越来越清晰，越来越大，我们也都更爱用照片或者视频记录美好生活
存储空间问题开始显露出来，手机存储空间报警，网盘空间备份不方便，并且也不能智能展示照片
比如根据头像分类，识别照片信息像素地址，展示相册位置地图，瞬间，Live图，等之类的
我们需要一个智能相册系统，能够清晰无损分类展示照片，可以很方便的同步最新照片到相册
避免手机突然故障，还希望能方便的备份相册照片到网盘，如果考虑更全面一点，增加异地备份
或者通过备份多个网盘来解决，经过最近的研究和探索，找到了一个完美解决方案，请往下看

## 介绍
这里我们会把前面所有的东西结合起来，打造一个相册系统+备份的解决方案，这里没有用NAS，是因为我觉得NAS跟ubuntu系统没什么区别
并且WSL是windows打造的linux系统工具，跟windows结合非常好，用起来也很方便丝滑 
- 基础环境：windows的WSL安装的ubuntu系统
- 面板系统：1panel
- 相册系统： photoprism 
- 同步工具：syncthing 
- 网盘系统：alist 
- 备份工具：rclone 
- 穿透工具：cloudflare tunnel
## ubuntu系统安装 
参考之前文章来进行安装
[https://wiki.budongshu.cn/homesystem/windows-wsl/#_1](https://wiki.budongshu.cn/homesystem/windows-wsl/#_1)

## 1panel 
### 安装1panel 
不得不说下，1panel的应用商店，我觉得选品都非常优秀，都是一些流行且兼具实用的软件，之前有博客写过安装步骤，这里不在重复，主要提供工具和思路，参考之前来进行安装

[https://wiki.budongshu.cn/homesystem/windows-ubuntu%E9%83%A8%E7%BD%B2%E6%9C%8D%E5%8A%A1/#_2](https://wiki.budongshu.cn/homesystem/windows-ubuntu%E9%83%A8%E7%BD%B2%E6%9C%8D%E5%8A%A1/#_2)

### 界面展示
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052111988.png)

## photoprism 
- 操作系统不限, Windows, MacOS以及Linux都支持
- 系统配置要求最低：2c3g
- 准备好docker环境和docker-compose环境
- 准备好大存储目录
- 使用Windows: 使用WSL Linux子系统ubuntu环境
- 注意photoprism数据库使用的是MariaDB 

### 安装数据库

首先我的windows电脑D盘是空闲盘，作为本次photoprism存储照片来使用，需要注意的是在ubuntu子系统（windows的WSL）中，通过`df -h`,可以看到`/mnt/d/`为对应windows的D盘，
所以我这里使用`/mnt/d`作为本次相册系统的存储目录

我们这里采用1panel面板来安装，有现成的轮子，我们当然要利用起来 

安装MariaDB数据库，我们通过1panel面板的应用商店来安装

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052133126.png)

安装配置，可以设置下资源配置
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052144597.png)


可以更改一下docker-compose，更改volume，比如我这里要改成`- /mnt/d/mariadb/data` 
```shell
mkdir -pv /mnt/d/mariadb/{data,conf}
```
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052145370.png)

如果上面更改目录，造成部署错误，可以先按照1panel默认建议目录来安装，问题不大，此数据库占用空间不会太大

创建一个db数据库，给photoprism服务使用

这里就很简单了，都是页面操作，按照提示输入相关信息，记住用户名和密码，后面在安装photoprism的时候，填入1panel配置中
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052255273.png)

### 安装photoprism服务

```shell
# 先创建主目录
mkdir /mnt/d/photoprism 
# 在这个目录下创建三个子目录,分别是import,originals,storage
mkdir /mnt/d/photoprism/{import,originals,storage} 
```

关于三个目录的作用
- originals: 原始照片目录, 你可以把你存在的照片放在这个目录
- import: 待导入照片, 这个目录的照片会等待导入并被photoprism处理 (处理是指对照片做分析与索引,存入原始目录等)
- storage: 这个目录是photoprism程序产生的各种文件,比如索引照片缩略图,缓存等

本次我们安装依然采用1panel来安装，下面我通过1panel如何来安装设置
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052244915.png)
选择我们刚刚创建的数据库，然后配置使用的db库，以及用户名和密码
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052245922.png)

选择资源配置cpu和内存，以及编辑docker-compose文件，更改volume配置 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052248495.png)

最后更改volume配置后，可以先关注红色框的三个配置，其他的俩个volume可以先忽略，后面会讲
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052253065.png)
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052254950.png)

### 启动
我们通过1panel容器查看，是否正常启动，如果启动失败，可以根据提示或者进入日志控制台查看具体报错

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052305286.png)

```shell
# 也可以手动处理报错，通过下面命令找到photoprism容器
docker ps -a
# 比如容器名字就叫做photoprism,查看日志报错来解决
docker logs -f photoprism
```

通过查看端口，通过本机IP地址，比如我的端口是12343（端口可以在上面配置自定义）访问地址是 http://localhost:12343 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052312122.png)
使用刚刚1panel配置中的管理员用户名密码登录就可以了. 登录进去会发现什么都没有，这是因为我们没有还没有导入照片。


### 导入照片
现在photoprism 就已经正常跑起来，下面就是如何把照片导入到这个系统中，通过学习了解，我们有多种方式来导入照片

但你现在会发现，你还没有任何照片视频，你现在的需求其实是两个:
- 导入旧有的存在的相片
- 后续如何同步新的相片



那么如何导入照片呢？
这里先讲下如何导入旧的存在的相片并完成索引，所谓索引是指对照片的信息元数据,比如地点,大小,拍摄时间等做提取, 同时对照片生成各种大小的缩略图照片. (你在UI上看到的小图,其实是缩略图,而不是原照片)

前面的配置文件中,提及了两个目录, 一个是`originals`,另一个是`import`, 旧有的相片可以放在`import`目录下, 也可以放在`originals`目录下.
这里我建议可以选择放在`originals`目录下 
`originals` 是photoprism存放相片原文件的地方,所有photoprism的相片最终的原文件,都在这个目录下。（这个就会用到后面说的同步工具软件syncthing）
所以你可以把已有的旧相片直接放到这个目录下，但是由于photoprism没有对你这些已存在的相片做索引， 所以你需要在WEB UI上触发一个全量索引操作。

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052343847.png)
进入资料库, 然后点击索引页

`import` 你也可以把存在的照片放到import目录下，然后在资料库的导入页中，选择你的目录，导入你的照片。
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412060015058.png)

俩个目录区别与选择
共同点：
- 都会对照片做索引

不同点：
- import目录下的照片，会按照日期重新进入命名，并将其存入originals目录下。
- import相当于一个待处理的临时目录, 而originals则是你原始照片存放的真正目录

### 导入照片建议选择
所以你的已有照片究竟是放import还是originals，关键点在于你是否希望photoprismt重新命名你的照片。
如果你的照片是已经有规则存放好的，就把它放在originals，否则把它放import中，让程序按照日期重新整理你的照片，是个非常不错的选择。

*需要注意的是，索引是非常耗时的操作, 我当时1500千张照片,索引花费了半小时.*

通过import目录导入后的目录结构，系统会根据时间进行目录规划切分
```shell
2021
├── 04
│   ├── 20210430_094239_4DC634BC.jpg
│   ├── 20210430_114039_E05E78F3.jpg
│   ├── 20210430_120718_3C8E92F2.jpg
│   └── 20210430_120734_2B014E86.jpg
├── 05
│   ├── 20210501_095824_0F2A7697.jpg
│   ├── 20210501_161637_02B094F6.jpg
│   ├── 20210501_161800_377F14E1.jpg
│   ├── 20210501_174414_9AE325CC.jpg
│   ├── 20210501_174511_79845FE1.jpg
│   ├── 20210501_192649_A99A0A1A.jpg
│   ├── 20210501_193137_E632007B.jpg
│   ├── 20210502_112902_6244D36B.jpg
│   ├── 20210502_113933_57D40310.jpg
│   ├── 20210504_114226_E40B20EB.jpg
│   └── 20210504_203742_43D611F7.jpg
├── 08
│   └── 20210829_213906_7E352431.jpg
└── 11
    └── 20211127_181818_D4A21A3A.jpg

5 directories, 17 files
上面这个就是`2021年的相片`, 每个相片都以时间 + 随机码来命名。
```
如果相片非常杂乱，迁移相片时， 经过photoprismt这么一处理，就会变成按日期存储的非常有规律的文件。

还有一个通过web页面控制台，导入，如果有临时导入需求，也可以通过这个导入功能来进行导入 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052324028.png)

### 中文
支持设置语言，设置成中文语言
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070903181.png)
### webdav
photoprism是支持webdav的，我们可以通过webdav客户端来访问我们已经导入的图片，此功能还是比较实用的
这样其实我们也可以通过webdav工具，导入照片，这个导入的照片是放在前面说的 `originals`目录的
导入后，我们需要手动执行下扫描，为了建立索引
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070904087.png)


### 删除照片
首先删除的功能默认是关闭，需要我们手动打开，如下
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070903181.png)

删除照片是通过归档的形式来删除，先选中要删除的照片，然后点击归档，在归档里面选择删除的照片 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070905781.png)

在归档里面，才可以删除
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070909345.png)
如果是删除相册里面的照片，需要先移出相册


### 使用体会
整体感觉眼前一亮，可以通过建立相册，分类整理我们的照片，但是操作功能和使用方便一般，
UI方面还算挺不错的，该有的功能也都有，不过感觉有些操作功能过于繁琐，就比如删除照片，
人脸识别和地图功能体验一般，对手机网页端适配比较友好

## syncthing 
这里我们介绍一个神级同步工具，synthing 
官网: [https://syncthing.net/](https://syncthing.net/)

### 前言
我们这里使用场景比较简单，该软件支持多对多或者中心媒介，多种方式来同步
我们今天只考虑一种单向同步照片 

我们熟悉了photoprism的俩个导入目录，其中我们是可以把新照片放在`originals`目录，然后在建立一个索引
这个事情就搞定了，索引ok后，在照片里面就能看到最新的同步过去的照片

但是对于新照片，我们怎么能更好的同步呢，最好是可以自动方便的同步，或者任意地点同步
所以接下来我们用syncthing把手机照片同步到photoprism的`originals`目录 

### 介绍

💻 Syncthing是什么？
Syncthing是一款去中心化的数据同步软件，适用于不同平台之间。它不需要通过云盘等中间服务器来传输数据，而是通过在线设备之间的点对点同步来实现。

📂 Syncthing的用途
我主要用Syncthing来处理以下几项任务：
- 1️⃣ 在Windows和安卓手机上同步Obsidian本地笔记；
- 2️⃣ 在Windows和安卓手机上同步本地歌曲；
- 3️⃣ 将Windows上的文件无感发送到手机，或从手机发送到Windows，确保两端文件一致；
- 4️⃣ 将手机端的通话录音自动备份到Windows，手动删除手机端文件不影响电脑端备份；
- 5️⃣ 用手机拍照、录视频后，文件自动同步到Windows，方便进行创作；
- 6️⃣ 在Windows上浏览摄影作品时，保存到电脑本地，并自动推送给手机；
- 7️⃣ 将Windows上的相册、视频回忆录自动同步到安卓电视盒子，方便在大屏上观看

### ubuntu 安装syncthing
这里我们通过docker-compose部署安装
```shell
# 创建目录
mkdir /mnt/d/syncthing
```

```shell
# cat /mnt/d/syncthing/docker-compose.yml

---
version: "3"
services:
  syncthing:
    image: syncthing/syncthing
    container_name: syncthing
    hostname: my-syncthing
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /mnt/d/syncthing:/var/syncthing  #注意这里的目录
    ports:
      - 8384:8384 # Web UI
      - 22000:22000/tcp # TCP file transfers
      - 22000:22000/udp # QUIC file transfers
      - 21027:21027/udp # Receive local discovery broadcasts
    network_mode: host 
    restart: unless-stopped
    healthcheck:
      test: curl -fkLsS -m 2 127.0.0.1:8384/rest/noauth/health | grep -o --color=never OK || exit 1
      interval: 1m
      timeout: 10s
      retries: 3
```
### 启动syncthing 

```shell
cd /mnt/d/synthing 
docker-compose up -d

docker ps  |grep syncthing  
```
也可以访问1panel面板来查看，即使非面板应用商店的应有，面板也可以管理，面板是服务于机器的 
只要是机器上的进程，容器，服务等，都也可以在面板被管理

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070914397.png)


### 手机安装syncthing 
手机安装包: 
donwload: [https://syncthing.en.softonic.com/android](https://syncthing.en.softonic.com/android)


### 配置同步文件夹
先添加远程设备，在手机上，我们添加ubuntu作为远端设备，这里添加设备是通过唯一ID或者二维码来添加的

首先看查看ubuntu上syncthing服务的 ID 或者二维码
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071002403.png)

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071002532.png)

然后打开手机，安装好syncthing app，然后我们按照如下图操作，先点击添加设备，通过扫描二维码方式来添加（比较方便）
当然也可以通过ID来添加
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071005442.png)
点击二维码图标，可以扫描
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071005862.png)
添加同步文件夹，选择相机图片存储目录，下面列一下重点项目，其他可以默认
- 填写一个名字，最好通俗易懂
- 选择小米手机照片存储目录
- 目录种类： 可以选择仅发送，代表单方向从手机到ubuntu 发送拷贝照片
- 监视更改：默认打开软件的时候，就会扫描最新图片
- 版本控制： 可以选择垃圾桶或者简单方式 ，这个需要提前看一下官网文档，防止那天误删照片或者同步出现问题
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071007861.png)

## alist 
一个支持多种存储，支持网页浏览和 WebDAV 的文件列表程序，由 gin 和 Solidjs 驱动
### 安装alist 
虽然1panel应用商店提供该软件，我们这次手动用docker来安装，因为也比较简单方便
```shell
# 这里我们还是要注意一下-v volume目录，要修改成自己的
docker run -d --restart=unless-stopped -v /mnt/d/alist/data:/opt/alist/data -p 5244:5244 -e PUID=0 -e PGID=0 -e UMASK=022 --name="alist" xhofe/alist:latest
```
检查一下是否启动成功,看一下容器状态，也可以通过1panel面板来查看
```shell
docker ps  |grep alist  
```

### 访问alist
其实默认第一次打开是没有存储后端的，会有报错，我们可以通过点击页面右下方`管理`进如管理控制台页面
```shell
# 控制台页面密码获取
root@DESKTOP-CK75KU2:/mnt/d/alist# docker logs -f alist 
INFO[2024-12-04 13:38:10] reading config file: data/config.json        
INFO[2024-12-04 13:38:10] config file not exists, creating default config file 
INFO[2024-12-04 13:38:10] load config from env with prefix:            
INFO[2024-12-04 13:38:10] init logrus...                               
INFO[2024-12-04 13:38:11] Successfully created the admin user and the initial password is: xxxxxxx  #密码在这里
INFO[2024-12-04 13:38:11] init tool pikpak success: ok                 
WARN[2024-12-04 13:38:11] init tool qBittorrent failed: Post "http://localhost:8080/api/v2/auth/login": dial tcp 127.0.0.1:8080: connect: connection refused 
WARN[2024-12-04 13:38:11] init tool transmission failed: failed get transmission version: can't get session values: 'session-get' rpc method failed: failed to execute HTTP request: Post "http://localhost:9091/transmission/rpc": dial tcp 127.0.0.1:9091: connect: connection refused 
INFO[2024-12-04 13:38:11] init tool 115 Cloud success: ok              
WARN[2024-12-04 13:38:11] init tool aria2 failed: failed get aria2 version: Post "http://localhost:6800/jsonrpc": dial tcp 127.0.0.1:6800: connect: connection refused 
INFO[2024-12-04 13:38:11] init tool SimpleHttp success: ok             
INFO[2024-12-04 13:38:11] start HTTP server @ 0.0.0.0:5244 
```

访问网页地址： http://localhost:5244 或者http://IP:port
用户名: admin
密码： 如上方法获取
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070928713.png)

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070929839.png)

### 密码重置
```shell
# 随机生成一个密码
docker exec -it alist ./alist admin random
# 手动设置一个密码,`NEW_PASSWORD`是指你需要设置的密码
docker exec -it alist ./alist admin set NEW_PASSWORD
```

### 存储阿里云盘
我这里用阿里云盘作为后端存储，其他网盘可以自行学习参考下
官网地址: [https://alist-doc.nn.ci/docs/intro](https://alist-doc.nn.ci/docs/intro)
#### 获取fold ID
我们在云盘里面创建一个目录，把这个目录作为跟Alist的挂载设备
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071144423.png)

#### Alist挂载路径
新建存储，选择阿里云盘OPEN ，其他阿里都已经不能用了。这个路径后面在创建目录的时候很重要，后面会有演示
其实我的理解，不一定对，相当于Alist把阿里云盘我们刚刚创建的目录`Homevideo`挂载到目录路径`aliyunopen`上 

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071145164.png)

我的配置如下：
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412070935280.png)

### alist阿里云盘webdav配置
这个依然可以参考aliast官网
[https://alist.nn.ci/zh/guide/webdav.html](https://alist.nn.ci/zh/guide/webdav.html)
- Url： http[s]://domain:port/dav   #我们这里把domain替换成IP地址
- Host： domain 
- 路径： dav
- 协议： http[s]
- 端口： 与网页一致
- WebDav用户名： 与网页端用户名一致
- WebDav密码： 与网页端密码一致
### 可以用来挂载WebDav的软件
1. windows
- Potplayer，kmplayer，RaiDrive，kodi，OneCommander，Mountain Duck，rclone，AIMP
2. MAC 
- VidHub，IINA，Mountain Duck，infuse，netdrive，rclone
3. Android
- Nplayer，kmplayer，ES文件管理器，kodi，nova魔改，reex，cx 文件管理器，Solid Explorer，X-plore File Manager，MiXplorer
4. IOS 
- VidHub，Nplayer，kmplayer，infuse，zFuse, Fileball文件管理器
5. linux 
- davfs , rclone 
6. 电视TV
- VidHub，Nplayer，kodi，nova魔改

## rclone 
本文让我们来看一看如何使用强大的 rclone 命令行工具配置挂载 WebDAV 云盘


### rclone 对接Alist网盘WebDav协议
我们通过rclone config配置，看起来复杂，其实很简单，只要根据提示，进行选择就好了
```shell
root@DESKTOP-CK75KU2:/mnt/d/rclone# rclone config
Current remotes:

Name                 Type
====                 ====
aliyunpan            webdav

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n

Enter name for new remote.
name> aliyunpan01

Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ (fichier)
 2 / Akamai NetStorage
   \ (netstorage)
 3 / Alias for an existing remote
   \ (alias)
 4 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, ArvanCloud, Ceph, ChinaMobile, Cloudflare, DigitalOcean, Dreamhost, GCS, HuaweiOBS, IBMCOS, IDrive, IONOS, LyveCloud, Leviia, Liara, Linode, Magalu, Minio, Netease, Petabox, RackCorp, Rclone, Scaleway, SeaweedFS, StackPath, Storj, Synology, TencentCOS, Wasabi, Qiniu and others
   \ (s3)
......
   \ (storj)
50 / Sugarsync
   \ (sugarsync)
51 / Transparently chunk/split large files
   \ (chunker)
52 / Uloz.to
   \ (ulozto)
53 / Union merges the contents of several upstream fs
   \ (union)
54 / Uptobox
   \ (uptobox)
55 / WebDAV
   \ (webdav)
56 / Yandex Disk
   \ (yandex)
57 / Zoho
   \ (zoho)
58 / premiumize.me
   \ (premiumizeme)
59 / seafile
   \ (seafile)
Storage> 55

Option url.
URL of http host to connect to.
E.g. https://example.com.
Enter a value.
url> http://192.168.31.239:5224/dav

Option vendor.
Name of the WebDAV site/service/software you are using.
Choose a number from below, or type in your own value.
Press Enter to leave empty.
 1 / Fastmail Files
   \ (fastmail)
 2 / Nextcloud
   \ (nextcloud)
 3 / Owncloud
   \ (owncloud)
 4 / Sharepoint Online, authenticated by Microsoft account
   \ (sharepoint)
 5 / Sharepoint with NTLM authentication, usually self-hosted or on-premises
   \ (sharepoint-ntlm)
 6 / rclone WebDAV server to serve a remote over HTTP via the WebDAV protocol
   \ (rclone)
 7 / Other site/service or software
   \ (other)
vendor> 6

Option user.
User name.
In case NTLM authentication is used, the username should be in the format 'Domain\User'.
Enter a value. Press Enter to leave empty.
user> admin

Option pass.
Password.
Choose an alternative below. Press Enter for the default (n).
y) Yes, type in my own password
g) Generate random password
n) No, leave this optional password blank (default)
y/g/n> y
Enter the password:
password:
Confirm the password:
password:

Option bearer_token.
Bearer token instead of user/pass (e.g. a Macaroon).
Enter a value. Press Enter to leave empty.
bearer_token>

Edit advanced config?
y) Yes
n) No (default)
y/n>

Configuration complete.
Options:
- type: webdav
- url: http://192.168.31.239:5224/dav
- vendor: rclone
- user: admin
- pass: *** ENCRYPTED ***
Keep this "aliyunpan01" remote?
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y

Current remotes:

Name                 Type
====                 ====
aliyunpan            webdav
aliyunpan01          webdav

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q>q

```
### rclone相关命令
```
# 查看配置信息
root@DESKTOP-CK75KU2:~# rclone config show 
[aliyunpan]
type = webdav
url = http://192.168.31.239:5244/dav
vendor = rclone
user = admin
pass = Tt2zB4jg2EIXHc8JYf3nXHdIvwua0HUC
# 查看配置文件路径
root@DESKTOP-CK75KU2:~# rclone config paths
Config file: /root/.config/rclone/rclone.conf
Cache dir:   /root/.cache/rclone
Temp dir:    /tmp
# 列出远程端设备
root@DESKTOP-CK75KU2:~# rclone listremotes 
aliyunpan:
# 列出远程端设备比如网盘里面的文件列表
root@DESKTOP-CK75KU2:~# rclone ls aliyunpan:  
      375 aliyunopen/mkdocs_start.sh
      335 aliyunopen/myopentoken.txt
       32 aliyunopen/mytoken.txt
 22492286 aliyunopen/rclone-v1.68.2-linux-amd64.zip
       40 aliyunopen/temp_transfer_folder_id.txt

```

### rclone 挂载
类似mount命令，把Alist存储的阿里云盘，通过WebDav挂载到机器上
```shell
mkdir /mnt/c/webdav 
rclone mount aliyunpan: /mnt/c/webdav  --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 022 --use-mmap --daemon
```
通过挂载命令，挂载到ubuntu机器上，基本上就可以作为linux系统存储目录了（类似mount）
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071039989.png)

### rclone 创建目录
其实看到这里，你应该能够想到，我们如果备份照片，就可以通过rclone把数据或者照片直接拷贝同步到阿里云盘了，如果阿里云盘备份还不够，我们通过alist在接入多个网盘，达到多个网盘
异地多备份的效果，充分保证数据或者照片的安全

**注意：目录层次这里很重要，Alist接入阿里云盘的时候，挂载路径我写的是`aliyunopen`,跟阿里云盘里面目录`Homevideo`是相对应的，所以我们创建目录要在`aliyunopen`目录下创建**
上面说的注意点，不知道大家能理解不，不理解可以先按照这么做

```
root@DESKTOP-CK75KU2:~# rclone mkdir aliyunpan:aliyunopen/wedding-photos
root@DESKTOP-CK75KU2:/mnt/c/webdav/aliyunopen# rclone lsd aliyunpan: --max-depth 2
          -1 2024-12-04 23:06:22        -1 aliyunopen
          -1 2024-12-07 11:32:53        -1 aliyunopen/wedding-photos
```
我们通过查看挂载目录也可以看到
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071133626.png)

### rclone 同步数据(或照片)

子命令 rsync 
> Sync the source to the destination, changing the destination only. Doesn't transfer files that are identical on source and destination, testing by size and modification time or MD5SUM. Destination is updated to match source, including deleting files if necessary (except duplicate objects, see below). If you don't want to delete files from destination, use the copy command instead.

阿里云盘： aliyunpan:aliyunopen/wedding-photos
本地目录： /mnt/d/images/ 

```shell
root@DESKTOP-CK75KU2:/mnt/d/images# rclone sync 定格时光相册×2/ aliyunpan:aliyunopen/wedding-photos/
root@DESKTOP-CK75KU2:/mnt/d/images/定格时光相册×2# rclone ls aliyunpan:aliyunopen/wedding-photos/  | wc -l
35
root@DESKTOP-CK75KU2:/mnt/d/images/定格时光相册×2# ls
1L9A3615.jpg  1L9A3759.jpg  1L9A3899.jpg  1L9A3965.jpg  倾城相框1整张1L9A3825.jpg     定格时光摆台1L9A4036.jpg
1L9A3629.jpg  1L9A3765.jpg  1L9A3925.jpg  1L9A4057.jpg  定格时光中框中上1L9A3678.jpg  定格时光摆台1L9A4048.jpg
1L9A3648.jpg  1L9A3791.jpg  1L9A3935.jpg  1L9A4070.jpg  定格时光中框中下1L9A3927.jpg  定格时光边框1L9A3668.jpg
1L9A3715.jpg  1L9A3822.jpg  1L9A3937.jpg  1L9A4092.jpg  定格时光中框中下1L9A4065.jpg  定格时光边框1L9A3833.jpg
1L9A3743.jpg  1L9A3830.jpg  1L9A3944.jpg  1L9A4095.jpg  定格时光摆台1L9A3832.jpg      海报1L9A3992.jpg
1L9A3751.jpg  1L9A3840.jpg  1L9A3948.jpg  1L9A4098.jpg  定格时光摆台1L9A3903.jpg
root@DESKTOP-CK75KU2:/mnt/d/images/定格时光相册×2# rclone lsl aliyunpan:aliyunopen/wedding-photos/
 14442675 2024-12-07 11:57:53.000000000 1L9A3615.jpg
 14179640 2024-12-07 11:57:58.000000000 1L9A3629.jpg
 14143112 2024-12-07 11:58:06.000000000 1L9A3648.jpg
```
我们看一下阿里云盘，已经有照片了。很棒，同步备份成功，经过这么一番折腾，我们就可以把照片备份到阿里云盘了 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071200951.png)

还有一个小Tips，需要说明下，其实/mnt/d/images 目录就是windows的D盘
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412071207549.png)
这就是用windows WSL的ubuntu的好处，俩个系统相当于是互通的






















