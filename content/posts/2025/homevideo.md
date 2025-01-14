---
title: "home| emby+auto_symlink+sd2+alist+nginx直链播放"
date: 2024-12-22
lastmod: 2024-12-22
author: 'bdsdc'
linkToMarkdown: true
categories: ['运维']
tags: ['AI']
---
## Alist 


```shell
mkdir -p /mnt/d/alist && cd /mnt/d/alist && vim docker-compose.yml
# docker-compose.yml
version: '3.3'
services:
    alist:
        image: 'xhofe/alist:latest'
        container_name: alist
        volumes:
            - './alist_data:/opt/alist/data'
        ports:
            - '5244:5244'
        environment:
            - PUID=0
            - PGID=0
            - UMASK=022
        restart: unless-stopped
```
## clouddrive2 
```
version: "2.3"
services:
  cloudnas:
    image: cloudnas/clouddrive2
    container_name: clouddrive2
    environment:
      - TZ=Asia/Shanghai
      - CLOUDDRIVE_HOME=/Config
    volumes:
      - /mnt/d/clouddrive2/mp_downloads:/CloudNAS:shared
      - /mnt/d/clouddrive2/Config:/Config
      - /mnt/d/clouddrive2/media:/media:shared #optional media path of host
    devices:
      - /dev/fuse:/dev/fuse
    restart: unless-stopped
    pid: "host"
    privileged: true
    network_mode: "host"

```

## auto_symlink

```shell
services:
  auto_symlink:
    image: shenxianmq/auto_symlink:latest
    container_name: auto_symlink
    environment:
      - TZ=Asia/Shanghai
    user: "0:0"
    ports:
      - 8095:8095
    volumes:
      - /mnt/d/clouddrive2/media:/mnt/d/clouddrive2/media:rslave     # cd2 media挂载目录
      - /mnt/d/clouddrive2/Moves/strm:/mnt/d/clouddrive2/Moves/strm  # emby扫描目录
      - /mnt/d/app/auto_symlink/config:/app/config
      - /mnt/d/app/auto_symlink/backup:/app/backup
    restart: unless-stopped

```
## embyserver 
```
version: '3'

services:
  emby-server:
    image: amilys/embyserver
    container_name: embyserver
    network_mode: bridge # DLNA and Wake-on-Lan需要bridge
    environment:
      - UID=0 # 设置容器用户 ID 为 0 (通常是 root)
      - GID=0 # 设置容器组 ID 为 0 (通常是 root)
      - GIDLIST=0 # 设置容器组 ID 列表为 0
      - TZ=Asia/Shanghai # 设置容器的时区为亚洲/上海
    devices:
      - /dev/dri:/dev/dri             # 将主机的 /dev/dri 设备挂载到容器 开启硬解
    ports:
      - 8098:8096 # 对外访问端口
    restart: unless-stopped
    privileged: true
    volumes:
      - /mnt/d/emby/emby-server/config:/config 
      - /mnt/d/clouddrive2/Moves/strm:/strm                      #
      - /mnt/d/clouddrive2/media:/mnt/d/clouddrive2/media:rslave #
```
## emby-nginx 
```
version: '3.5'
services:
    service.nginx-emby:
      image: nginx:1.27.1
      container_name: nginx-emby
      ports:
         - 8091:8091
         - 8092:8095
      volumes:
        - ../nginx/nginx.conf:/etc/nginx/nginx.conf
        - ../nginx/conf.d:/etc/nginx/conf.d
        - ../nginx/embyCache:/var/cache/nginx/emby
        - ../nginx/log:/var/log/nginx
      restart: always
```

