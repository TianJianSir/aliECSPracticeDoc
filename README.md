# 阿里云服务器实践

## 前端

使用[create-react-app](https://github.com/facebook/create-react-app)作为脚手架，开发前端项目

## 后台接口

使用[express](https://github.com/expressjs/express)搭建后台API

1, 在[blog_server](https://github.com/TianJianSir/blog_server)上,修改相关路由代码,逻辑处理,存储数据等

2, 在服务器上拉取最新代码

3, 使用`pm2 list`查看当前的node进程

4, 使用`pm2 restart [name]` 重启对应的服务
## 数据库

使用[mongDB](https://www.mongodb.com/)存储数据

## nginx 配置

nginx相关的配置，反向代理等

## 部署相关
使用[pm2](https://github.com/Unitech/PM2/)管理node进程

## 文档编写
使用[docsify](https://github.com/docsifyjs/docsify)作为文档编写的工具

1，在[DOCS](https://github.com/TianJianSir/DOCS.git)上,修改readme

2，在服务器上拉取最新代码

3，重启nginx,`nginx -s reload`(这一步不需要)

4, 访问[http://tianjian.work/nginxdoc](http://tianjian.work/nginxdoc)


## 其他