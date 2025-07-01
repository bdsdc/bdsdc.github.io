

#  构建GO 应用Docker镜像 
Go (Golang)[1] 语言因其编译成静态二进制文件、天然支持并发以及方便的跨平台编译等特性，非常适合容器化部署。本篇将重点介绍如何使用多阶段构建 (Multi-stage builds) 来为 Go 应用创建优化、小巧且安全的 Docker 镜像


Go 语言的特点使其非常适合容器化：
- 静态编译：生成的二进制文件包含所有依赖，不需要额外的运行时
- 高效并发：通过 goroutine 和 channel 机制提供简单易用的并发模型
- 跨平台能力：轻松实现不同操作系统和架构的编译

Go 语言的特点使其非常适合容器化：
- 静态编译：生成的二进制文件包含所有依赖，不需要额外的运行时
- 高效并发：通过 goroutine 和 channel 机制提供简单易用的并发模型
- 跨平台能力：轻松实现不同操作系统和架构的编译

## 构建GO 应用的编译环境

```
mkdir -p common/tools/golang
cd common/tools/golang 
```

### 基于Debian的GO编译环境
演示 Dockerfile如下
```
#syntax=harbor.bdser.cc/library/docker/dockerfile:1

FROM harbor.bdser.cc/common/os/debian:bullseye

ARG GOLANG_VERSION=1.24.2

LABEL GOLANG_VERSION="${GOLANG_VERSION}" \
      org.opencontainers.image.maintainer="ops@bdser.cc" \
      org.opencontainers.image.source="http://git.bdser.cc/ops/dockerfiles-base/common/tools/golang/Dockerfile" \
      org.opencontainers.image.description="golang ${GOLANG_VERSION} compiler environment."

ENV GOLANG_VERSION=$GOLANG_VERSION \
    GOPATH=/go \
    CGO_ENABLED=1 \
    GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct

ENV PATH=/usr/local/go/bin:$GOPATH/bin:$PATH

# install cgo-related dependencies
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
            git \
            g++ \
            gcc \
            make \
            cmake \
            pkg-config \
            binutils \
            libc6-dev \
            libstdc++6 \
            libssl-dev \
            libsasl2-dev \
            libzstd-dev \
            libcurl4-openssl-dev \
            ; \
    apt-get clean; \
    rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* /tmp/* /var/tmp/*; \
    truncate -s 0 /var/log/*log;


# install golang
RUN set -eux; \
    arch="$(dpkg --print-architecture)"; arch="${arch##*-}"; \
    url=; \
    case "$arch" in \
        'amd64') \
            url="https://dl.google.com/go/go${GOLANG_VERSION}.linux-amd64.tar.gz"; \
            ;; \
        'armel') \
            export GOARCH='arm' GOARM='5' GOOS='linux'; \
            ;; \
        'armhf') \
            url="https://dl.google.com/go/go${GOLANG_VERSION}.linux-armv6l.tar.gz"; \
            ;; \
        'arm64') \
            url="https://dl.google.com/go/go${GOLANG_VERSION}.linux-arm64.tar.gz"; \
            ;; \
        'i386') \
            url="https://dl.google.com/go/go${GOLANG_VERSION}.linux-386.tar.gz"; \
            ;; \
        'mips64el') \
            export GOARCH='mips64le' GOOS='linux'; \
            ;; \
        'ppc64el') \
            url="https://dl.google.com/go/go${GOLANG_VERSION}.linux-ppc64le.tar.gz"; \
            ;; \
        's390x') \
            url="https://dl.google.com/go/go${GOLANG_VERSION}.linux-s390x.tar.gz"; \
            ;; \
        *) echo >&2 "error: unsupported architecture '$arch' (likely packaging update needed)"; exit 1 ;; \
    esac; \
    build=; \
    if [ -z "$url" ]; then \
# https://github.com/golang/go/issues/38536#issuecomment-616897960
        build=1; \
        url="https://dl.google.com/go/go${GOLANG_VERSION}.src.tar.gz"; \
        echo >&2; \
        echo >&2 "warning: current architecture ($arch) does not have a compatible Go binary release; will be building from source"; \
        echo >&2; \
    fi; \
    \
    wget -O go.tgz "$url" --progress=dot:giga; \
    \
# https://github.com/golang/go/issues/14739#issuecomment-324767697
    GNUPGHOME="$(mktemp -d)"; export GNUPGHOME; \
    \
    tar -C /usr/local -xzf go.tgz; \
    rm go.tgz; \
    \
    if [ -n "$build" ]; then \
        savedAptMark="$(apt-mark showmanual)"; \
        apt-get update; \
        apt-get install -y --no-install-recommends golang-go; \
        \
        ( \
            cd /usr/local/go/src; \
# set GOROOT_BOOTSTRAP + GOHOST* such that we can build Go successfully
            export GOROOT_BOOTSTRAP="$(go env GOROOT)" GOHOSTOS="$GOOS" GOHOSTARCH="$GOARCH"; \
            ./make.bash; \
        ); \
        \
        apt-mark auto '.*' > /dev/null; \
        apt-mark manual $savedAptMark > /dev/null; \
        apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
        apt-get clean; \
        rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* /tmp/* /var/tmp/*; \
        truncate -s 0 /var/log/*log; \
        \
# remove a few intermediate / bootstrapping files the official binary release tarballs do not contain
        rm -rf \
            /usr/local/go/pkg/*/cmd \
            /usr/local/go/pkg/bootstrap \
            /usr/local/go/pkg/obj \
            /usr/local/go/pkg/tool/*/api \
            /usr/local/go/pkg/tool/*/go_bootstrap \
            /usr/local/go/src/cmd/dist/dist \
        ; \
    fi; \
    \
    go version


# install dependencies
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
            libprotobuf-dev; \
    PROTOC_VERSION="30.2"; \
    PROTOC_ZIP="protoc-${PROTOC_VERSION}-linux-x86_64.zip"; \
    curl -OL "https://gh-proxy.com/github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VERSION}/${PROTOC_ZIP}"; \
    unzip -o $PROTOC_ZIP -d /usr/local bin/protoc; \
    unzip -o $PROTOC_ZIP -d /usr/local 'include/*'; \
    go install honnef.co/go/tools/cmd/staticcheck@latest; \
    go install github.com/go-delve/delve/cmd/dlv@latest; \
    go install gotest.tools/gotestsum@latest; \
    go install github.com/boumenot/gocover-cobertura@latest; \
    go install github.com/elliots/protoc-gen-twirp_swagger@latest; \
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest; \
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest; \
    go clean -cache; \
    go clean -modcache; \
    apt-get clean; \
    rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/* /tmp/* /var/tmp/*; \
    truncate -s 0 /var/log/*log; \
    rm -rf $GOPATH/pkg/* $PROTOC_ZIP;


WORKDIR $GOPATH

```

