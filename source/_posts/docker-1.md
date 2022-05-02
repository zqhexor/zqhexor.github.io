---
title: Docker学习：从创建一个执行初始化sql脚本的mysql容器开始
date: 2021-06-30 09:00:03
tags: docker
---
## 前言

“我的机器上可以运行，为什么你的不可以？？？”

你可能会经常听见程序员之间出现这样的对话😂~~~

“芜湖~~~ 我来帮助你们！”Docker🐳突然闪现出来说道。

“果然，我的机器上也能跑了~”

“握草，以前搭一天的环境，我5分钟就搞定了。”

通过上面的对话，我们可以看出Docker的价值。Docker的出现不仅解决了我们运行环境不一致所导致的问题，同时它还可以帮助我们轻松共享我们的工作容器，让他人通过我们共享的容器轻松地创建一个相同功能的容器，节省了我们在环境搭建上所花费的时间🚀。既然Docker这么好用，那我们就开始学习吧！

本文通过创建一个能执行初始化sql脚本的mysql容器为例，帮助大家学习docker基础知识。文章从环境搭建、常用指令解析然后一步步过渡到实战演练，并最终完成我们的目标。其中实战分为三个部分，从基于基础镜像创建容器，到学会使用卷挂载进行数据共享创建容器，再到基于Dockerfile生成自己的镜像，通过自己的镜像创建容器。最后还介绍了，如何将一个镜像传送到dockerhub仓库供别人使用。

下图是个人对docker的常用指令的一个总结~

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f96d7f7a47bf47b191a0f3f956fba106~tplv-k3u1fbpfcp-watermark.image)

另外，学习Docker之前，我们还需要了解一点点Linux知识！

## 1.环境搭建
#### 1. 安装虚拟机
常用的虚拟机有：VMware Workstation、VirtualBox等，本教程安装的是VMware Workstation。

#### 2. 安装linux系统
下载linux系统镜像，通过镜像安装系统，本教程使用的是Ubuntu镜像。

#### 3. 配置Ubuntu的基础环境
- 设置root密码：使用以下命令可以设置root密码。
```shell
sudo passwd
```
- 查看系统shell：Ubuntu默认shell采用的是dash，可以用如下命令查看系统的shell。
```shell
ls -l /bin/sh
```
- 切换系统shell：
```shell
sudo dpkg-reconfigure dash
```
 运行命令，输入root账号的密码后出现如下界面，通过方向键选择No后按下回车，便可完成bash切换。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a177f0bf6e594df3bf7cdb93ba71b767~tplv-k3u1fbpfcp-watermark.image)

通过上面提到的查看系统shell命令可以查看shell是否切换成bash。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df49b26dee2d4008b8d690c6e31cb0e7~tplv-k3u1fbpfcp-watermark.image)

#### 4.安装Docker
- 安装curl依赖：

```linux
sudo apt-get install curl
```
- 安装Docker：

```linux
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
## 2.镜像
**镜像（Image）**：好比是一个只读的模板，而作为一名程序员，则可以将镜像理解为类。

#### 1.搜索镜像<span id="search"></span>：`docker search 镜像名`

```linux
docker search imageName
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a16629f4b3414db0abd03dcc93c8b0cb~tplv-k3u1fbpfcp-watermark.image)

#### 2.拉取/推送镜像<span id="pull"></span>：`docker pull/push 镜像名`
```linux
docker pull NAME:TAG
```

#### 3.查看镜像<span id="images"></span>： `docker images [镜像名]`
镜像名是可选参数，表示按名称过滤。

