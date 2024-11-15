---
title: "运维| mkdocs文档博客搭建"
date: 2024-11-14
lastmod: 2024-11-14
author: 'bdsdc'
linkToMarkdown: true
categories: ['web']
tags: ['blog']
comments: true 
---
## 前言
- centos7 系统
- mkdocs
- material 主题

## 介绍

kDocs 可以同时编译多个 markdown 文件，形成书籍一样的文件。有多种主题供你选择，很适合项目使用。

MkDocs 是快速，简单和华丽的静态网站生成器，可以构建项目文档。文档源文件在 Markdown 编写，使用单个 YAML 配置文件配置。

MkDocs 是一款快速、简单且极其出色的静态网站生成器，专门用于构建项目文档。文档源文件以 Markdown 编写，并使用单个 YAML 配置文件进行配置。它易于使用，并且可以使用第三方主题、插件和 Markdown 扩展进行扩展。
## 环境
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
[root@bdser mkdocs]# tree -L 3 | head -n 10 
.
├── docs
│   ├── index.md
│   └── overrides
│       └── partials
├── mkdocs.yml

```
#### gisus绑定github
建议github仓库中，新建库，选择公开
[https://github.com/apps/giscus](https://github.com/apps/giscus)

然后访问绑定仓库 - settings - Discussions（勾选启用），本仓库 Discussions 添加成功

#### comments.html代码
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


#### 生成giscus评论配置
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
#### 评论生效
每次编写markdown博客的时候，我们需要再开头编写元信息，需要加入如下元数据说明

```
---
title: 留言板 
comments: true #开启评论
---
```
### 图片放大
安装插件glightbox
```
pip install mkdocs-glightbox -i  https://pypi.tuna.tsinghua.edu.cn/simple/
```
配置mkdocs.yml
```
plugins:
  - search
  - glightbox   #加载插件
  - tags:
      tags_file: tags.md
```
重新启动，生效
### 添加TAG标签

```yaml title=mkdocs.yml 
plugins:
  - search
  - glightbox    
  - tags:                   #增加如下俩行
      tags_file: tags.md
......

nav:
  - 标签: tags.md 
```
把下面复制到tags.md文件中
```shell title=docs/tags.md 
# Tags

Following is a list of relevant tags:

<!-- material/tags -->
```
### 雪花效果

```yaml title=mkdocs.yml
extra_javascript:
  - https://vercount.one/js
  - javascript/extra.js    #添加脚本

```
脚本如下
如果`docs`,没有Javascript目录，可以新建 
```shell
mkdir /docs/javascript
## js代码
//雪花
const fps = 30;
const mspf = Math.floor(1000 / fps) ; 

let width = window.innerWidth || document.documentElement.clientWidth;
let height = window.innerHeight || document.documentElement.clientHeight;
let canvas;
window.addEventListener('resize', () => {
  width = window.innerWidth || document.documentElement.clientWidth;
  height = window.innerHeight || document.documentElement.clientHeight;
  if (canvas) {
    canvas.width = width;
    canvas.height = height;
  }
});

let particles = [];
let wind = [0, 0];
let cursor = [0, 0];

function velocity(r) {
  return 70 / r + 30;
}

function sine_component(h, a) {
  return [2 * Math.PI / h, Math.random() * a, Math.random() * 2 * Math.PI]; // [frequency, amplitude, phase]
}

function calc_sine(components, x) {
  let sum = 0;
  for (let i = 0; i < components.length; i++) {
    const [f, a, p] = components[i];
    sum += Math.sin(x * f + p) * a;
  }
  return sum;
}

function gen_particle() {
  let r = Math.random() * 4 + 1;
  return {
    radius: r,
    x: Math.random() * width,
    y: -r,
    opacity: Math.random(),
    sine_components: [sine_component(height, 3), sine_component(height / 2, 2), sine_component(height / 5, 1), sine_component(height / 10, 0.5)],
  };
}

function update_pos(dt) {
  const n = particles.length;
  for (let i = 0; i < n; i++) {
    const v = velocity(particles[i].radius);
    particles[i].x += calc_sine(particles[i].sine_components, particles[i].y) * v / 5 * dt;
    particles[i].y += v * dt;

    // const dist = Math.hypot(particles[i].x - cursor[0], particles[i].y - cursor[1]) + 1;
    // particles[i].x += wind[0] * dt / dist
    // particles[i].y += wind[1] * dt / dist;

    if (particles[i].y - particles[i].radius > height) {
      particles[i] = gen_particle();  
    }
  }
}

let context_cache;
function get_context() {
  if (context_cache)
    return context_cache;

  canvas = document.createElement('canvas');
  canvas.id = 'snow-canvas';
  canvas.width = width;
  canvas.height = height;
  canvas.style = 'position: fixed; top: 0; left: 0; overflow: hidden; pointer-events: none; z-index: 256;';
  if ((document.documentElement.dataset.darkreaderMode || "").startsWith('filter'))
    canvas.style.filter = 'invert(1)';
  document.body.appendChild(canvas);

  context_cache = canvas.getContext('2d');
  return context_cache;
}

function draw() {
  const ctx = get_context();

  ctx.clearRect(0, 0, width, height);

  const n = particles.length;
  for (let i = 0; i < n; i++) {
    const p = particles[i];
    ctx.fillStyle = `rgba(255, 255, 255, ${p.opacity})`;
    ctx.shadowColor = '#80EDF7';
    ctx.shadowBlur = 7;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.radius, 0, 2*Math.PI);
    ctx.fill();
  }
}

