---
title: windows上安装node-gyp
date: 2019-11-12
categories:
 - frontend
tags:
 - node-gyp
---

## 前言
gyp是一种根据c++源代码编译的工具，node-gyp就是为node编译c++扩展的时候使用的编译工具。

## 问题
一直有使用到bcrypt模块，而bcrypt依赖于node-gyp。但最近在windows部署却有奇怪的问题：
``` bash
error bcrypt@3.0.2 install: `node-pre-gyp install --fallback-to-build`
```

## 前提
需要安装Visual C++ Build Tools和python2.7，此处使用一行命令可以自动完成：
    ```bash
    npm install --global --production windows-build-tools
    ```

## 究极原因
因为之前一直可以，而此次不可以，所以我真是懵逼，经过好一番资料查找，终于找到一个issue：
https://github.com/nodejs/node-gyp/issues/1599

按照这位仁兄所述，貌似是因为node-gyp依赖旧版本的grpc，所以新版本的node下是无法编译成功的。然后我把node从12降到10（node最新稳定版是12，以前是10），就。。就OK了！~

真是蛋疼