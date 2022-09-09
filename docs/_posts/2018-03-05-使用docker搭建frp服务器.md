---
title: 使用docker搭建frp服务器
date: 2018-03-05
categories:
 - tool
tags:
 - docker
 - frp
---

## 前言
- 把本地的开发环境映射到外网，这是我们经常会碰到的一个要求，比方说展示给别人看啦，临时测试啦。尤其在微信开发中，因为微信必须要求80端口，所以不转发的情况下，我们只能把代码部署到服务器之后才能验证测试，非常麻烦。
- 最早的时候是花生壳，不过这家公司贼恶心。。后面开始用ngrok，然后现在又有了frp，相比来说frp的配置要更简单一点。
- 因为没有找到合适的docker镜像，所以在参考很多之后，就有了如下自写的image及compose

## 准备工作
-  具有外网ip的服务器
- 域名
> 我这里是准备了一个子域名，*.frp.thyiad.top，把这这个域名解析到服务器，这样可以支持同时映射多个域名到外网，具体的子域名在frp客户端配置，服务端配置前缀域名为frp.thyiad.top

- docker
> 需要注意的是，我这里是基于ngin-proxy镜像来解析域名的，此处不再赘述，可参照之前的文章：[使用docker搭建wordpress](https://www.thyiad.top/2018/02/28/%E4%BD%BF%E7%94%A8docker%E6%90%AD%E5%BB%BAwordpress/)

## docker file
> 镜像已经上传到docker的hub上了，所以你也可以跳过docker file直接使用compose

创建工作目录：
``` bash
cd /usr
mkdir frp && cd frp
mkdir frp_image && cd frp_image
```
先创建一个frp的默认配置文件：
``` bash
mkdir conf && vim conf/frps.ini
```
把以下内容填入 frps.ini：
```
[common]
bind_addr = 0.0.0.0
bind_port = 7000
kcp_bind_port = 7000
vhost_http_port = 80
vhost_https_port = 443
dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
authentication_timeout = 0
subdomain_host = frp.thyiad.top
```
创建dockerfile：
``` bash
vim dockerfile
```
把以下内容填入dockerfile：
```
FROM ubuntu
MAINTAINER thyiad <1520583107@qq.com>

ARG FRP_VERSION=0.16.0

RUN apt update \
    && apt install -y wget

WORKDIR /tmp
RUN set -x \
    && wget https://github.com/fatedier/frp/releases/download/v${FRP_VERSION}/frp_${FRP_VERSION}_linux_amd64.tar.gz \
    && tar -zxf frp_${FRP_VERSION}_linux_amd64.tar.gz \
    && mv frp_${FRP_VERSION}_linux_amd64 /var/frp \
    && mkdir -p /var/frp/conf \
    && apt remove -y wget \
    && apt autoremove -y \
    && rm -rf /var/lib/apt/lists/*

COPY conf/frps.ini /var/frp/conf/frps.ini

VOLUME /var/frp/conf    # conf被配置成了卷，方便以后修改frps.ini

WORKDIR /var/frp
ENTRYPOINT ./frps -c ./conf/frps.ini
```
这个dockerfile执行了以下操作：
- 从github上下载frp的release版本
- 解压
- 从conf目录中读取替换默认的frps.ini

此时就可以使用docker build命令进行编译镜像了，命令为：
``` bash
docker build -t="thyiad/my-frp" .
```

## docker compose
>在镜像编译好后，我们就可以开始compose文件了，毕竟compose比直接docker run要方便的多

创建工作目录：
```
mkdir /usr/frp/frp_compose && cd /usr/frp/frp_compose
vim docker-compose.yml
```
把以下内容填入docker-compose.yml：
```
version: '3'

services:
  frp:
    image: thyiad/my-frp:latest
    container_name: my-frp
    ports:
      - "7000:7000"
      - "7500:7500"
    expose:
      - 80
      - 443
    volumes:
      - frp_conf:/var/frp/conf
    restart: always
    environment:
      VIRTUAL_HOST: '*.frp.thyiad.top,frp.thyiad.top'   # 指定需要绑定的域名

volumes:
    frp_conf:

networks:
  default:
    external:
      name: nginx-proxy # 此处的nginx-proxy为之前创建的docker network
```
运行我们的compose：
``` bash
docker-compose up -d
```
此时，我们的frp服务器就已经OK了。
我们访问一下test.frp.thyiad.top试试：
![](https://static.yirenyian.com/blog/frp-unvisible.png)
显然，frp已经在运转了，只是该域名并没有绑定转发

## frp客户端

服务端搭好之后，我们就可以下载客户端进行使用了。需要前往frp的github上下载对应的版本，我这里是16.0，windows x64。

下载解压后，我们修改frpc.ini为以下内容：
```
[common]
server_addr = 你的服务器ip
server_port = 7000
# protocol = kcp

[web]
type = http
local_port = 52485
subdomain = test
```
然后打开cmd，运行frpc：
``` bash
cd /d d:\frp
frpc
```
此时会出现以下界面：
![](https://static.yirenyian.com/blog/frp-running.png)

说明已经连接成功了，我们再来访问test.frp.thyiad.top试试：
![](https://static.yirenyian.com/blog/frp-visible.png)

此时，我们的frp就已经搭建好了，很简单吧？
> ngrok的服务器搭建在这里：
> [使用docker搭建ngrok服务器](https://www.thyiad.top/2018/03/01/%E4%BD%BF%E7%94%A8docker%E6%90%AD%E5%BB%BAngrok%E6%9C%8D%E5%8A%A1%E5%99%A8/)

以上文件已经上传到github：
[https://github.com/Thyiad/docker](https://github.com/Thyiad/docker)