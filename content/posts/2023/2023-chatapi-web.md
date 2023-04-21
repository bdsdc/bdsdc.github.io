---
title: "道草|轻松打造免费专属的ChatGPT网站给朋友"
date: 2023-04-21
author: 'bdsdc'
linkToMarkdown: true
categories: ['AI']
tags: ['ChatGPT']
---
# 前言介绍

在中国大陆境内域名提供商购买的域名，需要实名认证才能开启 DNS 解析。而在国外域名提供商买的域名则不用。
指向中国大陆内的云服务器的域名提供的网站，必须备案才能正常被访问。但是任何指向国外服务器的域名却可以不受此规则的限制。可以利用这一点，绕过 __ 的限制。
Vercel 是一个无需云服务器即可快速部署现代 Web 应用程序的平台。
Vercel. App 本身是被墙了的网站，但是它的服务器 IP 却没有被墙。可以利用这点，让你的域名直接指向 Vercel 服务器，从而访问 Vercel 提供的服务。
此文章采用 github 的 Yidadaa/ChatGPT-Next-Web 项目来搭建 ChatGPT 网页服务

- 使用 vercel 一键部署 chatGPT 服务，项目使用的 Github 上的 ChatGPT-Next-Web 或者 chatbat-ui 项目
- 申请一个域名，然后通过 cloudflare 为该域名设置一个 CNAME，在 vercel 启用该域名访问

# 前期准备

- 需要一个 GitHub 账户，正常登录 GIthub
- 需要一个 Vercel 账户，正常登录 Vercel
- 需要一个 Cloudflare 账户，正常登录 Cloudflare
- 需要一个 ChatGPT 账户，正常登录 ChatGPT
- 需要一个免费域名（或者便宜的收费域名）
- 需要一个访问外面的工具（可有可无）

## GitHub 配置

项目地址：[github](https://github.com/Yidadaa/ChatGPT-Next-Web/blob/main/README_CN.md)

### Fork ChatGPT-Next-Web github 仓库

> 由于 Vercel 会默认为你创建一个新项目而不是 fork 本项目，这会导致无法正确地检测更新，项目方的*readm. Md*文件有说明

> 所以我们需要先 fork 本项目到我们的自己的 github 仓库中

![](https://fastly.jsdelivr.net/gh/filess/img7@main/2023/04/21/1682057464464-879fa58c-9c12-4604-af2a-4cf02696423f.png)

创建 fork

![](https://fastly.jsdelivr.net/gh/filess/img13@main/2023/04/21/1682058001657-b993f6b3-e14f-4694-a792-2a388ced78d6.png)

### 通过 actions 启用 workflow 自动更新

切换到 `Actions`，点击 `I understand my workflows, go ahead and enable them` 按钮：

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421202004.png)

切换到 `Upstream Sync`，点击 `Enable workflow`：
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421202018.png)


显示如下代表成功
本网页上左上角有 `success`

## Vercel 部署

网站地址： https://vercel.com/

我们可以通过邮件进行注册 vercel 网站，然后 vercel 绑定 github 账户，这样能够读取 github 项目中的仓库，从而把仓库代码部署在 vercel 上
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421202803.png)


首先查看 vercel 网站绑定的 github 账户
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421202820.png)

在 vercel 上进入 `dashboard`
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421202912.png)


新建 ` project ` 项目
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421202936.png)


然后目前已经识别到了 github 账号，然后找到 Fork 的代码仓库（ChatGPT-Next-Web），点击 `import`
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421203102.png)


添加环境变量（需要填入下面俩个变量）

- CODE： 建议填入，用户访问密码，想要给多少个用户分配访问码，需要用英文逗号分隔 `,` ,不要有空格
- OPENAI_API_KEY： 填写 chatapi 的 key ，前提是有了 chatgpt 账户，并且账户内有试用的额度，一般是 5$, 用完了可以换另外的号

范例如下

NAME： `CODE`  
VALUE: code 1, code 2, code 3

NAME: `OPENAI_API_KEY`  
VALUE: xxxxxxxx

然后更改名字，输入环境变量，开始准备部署 `deploy`

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421203952.png)


需要稍等个几分钟，正在部署，如下部署成功, 然后点击 `continue to daashboard`
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204105.png)