### Dockerfile 解析说明

```
这个编译环境 Dockerfile 有以下几个重要特点：
1. 基于 Debian：使用了我们之前创建的标准化 Debian 基础镜像
2. 动态版本：通过ARG GOLANG_VERSION参数化 Go 版本，便于维护和更新
3. 环境配置：设置了 Go 开发所需的环境变量（GOPATH、GO111MODULE 等）
4. CGO 支持：安装了 C/C++编译工具链，支持 CGO 功能
5. 多架构支持：检测当前 CPU 架构并安装相应版本的 Go
6. 工具链安装：包含了 protobuf、代码检查、调试等实用工具7. 缓存清理：每个步骤后清理不必要的缓存文件，减小镜像体积
```
### 多平台构建脚本

```
#!/bin/bash

set -e

# 配置
REGISTRY="harbor.bdser.cc"
IMAGE_BASE_NAME="common/tools/golang"
VERSION="1.24.2"


# 声明镜像地址数组
declare -a IMAGE_PATHS
IMAGE_PATHS+=(
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION}"
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION%.*}"
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION}-debian11"
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION%.*}-debian11"
)


build_image() {

    echo "Building and pushing image:"
    for img in "${IMAGE_PATHS[@]}"; do echo -n " $img"; done

    # 构建镜像
    docker buildx build \
      $(for img in "${IMAGE_PATHS[@]}"; do echo -n "-t $img "; done) \
      --label "org.opencontainers.image.created=$(date --rfc-3339=seconds)" \
      --network host \
      --add-host goproxy.bdser.cc=192.168.31.239 \
      --add-host dl.google.com=180.163.151.33 \
      --build-arg "GOLANG_VERSION=${VERSION}" \
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
脚本支持创建多个标签版本，包括完整版本号、主版本号以及带有系统标识的组合标签，确保镜像引用的灵活性。

## 构建 GO 应用的运行时镜像 

运行时镜像特点
- 最小化攻击面：减少不必要的组件和工具
- 合理的权限：避免使用 root 用户运行应用
- 适当的文件结构：提供标准化的应用目录结构
- 体积小：更快的分发和部署速度

```
mkdir -p common/runtime/golang
cd common/runtime/golang 
```
运行时镜像Dockerfile 
```
#syntax=harbor.bdser.cc/library/docker/dockerfile:1

