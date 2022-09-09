---
title: pm2的ignore_watch不生效
date: 2020-11-15
categories:
 - tool
tags:
 - pm2
---

### 前言
node监听文件变化并重启，本地开发时nodemon最佳，线上部署时，pm2最佳
pm2中为了方便，又最好使用ecosystem配置文件，不过发现其中的ignore_watch，修改后，死活不生效

### 具体配置

比如，一开始watch配置是这样的：
``` js
{
    watch: true,
    ignore_watch: [
      'node_modules',
    ]
}
```
加了一个文件夹之后：
``` js
{
    watch: true,
    ignore_watch: [
      'node_modules',
      'upload',
    ]
}
```
每次配置文件有修改，pm2都会自动重启，但是此时upload文件夹只要有文件变更，仍然会触发watch并重启进程

试过很多命令：
``` bash
pm2 reload ecosystem.config.js
```
``` bash
pm2 restart ecosystem.config.js
```
``` bash
pm2 reload ecosystem.config.js --watch
```
``` bash
pm2 restart ecosystem.config.js --watch
```

都没有用，最后试下来发现必须要这样：
``` bash
pm2 stop ecosystem.config.js --watch
pm2 start ecosystem.config.js --watch
```

看上去restart和reload不会重新读取应用配置文件，必须要start和stop才会应用新的配置文件，感觉怪怪的，与直觉不一样。
> pm2的官方文档唯一有提过的，就是stop时加上--watch才会停止监听。