let focused = true;
let disabled = false;
let lastTime = performance.now();
const requestFrame = () => setTimeout(loop, mspf);
function loop() {
  const dt = (performance.now() - lastTime) / 1000;

  if (particles.length < 120 && Math.random() < 0.1) {
    particles.push(gen_particle());
  }

  update_pos(dt);
  draw();

  lastTime = performance.now();
  if (focused && !disabled)
    requestFrame();
}


window.addEventListener('focus', () => {
  console.log('snow start');
  focused = true;
  lastTime = performance.now();
  requestFrame();
});

window.addEventListener('blur', () => {
  console.log('snow stop');
  focused = false;
});

window.addEventListener('keydown', e => {
  if (e.ctrlKey && e.key == 's') {
    e.preventDefault();
    disabled = !disabled;
    if (disabled) {
      canvas.style.display = 'none';
    } else {
      canvas.style.display = 'block';
      lastTime = performance.now();
      requestFrame();
    }
  }
});

requestFrame();
//雪花

```

### 添加404页面
docs/overrides文件下新建404.html即可

#### 树形结构
```
root@DESKTOP-CK75KU2:/mnt/d/mkdocs# tree -L 3 docs/overrides/ 
docs/overrides/
├── 404.html
└── partials
    └── comments.html

```
#### html代码

```
<html>
 <head> 
  <meta charset="UTF-8" /> 
  <meta name="description" content="公益404页面是由腾讯公司员工志愿者自主发起的互联网公益活动。" /> 
  <link rel="icon" href="data:image/gif;base64,R0lGODdhIAAgANUAAAAAAAgFBgYICAwMDBAPDxAQDxQTFBUYFxcaGRwcHCQkJCQoJykqKTQ0ND09PUJCQktMTFZWVltcXF1hYGNjY2doaGpqanNzc3d5eHp6eoODg4uLi5eXl5mamqOjo62tra+wr7S0tLe5uLu7u7/AwMPEw83NzdbX1tfa2dra2uTk5Ovr6+/w7/T09Pj39/f4+P///wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACH5BAkAADEALAAAAAAgACAAAAb/QJhwOGyZOpIGQ6FgNCQcU4tIrcJaIwlgy+1uJaOplZqydBOSzGaTkSS6lNN4ONoOAJRQajV0rVAgFF0jcyEAbw9hc1cjD4cAIVZ1DAAXKotEKxkAlIREJ48bYphFGwAKAHJCLWaVo6RFF1sWYnUDDpewVSoOd4QtWgCeMCmXLaqfUyopdF9GWxRiKQAVKh3CRHUdKhQEzFeCAEdbH6uOh28A3ypbCQhbEGIfWx2CCd8wJhB2CpFDIQvuAIBgYkiKN0kASOAzRJYjBwyFrGgAwNGFTFqUAMjwYogpA1woMFwRDgDIDUQ2LQGAcoiGLRr2QRgp8yUADURMMWFJRYMH3BglJOQSwq0EDA84c56ilOGVLisqKUaISKVFihIjRJRI4XTIiggAGggygKxICAoguSDI0/UEyAnXAJTLJMtLlwxUYcwD0MEEtIgtBKGi0OHDh3qn8IwKvEVKuGHXGAwI8apFCAOUOvxzBqNOAVzgthi1UgKasl7YrtS9MMXI6DEjpKieNWoaSA1dF7V4+QafkNKoLOlSIQsVCUkmATgYwWIRCxEOAByANKc0AAEAJnzYI6bFihQfBAUQjSlF3S0GIqjZgCFC2i0XfM/BUtJuSEVPjdRTwsQJYdljBAEAOw==" /> 
  <title>404 您访问的页面搞丢了</title> 
  <script src="https%3A//volunteer.cdn-go.cn/404/latest/404.js" rendertarget="404DlV"></script>
 </head> 
 <body style="overflow-x: hidden; max-width: 100vw;">
  <div style="margin:0px;padding:0px;background-color:rbga(0,0,0,0);">
   <div style="position:relative;left:50%;margin-left:-800px;width:1600px;height:1000px;">
    <img alt="404！您要访问的页面走丢了！" src="https://volunteer.cdn-go.cn/404/latest/img/dream4school.jpg" style="left:50%;width:1600px;margin-left:calc(50% - 800px);filter: brightness(75%);" />
    <div style="position:relative;margin-top:-1200px;width:100%;text-align:center;color:white;font-size:28px;">
     <div style="display:inline-block;width:98vw;max-width:1600px;height:calc(100vh - 250px);max-height:750px;">
      <div style="font-size:128px;font-weight: 800;">
       404
      </div>您访问的页面走丢在寻找梦想的路上了
      <br />不过您还可以和腾讯志愿者一起
      <br />
      <i><b style="font-size:1.2em">为孩子们点亮一个梦想</b></i>
     </div>
     <div style="display:inline-block;width:98vw;max-width:1600px;text-align:left;margin:0px 10px;font-size:0.75em">
      <a href="https://volunteer.cdn-go.cn/404/latest/img/dream4schoolQR.png"><img src="https://volunteer.cdn-go.cn/404/latest/img/dream4schoolQR.png" alt="点击进入支持页面" style="width:160px;" /></a>
      <br />扫码点亮一个梦想
      <div style="float:right;text-align:right;margin-right:1em;margin-top:-4.5em">
       <br>照片拍摄于湖南省岳阳市平江县三市镇新村完小
       <br>拍摄时间：二零二三年七月十一日
       <br/>（感恩基金会供稿）
       <br><a href="https://support.qq.com/products/378306" style="color:lightgray;font-size:0.8em;">我要反馈</a> 
      </div>
     </div>
    </div>
   </div>
  </div>
 </body>
</html>

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
### 访问
网页访问: [https://wiki.budongshu.cn](https://wiki.budongshu.cn)


