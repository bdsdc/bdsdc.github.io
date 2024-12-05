![image](https://github.com/user-attachments/assets/7624a9d9-2440-4d14-ae3b-86dc1ae20c59)# 个人照片神级相册系统和备份完美解决方案

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

建议选择
所以, 你的已有照片究竟是放import还是originals, 关键点在于你是否希望photoprismt重新命名你的照片.
如果你的照片是已经有规则存放好的, 就把它放在originals,否则把它放import中, 让程序按照日期重新整理你的照片,是个非常不错的选择.

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

### webdav

### 删除照片

## syncthing 





## alist 




## rclone 