FROM harbor.bdser.cc/common/os/debian:bullseye

LABEL org.opencontainers.image.authors="ops@bdser.cc"  \
      org.opencontainers.image.source="http://git.bdser.cc/ops/dockerfiles-base/common/runtime/go/Dockerfile" \
      org.opencontainers.image.description="Minimal base runtime for Go applications with non-root user."


RUN \
    groupadd -r nonroot \
    && useradd -r -g nonroot nonroot \
    && mkdir -p /app/logs \
    && chown nonroot:nonroot -R /app

USER nonroot:nonroot

```
### 运行时镜像Dockerfile解析
这个运行镜像的关键特点：
- 基于精简 Debian：继承了我们优化过的 Debian 基础镜像
- 非 root 用户：创建了专用的 nonroot 用户，提高安全性
- 标准目录结构：预先创建了应用和日志目录，并设置了正确的权限
- 最小依赖：不包含编译工具、开发库等不必要组件
- 权限切换：通过 USER 指令切换到非特权用户，防止容器内进程获取过高权

### 运行时镜像构建build脚本
```

#!/bin/bash

set -e

# 配置
REGISTRY="harbor.bdser.cc"
IMAGE_BASE_NAME="common/runtime/golang"
VERSION="debian11"


# 声明镜像地址数组
declare -a IMAGE_PATHS
IMAGE_PATHS+=(
    "${REGISTRY}/${IMAGE_BASE_NAME}:${VERSION}"           # 完整版本
)


