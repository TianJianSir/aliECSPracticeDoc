# 阿里云服务器实践
不是土豪,服务器只有一台,所以我们要榨干服务器,处理的方式是对使用nginx对不同的location进行反向代理。

`/blog`      前端页面

`/api`       后台接口

`/nginxdoc`  文档展示


## 前端

使用[create-react-app](https://github.com/facebook/create-react-app)作为脚手架，开发前端项目

使用`pm2 start server.js`开启一个node服务,返回build之后的html入口文件。此处若是想提高体验的话，可以针对index.html修改,作为一个服务端渲染。目前的话，index.html为空。内容由js插入。

nginx上将`/blog`反向代理到我们的node服务上,
```
location /blog {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_pass http://blog;
}
```
因为将`/blog`代理到了node服务器，所以需要将引用资源文件的地方都加上`/blog`。

这点webpack可以帮助我们,配置下config中的`publicPath:'/blog'`,使用create_react_app的话，可以在`package.json`中加入`"homepage":"/blog"`,这个其实就是cdn的意思, webpack编译的时候所有的资源路径前都会加上这个前缀。不然的话，资源请求会404。

+ ### 使用redux
m_react-redux分支 使用redux+redux-thunk(异步action),目录结构如下

![image.png](https://upload-images.jianshu.io/upload_images/2448841-d7bd7cecf9e87765.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/100)

主要就是增加了一个redux目录，放了action和reducer。在index.js中使用provide将整体结构包裹起来，接着在页面层次使用connect将view和data连接起来。
```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { Provider } from 'react-redux';
import store from './redux/store';

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
);
```

+ ### 使用dva
master分支,目录结构如下

![image.png](https://upload-images.jianshu.io/upload_images/2448841-aeebd66fd8ed4c9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

增加了model文件夹，存放state。
index.js改动较大，使用了dva的规范。在页面层次还是使用connect连接view和data,开发体验要好于原生redux.
```
// 1. Initialize
// 初始化dva对象的时候,要是传空的话，即使用了BrowserRouter URL后面还是会加上#/ 很丑
const app = dva({
    history: createHistory()
});

// 2. Plugins
app.use(createLoading());

// 3. Model
app.model(login);
app.model(register);

// 4. Router
app.router(router);

// 5. Start
app.start('#root');
```
使用dva-loading+nprocess处理转场效果

page高阶函数统一处理错误收集


## 后台接口

使用[express](https://github.com/expressjs/express)搭建后台API

1, 在[blog_server](https://github.com/TianJianSir/blog_server)上,修改相关路由代码,逻辑处理,存储数据等

2, 在服务器上拉取最新代码

3, 使用`pm2 list`查看当前的node进程

4, 使用`pm2 restart [name]` 重启对应的服务
## 数据库

使用[mongDB](https://docs.mongodb.com/guides/)存储数据

> db为某个数据库,inventory为某张表
### 直接使用命令操作mongodb

+ 查询

```
// 此处查询条件为空，得到的将是所有的数据
db.inventory.find( {} )
// 条件查询
db.inventory.find( { status: "D" } )
```

+ 插入

```
// 插入一条数据
db.inventory.insertOne(
   { "item" : "canvas",
     "qty" : 100,
     "tags" : ["cotton"],
     "size" : { "h" : 28, "w" : 35.5, "uom" : "cm" }
   }
)
// 插入多条数据
db.inventory.insertMany( [
   { "item": "journal", "qty": 25, "size": { "h": 14, "w": 21, "uom": "cm" }, "status": "A" },
   { "item": "notebook", "qty": 50, "size": { "h": 8.5, "w": 11, "uom": "in" }, "status": "A" },
   { "item": "paper", "qty": 100, "size": { "h": 8.5, "w": 11, "uom": "in" }, "status": "D" }
]);
```

+ 更新

```
// 更新第一条item=paper的数据
db.inventory.updateOne(
    { "item" : "paper" }, // specifies the document to update
    {
      $set: {  "size.uom" : "cm",  "status" : "P" },
      $currentDate: { "lastModified": true }
    }
)
// 更新所有的item=paper
db.inventory.updateMany(
    { "item" : "paper" }, // specifies the document to update
    {
      $set: {  "size.uom" : "cm",  "status" : "P" },
      $currentDate: { "lastModified": true }
    }
)
```

+ 删除

```
// 删除第一条 status = D
db.inventory.deleteOne(
    { "status": "D" }
)
// 删除所有 status = D
db.inventory.deleteMany(
    { "status": "D" }
)
```

### 在node中操作mongodb,API和sheel命令基本一致

+ 链接数据库

```
// npm i mongodb --save
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

// Connection URL
// 目前用的是const url = "mongodb://localhost:27017/";
const url = 'mongodb+srv://$[username]:$[password]@$[hostlist]/$[database]?retryWrites=true';

MongoClient.connect(url, function(err, client) {
  assert.equal(null, err);
  client.close();
});

```

+ 查询

```
db.collection('inventory').find({});
```

+ 插入

```
db.collection('inventory').insertOne({
  item: "canvas",
  qty: 100,
  tags: ["cotton"],
  size: { h: 28, w: 35.5, uom: "cm" }
})
.then(function(result) {
  // process result
})
```

+ 更新

```
db.collection('inventory').updateOne(
  { item: "paper" },
  { $set: { "size.uom": "cm", status: "P" },
    $currentDate: { lastModified: true } })
.then(function(result) {
  // process result
})   
```

+ 删除

```
db.collection('inventory').deleteOne({
  status: "D"
})
.then(function(result) {
  // process result
})

db.collection('inventory').deleteMany({
  status: "A"
})
.then(function(result) {
  // process result
})
```

## nginx 配置

nginx相关的配置，反向代理等

## 部署相关
使用[pm2](https://github.com/Unitech/PM2/)管理node进程

首次部署：`pm2 start server.js`

后续部署：`pm2 restart server`

## 其他
### 阿里云上配置ssl

1,申请证书（阿里云上可以申请免费的，但是不稳定，当然，学习是够了）

2,申请完毕之后，会在ssl证书管理中看到已签发的证书，下载下来

3,在服务器上的nginx目录下新建Cert目录，将证书传到该目录。
```
scp /path/local_filename username@servername:/path
```
4,修改nginx,http默认监听80端口，https默认监听443
```
server{
  #listen 80  //若想http和https共存，将这行打开，同时注释 ssl on
  listen 443 ssl;
  ssl on; 
  ssl_certificate "/etc/nginx/cert/cert.pem";
  ssl_certificate_key "/etc/nginx/cert/cert.key";
  ssl_session_cache shared:SSL:1m;
  ssl_session_timeout  10m;
  ssl_prefer_server_ciphers on;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
}
```

5,重启nginx,浏览器重新访问

6,免费的证书不稳定，会造成打不开网页的假象（可以多刷新两次）,还有香港和新加坡的服务器对https的支持不是很好。

### 文档编写
使用[docsify](https://github.com/docsifyjs/docsify)作为文档编写的工具

>本地使用`docsify serve DOCS` 启动服务，查看效果

1，在[DOCS](https://github.com/TianJianSir/DOCS.git)上,修改readme

2，在服务器上拉取最新代码

3，重启nginx,`nginx -s reload`(这一步不需要)

4, 访问[http://tianjian.work/nginxdoc](http://tianjian.work/nginxdoc)


