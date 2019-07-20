---
title: Docker 的前端持续集成开发环境
categories:  docker
tags: 
- docker
---

## 准备

  1. 注册DaoCloud账号 
  
  2. 使用DaoCloud Service
  
  3. 将vue.js项目push到Github

## 目的

 1. 代码无需在本地构建

 2. 只需将代码推上 Github ，自动构建 -> 部署

 3. 版本易管理，可轻松回退版本
 

## 步骤

### 1.基于vue.js的前端项目（例：使用vue-cli）
### 2. 在项目根目录下编 Dockerfile
```
	 FROM node:6.10.3-slim
    RUN apt-get update \    && apt-get install -y nginx
    WORKDIR /app
    COPY . /app/
    EXPOSE 80
    RUN  npm install \     && npm run build \     && cp -r dist/* /var/www/html \     && rm -rf /app
    CMD ["nginx","-g","daemon off;"]

```
### 3.使用 DaoCloud Service搭建 Devops 流程
* 授权DaoCloud Service可以访问github账号
* 在 Devops 项目中新建一个项目，并选择 Github 中对应刚才新创建的项目
* 先手动构建一个镜像版本，便于用这个镜像版本创建一个应用
* 连接自有主机（没有自有主机的，也可以使用云端测试环境，主机需要安装docker，将服务开启service docker start）
* 创建应用，进入【镜像仓库】选择刚才手动构建出来的镜像，并部署最新版本到自由主机或者云端测试环境
* 部署应用
* 等待镜像拉取成功，容器UP后即可访问vue项目
* 使用自动构建流程，自动发布任务
* 触发行为可以设置为branch及tag两种模式，当本地push更新后的代码，dcs将自动部署上线
* 完成啦！！！