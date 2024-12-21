

ollama是一个开源项目，它提供了一个平台和工具集，用于部署和运行各种大型语言模型。 

ollama官网：[https://ollama.com/](https://ollama.com/)

ollama下载地址：[https://ollama.com/download](https://ollama.com/download)

GitHub地址：[https://github.com/ollama/ollama](https://github.com/ollama/ollama)

## ollama 介绍

### ollama部署
我们这里选择直接部署docker,简单方便，docker部署忽略 

docker镜像地址： [https://hub.docker.com/r/ollama/ollama](https://hub.docker.com/r/ollama/ollama)

```shell
# 拉取docker镜像
docker pull ollama/ollama:latest

# AMD显卡
docker run -d   -v /mnt/c/ubuntu-wsl/data/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm 
# 英伟达显卡
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
# cpu only 
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```
### ollama模型
下面是一些受欢迎的模型：

|       Model        |     Tag     |  Parameters  |  Size   | Download                            |
|:------------------:|:-----------:|:------------:|:-------:|-------------------------------------|
|       llama3       |      -      |     70b      |  40GB   | `ollama run llama3:70b`             |
|       llama3       |      -      |      8b      |  4.7GB  | `ollama run llama3:8b`              |
|       gemma        |      -      |      7b      |  5.0GB  | `ollama run gemma:7b`               |
|       gemma        |      -      |      2b      |  1.7GB  | `ollama run gemma:2b`               |
|      mistral       |      -      |      7b      |  4.1GB  | `ollama run mistral:7b`             |
|        qwen        |      -      |     110b     |  63GB   | `ollama run qwen:110b`              |
|        phi3        |      -      |     3.8b     |  2.2GB  | `ollama run phi3:3.8b`              |
|       llama2       |      -      |      7b      |  3.8GB  | `ollama run llama2:7b`              |
|     codellama      |    Code     |     70b      |  39GB   | `ollama run codellama:70b`          |
|      llama3.1      |      -      |     405b     |  231GB  | `ollama run llama3.1:405b`          |
|       gemma2       |      -      |     27b      |  16GB   | `ollama run gemma2:27b`             |
|       qwen2        |      -      |     72b      |  41GB   | `ollama run qwen2:72b`              |
|       llava        |   Vision    |      7b      |  4.7GB  | `ollama run llava:7b`               |
|  nomic-embed-text  |  Embedding  |     v1.5     |  274MB  | `ollama pull nomic-embed-text:v1.5` |



### 本地模型启动
启动本地大模型，这个步骤会下载llama3模型，根据个人带宽网速预估时间
```
docker exec -it ollama ollama run llama3
```
### 交互对话 
执行完毕后，会进入交互模式，输入内容，就可以在线对话了，我们用docker安装的，所以通过docker命令启动对话
```shell
root@DESKTOP-CK75KU2:~# docker exec -it ollama ollama run  llama3
>>> 请介绍一下你自己
I'm just an AI, I don't have a personal identity or individual characteristics like humans do. However, I can introduce
myself and explain what I am and what I can do.

My name is LLaMA, and I'm a large language model trained by Meta AI. I was created to assist users in generating
human-like text based on the input they provide me.

```


## openweb ui 介绍
openweb ui参考仓库地址：https://github.com/ollama-webui/ollama-webui-lite

### 使用docker部署
```shell
docker run -d -p 8186:8080 --add-host=host.docker.internal:host-gateway -v /mnt/c/ubuntu-wsl/data/openwebui:/app/backend
```
之后点击端口访问，如下图所示。也可以直接在浏览器输入 http://localhost:8186/ ，打开后会出现登录到 Open WebUI，只需要邮箱注册一下就好了 
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412211313308.png)


### 选择模型
选型我们刚刚安装的模型，就可以发起对话了
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412211312949.png)

### 对话演示
这里我们选择 ollama3 模型，进行对话，然后发现都是英文回复，后面我们在讲一下怎么中文
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412211305377.png)

如果想探索更多功能可参考：[https://github.com/open-webui/open-webui](https://github.com/open-webui/open-webui)
## 调试中文模型

### 第一种要求中文回复
只需要在我们问的时候，要求回复的时候，加上，请用中文回复
![](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/202412211317549.png)

### 自定义模型配置

```


```










