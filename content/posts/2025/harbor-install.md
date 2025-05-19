


## 安装harbor

### 下载harbor仓库安装包
```
cd /data
wget https://gh-proxy.com/https://github.com/goharbor//harbor/releases/download/v2.13.0/harbor-offline-installer-v2.13.0.tgz
tar xf harbor-offline-installer-v2.13.0.tgz
cd harbor

```
### 配置harbor

复制配置文件模板
```
cp harbor.yml.toml  harbor.yml 

```
修改harbor.yml，需要修改的关键配置如下
```
http:
  port:8080
# 注意，这里不需要HTTPS，所以要注释掉 HTTPS 部分。
# https:
#port: 443
#certificate: /your/certificate/path
#private_key: /your/private/key/path
# Harbor 的访问地址(域名或 IP)
hostname：harbor.leops.local#路径二用户使用域名；路径一用户使用localhost 或 IP
# Harbor 的外部访问 URL (协议 + 主机名)
# 如果不打算配置 HTTPS，保持 http。路径一用户同样修改域名部分。
external_url: http://harbor.bdser.cc
#Harbor 管理员（admin）的初始密码（!!务必修改为强密码!!)
harbor_admin_password: Harbor12345
#数据库内部使用的密码（！!建议修改为强密码！！)
database:
  password: root123
# Harbor 存储镜像等数据的路径（确保存储空间足够)
data_volume：/data/harbor/harbor_data# 确保/data/harbor目录存在或有权限创建

```
如果没有配置https，那么需要Docker客户端新人这个HTTP Registry仓库 
需要修改/etc/docker/daemon.json文件，添加`insecure-registries` 配置，在重启docker服务
```
{
  "insecure-registries": ["harbor.bdser.cc"]

  //...其他配置...
}

systemctl daemon-reload
systemctl restart docker 

```
### 安装harbor
需要支持开启镜像扫描功能， Trivy 
```
# install.sh 会根据harbor.yml 生成部署需要的docker-compose.yml 等文件
bash ./install.sh --with-trivy 
```
**只有首次安装时需要执行`install.sh`，后续升级或修改配置只需执行`./prepare
--with-trivy && docker compose up-d` 即可。**

## 启动docker
```
# 注意目录下面有刚刚生成docker-compose.yml 文件 ， 旧版本需要用 docker-compose up -d 
docker compose up -d
# 查看状态
docker compose ps  
```
## Trivy 在线扫描配置
需要修改Trivy 环境变量,下面这个文件是在上面执行./prepare命令后，会生成的

```
cd /data/harbor
vim common/config/trivy-adapter/env 
```
添加或者修改下面
```
SCANNER_TRIVY_DB_REPOSITORY=ghcr.nju.edu.cn/ghcr.io/aquasecurity/trivy-db,m.daocloud.io/ghcr.io/aquasecurity/trivy-db,ghcr.io/aquasecurity/trivy-db
SCANNER_TRIVY_JAVA_DB_REPOSITORY=ghcr.nju.edu.cn/ghcr.io/aquasecurity/trivy-java-db,m.daocloud.io/ghcr.io/aquasecurity/trivy-java-db,ghcr.io/aquasecurity/trivy-java-db
```
重启Trivy相关服务,让配置生效
```
docker compose restart trivy-adapter coze 
```

## Trivy 离线扫描配置
如果harbor无法访问外网，可以通过配置Trivy 进行离线扫描

首先修改harbor.yml 
```
trivy:
  skip_update: true
  skip_java_db_update: true 
```
需要重新执行生效配置
```
sudo ./prepare --with-trivy 
```

需要手动下载数据库
> 需要临时开放外网权限或者代理下载，或者其他机器下载好，手动拷贝到harbor服务器上

```shell
#主数据下载
docker exec trivy-adapter trivy image --download-db-only --db-repository  ghcr.nju.edu.cn/ghcr.io/aquasecurity/trivy-db
#下载java数据库
docker exec trivy-adapter trivy image --download-java-db-only java-db-repository ghcr.nju.edu.cn/ghcr.io/aquasecurity/trivy-java-db
```
重启服务
```
docker compose up -d
#或者重启trivy-adapter 和 core 服务
docker compose restart trivy-adapter core
```
## harbor域名代理配置
配置nginx代理
```
server {
	listen 80;
	server_name harbor.bdser.cc;
	access_log /var/log/nginx/harbor-access.log;
	error_log  /var/log/nginx/harbor-error.log;
	client_max_body_size 1000m; 
	location / {
		proxy_pass http://192.168.31.236:8080;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_buffering off;
		proxy_request_buffering off;
	
		proxy_send_timeout 900;
		proxy_read_timeout 900;
	}
}
```
### 加载nginx配置
```
docker exec nginx-proxy sh -c "nginx -t && nginx -s reload" 
```
## 访问和使用harbor仓库

### 浏览器访问
通过浏览器访问刚刚配置的域名 http://harbor.bdser.cc

- 用户名： admin
- 密码： 配置文件中通过 harbor_admin_password 来设置 （harbor.yml）   
### 使用
创建project 

- 登录后，点击 项目 ， 然后 新建项目
- common： 用于存放通用的基础镜像（系统，工具，运行层），设置为 公开，方便拉取
- dev：用于存放开发的镜像，可以设置为 私有
- test：用于存放测试的镜像，可以设置为 私有
- prd： 用于存放生产的镜像，可以设置为 私有

### 命令行登录
```
docker login harbor.bdser.cc 
# docker login
# docker login <IP地址>

# 输入用户名：
# 输入密码：
```
### 推送镜像
```
# 给镜像打赏harbor 的标签 （ 格式： <harbor_hostname>/<项目名>/<镜像名>:<标签> 
docker tag nginx:1.27-alpine harbor.bdser.cc/library/nginx:1.27-alpine 
# 推送镜像到 harbor 的 library 项目
docker push harbor.bdser.cc/library/nginx:1.27-alpine 
```
### 拉取镜像
```
docker pull harbor.bdser.cc/library/nginx:1.27-alpine 
```

## harbor 日常维护
- 项目保留策略
- 系统清理服务