点击 `visit` 就可以在线访问体验了 （但是现在还没配置自定义域名，还需要翻墙才能进去）
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204058.png)


## 配置自定义域名

### 域名说明
> 如果现有域名，可以不需要购买，或者其他平台可以进行购买，我这里用的是纯数字域名，因为比较便宜，免费域名不确定的因素可能会比较多，花点小钱吧，还可以忍受

### 域名购买流程
- 购买地址 ： https://name.com  ，注册和登录 (国外老牌域名注册商，纯数字域名比较便宜)
- 搜索： 直接搜索想要的域名，最好是数字+xyz 后缀，因为在搞活动便宜，目前. xyz 域名已经支持在工信部稳定
- 价格： 一般是 1 美金用一年，如果比较贵的话，可以试试其他的数字
- 支付： 加入购物车，可以通过支付宝进行支付

### 网站注册

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204552.png)


### 搜索购买域名

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204706.png)



### 购物车提交
进入购物车界面，附加商品我们取消删除掉，然后我们就可以点击付款
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204812.png)


### 付款支付

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204746.png)


付款之后，等待页面跳转。域名购买成功之后，在我的域名列表就会看到刚刚购买的域名，可以取消勾选自动续费：

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204935.png)


### 进入详情页
点击这里进入域名详情页

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421204955.png)


## Cloudflare 配置

注册地址：<https://www.cloudflare.com/zh-cn>
右上角可以选简体中文
登录进去以后，，点击 `添加站点`

### 添加站点

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205031.png)


### 选择 FREE 免费计划

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205055.png)


### 添加 DNS 记录

Vercel. App 因为被大量使用，自然而然被墙掉了，不过好在 Vercel 官方提供了单独的 IP 和 CNAME 地址给大家，对于国内的用户来说，配置一下单独的解析，依然可以享受 Vercel 提供的服务。将上述步骤中用到的 ip 和 cname 地址替换成以下内容即可：

- A 记录地址：76.223.126.88 或 76.76.21.98 等
- CNAME 记录地址：cname-china.vercel-dns.com

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205129.png)

### 复制 cloudflare 名称服务器

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205150.png)


### 进入name. Com 域名详情页，点击	`管理域名服务器`
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205230.png)


### 替换 cloudflare 名称服务器和检测
把原来的四个地址删除掉，把刚刚提示俩个需要复制的 cloudflare 名称服务器地址，替换到这里，
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205449.png)


回到 cloudflare 界面，进行检测名称服务器，大概需要半个小时
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205459.png)


### 状态正常完成
我们进入 cloudlfare，点击网站，看到主页显示网站有效

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205538.png)


**状态更新后，域名服务器已经从 name.com 转到由 cloudflare 来进行维护，DNS 也指向了 Vercel 的服务器**


## 使用购买的域名
打开 Vercel 控制台，点击部署的项目，点击 Settings，侧边栏点击 Domains，把域名输入进去，点击 Add：


![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205641.png)




选择 Add xxxxx. Xyz and redirect www.xxxxx.xyz to it，然后 Add：
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205650.png)


因为在上一步中，DNS 已经修改并应用成功，等待刷新一会（或者手动刷新），就会看到域名成功添加，也可以点击 Edit，将默认的域名也可以删掉：

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205727.png)


此时就可以用新的域名访问啦~

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205803.png)


## ChatGPT 设置
网站可以访问后，需要进行一些简单的设置，如果你部署的时候加了访问码（CODE），填进去就可以了，其它的可以根据自己喜好设置：
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205926.png)
丰富的 prompt 提示
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421205948.png)


## 补充更换 code 或者 api key
> 配置更改后，我们需要重新进行部署应用。

打开 vercel dashboard 页面，选择 setting ，选择 environment variables 

![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421210109.png)

通过编辑来进行更改 Value 的值 
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421210325.png)

我们更改好配置后，肯定是需要重新发布一下，让网站应用我们的最新配置，针对最新的部署记录进行`Redeploy`
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421210706.png)

这里我们选择使用代码缓存构建，不更新代码，只更新我们变更的配置，部署更快，点击 `Redeploy `
![image.png](https://bdsblog.oss-cn-shanghai.aliyuncs.com/blog/20230421210559.png)

`over`
