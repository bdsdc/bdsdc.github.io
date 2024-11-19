![image](https://github.com/user-attachments/assets/8a49ac96-03d2-4678-901c-c860b6cf4593)# rustdesk部署




## 前言
正好想要一个可以远程访问windows电脑的软件，不想用windows自带的3389远程，可以跨网穿透访问，所以选择了rustdesk，开源免费还安全 

## 介绍
与 TeamViewer、ToDesk 等专有远程访问解决方案相比，RustDesk 作为一个开源软件，提供了几个显著的优势：

- RustDesk 完全免费使用，没有任何隐藏费用或订阅计划。
- 由于其开源特性，RustDesk 的代码是透明的，可以由社区审计，从而提供更高的安全性和可信度。
- RustDesk 使用 Rust 语言开发，从根本上确保了程序的内存安全和高性能。
- 跨平台，支持 Windows、macOS、Linux、Android 和 iOS 等多种操作系统。
- 支持远程唤醒机器，真正做到想连就连，无人值守

### rustdesk架构

由于国内原因，目前只能自建，要自建 RustDesk，首先要理解下 RustDesk 的架构。RustDesk 由三个组件组成：

- 客户端：运行在你的设备上（Windows，macOS，Linux，Android， iPhone）用于连接两个设备的软件，它负责监听来自客户端的连接请求，并在建立连接后向客户端发送屏幕更新和接收输入事件。
- 中继服务器（Relay Server）：运行在服务器上，充当客户端之间的桥梁，转发来自一方的数据包到另一方。在某些环境中（如经过 NAT 出网）设备之间无法进行 P2P 连接，可以用服务器来中转
- ID 服务器（ID Server）：运行在服务器上，用于维护客户端及中继服务器的连接信息，促进设备之间建立 P2P 连接。
值得一提的是，即使通过 Relay Server 进行通信，RustDesk 也会维护端到端加密，确保中继服务器无法访问明文数据。Relay Server 只是盲目地转发加密的数据包，而不能查看或修改其内容。

### 常用端口如下

- 21115是hbbs用作NAT类型测试
- 21116/UDP是hbbs用作ID注册与心跳服务 
- 21116/TCP是hbbs用作TCP打洞与连接服务 
- 21117是hbbr用作中继服务
- 21118和21119是为了支持网页客户端

## 环境准备
- 一台1c1g的centos7
- docker环境
现在云主机，一年100块钱， 腾讯和阿里以及华为，都有活动，还算是比较便宜，我们可以购买一台，有资源剩余，还可以部署其他平台玩玩


## 安装
安装hbbs 和 hbbr 俩个服务，我们通过docker来安装  
- hbbs: 对应前面说的 ID Server，用于分配和注册ID；
- hbbr: 对应前面说的Relay Server，如果P2P无法连接，会使用hbbr进行流量中继。

### docker环境安装
```shell
yum install docker docker-io -y
```
由于现在国内原因，docker源好多已经被ban，下面是最近可以正常访问的配置 
```shell
cat /etc/docker/daemon.json 
{
    "registry-mirrors": [
        "https://docker.1ms.run",
	"https://docker.fxxk.dedyn.io",
	"https://dockerpull.com"
    ]
    	
}
# systemctl daemon-reload
# systemctl start docker 
```
### hbbs
我们这里也可以指定key ，也可用自动生成的pub文件里面的key 
```shell
mkdir /data/rustdesk/{hbbs,hbbr}
cd /data/rustdesk/hbbs
sudo docker run --name hbbs -p 21115:21115 -p 21116:21116 -p 21116:21116/udp -p 21118:21118 -v `pwd`:/root -td --net=host rustdesk/rustdesk-server hbbs -k 123456

```
### hbbr

```shell
cd /data/rustdesk/hbbr
sudo docker run --name hbbr -p 21117:21117 -p 21119:21119 -v `pwd`:/root -td --net=host rustdesk/rustdesk-server hbbr

```
**据我所知，–net=host 仅适用于 Linux，它让 hbbs/hbbr 可以看到对方真实的ip, 而不是固定的容器ip (172.17.0.1)。 如果–net=host运行正常，-p选项就不起作用了, 可以去掉。**

### 确认服务启动
查看服务器是否都已经正常启动 
```shell
docker ps -a
[root@bdser hbbs]# docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED        STATUS                 PORTS                                         NAMES
79e5ae74c58a   rustdesk/rustdesk-server    "hbbs -k 123456"         2 hours ago    Up 2 hours                                                           hbbs
cf9659426c11   rustdesk/rustdesk-server    "hbbr"                   3 hours ago    Up 3 hours          
```

查看key 

第一种通过日志
```
[root@bdser hbbs]# docker logs -f hbbs
[2024-11-19 13:12:52.138240 +00:00] INFO [src/rendezvous_server.rs:1205] Key: 123456
[2024-11-19 13:12:52.138264 +00:00] INFO [src/peer.rs:84] DB_URL=./db_v2.sqlite3
[2024-11-19 13:12:52.139233 +00:00] INFO [src/rendezvous_server.rs:99] serial=0
[2024-11-19 13:12:52.139249 +00:00] INFO [src/common.rs:46] rendezvous-servers=[]
[2024-11-19 13:12:52.139253 +00:00] INFO [src/rendezvous_server.rs:101] Listening on tcp/udp :21116
[2024-11-19 13:12:52.139256 +00:00] INFO [src/rendezvous_server.rs:102] Listening on tcp :21115, extra port for NAT test
[2024-11-19 13:12:52.139259 +00:00] INFO [src/rendezvous_server.rs:103] Listening on websocket :21118
[2024-11-19 13:12:52.139284 +00:00] INFO [libs/hbb_common/src/udp.rs:36] Receive buf size of udp [::]:21116: Ok(212992)
[2024-11-19 13:12:52.139324 +00:00] INFO [src/rendezvous_server.rs:138] mask: None
[2024-11-19 13:12:52.139327 +00:00] INFO [src/rendezvous_server.rs:139] local-ip: ""
[2024-11-19 13:12:52.139333 +00:00] INFO [src/common.rs:46] relay-servers=[]
[2024-11-19 13:12:52.139378 +00:00] INFO [src/rendezvous_server.rs:153] ALWAYS_USE_RELAY=N
[2024-11-19 13:12:52.139401 +00:00] INFO [src/rendezvous_server.rs:185] Start
[2024-11-19 13:12:52.139440 +00:00] INFO [libs/hbb_common/src/udp.rs:36] Receive buf size of udp [::]:0: Ok(212992)


```
第二种通过查看pub文件
```shell
[root@bdser hbbs]# cat /data/rustdesk/hbbs/id_ed25519.pub 
```
## 演示
下载window客户端地址: 


## 总结