### 更改constant.js 
需要更改下面三项
```
const embyHost = "http://192.168.xxx.xxx:8098"; 
const embyApiKey = "18c503639f62492exxxxxxxxx";
const mediaMountPath = [""];
```
原文件如下
```javascript
// 如果使用拆分配置,请注意填写 config 下使用到的配置文件
  
import commonConfig from "./config/constant-common.js";
import mountConfig from "./config/constant-mount.js";
import proConfig from "./config/constant-pro.js";
import symlinkConfig from "./config/constant-symlink.js";
import strmConfig from "./config/constant-strm.js";
import transcodeConfig from "./config/constant-transcode.js";
import extConfig from "./config/constant-ext.js";
import nginxConfig from "./config/constant-nginx.js";
  
// 必填项,根据实际情况修改下面的设置
  
// 这里默认 emby/jellyfin 的地址是宿主机,要注意 iptables 给容器放行端口
const embyHost = "http://192.168.31.239:8098";
  
// emby/jellyfin api key, 在 emby/jellyfin 后台设置
const embyApiKey = "18c503639f62492ea4144e58ebdc48c3";
  
// 挂载工具 rclone/CD2 多出来的挂载目录, 例如将 od,gd 挂载到 /mnt 目录下: /mnt/onedrive /mnt/gd ,那么这里就填写 /mnt
// 通常配置一个远程挂载根路径就够了,默认非此路径开头文件将转给原始 emby 处理
// 如果没有挂载,全部使用 strm 文件,此项填[""],必须要是数组
const mediaMountPath = [""];    
```
### 更改constant-mount.js
需要更改如下几项 

```javascript
// 选填项,用不到保持默认即可

// rclone/CD2 挂载的 alist 文件配置,根据实际情况修改下面的设置
// 访问宿主机上 5244 端口的 alist 地址, 要注意 iptables 给容器放行端口
const alistAddr = "http://192.168.31.239:5244";

// alist token, 在 alist 后台查看
const alistToken = "alist-4e440c66-3eac-4e77-9e08-906724122ea3OcuJ797zfyLR7YlZJIIB1VUzr9vWow2fQVvanyjqzSrmXa0mZ2FjdqIQWUGl1EVY";

// alist 是否启用了 sign
const alistSignEnable = true;

// alist 中设置的直链过期时间,以小时为单位
const alistSignExpireTime = 12;
```
### 更改constant-pro.js

需要新增如下配置，注意位置和缩进
```
const mediaPathMapping = [
     [0,1,"/mnt/d/clouddrive2/media/115",""],   //新增一行，注意这个配置路径需要对应上面配置
```
源文件如下
```
// 路径映射,会在 mediaMountPath 之后从上到下依次全部替换一遍,不要有重叠,注意 /mnt 会先被移除掉了
// 参数?.1: 生效规则三维数组,有时下列参数序号加一,优先级在参数2之后,需同时满足,多个组是或关系(任一匹配)
// 参数1: 0: 默认做字符串替换replace一次, 1: 前插, 2: 尾插, 3: replaceAll替换全部
// 参数2: 0: 默认只处理本地路径且不为 strm, 1: 只处理 strm 内部为/开头的相对路径, 2: 只处理 strm 内部为远程链接的
// 参数3: 来源, 参数4: 目标
const mediaPathMapping = [
     [0,1,"/mnt/d/clouddrive2/media/115",""],
  // [0, 0, "/aliyun-01", "/aliyun-02"],
  // [0, 2, "http:", "https:"],
  // [0, 2, ":5244", "/alist"],
  // [0, 0, "D:", "F:"],
  // [0, 0, /blue/g, "red"], // 此处正则不要加引号
  // [1, 1, `${alistPublicAddr}/d`],
  // [2, 2, "?xxx"],
  // 此条是一个规则变量引用,方便将规则汇合到同一处进行管理
  // [ruleRef.mediaPathMappingGroup01, 0, 0, "/aliyun-01", "/aliyun-02"],
  // 路径映射多条规则会从上至下依次执行,如下有同一个业务关系集的,注意带上区间的闭合条件,不然会被后续重复替换会覆盖
  // 以下是按码率条件进行路径映射,全用户设备强制,区分用户和设备可再精确添加条件
  // [[["4K 目录映射到 1080P 目录", "r.XMedia.Bitrate", ">", 10 * 1000 ** 2],
  // ], 0, 0, "/4K/", "/1080P/"],
  // [[["1080P 目录映射到 720P 目录", "r.XMedia.Bitrate", ">", 6 * 1000 ** 2],
  //   ["1080P 目录映射到 720P 目录", "r.XMedia.Bitrate", "<=", 10 * 1000 ** 2],

```




