---
title: "运维| mkdocs文档博客搭建"
date: 2024-11-06
lastmod: 2024-11-06
author: 'bdsdc'
linkToMarkdown: true
categories: ['web']
tags: ['blog']
---
## 前言
- centos7 系统
- mkdocs
- material 主题

## 介绍

kDocs 可以同时编译多个 markdown 文件，形成书籍一样的文件。有多种主题供你选择，很适合项目使用。

MkDocs 是快速，简单和华丽的静态网站生成器，可以构建项目文档。文档源文件在 Markdown 编写，使用单个 YAML 配置文件配置。

MkDocs 是一款快速、简单且极其出色的静态网站生成器，专门用于构建项目文档。文档源文件以 Markdown 编写，并使用单个 YAML 配置文件进行配置。它易于使用，并且可以使用第三方主题、插件和 Markdown 扩展进行扩展。
## 安装


### pip3 
现在都在使用python3，先安装pip3 
```
python --version
yum install python3-pip
```

## 安装
```
python3 -m pip install mkdocs  -i https://pypi.tuna.tsinghua.edu.cn/simple/
# 先选择好目录，我这里把博客文档放到/data/blog下面，根据自己要求更改
# 生成项目文档
cd /data/blog
python3 -m mkdocs new . 
```
## 开始
```
[root@bdser mkdocs]# python3 -m mkdocs new . 
INFO     -  Writing config file: ./mkdocs.yml
INFO     -  Writing initial docs: ./docs/index.md
[root@bdser mkdocs]# tree -L 2 
.
├── docs
│   └── index.md
├── mkdocs.yml
## 启动，命令后可以自定义监听端口
[root@bdser mkdocs]# python3 -m mkdocs serve -a 0.0.0.0:8001
INFO     -  Building documentation...
WARNING  -  Config value: 'dev_addr'. Warning: The use of the IP address '0.0.0.0' suggests a production environment or the use of a proxy to
            connect to the MkDocs server. However, the MkDocs' server is intended for local development purposes only. Please use a third party
            production-ready server instead.
INFO     -  Cleaning site directory
INFO     -  Documentation built in 0.11 seconds
INFO     -  [15:11:22] Serving on http://0.0.0.0:8001/
```
<补图>

简单修改配置,在启动，就可以看到最新配置过的界面了
```
site_name: My Docs
nav:
  - Home: index.md
  - About: about.md
```

## 配置
> 下面我们重点讲解优化mkdocs.yaml配置,在mkdocs.yml中配置站点名称、描述、作者、ur、导航菜单等信息等。

基础配置如下：
- site name:站点名称
- site ur:站点 URL链接
- site author:站点作者
- site description:站点描述
- copyright:版权信息
- repo ur:站点仓库 URL
- nav: 站点导航
- theme: 站点主题
- markdown extensions:Markdown扩展

### 主题
```
python3 -m pip   install mkdocs-material  -i https://pypi.tuna.tsinghua.edu.cn/simple/ 
```
```
# 编辑mkdocs.yml
# theme: material 
```

