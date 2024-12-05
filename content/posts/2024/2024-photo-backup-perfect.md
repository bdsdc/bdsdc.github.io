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
不得不说下，1panel的应用商店，我觉得选品都非常优秀，都是一些流行且兼具实用的软件，之前有博客写过安装文件，这里不在重复安装步骤，主要提供工具和思路，参考之前来进行安装

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
```
mkdir -pv /mnt/d/mariadb/{data,conf}
```
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052145370.png)

如果上面更改目录，造成部署错误，可以先按照1panel默认建议目录来安装，问题不大，此数据库占用空间不会太大

创建一个db数据库，给photoprism服务使用

这里就很简单了，都是页面操作，按照提示输入相关信息，记住用户名和密码，后面在安装photoprism的时候，填入1panel配置中
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412052255273.png)

### 安装photoprism服务

```
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



### 配置 


### 功能 


### 上传照片

### 索引建立

### webdav

### 删除照片

## syncthing 





## alist 




## rclone 



