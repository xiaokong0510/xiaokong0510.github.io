---
title: Docker 基本操作
categories:
  - 工具
tags:
  - Docker
date: 2022-02-10
description: Docker 入门，先掌握一个操作镜像、容器的常用指令。
---

Docker 基本知识，包括：

- 镜像与容器的概念

- 安装与启动

- 镜像与容器相关命令

- docker 部署 Tomcat、Nginx 、MySQL、Redis


## 1. Docker简介

> 虚拟化

​	Virtualization，是一种资源管理技术，是将计算机的各种实体资源，如服务器、网络、内存及存储等，予以抽象、转换后呈现出来，打破实体结构间的不可切割的障碍，使用户可以比原本的组态更好的方式来应用这些资源。这些资源的新虚拟部份是不受现有资源的架设方式，地域或物理组态所限制。一般所指的虚拟化资源包括计算能力和资料存储。

​	在实际的生产环境中，虚拟化技术主要用来解决高性能的物理硬件产能过剩和老的旧的硬件产能过低的重组重用，透明化底层物理硬件，从而最大化的利用物理硬件   对资源充分利用

​	虚拟化技术种类很多，例如：软件虚拟化、硬件虚拟化、内存虚拟化、网络虚拟化(vip)、桌面虚拟化、服务虚拟化、虚拟机等等。

>什么是Docker

Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。

项目地址： https://github.com/docker/docker

特点：

1. 启动快；去除了管理程序的开销同一台宿主机中也可以运行更多的容器，尽可能的充分利用系统资源。
2. 职责的逻辑分类；加强开发人员写代码的开发环境与应用程序要部署的生产环境一致性。
3. 可移植性，易于构建，并易于协作。
4. 鼓励使用面向服务的架构，推荐单个容器只运行一个应用程序或进程，形成了一个分布式的应用程序模型。

> Docker 镜像与容器

Docker是一个客户端-服务器（C/S）架构程序。Docker客户端只需要向Docker服务器或者守护进程发出请求，服务器或者守护进程将完成所有工作并返回结果Docker提供了一个命令行工具Docker以及一整套RESTful API。可以在同一台宿主机上运行Docker守护进程和客户端，也可以从本地的Docker客户端连接到运行在另一台宿主机上的远程Docker守护进程

**镜像**： 将镜像当作容器的“源代码”。镜像体积很小，非常“便携”，易于分享、存储和更新。

**容器：** 类似于集装箱

**Registry： ** 镜像仓库，分为公共和私有两种。

Docker公司运营公共的Registry：Docker Hub。https://hub.docker.com/

也可以自己构建私有的Registry

## 2. 安装 Docker（yum安装）

### 2.1 安装步骤

1. 官网安装参考手册：https://docs.docker.com/install/linux/docker-ce/centos/

2. 确定是 CentOS7 及以上版本

   ```shell
   cat /etc/redhat-release  #查看版本
   CentOS Linux release 7.7.1908 (Core)
   ```

3. yum 安装 gcc 相关（需要确保 虚拟机可以上外网 ）

   ```shell
   yum -y install gcc
   yum -y install gcc-c++
   ```

4. 卸载旧版本

   ```shell
   $yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```
   