### 谷歌分析
谷歌分析地址： [https://analytics.google.com/](https://analytics.google.com/)
先注册账号，可能需要谷歌邮箱注册，成功登录后账号后，新建媒体资源，媒体资源就是你的网站 

`媒体资源`-`媒体收集和分析`-`选择并打开数据流`-`衡量ID`,把这个ID复制到配置中 
```
google_analytics:
  - 'UA-XXXXXXXX-X'
  - 'auto'
```
### 评论系统
推荐giscus
利用 GitHub Discussions 实现的评论系统，让访客借助 GitHub 在你的网站上留下评论和反应吧！本项目深受 utterances 的启发。

- 开源。🌏
- 无跟踪，无广告，永久免费。📡 🚫
- 无需数据库。所有数据均储存在 GitHub Discussions 中。:octocat:
- 支持自定义主题！🌗
- 支持多种语言。🌐
- 高可配置性。🔧
- 自动从 GitHub 拉取新评论与编辑。🔃
- 可自建服务！🤳
在mkdocs添加配置
```
theme:
  name: material
  custom_dir: docs/overrides  #主要是这一行

```
参考下图新建overrides文件，结构图如下：
```
tree -a
```
### gisus绑定github
建议github仓库中，新建库，选择公开
[https://github.com/apps/giscus](https://github.com/apps/giscus)

然后访问绑定仓库 - settings - Discussions（勾选启用），本仓库 Discussions 添加成功

### comments.html代码
```
{% if page.meta.comments %}
  <h2 id="__comments">{{ lang.t("meta.comments") }}</h2>


  <!-- Insert generated snippet here 高亮这里 --> 
  <script src="https://giscus.app/client.js"
  data-repo="你的仓库名称（如Wcowin/hexo-site-comments）"
  data-repo-id=" "
  data-category=" "
  data-category-id=" "
  data-mapping="pathname"
  data-strict="0"
  data-reactions-enabled="1"
  data-emit-metadata="0"
  data-input-position="bottom"
  data-theme="preferred_color_scheme"
  data-lang="zh-CN"
  crossorigin="anonymous"
  async>


</script>
  <!-- Synchronize Giscus theme with palette -->
  <script>
    var giscus = document.querySelector("script[src*=giscus]")

    // Set palette on initial load
    var palette = __md_get("__palette")
    if (palette && typeof palette.color === "object") {
      var theme = palette.color.scheme === "slate"
        ? "transparent_dark"
        : "light"

      // Instruct Giscus to set theme
      giscus.setAttribute("data-theme", theme) 
    }

    // Register event handlers after documented loaded
    document.addEventListener("DOMContentLoaded", function() {
      var ref = document.querySelector("[data-md-component=palette]")
      ref.addEventListener("change", function() {
        var palette = __md_get("__palette")
        if (palette && typeof palette.color === "object") {
          var theme = palette.color.scheme === "slate"
            ? "transparent_dark"
            : "light"

          // Instruct Giscus to change theme
          var frame = document.querySelector(".giscus-frame")
          frame.contentWindow.postMessage(
            { giscus: { setConfig: { theme } } },
            "https://giscus.app"
          )
        }
      })
    })
  </script>
{% endif %}
```


### 生成giscus评论配置
打开https://giscus.app/zh-CN 走完这个页面的流程就会得到(这会在你的Github创建新的仓库，建议自己先去新建个 Discussions)
```
<script src="https://giscus.app/client.js"
        data-repo="[在此输入仓库]"
        data-repo-id="[在此输入仓库 ID]"
        data-category="[在此输入分类名]"
        data-category-id="[在此输入分类 ID]"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
```
复制将此代码，替换最上面👆🏻comments.html中高亮的代码
### 评论生效
每次编写markdown博客的时候，我们需要再开头编写元信息，需要加入如下元数据说明
```
---
title: 留言板 
comments: true #开启评论
---

```

## 生成网页站点

### 启动服务
```
在 mkdocs.yml 同一路径下，使用以下命令启动服务：
python3 -m mkdocs serve -a 0.0.0.0:8001

```
### 生成网页
在 mkdocs.yml 同一路径下，使用以下命令依据 mkdocs.yml 配置文件，生成网页：
```
mkdocs build
```
成功后，将在项目文件夹下生成一个名为 site 的文件夹，其中包含了所有生成的网页。

将 site 文件夹下的所有文件拷贝到项目根目录下，与 docs文件夹及其包含的文本，共同组成网页资源。

当我们部署网页时，只需要将上述网页资源进行部署即可（忽略 mkdocs.yml 和 site 文件夹）
### 拷贝远程服务器
```
scp -rp /data/mkdocs/site  root@10.x.x.x:/data/mkdocs/site   
```
### 远程服务器nginx
通过certbot生成了ssl证书，本站写过文档，可以参考下

下面是nginx配置，/data/mkdocs/site: 网站目录，可以自定义，启动nginx，可以正常访问 

```
server {
  charset utf-8;
  server_name wiki.budongshu.cn;  # 改成你的 IP
  access_log         /data/nginx/logs/wiki.budongshu.cn-access.log;

  location / {
     root /data/mkdocs/site; 
     index index.html; 
     try_files $uri  $uri/ /index.html; 

  }
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/wiki.budongshu.cn/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/wiki.budongshu.cn/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = wiki.budongshu.cn) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
  listen 80;
  server_name wiki.budongshu.cn;
    return 404; # managed by Certbot


}

```



