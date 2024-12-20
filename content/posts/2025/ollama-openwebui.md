

ollama是一个开源项目，它提供了一个平台和工具集，用于部署和运行各种大型语言模型。 


## ollama部署
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
## 本地模型启动
启动本地大模型，这个步骤会下载llama3模型，根据个人带宽网速预估时间
```
docker exec -it ollama ollama run llama3
```
## 交互对话 
执行完毕后，会进入交互模式，输入内容，就可以在线对话了

openweb ui

github: []()