5. 安装需要的软件包

   ```shell
   yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

6. 设置stable镜像仓库

   ```shell
   # 推荐使用国内的阿里云
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```
   
7. 更新yum软件包索引

   ```shell
   yum makecache fast
   ```

8. 安装最新版本Docker Engine和容器

   ```shell
   yum -y install docker-ce docker-ce-cli containerd.io
   ```


9. 启动并测试 hello-world

   ```shell
   #启动
   systemctl start docker   
   #运行helloworld,会提示先在本地找没有这个镜像，然后拉取进镜像
   docker run hello-world
   ```

10. 设置ustc的镜像 

    ustc是老牌的linux镜像服务提供者：https://lug.ustc.edu.cn/wiki/mirrors/help/docker

    新建并编辑文件：

   ```shell
   vi /etc/docker/daemon.json  
   ```

在该文件中输入如下内容：

   ```shell
{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
   ```

或者注册一个自己的阿里云账号，配置阿里云中的镜像加速器

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["****"]
}
EOF
sudo systemctl daemon-reload 
sudo systemctl restart docker   #重启docker
```

11. docker相关操作

   ```shell
#启动docker
systemctl start docker  
#停止docker
systemctl stop docker	
#重启docker
systemctl restart docker 
#查看docker状态：
systemctl status docker
#设置开机启动：
systemctl enable docker  
# 帮助命令,#查看docker版本
docker verion 
#查看docker状态
docker info   
#查看docker帮助文档
docker --help 
   ```

### 2.2 镜像相关命令

#### 2.2.1 查看镜像

```shell
docker images
#列出本地所有镜像(含中间映像层)
docker images -a  
#只显示镜像ID
docker images -q  
#显示镜像的摘要信息
docker images --digests  
#显示完整的镜像信息
docker images --no-trunc  
```

- REPOSITORY：镜像名称， 唯一的
- TAG：镜像标签，如果不指定一个镜像的版本标签将默认使用latest 镜像
- IMAGE ID：镜像ID， 唯一的
- CREATED：镜像的创建日期（不是获取该镜像的日期）
- SIZE：镜像大小
- 这些镜像都是存储在Docker宿主机的/var/lib/docker目录下

#### 2.2.2 搜索镜像

```shell
#搜索某个镜像
docker search 镜像名称  
#搜索点赞数超过30的镜像
docker search -s 30 镜像名称   
```

- NAME：仓库名称

- DESCRIPTION：镜像描述

- STARS：用户评价，反应一个镜像的受欢迎程度

- OFFICIAL：是否官方

- AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的

#### 2.2.3 拉取镜像

```shell
 #拉取某个镜像，等价于： docker pull 镜像名称：latest
docker pull 镜像名称  
```

- 还可以拉 dokcer pull centos ，只有不到200M的大小

#### 2.2.4 删除镜像

```shell
 #删除镜像，先删容器才能删镜像
docker rmi  镜像ID(或者镜像名)  
 #强制删除
docker rmi -f 镜像ID/(或者镜像名)  
#删除多个
docker rmi -f 镜像名 镜像名 镜像名
#组合命令，删除所有镜像
docker rmi -f $(docker images -q)    
```

### 2.3 容器相关命令

#### 2.3.1 查看容器

```shell
   #查看所有正在运行的容器
   docker ps 		
   #查看所有容器,运行过的或者正在运行的
   docker ps -a  	
   #查看最后一次运行的容器
   docker ps –l 	
   #查看最近n个创建的容器
   docker ps –n 数字	
   #静默模式，只显示容器编号
   docker ps –q 	
   #查看停止的容器
   docker ps -f status=exited  
```

#### 2.3.2 创建与启动容器

1. 创建容器命令：

```shell
docker run  [OPTIONS]  image  [COMMAND]
```

**[OPTIONS]说明：**

- --name="容器新名字"：为容器指定一个名称

- -d：后台运行容器，并返回去容器ID，即启动守护式容器（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。

- ==-i：以交互模式运行容器，常与-t同时使用==

- ==-t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。==

- -v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。

- -p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射

2. 交互式方式创建容器：

```shell
docker run -it --name=容器名称 镜像名称:标签 /bin/bash

exit  # 退出并关闭当前容器
Ctrl+Q+P   # 退出当前容器,并不停止
```
3. 守护式方式创建容器：

```shell
docker run -d --name=容器名称 镜像名称:标签

#登录守护式容器方式：
docker exec -it 容器名称 (或者容器ID)  /bin/bash
```

【注意】：此时docker ps -a 进行查看, 会发现容器已经退出。

Docker容器后台运行,就必须有一个前台进程。

容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。

4. 停止与启动容器

```shell
#停止容器：
docker stop 容器名称（或者容器ID）
#强制停止容器：
docker kill 容器名称（或者容器ID）
#启动容器：
docker start 容器名称（或者容器ID）
#重启容器：
docker restart 容器名称（或者容器ID）

```

5. 删除容器

```shell
  docker rm 容器ID  #，删除容器
  docker rm -f $(docker ps -aq)  #一次性删除所有容器
  docker pa -aq | xargs docker rm
```

6. 文件拷贝

```shell
#拷进
docker cp 需要拷贝的文件或目录 容器名称:容器目录
#拷出
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

### 2.4 镜像补充命令 

```shell
docker commit #提交容器副本使之成为一个新的镜像
docker commit -m="描述信息" -a="作者信息" 容器id 要创建的目标镜像名:[标签名]
```

### 2.5 Docker容器数据卷

#### 2.5.1 联合文件系统(Union File System)

​		Docker镜像被存储在一系列的只读层。当开启一个容器，Docker读取只读镜像并添加一个读写层在顶部。如果正在运行的容器修改了现有的文件，该文件将被拷贝出底层的只读层到最顶层的读写层。在读写层中的旧版本文件隐藏于该文件之下，但并没有被不破坏 - 它仍然存在于镜像以下。当Docker的容器被删除，然后重新启动镜像时，将开启一个没有任何更改的新的容器 - 这些更改会丢失。此只读层及在顶部的读写层的组合被Docker称为联合文件系统

#### 2.5.2 容器数据卷(Volumes)

​        DVolumes是目录（或者文件），它们是外部默认的联合文件系统或者是存在于宿主文件系统正常的目录和文件，**能够持久数据以及共享容器间的数据**。通俗地来说，docker容器数据卷可以看成是生活中常用的u盘，它存在于一个或多个的容器中，由docker挂载到容器，但不属于联合文件系统，Docker不会在容器删除时删除其挂载的数据卷。

特点：

- 数据卷可以在容器之间共享或重用数据

- 数据卷中的更改可以直接生效

- 数据卷中的更改不会包含在镜像的更新中

- 数据卷的生命周期一直持续到没有容器使用它为止

#### 2.5.3 添加数据卷

1. 命令添加，创建容器时在容器内添加数据卷：

   ```shell
   docker run -it -v /宿主机绝对路径:/容器内目录 镜像名
   ```

- 将宿主机的目录与容器内的目录进行映射，这样就可以通过修改宿主机某个目录的文件从而去影响容器。把容器关掉，修改宿主机的目录，再次启动容器时，也会同步。

- 如果共享的是多级的目录，可能会出现权限不足的提示。这是因为CentOS7中的安全模块selinux把权限禁掉了，需要添加参数  --privileged=true  来解决挂载的目录没有权限的问题。

- -v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现，因为并不能够保证在所有的宿主机上都存在这样的特定目录。

   ```shell
   docker inspect 容器id  #查看是否挂载成功
   ```

2. DockerFile添加：

   - 容器下新建一个文件夹mydocker，编辑Dockerfile文件，添加两个数据卷;
   - 执行，获得一个新的镜像;
   - 宿主机的默认路径可以用`docker inspect 容器id`指令查看
```shell
   #容器下新建一个文件夹mydocker，编辑Dockerfile文件，添加两个数据卷
   vim Dockerfile
   
   FROM centos    #来源于镜像centos
   VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]  #添加两个数据卷
   CMD echo "finished,--------success1" #容器卷运行成功提示
   CMD /bin/bash
   #---------------------------
   #以上指令相当于:
   docker run -it -v /host1:/dataVolumeContainer1 /host2:/dataVolumeContainer2 centos /bin/bash
   
   #执行，获得一个新的镜像
   docker bulid -f /mydocker/Dockerfile -t 自定义镜像名
   
   #宿主机的默认路径可以用docker inspect 容器id指令查看
