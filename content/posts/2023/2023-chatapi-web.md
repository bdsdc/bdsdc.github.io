# 前言介绍

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

### fork ChatGPT-Next-Web github 仓库

> 由于 Vercel 会默认为你创建一个新项目而不是 fork 本项目，这会导致无法正确地检测更新，项目方的*readm.md*文件有说明

> 所以我们需要先 fork 本项目到我们的自己的 github 仓库中

![](https://fastly.jsdelivr.net/gh/filess/img7@main/2023/04/21/1682057464464-879fa58c-9c12-4604-af2a-4cf02696423f.png)

创建 fork

![](https://fastly.jsdelivr.net/gh/filess/img13@main/2023/04/21/1682058001657-b993f6b3-e14f-4694-a792-2a388ced78d6.png)

### 通过 actions 启用 workflow 自动更新

切换到 `Actions`，点击`I understand my workflows, go ahead and enable them`按钮：

![](https://fastly.jsdelivr.net/gh/filess/img2@main/2023/04/21/1682058102489-f41ea53f-e94e-49a4-94e0-ef10df702dac.png)

切换到 `Upstream Sync`，点击 `Enable workflow`：

![](https://fastly.jsdelivr.net/gh/filess/img4@main/2023/04/21/1682058109825-651c665d-09c1-476c-a2b9-c341c6d57580.png)

显示如下代表成功
![](https://fastly.jsdelivr.net/gh/filess/img10@main/2023/04/21/1682058189191-3b13e1aa-0628-4f7f-bf56-8aa85da5f340.png)

## Vercel 部署

网站地址： https://vercel.com/

我们可以通过 github 来进行注册 vercel 网站，然后 vercel 能够读取 github 项目中的仓库，从而把仓库代码部署在 vercel 上

![](https://fastly.jsdelivr.net/gh/filess/img8@main/2023/04/21/1682058740440-e18d73d4-f01c-40a3-8e82-877b25e18298.png)

首先查看 vercel 网站绑定的 github 账户
![](https://fastly.jsdelivr.net/gh/filess/img9@main/2023/04/21/1682063356902-3d3e8694-8997-4d3f-93c1-2170038765c2.png)

新建project 项目
![](https://fastly.jsdelivr.net/gh/filess/img5@main/2023/04/21/1682063898604-c7fd7b82-0b8b-4e2d-9df3-74d5609699ae.png)


![](https://fastly.jsdelivr.net/gh/filess/img17@main/2023/04/21/1682063950527-87e36061-b50a-49c2-943d-34823f02a5c0.png)


然后目前已经识别到了 github 账号，然后 Fork 的代码仓库（ChatGPT-Next-Web），点击`import`
![](https://fastly.jsdelivr.net/gh/filess/img13@main/2023/04/21/1682063399617-55a3c98d-269e-43f4-b9b7-d63eacf6465a.png)

添加环境变量

- CODE： 建议填入，用户访问密码，想要给多少个用户分配访问码，需要用英文逗号分隔 `,` ,不要有空格
- OPENAI_API_KEY： 填写 chatapi 的 key ，前提是有了 chatgpt 账户，并且账户内有试用的额度，一般是 5$,用完了可以换另外的号

NAME： `CODE`  
VALUE: code1,code2,code3

NAME: `OPENAI_API_KEY`  
VALUE: xxxxxxxx

然后更改名字开始部署

![](https://fastly.jsdelivr.net/gh/filess/img3@main/2023/04/21/1682064010240-351fecc6-e8d6-4b00-ab71-bd642a54b810.png)

稍等个几分钟，正在部署，如下部署成功
![](https://fastly.jsdelivr.net/gh/filess/img5@main/2023/04/21/1682064036976-f8507875-8ee1-4d66-b89e-1b1bee57356b.png)

点击visit 就可以在线访问体验了 （但是现在还没配置自定义域名，还需要翻墙才能进去）
![](https://fastly.jsdelivr.net/gh/filess/img11@main/2023/04/21/1682064095381-e0546d94-e0bc-43c6-aeae-6c6ef65f1b00.png)

## 配置自定义域名

### 购买域名
〉如果现有域名，可以不需要购买，或者其他平台可以进行购买，我这里用的是纯数字域名，因为比较便宜

- 购买地址 ： https://name.com  ，注册和登录
- 搜索： 直接搜索想要的域名，最好是数字+xyz后缀，因为在搞活动，目前.xyz域名已经支持在工信部稳定
- 价格： 一般是1美金左右，如果比较贵的话，可以试试其他的数字
- 支付： 加入购物车，可以通过支付宝进行支付

### 网站注册

![](https://fastly.jsdelivr.net/gh/filess/img18@main/2023/04/21/1682068526825-2db8ae75-6087-4cd8-beb3-3f1ae320b50a.png)

### 搜索购买域名

![](https://fastly.jsdelivr.net/gh/filess/img16@main/2023/04/21/1682069022505-e3699fb1-ddd5-41d8-bc82-8d5bbe6304f5.png)

### 购物车提交
进入购物车界面，附加商品我们取消删除掉，然后我们就可以点击付款

![](https://fastly.jsdelivr.net/gh/filess/img17@main/2023/04/21/1682068574327-e422315f-bb41-4404-9ed6-2decb67d1be0.png)

### 付款支付

![](https://fastly.jsdelivr.net/gh/filess/img0@main/2023/04/21/1682068735660-5ebe0633-55f7-4164-aa74-3be19206ed4a.png)

付款之后，等待页面跳转。域名购买成功之后，在我的域名列表就会看到刚刚购买的域名，可以取消勾选自动续费：

![](https://fastly.jsdelivr.net/gh/filess/img0@main/2023/04/21/1682068958260-9ee099ff-5e31-44a3-bbca-9ecfa7ca240d.png)

### 进入详情页
点击这里进入域名详情页

![](https://fastly.jsdelivr.net/gh/filess/img16@main/2023/04/21/1682069215602-0a07cf8f-0f09-44a8-b65c-7a32f6498176.png)

## Cloudflare 配置

注册地址： https://www.cloudflare.com/zh-cn/
右上角可以选简体中文
登录进去以后，，点击`添加站点`

### 添加站点

![](https://fastly.jsdelivr.net/gh/filess/img13@main/2023/04/21/1682070611864-e74afd32-7d73-4519-8d1a-0f2405744d7b.png)

### 选择FREE免费计划

![](https://fastly.jsdelivr.net/gh/filess/img17@main/2023/04/21/1682070283762-bbd0d1a7-c7c1-4a60-ba84-462c87ebd523.png)

### 添加DNS记录
vercel.app因为被大量使用，自然而然被墙掉了，不过好在 Vercel 官方提供了单独的 IP 和 CNAME 地址给大家，对于国内的用户来说，配置一下单独的解析，依然可以享受 Vercel 提供的服务。
将上述步骤中用到的 ip和 cname地址替换成以下内容即可：

- A记录地址：76.223.126.88 或 76.76.21.98 等
- CNAME 记录地址：cname-china.vercel-dns.com

![](https://fastly.jsdelivr.net/gh/filess/img4@main/2023/04/21/1682070306683-e9aa9792-1c51-4539-9df8-85c636f36c99.png)
### 复制cloudflare名称服务器

![](https://fastly.jsdelivr.net/gh/filess/img11@main/2023/04/21/1682070604722-1471882e-e8c3-41b2-886e-811f284c972c.png)

### 修改name.com 域名详情页，点击	`管理域名服务器`

![](https://fastly.jsdelivr.net/gh/filess/img19@main/2023/04/21/1682070396157-e68d55d2-bcb4-4dd2-a1a6-03030d8db5e0.png)

### 替换cloudflare名称服务器和检测
大概需要半个小时左右

![](https://fastly.jsdelivr.net/gh/filess/img19@main/2023/04/21/1682070409857-219c5eec-a06e-4ff8-b440-c85d4f0ad2b6.png)

回到cloudflare界面，进行检测名称服务器
![](https://fastly.jsdelivr.net/gh/filess/img15@main/2023/04/21/1682072032235-54dbcfb0-a920-44cf-a78b-6bcc0687bcce.png)


### 状态正常完成

![](https://fastly.jsdelivr.net/gh/filess/img10@main/2023/04/21/1682072011017-25d4b7a6-3d4f-4761-9dc6-ca41ee635427.png)

状态更新后，域名服务器已经从name.com 转到 cloudflare来进行维护，DNS也指向了Vercel的服务器


## 应用域名
打开Vercel控制台，点击部署的项目，点击 Settings，侧边栏点击 Domains，把域名输入进去，点击 Add：


![](https://fastly.jsdelivr.net/gh/filess/img14@main/2023/04/21/1682073540044-30848122-7029-4752-8a4c-011e221f0805.png)



选择 Add xxxxx.xyz and redirect www.xxxxx.xyz to it，然后 Add：
![](https://fastly.jsdelivr.net/gh/filess/img4@main/2023/04/21/1682073492441-107f2ff2-8a18-4837-95a8-48c9fcbfbee7.png)

因为在上一步中，DNS 已经修改并应用成功，等待刷新一会（或者手动刷新），就会看到域名成功添加，也可以点击 Edit，将默认的域名删掉：

![](https://fastly.jsdelivr.net/gh/filess/img11@main/2023/04/21/1682073599252-bc81d405-439f-488a-a9be-6bba08a485dc.png)

此时就可以用新的域名访问啦~

![](https://fastly.jsdelivr.net/gh/filess/img0@main/2023/04/21/1682073688666-b56e4fa7-af88-4692-baac-0693aeb76d8a.png)

## ChatGPT 设置
网站可以访问后，需要进行一些简单的设置，如果你部署的时候加了访问码（CODE），填进去就可以了，其它的可以根据自己喜好设置：

![](https://fastly.jsdelivr.net/gh/filess/img13@main/2023/04/21/1682073772660-7e7abe08-24ba-4e67-8dc0-5c4b5d4a55e0.png)


## 补充
通过更改
![](https://fastly.jsdelivr.net/gh/filess/img6@main/2023/04/21/1682073929236-c45405ca-cbee-4441-94ec-a9a993789d94.png)












