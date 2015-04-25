---
layout: post
title: "run own ngrokd server"
description: ""
category: Linux
tags: [Linux ngrok]
summary: ngrok.com所在的ip地址在国内已被封，搭建自己的ngrokd服务更靠谱。
---
{% include JB/setup %}

**Overview**

{{ page.summary }}

#ngrok是什么

实际上，我们经常面临这样的需求，在进行Web开发等工作时，需要将本地内网中部署的网站能够直接被外网访问到；在集群中工作时，登录内网节点需要以具备外网地址的登录节点为跳板，向内网节点传送本地文件时，往往需要以登录节点为中转，或者只能从内网拉取本地的文件。有没有更简便的方法呢？防火墙上的端口映射是一种，但这个方法面临着权限等要求。[ngrok](https://ngrok.com)同样适合于这种工作，并且更为方便。ngrok是一个反向代理工具，可以提供在外网中安全访问内网Web服务的方法，并且可以捕获所有请求的http内容，可以支持TCP层的端口映射……以下以ngrok 1.x 版本为例。

#常用示例

ngrok下载即可用。在shell中执行`./ngrok 80`择可以为本地的80端口提供一个ngrok.com的随机二级域名以供外网访问，在ngrok.com注册的用户还可以自定义一个没有被用过的二级域名。

注册用户支持tcp端口转发，例如`./ngrok -proto=tcp 22`则可以将22号（sshd服务的默认端口）上的tcp服务转发到ngrok.com分配的一个随机端口上。

……

#被墙了

可惜15年初，ngrok的官方域名ngrok.com所指向的ip被墙，这意味着官方提供的服务全挂了……不过ngrok作为开源软件，在github上提供了完整代码，也在FAQ中讲述了如何自建ngrokd服务端。自建ngrokd服务器可以解决ngrok.com被墙后ngrok无法使用的问题，同时自己本地搭建的服务器也可以带来速度上的优势。

##需要一个域名

[freenom](freenom.com)上提供了tk、ml等顶级域名的免费注册使用12个月，具体注册方法可以参考[freehao123](http://www.freehao123.com/freenom-gq/)上的介绍。

注册完成后可以迁移dns服务到dnspod上，并设置顶级域名指向的地址以及一条*记录以保证所有二级域名都能被自动导向到服务端的地址。

##配置及编译
自建服务端需要有ssl设置，如果没有购买ssl证书或者没有找到合适的免费证书发行商，可以搭建自验证的证书服务。
```
NGROK_DOMAIN="my.domain.com"
git clone https://github.com/inconshreveable/ngrok.git
cd ngrok

openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000

cp rootCA.pem assets/client/tls/ngrokroot.crt
# make clean
make release-server release-client
```

参考以上示例，并根据自身情况修改NGROK_DOMAIN。
使用刚编译好的bin/ngrokd则可以启动子集的ngrokd服务程序。`bin/ngrokd -tlsKey=device.key -tlsCrt=device.crt -domain="$NGROK_DOMAIN" -httpAddr=":8000" -httpsAddr=":8001"`

将刚编译好的bin/ngrok程序分发到需要ngrok服务的内网机器上，并通过`echo -e "server_addr: $NGROK_DOMAIN:4443\ntrust_host_root_certs: false" > ngrok-config`配置ngrok配置文件，则可以使用`./ngrok -config=配置文件 ***`来使用ngrok所具备的功能。

ngrok的github仓库上【SELFHOSTING](https://github.com/inconshreveable/ngrok/blob/master/docs/SELFHOSTING.md)也提供了搭建ngrok的的较完善的指导。
