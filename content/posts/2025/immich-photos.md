# 开源相册IMMICH私有化本地部署


## 简介
可以直接从手机后台备份照片和视频的相册解决方案，支持苹果（Live）和安卓，支持人脸，地图，地点，搜索等大模型功能

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250615225448.png)
## 说明
- 个人照片管理：immich提供了一个简单而直观的界面，让个人用户能够方便地管理自己的照片集合。用户可以通过标签、日期和描述等元数据对照片进行分类和搜索，同时可以进行批量处理和编辑操作。

- 团队协作：immich支持多用户使用，可以作为团队内部的照片管理工具。团队成员可以共享照片集合，并进行评论和讨论。通过权限管理，可以精确控制不同用户对照片的访问和编辑权限。

- 专业摄影师：对于摄影师来说，照片管理是不可或缺的一环。immich提供了针对摄影师的一些特殊功能，如支持原始RAW文件的管理和预览、批量导出和水印处理等。摄影师可以方便地对照片进行整理、筛选和处理，从而节省时间和提高工作效率。

- 图片库管理：immich的强大搜索和分类功能使其成为一个理想的图片库管理工具。无论是个人图片库、企业图片库还是公共图片库，都可以通过immich来进行统一管理和检索。用户可以根据需要自定义标签和分类方式，从而更方便地找到所需的图片。

## 环境需求
硬件
- 操作系统：推荐使用Linux或*nix操作系统（如Ubuntu、Debian等）。非Linux操作系统往往提供较差的Docker体验，强烈建议不要使用。我们在非Linux操作系统上的设置或故障排除支持能力将大幅降低。如果您仍然想尝试使用非Linux操作系统，可以按照以下方式设置：
  - Windows：Windows上的Docker Desktop或WSL 2。
  - macOS：Mac上的Docker Desktop。
。
- RAM：最小4GB，推荐6GB。
- CPU：最小2核，推荐4核。
- 存储：推荐使用兼容Unix的文件系统（如EXT4、ZFS、APFS等），支持用户/组所有权和权限。
- 生成缩略图和转码视频可能会使照片库的大小平均增加10-20%。

`注意`：**当在完整虚拟机中运行时，Immich在虚拟化环境中运行良好。不建议在LXC容器中使用Docker，但对于高级用户可能可行。如果您遇到问题，我们建议您切换到受支持的虚拟机部署**

## 安装
仓库地址： <https://github.com/immich-app/immich>

配置文件地址：<https://github.com/immich-app/immich/tree/main/docker>

我们从仓库中，下载docker-compose文件和env文件 

### 环境
我们需要准备好docker ，docker compose 环境，这个环境准备，可以看我博客的相关文章，这里不做阐述，现在所有的环境都已经安装好

首先我们先建立目录，准备放在下面目录
```
mkdir /mnt/d/immich-app
cd /mnt/d/immich-app
```

### 编辑docker-compose.yml文件

我们这里由于没有gpu，并且个人也没有转码，解码需求，所以这里关于解码等功能先不开启，大模型功能如 搜索和人脸我们后面会用到

```
# WARNING: To install Immich, follow our guide: https://immich.app/docs/install/docker-compose
#
# Make sure to use the docker-compose.yml of the current release:
#
# https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
#
# The compose file on main may not be compatible with the latest release.

name: immich

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    # extends:
    #   file: hwaccel.transcoding.yml
    #   service: cpu # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    restart: always
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    # For hardware acceleration, add one of -[armnn, cuda, rocm, openvino, rknn] to the image tag.
    # Example tag: ${IMMICH_VERSION:-release}-cuda
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    # extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
    #   file: hwaccel.ml.yml
    #   service: cpu # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference - use the `-wsl` version for WSL2 where applicable
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: always
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:ff21bc0f8194dc9c105b769aeabf9585fea6a8ed649c0781caeac5cb3c247884
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.3.0-pgvectors0.2.0@sha256:fa4f6e0971f454cd95fec5a9aaed2ed93d8f46725cc6bc61e0698e97dba96da1
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    restart: always

volumes:
  model-cache:
```
### 编辑.env配置文件
```
# You can find documentation for all the supported env variables at https://immich.app/docs/install/environment-variables

# The location where your uploaded files are stored
UPLOAD_LOCATION=./library

# The location where your database files are stored. Network shares are not supported for the database
DB_DATA_LOCATION=/data/immich/postgres

# To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
# TZ=Etc/UTC

# The Immich version to use. You can pin this to a specific version like "v1.71.0"
IMMICH_VERSION=release

# Connection secret for postgres. You should change it to a random password
# Please use only the characters `A-Za-z0-9`, without special characters or spaces
DB_PASSWORD=postgres

# The values below this line do not need to be changed
###################################################################################
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
TZ=Asia/Shanghai
MODEL_CACHE=./model-cache
```
## 模型下载
目前国内大模型下载方法
大模型地址: <https://huggingface.co> 
### 模型介绍
- 搜索模型：immich-app/XLM-Roberta-Large-Vit-B-16Plus
- 人脸识别模型： buffalo_l

