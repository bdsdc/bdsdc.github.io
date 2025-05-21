
## 介绍说明
- go 依赖: goproxy
- Node.js: 使用Verdaccio
- Python: Nexus
- Java(maven/Gradle): Nexus 

**通过nginx 代理域名的方式，本地配置hosts解析，来进行访问和使用**

## 开始部署

### 创建目录结构
```shell
root@base:~# mkdir /data/dependency-proxies/goproxy/cache -p 
root@base:~# mkdir /data/dependency-proxies/verdaccio/{conf,storage,plugins} -p 
root@base:~# mkdir /data/dependency-proxies/nexus/data -p

```
### 准备Verdaccio配置文件
创建 verdaccio/conf/config.yaml 文件，并填入以下内容，配置上游代理为淘宝镜像源:

```shell
# 包存储地址
storage: /verdaccio/storage

# 插件存储地址
plugins: /verdaccio/plugins

# UI相关信息
web:
 enable: true # 开启 Web 页面
 title: Verdaccio  # Web 页面标题

# 身份认证
auth:
 htpasswd:                           # 默认情况下使用的 htpasswd 插件作为身份认证
   file: /verdaccio/storage/htpasswd # htpasswd 文件为加密认证信息文件

# 上行链路
uplinks:
 npmjs:                                    # 上行名称，随便定义
   url: https://registry.npmmirror.com/        # 上行地址
   timeout: 30s                            # 超时时间

                         
 taobao:                                   # 上行名称
   url: https://registry.npm.taobao.org/   # 上行地址
   timeout: 30s                          

# 包访问设置, 可以根据名称对包做不同权限设置
packages:
 '@*/*':

   access: $                             # 登录用户才允许访问
   publish: $authenticated               # 登录用户才允许发布
   proxy: npmjs                          # 代理上行链路地址

 '**':
   access: $all                          # 登录用户才允许访问
   publish: $authenticated               # 登录用户才允许发布
   proxy: npmjs                          # 代理上行链路地址

server:
 keepAliveTimeout: 30                    # 服务器保持活动链接的时间,较大的包可能会消耗一定时间,此属性就是设置活动链接时间

listen: 0.0.0.0:4873 


max_body_size: 100mb

middlewares:
 audit:
   enabled: true

# 日志
logs: { type: stdout, format: pretty, level: info  }
i18n:
  web: zh-CN 

```
### 创建docker-compose文件
```shell
services:

  goproxy:
    container_name: goproxy 
    image: goproxy/goproxy:latest
    restart: always 

    command: "-listen=0.0.0.0:80 -cacheDir=/go/cache -proxy https://goproxy.cn -exclude 'git.bdser.cc'"
    ports:
      - "8084:80"
    volumes:
      - ./goproxy/cache:/go/cache 
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
    environment:
      - TZ=Asia/Shanghai 

  verdaccio:
    container_name: verdaccio 
    image: verdaccio/verdaccio:6.1
    restart: always
    ports:
      - "8085:4873"
    volumes:
      - ./verdaccio/storage:/verdaccio/storage 
      - ./verdaccio/conf:/verdaccio/conf 
      - ./verdaccio/plugins:/verdaccio/plugins
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro 
    environment:
      - TZ=Asia/Shanghai
      - VERDACCIO_PORT=4873
      - VERDACCIO_PUBLIC_URL=//verdaccio.bdser.cc 

  nexus:
    container_name: nexus
    image: sonatype/nexus3:3.79.1-alpine
    restart: always
    user: root
    ports:
      - "8086:8081"


    volumes:
      - ./nexus/data:/nexus-data
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro 
    environment:
      - TZ=Asia/Shanghai 

```
