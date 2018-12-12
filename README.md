# 阿里云服务器实践

## 前端

使用[create-react-app](https://github.com/facebook/create-react-app)作为脚手架，开发前端项目

使用`pm2 start server.js`开启一个node服务,返回build之后的html入口文件，这里若是想提高体验的话，可以针对index.html修改,作为一个服务端渲染

nginx上将`/blog`反向代理到我们的node服务上,
```
location /blog {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_pass http://blog;
        }
```
因为将`/blog`代理到了node服务器，所以需要将引用资源文件的地方都加上`/blog`,这点webpack可以帮助我们,配置下config中的`publicPath:'/blog'`,不然的话，资源请求会404

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

## 文档编写
使用[docsify](https://github.com/docsifyjs/docsify)作为文档编写的工具

>本地使用`docsify serve DOCS` 启动服务，查看效果

1，在[DOCS](https://github.com/TianJianSir/DOCS.git)上,修改readme

2，在服务器上拉取最新代码

3，重启nginx,`nginx -s reload`(这一步不需要)

4, 访问[http://tianjian.work/nginxdoc](http://tianjian.work/nginxdoc)


## 其他