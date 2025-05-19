## 基础dockerfile
```shell
# syntax=harbor.bdser.cc/library/docker/dockerfile:1

ARG OS_VERSION=bullseye-slim

FROM debian:$OS_VERSION

ARG OS_VERSION

LABEL org.opencontainers.image.authors="ops.leops.local" \
      org.opencontainers.image.source="http://git.leops.local/ops/dockerfiles-base/common/os/debian/bullseye-slim/Dockerfile" \
      org.opencontainers.image.description="Standard Debian $OS_VERSION base image for internal use."

ENV LANG='en_US.UTF-8' \
    LANGUAGE='en_US.UTF-8' \
    TZ='Asia/Shanghai' \
    OS_VERSION=$OS_VERSION

WORKDIR /app

RUN sed -i -e 's#deb.debian.org#mirrors.aliyun.com#g' \
           -e 's#security.debian.org#mirrors.aliyun.com#g' \
           -e 's#ftp.debian.org#mirrors.aliyun.com#g' \
           /etc/apt/sources.list \
  && apt-get update \
  && apt-get install -y --only-upgrade $(apt-get --just-print upgrade | awk 'tolower($4) ~ /.*security.*/ || tolower($5) ~ /.*security.*/ {print $2}' | sort | uniq)  \
  && DEBIAN_FRONTEND=noninteractive DEBCONF_NOWARNINGS=yes apt-get install -y --no-install-recommends \
                        ca-certificates openssl \
                        tzdata wget curl procps psmisc dnsutils iproute2 telnet vim-tiny zip unzip inetutils-ping tcpdump lsof \
                        net-tools traceroute strace fonts-noto-cjk fontconfig locales \
  && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
  && dpkg-reconfigure --frontend=noninteractive locales \
  && update-locale LANG=en_US.UTF-8 \
  && fc-cache \
  && apt-get clean \
  && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log \
  && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
  && echo ${TZ} > /etc/timezone \
  && echo 'export PS1="\e[1;34m[\e[1;33m\u@\e[1;32m\h\e[1;37m:\w\[\e[1;34m]\e[1;36m\\$ \e[0m"' >> /etc/bash.bashrc \
  && echo 'transfer(){ if [ $# -eq 0 ];then echo "No arguments specified.\nUsage:\n transfer <file|directory>\n ... | transfer <file_name>">&2;return 1;fi;if tty -s;then file="$1";file_name=$(basename "$file");if [ ! -e "$file" ];then echo "$file: No such file or directory">&2;return 1;fi;if [ -d "$file" ];then file_name="$file_name.zip" ,;(cd "$file"&&zip -r -q - .)|curl --progress-bar --upload-file "-" "http://transfer.leops.local/$file_name"|tee /dev/null,;else cat "$file"|curl --progress-bar --upload-file "-" "http://transfer.leops.local/$file_name"|tee /dev/null;fi;else file_name=$1;curl --progress-bar --upload-file "-" "http://transfer.leops.local/$file_name"|tee /dev/null;fi;echo;}' >> /etc/bash.bashrc
```
## 配置多平台构建环境

```shell
mkdir /etc/buildkit
# vim /etc/buildkit/buildkitd.toml 
insecure-entitlements = [ "network.host", "security.insecure" ]
debug = true
trace = true 

[registry."harbor.bdser.cc"]
  http = true
  insecure = true 

[registry."docker.io"]
  mirrors = ["https://dockerpull.com","https://docker.1ms.run","https://docker.1panel.live"]
```
## 多平台构建
```shell
docker buildx create --name container-builder --driver docker-container --driver-opt "network=host" --buildkitd-config /etc/buildkit/buildkitd.toml --bootstrap --use
```
## 查看多平台基础镜像
```
root@base:/data/project/debian# docker buildx inspect --bootstrap 
Name:          container-builder
Driver:        docker-container
Last Activity: 2025-05-19 10:07:09 +0000 UTC

Nodes:
Name:                  container-builder0
Endpoint:              unix:///var/run/docker.sock
Driver Options:        network="host"
Status:                running
BuildKit daemon flags: --allow-insecure-entitlement=network.host
BuildKit version:      v0.21.1
Platforms:             linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/386
Labels:
 org.mobyproject.buildkit.worker.executor:         oci
 org.mobyproject.buildkit.worker.hostname:         base
 org.mobyproject.buildkit.worker.network:          host
 org.mobyproject.buildkit.worker.oci.process-mode: sandbox
 org.mobyproject.buildkit.worker.selinux.enabled:  false
 org.mobyproject.buildkit.worker.snapshotter:      overlayfs
......
```
## 构建脚本

```shell
#!/bin/bash
set -e
# 配置
REGISTRY="harbor.leops.local"
IMAGE_BASE_NAME="common/os/debian"
VERSION="bullseye-20250407-slim"
# 声明镜像地址数组
declare -a IMAGE_PATHS
IMAGE_PATHS+=(
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION}"           # 完整版本
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION%%-*}"       # 主版本
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION%%-*}-slim"  # 主版本
)


# 构建镜像
build_image() {

    echo "Building and pushing image:"
    for img in "${IMAGE_PATHS[@]}"; do echo -n " $img"; done

    docker buildx build --platform linux/amd64,linux/arm64 \
      $(for img in "${IMAGE_PATHS[@]}"; do echo "-t $img "; done) \
      --build-arg "OS_VERSION=${VERSION}" \
      --label "OS_VERSION=${VERSION}" \
      --label "org.opencontainers.image.created=$(date --rfc-3339=seconds)" \
      --provenance=false \
      --pull \
      --push \
      .

    echo "Build complete."
}
# 参数处理
case "$1" in
    "list-tags")
        # 输出镜像标签列表
        printf '%s\n' "${IMAGE_PATHS[@]}"
        ;;
    *)
    build_image
    ;;
esac

```

