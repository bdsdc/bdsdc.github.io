# 目前国内大模型下载方法

## 虚拟环境
```
python3 -m venv huggingface-venv
# 激活环境
source huggingface-venv/bin/activate
```

## 安装 huggingface-clit 

```
python3  -m pip install huggingface-hub --index-url=https://mirrors.aliyun.com/pypi/simple
```

## 下载模型
```
export HF_ENDPOINT=https://hf-mirror.com
huggingface-cli download   immich-app/XLM-Roberta-Large-Vit-B-16Plus
```