使用以下命令可查看本地已下载的所有镜像。
```linux
docker images
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33183267bd054cb1891662d2188590e7~tplv-k3u1fbpfcp-watermark.image)

#### 4.删除镜像<span id="rmi"></span>： `docker rmi [镜像名/镜像ID...]`
该命令可以删除多个镜像，多个镜像之间用空格隔开。

使用以下命令可删除所有镜像。
```linux
docker rmi -f $(docker image ls)
```

#### 5.使用 Dockerfile 创建镜像<span id="build"></span>： `docker build -t 镜像名 -f Dockerfile路径 .` 
-f :指定要使用的Dockerfile的路径，不使用此可选参数，则使用当前目录下的Dockerfile创建镜像；

-t，-tag: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签；

不要忘记最后的.哦~~~
```linux
docker build -t NAME:TAG -f Dockerfile路径 .
```

#### 6.通过容器生成镜像<span id="commit"></span>： `docker commit 容器id 镜像名` 
```linux
docker commit containerId imageNAME
```
#### 7.重命名镜像<span id="tag"></span>： `docker tag 源镜像Tag 目标镜像Tag` 

给本地镜像打一个标记（tag），可将其归入某一仓库。打了新的 TAG 虽然会多了一条记录，但是 IMAGE ID 不会变，他们还是同一个镜像。
```linux
docker tag 源镜像Tag 目标镜像Tag`
```
## 3.容器
**容器（Container）**：基于镜像创建的, 独立运行的一个或一组应用，可以看成是镜像运行时的实体。容器可以被创建、启动、停止、删除等。

#### 1.创建并运行容器<span id="run"></span>：`docker run -itd --name 容器名 -p 主机(宿主)端口:容器端口 -v 主机文件路径:/容器文件路径 -e 环境变量 镜像名`

-i: 以交互模式运行容器，通常与 -t 同时使用。

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用。

-d: 后台运行容器，并返回容器ID。

--name 容器名: 为容器指定一个名称。

-p 主机(宿主)端口:容器端口：指定端口映射。

-v 主机文件路径:/容器文件路径:绑定一个卷，也就是将容器内的某个文件或者某个文件夹绑定到宿主机的某个文件或者文件夹，卷不仅共享，而且同步，修改其中任意一个卷，另外的卷立马作出相应变化。

-e 环境变量: 设置环境变量。

镜像名: 基于哪个镜像创建的。

注意：如果只创建不运行容器的话，可以用create命令代替run命令，create命令的参数和run一样，之后可以通过start命令启动。

#### 2.启动/停止/重启<span id="start"></span>：`docker start/stop/restart 容器名/容器id`

#### 3.查看所有容器<span id="ps"></span>：`docker ps -a`

通过以上命令返回如下信息。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc591c0e143f4787b0c8962973e4b89f~tplv-k3u1fbpfcp-watermark.image)
返回信息详情介绍如下：

CONTAINER ID: 容器 ID。

IMAGE: 使用的镜像。

COMMAND: 启动容器时运行的命令。

CREATED: 容器的创建时间。

STATUS: 容器状态。状态有7种：created（已创建）
restarting（重启中）
running（运行中）
removing（迁移中）
paused（暂停）
exited（停止）
dead（死亡）。

PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。

NAMES: 自动分配的容器名称。

#### 4.查看容器日志<span id="logs"></span>:`docker logs -f --tail 行数 容器名/容器id`

-f：跟踪实时日志。

--tail 行数：从日志末尾显示多少行日志， 默认是all。

#### 5.删除容器<span id="rm"></span>:`docker rm 容器名/容器id`

注意： **删除容器前，需先停止容器。**

#### 6.进入容器内部<span id="exec"></span>:`docker exec -it 容器名/容器id bash`
可以在[创建并启动容器的命令](#run)后也加上bash，从而达到容器启动后直接进入容器的目的。

#### 7.退出容器<span id="exit"></span>:`exit`

使用exit命令可从容器内部退出。

## 4.Dockerfile常用指令<span id="Dockerfile"></span>

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e979565927944b6a8c07dafc146eb15e~tplv-k3u1fbpfcp-watermark.image)

#### RUN、CMD、ENTRYPOINT的对比
RUN 指令是在容器被构建时运行的命令；CMD 、 ENTRYPOINT 是启动容器时执行 shell 命令。

