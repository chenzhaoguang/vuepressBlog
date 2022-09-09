---
title: 使用docker搭建wordpress
date: 2018-02-28
categories:
 - frontend
tags:
 - docker
 - wordpress
---

## 前言
去年在学习docker，在看完菜鸟教程和第一本docker书后，一直想实战用一下这个技术，多用用才能熟能生巧，真正体验它的利弊。正好傅老板用docker搭完了wordpress，我也就手痒跟着搭建了一下（也就是现在的这个博客网站）。
此处记录一下搭建过程。

<!-- more -->

## 搭建环境
- 阿里云ECS
> 去年双11买的，720/3年，1核1G1M香港服务器，centos 7.4
有个小插曲，阿里云的工作人员还给我打电话，问我用的怎么样。。阿里云什么时候有这种回访了。。。

- 域名
> 阿里云购买即可，像我申请的 .top 域名更是便宜，丧心病狂只要2块钱。。

- ssl证书
> https用的证书，我是在腾讯云免费申请的，地址为：[腾讯云证书管理](https://console.cloud.tencent.com/ssl)，此处就不详细描述申请过程了，很简单的

- Docker
> 这里要注意，centos中不要直接使用yum install docker，yum中的是旧的docker版本，升级参考我的这篇博文：[CentOS更新Docker至最新版本](https://www.thyiad.top/2017/12/29/centos-upgrade-docker/)

- Docker Compose
> compose原本是一个第三方公司写的，用来在docker中定义和运行复杂应用的小工具，后来被docker收购了，正式用来替代最早的fig。
> 通过以下命令安装：
> ``` bash
> # 下载compose
> curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname > -s`-`uname -m` -o /usr/bin/docker-compose
>
> # 赋予可执行权限，确保compose可执行
> chmod +x /usr/bin/docker-compose
> ```

## 添加一个docker network
网站需要占据80端口，显然，我们的服务器不可能只有一个网站，所以部署一个nginx容器是必须的，让这个nginx来监听80以及443端口，再根据域名转发到对应的网站容器。容器之间的通信是通过network的，所以我们需要先添加一个network：
``` bash
 docker network create nginx-proxy
```

## compose部署WordPress和MySql容器
- 创建工作目录，创建docker-compose.yml文件：
``` bash
cd /usr
mkdir myblog && cd myblog
vim docker-compose.yml
```

- docker-compose.yml输入以下内容：
```
version: '3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: your-mysql-root-password    # 在这里输入你要设置的mysql密码
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
        - wp_site:/var/www/html     # 定义卷后，compose down之类的操作不会导致你的文章等数据丢失
     expose:
       - 80
     restart: always
     environment:
       VIRTUAL_HOST: www.thyiad.top,thyiad.top
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
   nginx-proxy:
     image: jwilder/nginx-proxy
     container_name: nginx-proxy
     restart: always
     ports:
       - "80:80"
       - "443:443"
     volumes:
       - /var/run/docker.sock:/tmp/docker.sock:ro   # docker.sock是docker守护进程默认监听的Unix域套接字，容器进程通过它与守护进程进行通信。以后添加新的站点时，nginx将会自动发现并重启服务
       - nginx_certs:/etc/nginx/certs:ro    # nginx的证书目录，:ro指定为只读
volumes:
    db_data:
    wp_site:
    nginx_certs:
networks:
   default:
     external:
       name: nginx-proxy
```

## 添加ssl证书
``` bash
# 列出所有的卷信息，查找到xxx_wp_certs(xxx是docker自动添加的)
docker volume ls

# 查询出xxx_wp_certs的真实路径，一般是在 /var/lib/docker/volumes/xxx_wp_certs/_data
docker volume inspect --format '{{ .Mountpoint }}' xxx_wp_certs

# 创建www.thyiad.top.key，并且把ssl证书的xxx.key内容复制粘贴进来
cd /var/lib/docker/volumes/xxx_wp_certs/_data && sudo vim www.thyiad.top.key

# 创建www.thyiad.top.crt，并且把ssl证书的xxx.crt内容复制粘贴进来
cd /var/lib/docker/volumes/xxx_wp_certs/_data && sudo vim www.thyiad.top.crt
``` 
nginx-proxy如果发现在certs文件夹中存在当前域名的.crt和.key文件，将自动转为https协议

## 运行wordpress
``` bash
docker-compose up -d
```

此时，我们的网站就可以访问了，很简单吧？

参考资料：
[傅老板的博客](https://www.fujiabin.com/2017/11/07/deploy-wordpress-with-docker-compose-in-centos7/)
[docker-compose](https://github.com/docker/compose)
[nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/)
