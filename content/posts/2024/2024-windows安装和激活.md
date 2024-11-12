---
title: "运维| Windows安装和激活"
date: 2024-11-12
lastmod: 2024-11-12
author: 'bdsdc'
linkToMarkdown: true
categories: ['windows']
tags: ['tools']
---
windows
现在windows11用起来也算是比较稳定，并且windows11新增了一些关于WSL的新功能，值得尝试和体验 

## 前言

## 准备
- 准备16g的U盘
- 一台电脑
- 系统镜像
- U盘装机工具
- 激活工具


## 系统镜像

> 最近这个作者的网站更新了，需要登录，LTSC是长期稳定版本，也可以体验最新版本

纯净版系统下载： https://next.itellyou.cn

`补充图片`

## 制作ventory启动盘
这里我们选择开源运维工具Ventory，一个超级棒的U盘启动工具，有了Ventoy你就无需反复地格式化U盘，你只需要把 ISO/WIM/IMG/VHD(x)/EFI 等类型的文件直接拷贝到U盘里面就可以启动了，无需其他操作。
你可以一次性拷贝很多个不同类型的镜像文件，Ventoy 会在启动时显示一个菜单来供你进行选择
总结如下：
- 不需要每次对U盘初始化
- 直接拷贝iso文件到u盘中
- 可以支持集中多种操作系统安装 

`补充图片`


### ventory下载
下载地址：(https://www.ventoy.net/cn/download.html)[https://www.ventoy.net/cn/download.html] 

1.1 首先看是否有特殊需求，则可到配置选项中，进行一些符合自己需求的调整。如设置安全启动、分区类型、修改分区设置等 
如果没有特殊配置需求，我们选择默认安装就可以

1.2 在先插好u盘，选中u盘直接双击安装即可 

### 制作u盘步骤

- 准备一个U盘，里面的东西先拷走。
- 插入电脑，给它格式化并安装Ventoy。
- 把下载好的系统ISO安装盘（Windows、Linux均可）、PE的ISO文件直接拷贝到u盘中
- 开机启动时ventoy会列出可启动的ISO列表
- 选择ISO进行安装，或选择PE进行PE操作，或者选择其它ISO进行更多其它操作

### 开启启动
这个基本上不用说了，每个电脑根据型号不太一样，大家百度查一下，都懂得

开机时按下F12、F8或者ESC进入Boot Menu启动菜单，用方向键选中U盘，回车键

然后根据提示，一步一步安装就可以了

## windows激活 
windows最棒的激活软件
激活工具地址： https://cmwtat.cloudmoe.com/cn.html