CMD  会被 docker run 命令覆盖，但是  ENTRYPOINT 不会被覆盖。事实上，docker run 命令指定的任何参数都会被当作参数再次传递给 ENTRYPOINT 指令。CMD 、 ENTRYPOINT 两个指令之间也可以一起使用。
#### AND、COPY的对比
ADD 、 COPY 指令用法一样，唯一不同的是 ADD 支持将归档文件（tar, gzip, bzip2, etc）做提取和解压操作。注意的是，COPY 指令需要复制的目录一定要放在 Dockerfile 文件的同级目录下

## 5.实战演练：docker容器方式启动一个mysql
本栗子演示基于mysql:5.7镜像创建并运行mysql的过程。
首先我们输入命令 `su`，然后输入root账户的密码，将用户切至root账户，这样可避免我们在docker命令前加sudo，使我们接下来的操作更加便捷。

### 1.搭建基础版mysql
首先我们基于mysq:5.7镜像生成一个空数据库。

[搜索镜像](#search)：搜索mysql:5.7的镜像，命令如下：
```docker
docker search mysql:5.7
```
[拉取镜像](#pull)：拉取mysql:5.7的镜像到本地，命令如下：
```linux
docker pull mysql:5.7
```

[创建并运行容器](#run)：创建并启动了一个基于mysql:5.7镜像创建的且容器名为mysql-test的mysql应用，这个mysql应用root账号的初始密码是123456，暴露的端口为3306，命令如下：

```linux
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```
[查看容器转态](#ps)：通过以下命令，我们可以查看所有容器的状态
```linux
docker ps -a
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc591c0e143f4787b0c8962973e4b89f~tplv-k3u1fbpfcp-watermark.image)
在status栏我们可以看出容器已运行两分钟。

我们也可以用mysql连接工具来测试一下这个mysql应用是否正常运行。首先通过 `ifconfig` 命令查出宿主机的ip，然后输入端口为3306，账号为root，密码为123456进行测试。

### 2.使用挂载卷创建并运行一个有初始化数据的mysql

我们连接上容器内的mysql，然后可以创建一个数据库，并在其中建一些表和数据，最后导出成sql脚本，此处案例导出的脚本文件名为test.sql。

接下来，我们需思考怎样让创建容器的时候同时运行这个初始化脚本呢？

我们通过以下命令[进入容器](#exec)（mysql-test）内部：
```linux
docker exec -it mysql-test bash
```
然后输入命令` ls -al`，查看容器中的目录及文件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a389e5efb8dc408b9ffd452d39eaf28b~tplv-k3u1fbpfcp-watermark.image)

我们发现容器中有一个/**docker-entrypoint-initdb.d** 目录, **此目录可用于执行初始化脚本, 且执行顺序是按照文件名字母顺序执行**。 因此我们**只要挂载一个宿主机目录到容器中目录 /docker-entrypoint-initdb.d**，然后**在挂载宿主机目录中添加自己的初始化脚本**即可。 执行一次的保证是，容器启动会检查/var/lib/mysql 目录下是否有数据，如果mysql不是第一次启动，那么该文件夹下就有数据，/docker-entrypoint-initdb.d 目录中脚本就不会执行；如果mysql第一次启动，由于没有数据，所以/var/lib/mysql内容为空，初始化脚本就会执行。当然，由于/var/lib/mysql 目录一般是挂载在宿主机器上的，所以清空该目录也可触发初始化脚本执行。

接下来我们删除该容器，创建一个同名的mysql-test容器，然后通过添加挂载卷参数启动容器。

首先我们通过 `exit` 命令退出容器。

接下来我们在删除容器之前需要用[停止容器](#start)命令，停止mysql-test容器，命令如下:
```linux
docker stop mysql-test
```
然后再通过[删除容器](#rm)命令，删除mysql-test容器，命令如下：
```linux
docker rm mysql-test
```
紧接着我们通过 `cd /home` 命令，将目录切到/home，我们在该目录下创建出一个 /home/mysql/init-data 的目录，然后将test.sql文件复制到该目录下，准备好宿主目录，我们接下来使用[创建并运行容器](#run)命令，创建并启动了一个基于mysql:5.7镜像创建的且容器名为mysql-test的mysql应用，这个mysql应用root账号的初始密码是123456，暴露的端口为3306，并将宿主机上/home/mysql/init-data/下的文件挂载到了容器内的/docker-entrypoint-initdb.d目录下，命令如下：
```linux
docker run -itd --name mysql-test -p 3306:3306 -v /home/mysql/init-data/:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```
我们通过以下命令[进入容器](#exec)（mysql-test）内部：
```linux
docker exec -it mysql-test bash
```
然后切至/docker-entrypoint-initdb.d目录下，发现该目录下存在test.sql文件，说明文件挂载成功。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/647c6d0b2a14416f99d44f68387c81d3~tplv-k3u1fbpfcp-watermark.image)
通过用mysql连接工具，进入mysql应用内部，发现数据库中的表和数据均初始化。

### 3.通过Dockerfile重新打包镜像，创建并运行一个有初始化数据的mysql

我们一个启动命令这么复杂，我们能不能简化启动命令，把一些固定的东西交给脚本去做了？

这里我就使用Dockerfile脚本打包成新的镜像，然后基于新的镜像，创建一个mysql容器。

首先我们通过`cd /home`切换至/home目录下，在该目录下创建Dockerfile目录，同时在Dockerfile目录下新建一个[Dockerfile](#Dockerfile)文件，然后在文件里写入如下脚本。
```linux
FROM mysql:5.7
ENV MYSQL_ROOT_PASSWORD=123456
COPY ./init-data /docker-entrypoint-initdb.d
```
这里我们采用COPY命令将宿主机器Dockerfile目录下的init-data文件夹下的文件（因此我们还需在Dockerfile目录下创建一个init-data文件夹，将test.sql放入其中）复制到容器的/docker-entrypoint-initdb.d文件夹下，不用卷挂载是由于Dockerfile脚本只支持匿名挂载。

于是我们通过[Dockerfile生成镜像命令](#build)，生成一个新的镜像，这里我们取名镜像为mysql:v1，命令如下：
```linux
docker build -t mysql:v1 .
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9bc13ca2c634276ae84db9e17a33ea1~tplv-k3u1fbpfcp-watermark.image)

镜像生成成功后，我们可以通过[查看镜像命令](#images)，查看镜像状态，命令如下：
```linux
docker images
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a59167811e8413daa53fb55796536dc~tplv-k3u1fbpfcp-watermark.image)

接下我们就可以基于mysql:v1镜像，[创建并运行一个容器](#run)mysql-test1，此时我们创建的容器就不需要带入环境变量和卷挂载参数了，命令如下：
```linux
docker run -itd --name mysql-test1 -p 3306:3306 mysql:v1
```
我们这个容器mysql-test1同样也实现了mysql数据库的初始化。

## 6.上传镜像到dockerhub

最后我们将mysql:v1镜像上传到dockerhub。
首先我们得在[dockerhub](https://hub.docker.com/)上注册一个账号，然后我们通过以下命令进行登录。
```linux
docker login
```
输入用户和密码后，提示登录成功。
然后我们再用`docker push`命令将镜像推送至仓库。但是推送至dockerhub的镜像是有规范的，规范是：

```linux
docker push 注册用户名/镜像名
```
于是我们先需要用[`docker tag`命令](#tag)将mysql:v1镜像改名，命令如下：
```linux
docker tag mysql:v1 zqhexor/mysql:v1
```
改名完成后，我们就可以使用`docker push`命令，将镜像推送至dockerhub仓库。
```linux
docker push zqhexor/mysql:v1
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adf5cb4f4ea44b0da31d4583a3d1bcfc~tplv-k3u1fbpfcp-watermark.image)

推送成功后，我们可以在dockerhub中看到我们上传的镜像。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d64ec5d2f974313b7e71194eebda2a5~tplv-k3u1fbpfcp-watermark.image)

完结~👻