```

   

### 2.6 Dockerfile

#### 2.6.1 概念

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。

1. 对于开发人员：可以为开发团队提供一个完全一致的开发环境； 
2. 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了； 
3. 对于运维人员：在部署时，可以实现应用的无缝移植

区分几个概念：

- Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;

- Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;

- Docker容器，容器是直接提供服务的。

#### 2.6.2 常用命令

| 命令                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| FROM image_name:tag                | 定义了使用哪个基础镜像启动构建流程                           |
| MAINTAINER user_name               | 声明镜像的创建者                                             |
| ENV key value                      | 设置环境变量 (可以写多条)                                    |
| RUN command                        | 是Dockerfile的核心部分(可以写多条)                           |
| EXPOSE                             | 当前容器对外暴露出的端口                                     |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压                      |
| VOLUME                             | 容器数据卷，用于数据保持和持久化工作                         |
| CMD                                | 指定一个容器启动时要运行的命令，Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替代 |
| ENTRYPOINT                         | 指定一个容器启动时要运行的命令，docker run之后的会追加       |
| WORKDIR path_dir                   | 设置工作目录                                                 |



### 2.7 centos镜像案例

```shell
FROM scratch    #依赖镜像名称和ID
MAINTAINER ...   #指定镜像创建者信息
ADD centos-7-x86_64-docker.tar.xz /    #执对容器作出修改

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20191001"

CMD ["/bin/bash"]
```

## 3. 应用部署

### 3.1 Tomcat 部署

```shell
docker search tomcat   #搜索Tomcat镜像
docker pull tomcat    #拉取tomcat镜像
docker run -it -p 8080:8080 tomcat  #交互式方式创建容器，-p表示端口映射，前者是宿主机端口，后者是容器内的映射端口
```

最新版本的webapp目录下没有页面，浏览器连接会显示404。

### 3.2 MySQL 部署

1. 拉取镜像

```SHELL
   docker pull mysql:5.7   #拉取mysql:5.7镜像
```

2. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql

docker run -p 12345:3306 --name mysql -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/logs:/logs -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 

docker exec –it 容器id /bin/bash  #进入容器，操作mysql
mysql -uroot -p123456  #进入mysql命令操作
```

3. 数据备份：

```shell
#所有的数据库文件导出到宿主机的/root/mysql/all-databases.sql文件备份
docker exec 容器ID sh -c ' exec mysqldump --all-databases -uroot -p"123456" ' > /root/mysql/all-databases.sql
```

### 3.3 Redis 部署

1. 搜索redis镜像

```shell
docker search redis
```

2. 拉取redis镜像

```shell
docker pull redis:3.2
```

3. 创建容器，设置端口映射

```shell
docker run -p 6379:6379 -v /root/redis/data:/data -v /root/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis:3.2 redis-server /usr/local/etc/redis/redis.conf --appendonly yes  #开启aof
```

4. 测试连接

```shell
docker exec -it 容器ID  redis-cli
```

### 3.4 Ngnix 部署

1. 搜索nginx镜像

```shell
docker search nginx
```

2. 拉取nginx镜像

```shell
docker pull nginx
```

3. 创建容器，设置端口映射、目录映射


```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
vim nginx.conf
```
```shell

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


```


```shell
docker run -id --name=c_nginx -p 80:80  -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/conf/logs:/var/log/nginx -v /root/nginx/conf/html:/usr/share/nginx/html nginx
```

4. 使用外部机器访问 nginx

