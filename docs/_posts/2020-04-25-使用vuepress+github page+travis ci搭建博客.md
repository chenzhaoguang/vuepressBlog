---
title: 使用vuepress+github page+travis ci搭建博客
date: 2020-04-25
categories:
 - frontend
tags:
 - vuepress
 - github page
 - travis ci
---

## 前言
其实，我博客写的很少了，相对来说，流水账笔记记的更多一点（[笔记](https://github.com/thyiad/learn)）。17年底研究docker的时候，顺便用docker搭建了wordpress，于是就有了这个网站。
然而前段时间发现wordpress经常挂，docker log排查后，发现是因为mysql太吃内存，我这台1G1核的服务器带不动了。。。
于是乎花了1天假期用vuepress+github page+travis ci重新搭建，此处简单记录一下

## vuepress
具体使用就不说了，[官方文档](https://vuepress.vuejs.org/zh/)很详细。
值得一提的是踩了一个坑：进入详情页后再F5刷新，此时会404。最后使用[vuepress-plugin-dehydrate](https://vuepress.github.io/zh/plugins/dehydrate/#ssr-%E9%94%99%E9%85%8D)这个官方插件解决了
本质上vuepress是一个带有ssr的spa应用，ssr, spa都需要服务端的支持，但部署到github page显然是无服务端支持的，只能全部静态化解决。
而这个插件，解决了ssr错配的问题，同时支持全部静态化。

## github page
按照github要求，创建一个 [name].github.io仓库即可
[官方文档](https://help.github.com/cn/github/working-with-github-pages/getting-started-with-github-pages)

## travis ci
步骤如下：
- 注册并开通travis，打开blog仓库的自动构建
- blog仓库下创建.travis.yml文件, 我的完整命令如下
    ``` bash
    language: node_js
    node_js:
    - lts/*
    install:
    - npm install
    script:
    - npm run build
    after_script:
    - cd dist
    - echo 'www.thyiad.top' > CNAME
    - git init
    - git add -A
    - git commit -m 'deploy'
    - git push -f "https://${GITHUB_TOKEN}@${GH_REF}" master
    env:
    global:
        - GH_REF: github.com/Thyiad/thyiad.github.io.git
        - secure: "uhAf/b6TBULEd7DEbT0ggtrZW4ZUW1FxZVlk6mKTpcYdm0Iz0jObLohUPbKxt0uC2Ud5Z1ZAepAuJoPMxf/Ten12s2MW3yp7iihCnLZhE1SHm31mvrv8v0TdcFFKlLPLK/KKh7l1xShJFTVvYyFQfO7dvxdcTCAYmDY3jobPpfAlLdtnWzPN7UijpA3uo/Jp5i+DsNtwVLJQHOfQcO948NILd20PwXY0CbYvNl7MsCtFcFZTxHXt5XTcbOTIpk874Q+6W/qzM95mWgtKSqKLJAjhx4tj6Nbksn7XtFggGP9x0n/EB1Kid8jvX3FZ71rvQwJuHWgXS4XJXKz7Fdl1zYfRlbGifPHpArKfzH7pvae09VvHKG6+gTY28bPeyY8PtWIwmiFLbJHNT6jeKCXpZU34IlIc4lvsT5hgftuqa2yPXCTltYTU4uWR7KhNFLcvsSaxcjFCfKZjGnwcwrTDgGEgJS9DIGprbbZ8t+DSgi3Rsxt/yeT8yzTxyB2ALr+AEc5d7m+Qt4spDcnTpTSPHZJIyRPgOE5VTGV4YxrndqDOJ+NlvqDczCl01XrNXpIlLn+0K4oS0PpoN2WQ6TG7S+BFsgiYglBMg2tuVlXwc9qWybmhx9VFjO9dFZxZQeCHlvnUpVQ8kG5AVz2Ur21WcovzV8hu9UAWNRQEh5YnJoU="
    ```
这里在监听到push代码后，执行了以下动作：
- install 安装依赖
- build 打包代码
- 把打包后的文件push到thyiad.github.io仓库

> 推送到仓库需要在github的settings/Developer settings/Personnal access tokens中添加一条token
> 使用 ``travis encrypt -r 用户名/仓库 GITHUB_TOKEN=XXX`` 生成token，保证token只在这个仓库中能被解密使用，如果是在该仓库下面执行命令，则可以省略
