---
title: docker发布一个springboot工程
date: 2022-03-07 01:41:18
tags: docker
---

# 前言✨

本文介绍如何使用docker发布一个springboot工程到腾讯云

#  一、搭建mysql数据库
## 1.拉取mysql镜像
```
docker pull mysql8.0.0
```
## 2.启动mysql
```
docker run -p 3306:3306 --name mysql-1 --privileged=true --restart unless-stopped -v /opt/mysql_docker/logs:/logs -v /opt/mysql_docker/localtime -e MYSQL_ROOT_PASSWORD=123456 -itd mysql:8.0.0
```
--privileged=true 挂载文件权限设置

--restart unless-stopped 设置 开机后自动重启容器


-v /opt/mysql_docker/logs:/logs 挂载日志

-v /etc/localtime:/etc/localtime 容器时间与宿主机同步

-e MYSQL_ROOT_PASSWORD=123456 设置密码

-itd mysql:8.0.0 后台启动,mysql

-v /opt/mysql_docker/mysql:/etc/mysql 挂载配置文件

-v /opt/mysql_docker/data:/var/lib/mysql 挂载数据文件 持久化到主机

## 3.修改mysql的密码
**注意：腾讯云安全组默认不添加开放 3306 端口，需要手动添加，才能连接远程工具**

**另外，当远程工具不能连接时，需按下面的操作更改加密规则为：`mysql_native_password`**
1. 进入容器内部
```
docker exec -it mysql-1 bash
```
2. 输入用户名
```
mysql -u root -p
```
3. 修改密码
```
use mysql;
select host,user,plugin from user;
alter user 'root'@'%' identified with mysql_native_password by '123456';
flush privileges;
```

# 二、创建后端镜像并启动
- 方式一：通过--link
## 1.打包springboot工程并上传
1. 将关联的数据库`ip地址`改成`别名`（后面会提到），以下把ip改成了emysql
```
spring.datasource.url=jdbc:mysql://emysql:3306/music?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=false
```
2. 然后用maven对数据库进行package打包，然后上传至腾讯云

## 2.编写Dockerfile并打包镜像
1. Dockerfile配置如下：
```
FROM java:8
EXPOSE 8090

VOLUME /tmp

ADD music-1.0-SNAPSHOT.jar /app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
2. 然后通过以下命令打包镜像
```
docker build -t music .
```
## 3.启动镜像
通过`--link`命令连接mysql-1容器并取别名为emysql(用于上述提到的数据库地址配置)
```
docker run -p 8090:8090 --name music --link mysql-1:emysql -d music
```
- 方式二：docker-compose方案：

学不动了，后续在更新~~~😆
