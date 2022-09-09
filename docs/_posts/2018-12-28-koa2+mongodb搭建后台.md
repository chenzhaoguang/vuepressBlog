---
title: koa2+mongodb搭建后台
date: 2018-12-28
categories:
 - frontend
tags:
 - koa2
 - mongodb
 - nodejs
---

## 前言
毫无疑问，目前nodejs里面用来开发后台的首选就是koa2+mongodb的组合了。参考过很多资料，都是零零碎碎不齐全，要么很简单只是教你如何运行一个demo，要么只讲了简单的一方面，要么就是一个复杂的koa项目生成器，我设想的一个最基础的后台应该具有以下内容：
> 此处只讨论前后端分离，后台项目提供接口，不考虑模板渲染之类的，毕竟你都用nodejs做后台了，还不做前后端分离也太说不过去了
- 对传入、返回及错误数据做统一处理
- 支持跨域
- 使用 token 做身份验证
- 完善的日志记录
- 支持发送邮件
- 上传文件
- 常见的数据库操作，对列表数据分页，返回指定行数据做封装
- 调试

所以在折腾完之后，就想把整个过程记录一下，如果你正好是刚开始摸索，应该能让你避免不少弯路。

## 创建基础项目
- 创建koa-mongo目录，并运行 npm init 创建package.json
``` bash
mkdir koa-mongo
cd ./koa-mongo
npm init
```
- 安装基础包
``` bash
npm install koa # koa，必须的
npm install koa-router  # 路由，必须的，这里要注意的是还有一个koa-route，这两个是不同的，不要用koa-route
npm install koa-static  # 静态资源，必须的
```

- 创建app.js，填入以下内容：
``` js
// koa
const Koa = require('koa')
const app = new Koa()

// midleware
const serve = require('koa-static')

app.use(serve('./assets'))

var server = app.listen(3000, function (){
    const host = server.address().address;
    const port = server.address().port;
    console.log('app start listening at http://%s:%s', host, port);
});
```
此处创建了一个实例，监听3000端口，将assets目录作为静态资源运行，我们创建一个assets目录，里面创建一个index.html，然后我们运行起来试试：
``` bash
node ./app.js
```
此时会打印一行日志：app start listening at http://:::3000，让我们来访问试试：
![](https://static.yirenyian.com/blog/koa-mongo-home.jpg)
ok，koa启动一个项目就是这么简单。。

## 添加路由
在koa-router的使用说明中，我们可以看到是这样使用的：
```js
var router = new Router();
 
router.get('/', (ctx, next) => {
  // ctx.router available
});
 
app
  .use(router.routes())
  .use(router.allowedMethods());
```
因为正常项目中，controller肯定不止一个的，所以我把目录写成这样：
```
├─ controller
    ├─ test-controller
├─ router.js
```
controller目录用来放置所有的controller，在router.js中统一汇总，app.js中只需要使用router.js即可。
test-controller:
``` js
const hello = async (ctx, next) => {
    ctx.body = 'hello world'
    ctx.status = 200;
}

module.exports = {
    'test/hello': hello,
}
```
router.js:
```js
const Router = require('koa-router')
const router = new Router({
    prefix: '/api', // 统一前缀，接口全部为 /api/xxx 格式
})

const testController = require('../controller/test-controller')

Object.keys(testController).forEach(key=>{
    router.all("/"+key, testController[key]);   // router.all是允许所有的访问方式，如果需要限定则改为指定方式即可
})

module.exports = router;
```
app.js:
```js
// router
const router = require('./router')
app.use(router.routes()).use(router.allowedMethods())
```
这个时候让我们重新启动一下，访问localhost:3000/api/test/hello试试：
![](https://static.yirenyian.com/blog/koa-mongo-hello.jpg)
可以看到正确返回了hello world。

## 调试
到这里了，有没有感觉哪里不对劲。每修改一次，都需要手动敲命令重启一次，这简直太烦了好嘛，我们程序猿哪能忍受这个。答案就是使用nodemon，这个玩意儿能监听我们的文件变更，自动运行命令重启应用。使用方式也很简单，这个我们直接全局安装就好了：
``` bash
npm install -g nodemon
```
然后去package.json的scripts中添加一行脚本：
```json
"dev": "nodemon ./app.js",
```
然后 npm run dev，把hello函数修改一下返回值，保存，就会看到nodemon自动帮我们重启应用了。
> 除了nodemon，类似的工具还有很多，这里就不展开说了，现在的你只需要知道开发用nodemon，线上用pm2就ok了

解决了自动运行之后，我们来说一下调试。其实nodejs自带了调试的，只需要一个inspect参数，调试的时候就跟我们在chrome中调试是一模一样的。
我们先去package.json的scripts中添加一行脚本：
```json
"debug": "nodemon --inspect ./app.js",
```
然后我们运行 npm run debug，刷新我们的网页，用f12打开，此时我们能看到开发者工具上面多了一个nodejs的图标：
![](https://static.yirenyian.com/blog/koa-mongo-debug-1.jpg)
点击这个图标，就可以跟调试网页一样调试nodejs代码了：
![](https://static.yirenyian.com/blog/koa-mongo-debug-2.jpg)
## 配置化
代码就是一步步总结，边写边优化，重构。到目前为止，我们会发现有不少配置性的东西是散乱在不同文件中，比如说项目启动时监听的端口，接口的统一前缀，考虑到我们还会有很多配置项，我们应该把这些写到一个配置文件中集中管理。
新建一个config.js:
```js
module.exports= {
    port: 3000,
    apiPrefix: '/api',
}
```
然后把用到的地方全部更改为变量

## 跨域
为了防止跨域问题，我们需要使用koa2-cors类库，使用方式很简单：
```bash
npm install koa2-cors
```
然后在app.js中添加以下内容：
```js
const cors = require('koa2-cors')
app.use(cors())
```
一行代码搞定，cors的具体配置此处就不细说了，有兴趣的可以自己去看看

## 处理参数，上传文件
一般来说，我们是使用koa-bodyparser 和 koa-multer来分别处理表单数据和文件数据的。这两个分别集成也没什么问题，但我们可以直接使用koa-body来完成。koa-body是基于co-body和formidable做了封装，同时支持参数解析和文件上传。最后是把参数和文件分别放到ctx.request.body和ctx.request.files变量中。
我这里是把上传文件统一放到assets/upload目录中:
```js
const uploadDir = path.join(__dirname, 'assets/upload/')
// 此处还需要判断文件夹是否存在，不存在的话就创建
app.use(koaBody({
    multipart: true,
    encoding: 'utf-8',
    formidable:{
        uploadDir: uploadDir,
        keepExtensions: true,
        maxFieldsSize: 5*1024*1024,
        onFileBegin:(name, file)=>{

        }
    }
}))
```
然后添加一个upload：
```js
const upload = async (ctx, next)=>{
   const files = ctx.request.files || {};   # 文件会被解析到ctx.request.files中，是个object
   let fileNames = Object.keys(files);
   if(fileNames.length<=0){
       throw new ApiError('上传文件不能为空')
   }else{
       if(fileNames.length===1){
           ctx.body = files[fileNames[0]].path.replace(config.baseDir+'/assets', "")
       }else {
           ctx.body = fileNames.map(key => files[key].path.replace(config.baseDir+'/assets', ""))
       }
   }
}
```

## token验证
使用jwt，待续

## 统一返回格式
待续

## mongodb
待续

## 日志模块
待续

## 微信支付
待续