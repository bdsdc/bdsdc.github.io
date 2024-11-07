---
title: "运维| Windows子系统wsl安装ubuntu"
date: 2024-10-20
lastmod: 2024-10-20
author: 'bdsdc'
linkToMarkdown: true
categories: ['运维']
tags: ['windows']
---

## 前言

## 环境准备


## 查看windows系统版本
打开powershell，管理员权限执行
```
PS C:\Windows\system32> winver 
```
## WSL版本

打开powershell 管理员权限执行

1.1 启用WSL，适用于Linux的window子系统
```
PS C:\Windows\system32> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

部署映像服务和管理工具
版本: 10.0.19041.3636

映像版本: 10.0.19045.5011

启用一个或多个功能
[==========================100.0%==========================]
操作成功完成。
```
1.2 启用虚拟化平台支持
```
 dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
1.3 更新WSL
```
wsl --update
## 将 wsl 版本设置为 wsl2
wsl --set-default-version 2
```

## WSL安装ubuntu

打开powershell 管理员权限执行
```
# 列出可安装的 Linux 版本
wsl --list --online
# 安装
wsl --install ubuntu-24.04 --web-download
wsl --set-default ubuntu-24.04
# 查看
wsl --list -v
# 卸载旧版
wsl --unregister ubuntu-24.04

```
当然，也可以打开Microsoft Store，安装最新的Ubuntu 22.04的发行版本

### ubuntu更改镜像源

```
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
```

### ubuntu安装docker
```
apt install docker
```
docker镜像地址配置
```
[root@bdser ~]# cat /etc/docker/daemon.json 
{
    "registry-mirrors": [
        "https://docker.1ms.run",
	      "https://docker.fxxk.dedyn.io",
	      "https://dockerpull.com"
    ]
    	
}
```
systemctl daemon-reload
systemctl start docker
```

### ubuntu用ssh访问 
```
重新生成key
sudo ssh-keygen -A
sudo vim /etc/ssh/sshd_config  
修改成PasswordAuthentication yes
启动服务
sudo service ssh start
```
## 安装Hyper-v和管理器
这个可以通过图像界面方便的管理虚拟网络等
```
# 方法一：
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All 
# 方法二：
DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
```

## ubuntu开启镜像网络模式
在运行 Windows 11 22H2 及更高版本的计算机上，可以在 .wslconfig 文件中的 [wsl2] 下设置 networkingMode=mirrored，以启用镜像模式网络。 启用此功能会将 WSL 更改为全新的网络体系结构，其目标是将 Windows 上的网络接口“镜像”到 Linux 中，以添加新的网络功能并提高兼容性


以下是启用此模式的当前优势：
- IPv6 支持
- 使用 localhost 地址 127.0.0.1 从 Linux 内部连接到 Windows 服务器。 不支持 IPv6 localhost 地址 ::1
- 改进了 VPN 的网络兼容性
- 多播支持
- 直接从局域网 (LAN) 连接到 WSL

在家目录下创建.wslconfig配置文件，家目录可以通过下面命令来确定,注意要用cmd里面执行 
```
C:\Users\dongshu.bu>echo %USERPROFILE%
C:\Users\dongshu.bu
```

.wslconfig 配置如下：
```
[wsl2]
memory=4GB 
processors=4 
localhostForwarding=true 

[experimental]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
hostAddressLoopback=true

```

使用管理员权限在 PowerShell 窗口中运行以下命令，以配置 Hyper-V 防火墙设置，从而允许入站连接：Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow 或 New-NetFirewallHyperVRule -Name "MyWebServer" -DisplayName "My Web Server" -Direction Inbound -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -Protocol TCP -LocalPorts 80

## ubuntu安装服务 

## ubuntu安装1panel面板

### 安装photoprism 

### 安装cloudreve 

### 安装openAI UI平替平台

### 安装视频影音平台




