---
title: "道草|白嫖GPTs和Dalle3，免费体验要趁早"
date: 2023-12-18
author: 'bdsdc'
linkToMarkdown: true
categories: ['AI']
tags: ['ChatGPT']
---
# 前言
Coze是由字节跳动在海外推出的一个AL聊天机器人和应用程序编辑开发平台，可以理解为字节跳动版的GPTS。无论用户是否有编程经验，都可以通过该平台快速创建各种类型的聊天机器人、智能体、A应用和插件，并将其部署在社交平台和即时聊天应用程序中，如Discord、WhatsApp、Twitter。
有意思的地方在于，目前Coze提供的是基于OpenAl GPT-4和GPT-3.5的AP来创建和使用A聊天机器人，并未使用自研的云雀大模型，而此前媒体报道字节将于12月底推出一个开放平台并开启公测，允许用户自主创建自定义聊天机器人。如同此前推出的聊天机器人豆包国际版为Cic，后续字节可能推出一个国内版
所以我们现在可以趁早来免费体验

## 前期条件
- discord账号（访问需要科学上网，前面我们有介绍过哦，在玩midjourney的时候介绍过）
- 注册下coze网站
- 准备一个GPTs机器人的产品思路

## 开始coze网站之旅
这里是重点哦 ，主要是coze这个产品提供的功能，我们也可以参考文章的文档来操作，本文也是基于各种文档参考操作成功的

### 注册coze
网站： https://www.coze.com/docs/zh_cn/welcome.html
首先注册一下网站，我这里通过google邮箱进行注册的，正好我有google邮箱，如果前面买了wildcard的同学应该也会有谷歌邮箱，不过也可以是电话
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/aa92b54f-efdb-4124-8c2c-1b2c498ae9e9)
### 创建机器人
这是目前我创建的机器人
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/eb3d5d65-9220-4daf-9335-2fefb1fc0442)
然后根据我们之前打造GPTs的经验，可以创建一个机器人产品，如果不清楚的，可以去破解别人的思路，或者还可以参考官方coze的explore
网站：https://www.coze.com/explore
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/b39867a5-15bb-4503-a0b5-66234ab096e7)
### 打造个性产品
支持的功能目前看，比chatgpt灵活很多，自定义配置也很灵活，比chatgpt好一点的就是不需要额外编程也可以
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/c1f4b506-26a3-4319-ae52-0327a90f056c)
翻译了一下，大家参考看一下功能
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/661583f7-88a9-42e3-94d9-8bcdc9ca7b08)
### coze 对接discord 
下面我们就需要分别登录进去操作了，涉及到要来回切换，大致的配置流程顺序如下
- 在coze界面，我们选择对接 discord，然后进入配置界面
- 在登录discord中找到一个token ，一个public key，然后填入配置中
- 在coze界面，进行Publish发布，主页bots机器人界面，会显示绿色，已经发布 （这个一定要先发布，下一条才可以配置成功，要不然一直会报错）
- 然后在discord中配置一个coze提供的url地址，填入进去，这样discord就可以调用coze了
- 然后在discord中通过/start启动机器人，通过@机器人名字 方式进行会话聊天了
#### 进入coze，选择discord配置
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/7548a7c1-c8ad-43f4-a300-ebd8017d8f7a)
#### 进入discord，找到token和public key
首先进入 https://discord.com/developers/applications 这里建立discord的机器人
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/736210db-26e5-4b2a-a73c-a3e18588904a)
随便定义名字
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/a5723dbf-2efc-4f5e-84d6-a936a009c679)
找到public key
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/690073f7-44de-46b0-8483-e369b50b9287)
配置一下discord机器人 ，按照图中进行选择配置，里面有token，进行生成一个token 
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/2a36a2d2-e1af-4262-9f03-7bbd0dea2f47)
#### 发布coze 
复制下面url，后面会用到
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/23ae9477-d450-420d-9e86-c1aab9c25d48)
发布coze
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/dae12926-9f2c-4d35-8fa0-4d616e732df5)
看下coze状态，绿色代表正常
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/1b4d0940-c908-470c-92b5-79dbdf0f8c6f)
#### 配置discord连接coze 
上面复制的url ，粘贴到discord中，如下
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/3fb8f61a-fda3-4943-8ccc-3932c047bf0f)
然后配置discord 机器人授予管理员权限
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/1351f91d-b7e7-4a6b-aee1-39ad947e05fd)
## 在discord 开始使用
![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/d88374ed-5253-4797-926a-12a14444e462)

![image](https://github.com/bdsdc/bdsdc.github.io/assets/129520887/875f3f61-8d24-40dd-8035-395e90d7e670)