build_image() {

    echo "Building and pushing image:"
    for img in "${IMAGE_PATHS[@]}"; do echo -e " $img"; done

    # 构建镜像
    docker buildx build \
      $(for img in "${IMAGE_PATHS[@]}"; do echo -n "-t $img "; done) \
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

## 多阶段构建镜像-应用镜像
在准备好编译环境和运行环境后，我们可以通过多阶段构建来创建最终的应用镜像。多阶段构建允许我们在同一个 Dockerfile 中使用多个 FROM 指令，每个阶段可以独立运行并复用结果。

### docker 多阶段构建说明
多阶段构建的主要优势：
- 分离构建和运行环境：编译阶段可以包含所有开发工具，而运行阶段仅包含必要组件
- 减小最终镜像体积：最终镜像只包含运行所需文件，不含编译工具和中间产物
- 优化缓存：合理使用构建缓存加速重复构建
- 简化流程：将多个构建步骤整合到单个 Dockerfile 中

```
mkdir /web
cd /web
git clone https://github.com/lework/ci-demo-go.git
cd ci-demo-go

```
Dockerfile
```
#syntax=harbor.bdser.cc/library/docker/dockerfile:1

# ---- 编译环境 ----
FROM harbor.bdser.cc/common/tools/golang:1.24 AS builder

ARG APP_ENV=test \
  APP=undefine \
  GIT_BRANCH= \
  GIT_COMMIT_ID=

ENV APP_ENV=$APP_ENV \
  APP=$APP \
  GIT_BRANCH=$GIT_BRANCH \
  GIT_COMMIT_ID=$GIT_COMMIT_ID

WORKDIR /app_build

# 编译
COPY . .
RUN --mount=type=cache,id=gomod,target=/go/pkg/mod \
  --mount=type=cache,id=gobuild,target=/root/.cache/go-build \
  go build  -tags 'osusergo,netgo' \
  -ldflags "-X main.branch=$GIT_BRANCH -X main.commit=$GIT_COMMIT_ID" \
  -v -o bin/${APP} *.go \
  && cp -rf etc bin/etc \
  && chown 999.999 -R bin

#
# ---- 运行环境 ----
FROM harbor.bdser.cc/common/runtime/golang:debian11 AS running

ARG APP_ENV=test \
  APP=undefine \
  GIT_BRANCH= \
  GIT_COMMIT_ID=

ENV APP_ENV=$APP_ENV \
  APP=$APP \
  GIT_BRANCH=$GIT_BRANCH \
  GIT_COMMIT_ID=$GIT_COMMIT_ID

WORKDIR /app

COPY --from=builder --link /app_build/bin /app/

CMD ["bash", "-c", "exec /app/${APP} -f /app/etc/app_${APP_ENV}.yaml"]
```
### 多阶段构建应用镜像解析
这个 Dockerfile 分为两个明确的阶段：
1. 编译阶段 (builder)：
  - 使用我们的 Go 编译环境镜像
  - 接收构建参数（环境、应用名称、Git 信息等）
  - 复制源代码到工作目录
  - 使用 BuildKit 缓存加速依赖下载和编译
  - 通过 ldflags 注入版本信息
  - 为生成的二进制文件和配置设置正确权限

2. 运行阶段 (running)：
  - 使用我们的精简运行环境镜像
  - 仅从编译阶段复制编译结果
  - 使用--link确保构建缓存的高效利用
  - 通过 CMD 指定启动命令，支持不同环境配置


### 构建脚本buil-app.sh

脚本实现了以下功能：
- 接手环境和应用名称参数
- 自动获取Git分支和提交信息
- 生成包含所有镜像相关信息的镜像标签
- 构建并推送镜像
- 对生产环境主分支构建添加latest标签

```
#!/bin/bash

set -e

APP_ENV="$1"
APP="$2"

REGISTRY="harbor.bdser.cc"
APP_GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
APP_GIT_COMMITID=$(git rev-parse --short HEAD)

APP_IMAGE="${REGISTRY}/${APP_ENV}/${APP}"
APP_IMAGE_TAG="${APP_GIT_BRANCH}-${APP_GIT_COMMITID}-$(date +%Y%m%d%H%M)"

docker buildx build \
-t "${APP_IMAGE}:${APP_IMAGE_TAG}" \
--add-host goproxy.bdser.cc=192.168.31.239 \
--add-host nexus.bdser.cc=192.168.31.239 \
--add-host verdaccio.bdser.cc=192.168.31.239 \
--build-arg APP=${APP} \
--build-arg APP_ENV=${APP_ENV} \
--build-arg GIT_BRANCH=${APP_GIT_BRANCH} \
--build-arg GIT_COMMIT_ID=${APP_GIT_COMMITID} \
--progress plain \
--push .

if [[ "$APP_ENV" == "prd" && ${APP_GIT_BRANCH} == "master" ]]; then
  docker tag "${APP_IMAGE}:${APP_IMAGE_TAG}" "${APP_IMAGE}:latest"
  docker push "${APP_IMAGE}:latest"
fi

```
### 开始构建应用
```
cd web/ci-demo-go 
./build-app.sh dev ci-demo-go 
```

## 运行Go应用容器
```
# 启动容器
docker run -d --name ci-demo-go -p 18080:8080 harbor.bdser.cc/dev/ci-demo-go:master-4ff8dc8-202507012318
# 访问应用
curl http://192.168.31.239:18080/health
# 查看日志
docker logs ci-demo-go
# 停止和启动
docker stop ci-demo-go
docker start ci-demo-go 
```

### 生产环境实战解析
在生产环境中部署 Go 应用容器时，可以考虑以下最佳实践：
- 资源限制：使用--memory和--cpus设置容器资源上限
- 健康检查：添加--health-cmd配置容器健康检查
- 日志管理：配置日志驱动将日志发送到集中式日志系统
-  网络安全：仅暴露必要端口，使用网络隔离
- 存储持久化：对需要持久化的数据使用卷挂载
- 容器编排：在生产环境中使用 Kubernetes 等工具进行编排总结

## 总结
通过采用多阶段构建和遵循最佳实战，我们成功为Go 应用程序创建了优化，小巧而且安全的Docker镜像
- 镜像体积小
- 构建速度快
- 安全性高
- 标准化流程
- 资源效率

通过这种方式构建的Go 应用程序容器 ，保留了Go语言的高性能特性，又充分发挥了容器技术的优势，为微服务架构和云原生应用提供了理想的部署方案
