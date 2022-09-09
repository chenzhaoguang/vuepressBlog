---
title: 使用docker搭建ngrok服务器
date: 2018-03-05
categories:
 - tool
tags:
 - docker
 - ngrok
---

## 前言
前面有一篇文章已经写了如何搭建frp：[使用docker搭建frp服务器](https://www.thyiad.top/2018/03/01/%E4%BD%BF%E7%94%A8docker%E6%90%AD%E5%BB%BAfrp%E6%9C%8D%E5%8A%A1%E5%99%A8/)，但对于ngrok这个用的最久的转发工具也忍不住想搭个试试，所以也就有了这篇文章。

## 准备工作
- ngrok客户端、服务端
> 我没有自己编译，直接用的sunny编译好的ngrok客户端，已经全部放到了这里：[github地址](https://github.com/Thyiad/tool/tree/master/ngrock/my-ngrok/server%20%26%20client)

- 具有外网ip的服务器
- 域名
> 我这里用的是*.tunnel.thyiad.top，需要把这个域名解析到服务器上，具体的子域名在客户端配置，服务端配置前缀域名为tunnel.thyiad.top

- docker
> 需要注意的是，我这里是基于ngin-proxy镜像来解析域名的，此处不再赘述，可参照之前的文章：[使用docker搭建wordpress](https://www.thyiad.top/2018/02/28/%E4%BD%BF%E7%94%A8docker%E6%90%AD%E5%BB%BAwordpress/)

## docker file
> 镜像已经上传到docker的hub上了，所以你也可以跳过docker file直接使用compose

创建工作目录与之前frp类似，dockerfile在这里就更简单了：
```
FROM ubuntu
MAINTAINER Thyiad <1520583107@qq.com>

COPY conf/ngrokd /ngrokd

RUN chmod +x /ngrokd

ENTRYPOINT /ngrokd --domain="tunnel.thyiad.top"
```
其实就是直接把conf下面的ngrok服务端复制过来，然后运行起来

## docker compose
创建工作目录与之前frp类似，compose文件为以下内容：
```
version: '3'

services:
  ngrok:
    image: thyiad/my-ngrok:latest
    container_name: my-ngrok
    ports:
      - "4443:4443"
    expose:
      - 80
      - 443
    restart: always
    environment:
      VIRTUAL_HOST: '*.tunnel.thyiad.top,tunnel.thyiad.top'

networks:
  default:
    external:
      name: nginx-proxy
```
运行我们的compose：
``` bash
docker-compose up -d
```
此时，我们的ngrok服务器就已经OK了。

## ngrok客户端
服务端搭好之后，我们就可以使用客户端来进行使用了，解压对应的平台客户端，执行命令进行隧道连通。我这里是windows x64，执行以下命令：
```
ngrok.exe -server_addr=tunnel.thyiad.top:4443 -subdomain=test -proto=http 52485
```
敲完回车后会出现以下界面：
![](https://static.yirenyian.com/blog/ngrok-running.png)
我们来访问test.tunnel.thyiad.top试试：
![](https://static.yirenyian.com/blog/ngrok-visible.png)
OK，现在就已经大功告成了！

以上文件已经上传到github：
[https://github.com/Thyiad/docker](https://github.com/Thyiad/docker)