### 方法

- 上游出国: 比如有openwrt，ikuai，istoreos等
- 本地出国: v2ay,socket5,ssr,proxy等
- 手动获取：自己想办法下载

### 获取方式
- huggingface.co网站，直接文件一个一个下载
- 自己出国用proxy代理，通过git clone 方式下载，需要安装git lfs,没有断电续传
- Python环境安装huggingface-cli，通过huggingface-cli和代理proxy方式
- 通过公益项目 <https://hf-mirror.com/>

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250615225334.png)

### 模型位置
这个请看docker-compose中volume的配置，这个可以自定义，我这里放在
```
/mnt/d/immich-app/model-cache/ 
```

## 用huggingface-cli方式

```
python3 -m venv huggingface-venv
# 激活环境
source huggingface-venv/bin/activate
```
### 安装 huggingface-clit 
```
python3  -m pip install huggingface-hub --index-url=https://mirrors.aliyun.com/pypi/simple
```

### 下载模型
```
export HF_ENDPOINT=https://hf-mirror.com
huggingface-cli download   immich-app/XLM-Roberta-Large-Vit-B-16Plus
```
## 个人出国
```
# linux终端配置proxy
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
# 安装git-lfs  
apt install git-lfs 
git clone https://huggingface.co/immich-app/XLM-Roberta-Large-Vit-B-16Plus
```
我们可以先使用然后整理照片，模型下载比较慢，后面下载完成后，我们在使用

## 使用
我们通过浏览器访问 http://localhost:2283


注册邮件，设置用户名和密码
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250615223431.png)

用户界面配置
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250615223603.png)

系统界面配置
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250615223821.png)


## 手机app

### apple手机 
可以从商店直接下载 ，然后配置服务器地址


### 安卓手机
可以从 <https://github.com/immich-app/immich/releases> 下载 ,手机一般选择arm-v8a 

## 备份到相册

### 苹果apple手机备份

我们配置好服务器地址,赋予手机相册权限,点击右上角 `云`的图片,就可以进行备份了

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/b7caba1dff6d3d94aba6f91ec370c1f.jpg)

![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/3f7bb18b97adcb98ffc0f5bf7ae191d.jpg)

### 后台备份
可以忽略上传到icloud中的照片 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/3a95e60c7ca1e106b42730a06187351.jpg)

### 命令行备份
可以npm 和 docker俩种形式安装 immich-cli终端命令
```
docker run -it -v /mnt/d/syncthing/image:/mnt:ro -e IMMICH_INSTANCE_URL=http://IP:port/api -e IMMICH_API_KEY=you-key-api  ghcr.io/immich-app/immich-cli:latest  upload /mnt/mmexport1726623504420.jpg
```
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20250615234204.png)

## 总结
经过一番简单折腾，我们终于有了自己的开源本地相册，你感觉怎么样？我正在逐步使用起来，综合感觉还是不错，安卓手机开启备份，苹果手机上也能看到图片 也可以支持后台备份，设置定时任务，手机存储终于可以腾出来了

所以，有兴趣就来试试吧！可以在动手前看看，建议再看看官方的文档






