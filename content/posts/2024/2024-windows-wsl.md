---
title: "运维| Windows子系统wsl安装ubuntu"
date: 2024-11-07
lastmod: 2024-11-07
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
1.3 安装WSL 

官方参考地址: <https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package>

新版现在只是安装一个命令，但是官方安装比较慢
```
# windows power shell 管理员运行
wsl --install
```
我们可以去github下载最新版本
https://github.com/microsoft/WSL/releases

查看版本
```
PS C:\Users\bdser> wsl --version
WSL 版本: 2.5.7.0
内核版本: 6.6.87.1-1
WSLg 版本: 1.0.66
MSRDC 版本: 1.2.6074
Direct3D 版本: 1.611.1-81528511
DXCore 版本: 10.0.26100.1-240331-1435.ge-release
Windows: 10.0.26100.4349
```

1.4 更新WSL
```
wsl --update
## 将 wsl 版本设置为 wsl2
wsl --set-default-version 2
```
1.5 问题
如果`wsl --install`或者`wsl --update`遇到禁止403 
```
PS C:\Users\32956> wsl --install
已禁止(403)
```
这是开了代理，需要换成直连或者关掉代理。

## WSL安装ubuntu和使用

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
当然，也可以打开Microsoft Store，安装最新的Ubuntu 24.04的发行版本

### ubuntu更改镜像源
ubuntu 24.0版本更换源的方式有更新，所以之前版本请自行百度参考下

```
# 备份系统默认源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 替换成阿里云源（/etc/apt/sources.list）
deb https://mirrors.aliyun.com/ubuntu/ noble main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ noble main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ noble-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ noble-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ noble-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ noble-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ noble-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ noble-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ noble-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ noble-backports main restricted universe multiverse
```
更新源

```
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
```
### ubuntu安装docker
```
# 解决安装依赖
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common lsb_release 
# 导入密钥公钥
sudo curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg  > 1.gpg
sudo apt-key add 1.gpg
# 进入到/etc/apt/sources.list.d目录
cd /etc/apt/sources.list.d
# 在/etc/apt/sources.list.d目录下的docker.list文件中添加阿里云的镜像下载源
sudo echo "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" > docker.list
# 更新
sudo apt update
```

### 解决密钥警告

警告信息

```
W: https://mirrors.aliyun.com/docker-ce/linux/ubuntu/dists/noble/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```
密钥管理机制变更：Ubuntu 22.04+ 推荐使用 `/etc/apt/trusted.gpg.d/` 目录管理密钥，而非传统的 /etc/apt/trusted.gpg
```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
```


安装最新版的docker 
```
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose
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
加载配置，启动docker
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

### 安装Hyper-v和管理器
这个可以通过图像界面方便的管理虚拟网络等

```
# 方法一：
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All 
# 方法二：
DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
```

### ubuntu开启镜像网络模式
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

使用管理员权限在 PowerShell 窗口中运行以下命令，以配置 Hyper-V 防火墙设置，从而允许入站连接：
```
Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow 
或 
New-NetFirewallHyperVRule -Name "MyWebServer" -DisplayName "My Web Server" -Direction Inbound -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -Protocol TCP -LocalPorts 80
```

## 开机启动
使用`文件资源管理器`添加自定义启动项，如果有脚本的话，可以编写好脚本放入启动文件夹中

- 打开“文件资源管理器”：按下“Win + E”快捷键，打开“文件资源管理器”。
- 导航到启动文件夹：在地址栏中输入“shell:startup”并按下回车键，进入用户启动文件夹。
- 添加快捷方式：将您希望开机启动的程序的快捷方式拖放到此文件夹中。这样，程序将在下次开机时自动启动。
- 注意事项：确保您添加的程序是可信的，以免影响系统安全

### WSL开机启动
方法如下：

- WIN+R 运行 shell:startup 打开启动目录
- 在此目录中创建文件 wsl-startup.vbs
- 在 wsl-startup.vbs 中填充如下内容，需替换为你使用的发行版名称。

查看wsl-startup.vbc脚本内容
```
Set ws = WScript.CreateObject("WScript.Shell")        
ws.run "wsl -d ubuntu",0 
```
后面再开机的时候，就会自动启动Ubuntu系统，放到后台启动 

## 补充
要想要ubuntu系统里面的docker服务，被本地主机能访问到，需要在docker配置中 ，增加配置`"iptables": false`

全部配置如下
```
vi /etc/docker/daemon.json
{
	
	"iptables": false,
	"log-driver": "json-file",
	"log-opts": {
		"max-file": "3",
		"max-size": "50m"
	},
	"registry-mirrors": [
                "https://dockerpull.com",
	        "https://docker.1ms.run",
		"https://docker.1panelproxy.com"
	]
}
```
## 总结
后面我们可以体验下ubuntu给我们带来的方便，轻轻体验把~


