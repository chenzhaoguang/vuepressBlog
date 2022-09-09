---
title: iisnode运行koa2项目
date: 2018-12-18
categories:
 - frontend
tags:
 - iisnode
 - nodejs
---

## 前言
前段时间在研究koa2+mongodb，捣鼓完一个项目之后，就是发布的问题了。一般来说，nodejs的项目，推荐开发用nodemon，生产用pm2。两者都可以监听文件变更重启node项目，而pm2则更为强大，更拥有性能监控，日志，负载均衡等等高级功能。
> 在linux服务器上，我们一般会选择使用pm2启动，然后使用nginx转发。而在windows上，我们首选的则是这个iisnode啦，不为别的，就因为这货是iis插件，使用更方便，也能够监听文件变更自动重启node项目，至于作者宣称的其他各种优点，我们就不用管了，毕竟真要说性能的话，我在stackoverflow上面还见过吐槽iisnode性能差的问题呢。

## 准备工作
- iis7.x/8.x
> 请注意不要低于iis7，至于iis express/WebMatrix，iisnode也是支持的
- nodejs
> 下载并安装nodejs的最新windows版本

## 安装url-rewrite
因为需要用到路由重写，所以必须为iis安装URL Rewrite插件，下载地址为：
[https://www.iis.net/downloads/microsoft/url-rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)

## 安装iisnode
[https://github.com/Azure/iisnode](https://github.com/Azure/iisnode)
> 直接在github项目的release中下载最新版，我们直接下载iisnode-full系列就ok了。
这货最早是tjanczuk私人开发的，后面捐赠给微软了，旧地址为
[https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode)

## 初步使用
- 具体安装步骤就不说了，直接让他下一步下一步安装完，默认安装路径为：C:
\Program Files\iisnode

- 然后我们直接运行setupsamples.bat文件，它会自动在default web site中添加一个node项目
![](https://static.yirenyian.com/blog/iisnode-sample.jpg)
- 此时我们访问http://localhost/node，如果能看到下面的页面就说明大功告成了
![](https://static.yirenyian.com/blog/iisnode-suc.jpg)
## 部署koa2项目
还是老样子创建站点，然后将路径指向koa2项目的根路径，只需要添加一个web.config文件，填入以下内容：
``` xml
<configuration>
   <system.webServer>

     <handlers>
       <add name="iisnode" path="app.js" verb="*" modules="iisnode" />
     </handlers>

     <rewrite>
       <rules>
         <rule name="app">
           <match url="/*" />
           <action type="Rewrite" url="app.js" />
         </rule>
       </rules>
     </rewrite>

     <security>
       <requestFiltering>
         <hiddenSegments>
           <add segment="node_modules" />
         </hiddenSegments>
       </requestFiltering>
     </security>    
     
	 <iisnode
      nodeProcessCommandLine="&quot;D:\Program Files\nodejs\node.exe&quot;" 
      interceptor="&quot;%programfiles%\iisnode\interceptor.js&quot;" />
	 
   </system.webServer>
 </configuration>
```
> 请注意最后一段，我在里面指明了nodejs的执行路径，因为他的默认配置是指向C盘的，而我安装时安装在了D盘，所以会报一个无法执行nodejs的错误。
所有的可配置选项，在sample中也有列出来：
[https://github.com/Azure/iisnode/blob/master/src/samples/configuration/web.config](https://github.com/Azure/iisnode/blob/master/src/samples/configuration/web.config)