

## 

```
curl https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo > /etc/yum.repos.d/Centos-vault-8.5.2111.repo
curl https://mirrors.aliyun.com/repo/epel-archive-8.repo > /etc/yum.repos.d/epel-archive-8.repo


```

```
yum install docker-ce docker-compose
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y yum-utils device-mapper-persistent-data lvm2
yum install docker-ce docker-compose
```

dnf
```
systemctl disable --now podmansudo dnf remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
dnf remove -y podman podman-docker buildah
dnf remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
dnf remove -y podman buildah runc containers-common

``
sync-in
```
curl -L -o sync-in-docker.zip https://github.com/Sync-in/server/releases/latest/download/sync-in-docker.zip && unzip sync-in-docker.zip
cd sync-in-docker/
```
#### env yaml配置
```
server:
  # It will become required in a future major release.
  # e.g.: https://sync-in.domain.com
  publicUrl: http://xx.xx.xx.xx
  title: 'xxx'        # 网站标题/名称
  description: 'YKT内部文档知识库'   # 网站描述
mysql:
  url: 'mysql://root:4rfv5TGB@mariadb:3306/sync_in'
auth:
  # Key used to encrypt MFA secrets at rest.
  # Optional but strongly recommended.
  # Use a long, randomly generated value and do not change it after enabling MFA.
  encryptionKey: 'f1e2d3c4b5a69788796a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e'
  token:
    access:
      # Required. Use a long, randomly generated secret.
      secret: '8a2f4b8e9c1d7a6b3f5e2d1c4b8a9f7e6d5c3b2a1f4e8d7c6b9a5f2e1d4c8b7a'
    refresh:
      # Required. Use a different long, randomly generated secret.
      secret: 'a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2'
  provider: ldap
  ldap:
    servers:
      - 'ldap://xxx.xxx.xxx.xxx:389'
    baseDN: 'OU=xxx,DC=xxx,DC=local'
    serviceBindDN: 'CN=xxx,OU=ServicesAccounts,OU=xxx,DC=xxx,DC=local' # 注意要是完整的DN
    serviceBindPassword: 'xxx'
    attributes:
      login: 'sAMAccountName'
      email: 'mail'
      name: 'displayName'
      # 为首次登录的 AD 用户自动分配基础权限，防止进来后什么都点不了
    options:
      autoCreatePermissions:
        - 'personal_space'   # 自动开通个人空间（拥有完整的上传、新建权限）
        - 'spaces_access'    # 允许访问协作空间
        - 'shares_access'    # 允许访问“共享给我”
        - 'webdav_access'    # 允许使用 WebDAV 协议
        - 'desktop_app_access' # 允许使用桌面客户端


applications:
  files:
    editors:
      collabora:
        enabled: false
      drawio:
        enabled: true
      onlyoffice:
        enabled: false
        # Required when enabled. Must match JWT_SECRET in the OnlyOffice container.
        secret:
      eurooffice:
        enabled: true
        # Required when enabled. Must match JWT_SECRET in the Euro-Office container.
        secret: '9f4c7a2e8b1d6f3a5c0e9b7d2a4f8c1e6b3d9a7f5c2e8b4d1a6f9c3e7b2d5a8f'

```
#### docker compose yaml

```
include:
  - ./config/nginx/docker-compose.nginx.yaml
#  - ./config/onlyoffice/docker-compose.onlyoffice.yaml
#  - ./config/collabora/docker-compose.collabora.yaml
  - ./config/eurooffice/docker-compose.eurooffice.yaml
  - ./config/drawio/docker-compose.drawio.yaml
#  - ./config/sync-in-desktop-releases/docker-compose.sync-in-desktop-releases.yaml

services:
  sync_in:
    image: syncin/server:2
    container_name: sync-in
    restart: always
    environment:
      - INIT_ADMIN=true
      - INIT_ADMIN_LOGIN=admin
      - INIT_ADMIN_PASSWORD=4rfv5TGB
      - PUID=8888
      - PGID=8888
    ports:
      - "8080:8080"
    volumes:
      # 配置文件（必须与宿主机文件同步）
      - ./environment.yaml:/app/environment/environment.yaml
      # 用户文件存储：将容器内 /app/data 映射到宿主机，便于备份
      - /data/sync-in/data:/app/data
    depends_on:
      mariadb:
        condition: service_healthy
    networks:
      - sync_in_network
    logging:
      driver: "json-file"
      options:
        max-size: "25m"
        max-file: "5"

  mariadb:
    image: mariadb:11
    container_name: mariadb
    restart: always
    command:
      - --innodb_ft_cache_size=16000000
      - --max-allowed-packet=1G
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_uca1400_ai_ci
    environment:
      MYSQL_USER: root
      MYSQL_ROOT_PASSWORD: 4rfv5TGB  # 务必修改为强密码
      MYSQL_DATABASE: sync_in
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    volumes:
      # MySQL 数据目录映射到宿主机
      - /data/sync-in/mysql:/var/lib/mysql
    networks:
      - sync_in_network

networks:
  sync_in_network:
    name: sync_in_network
    driver: bridge

volumes:
  desktop_releases:
```
#### euroffice 
```
services:
  eurooffice:
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/euro-office/documentserver:v9.3.3
    container_name: eurooffice
    restart: always
    environment:
      # Must match applications.files.editors.eurooffice.secret in environment.yaml.
      - JWT_SECRET=9f4c7a2e8b1d6f3a5c0e9b7d2a4f8c1e6b3d9a7f5c2e8b4d1a6f9c3e7b2d5a8f
      - JWT_IN_BODY=true
      - USE_UNAUTHORIZED_STORAGE=true
    logging:
      driver: json-file
      options:
        max-size: "25m"
        max-file: "5"
    networks:
      - sync_in_network
```
#### onlyoffice docker compose 
```
services:
  onlyoffice:
    # Version 9.4.0.1 may fail in Firefox when Analytics.js is blocked: https://github.com/ONLYOFFICE/DocumentServer/issues/3686
    image: onlyoffice/documentserver:9.3.1.2
    container_name: onlyoffice
    restart: always
    environment:
      # Must match applications.files.editors.onlyoffice.secret in environment.yaml.
      - JWT_SECRET=
      - JWT_IN_BODY=true
      - USE_UNAUTHORIZED_STORAGE=true
    logging:
      driver: json-file
      options:
        max-size: "25m"
        max-file: "5"
    networks:
      - sync_in_network
      

```

### docker daemon.json 
```
[root@VM_96_40_centos sync-in-docker]# cat /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.1panel.live"
  ]
}
```


