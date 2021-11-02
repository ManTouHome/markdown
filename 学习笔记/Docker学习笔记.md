# Docker学习笔记:

## 1.docker安装

​	注册中心:[https:www.hub.docker.com]()

1.yum包更新到最新

```linux
sudo yum update
```

 2.安装需要的软件包，yum-util提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```linux
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

3.设置yum源为阿里云

```linux
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4.安装docker(docker-ce是社区版免费，docker-ee是企业版收费)

```linux
sudo yum install docker-ce
```

5.查看docker版本

```linux
docker -v
```

6.设置ustc的镜像

ustc是docker镜像的加速器

[https://lug.ustc.edu.cn/wiki/mirrors/help/docker]()

编辑该文件(先创建该目录和文件再编辑):

```linux
vi /etc/docker/daemon.json
```

再该文件中输入如下内容:

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

## 2.Docker的启动与停止

1，启动docker

```linux
systemctl start docker
```

2.停止docker

```linux
systemctl stop docker
```

3.重启docker

```linux
systemctl restart docker
```

4.查看docker的状态

```linux
systemctl status docker
```

5.开机启动

```linux
systemctl enable docker
```

6.查看docker概要信息

```linux
docker info
```

7.查看docker帮助文档

```linux
docker --help
```

## 3.镜像相关的命令

1.查看镜像

```docker
docker images
```

REPOSITORY：镜像名称

TAG：镜像标签

IMAGE ID：镜像ID

CREATED：镜像创建日期

SIZE：镜像大小

镜像存储位置：/var/lib/docker

2.搜索镜像

```docker
docker search 镜像名称
```

NAME：仓库名称

DESCRIPTION：镜像描述

STARS:用户评价，反应一个镜像的受欢迎程度

OFFICIAL：是否官方

AUTOMATED：自动构建

3.拉取镜像

```docker
docker pull 镜像名称
```

4.删除镜像

```docker
docker rmi 镜像ID
```

   删除所有的镜像

```docker
docker rmi `docker images` -q
```

## 4.容器相关的命令

### 1.查看容器

1.查看正在运行的容器

```docker
docker ps
```

2.查看所有容器

```docker
docker ps -a
```

3.查看最后一次运行的容器

```docker
docker ps -l
```

4.查看停止的容器

```docker
docker ps -f stasus=exited
```

### 2.创建容器

创建容器常用的参数说明:

创建容器的命令：

```docker
docker run
```

-i:表示运行容器

-t:表示容器启动后进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。

--name:为创建的容器命名。

-v:表示目录映射关系（前者为宿主机目录，后者是映射到宿主机上的目录），可以使用多个-v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。

-d:创建一个守护式容器在后台运行

-p:表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射。

1.交互式方式创建容器

```docker
docker run -it --name=容器名称 镜像名称：标签 /bin/bash
```

退出容器

exit

交互式创建容器退出后容器自动关闭

2.守护式创建容器

```docker
docker run -id --name=容器名称 镜像名称：标签
```

登录守护式容器：

```docker
docker exec -it 容器名称（或容器ID） /bin/bash
```

查看容器中linux版本

```docker
cat /etc/issue
```

### 3.停止与启动容器

1.停止容器

```docker
docker stop 容器名称(或者容器ID)
```

2.启动容器

```docker
docker start 容器名称（或容器ID）
```

### 4.文件拷贝

1.将文件拷贝到容器内

```docker
docker cp 需要拷贝的文件或目录 容器名称：容器目录
```

2.将文件从容器中拷贝出来

```docker
docker cp 容器名称：容器目录 需要拷贝的文件或目录
```

### 5.目录挂载

将宿主机的目录和容器内的目录进行映射，修改宿主机某个目录的文件从而去影响容器。

创建容器加上-v参数

```docker
docker run -id -v 宿主机目录:容器目录 --name=容器名称 镜像名称：标签
```

如果是多级目录可能会出现权限不足的提示，需要加上参数 --privileged=true

### 6.查看容器ip地址

1.查看容器运行的各种数据

```docker
docker inspect 容器名称（容器ID）
```

2.查看容器的ip地址

```docker
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```

###  7.删除容器

删除指定的容器

```docker
docker rm 容器名称（容器ID）
```

## 5.应用部署

### 1.MySQL部署

1.拉取镜像

```docker
docker pull centos/mysql-57-centos7
```

2.创建容器

```docker
docker run -id --name=tensquare_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
```

-p 代表端口映射，格式: 宿主机映射端口:容器运行端口

-e 代表添加环境变量  MYSQL_ROOT_PASSWORD 是root用户的密码

3.进入mysql容器

```docker
docker exec -it tensquare_mysql /bin/bash
#如果遇到docker容器中无法输入中文的问题则加上参数
docker exec -it mantou_mysql env LANG=C.UTF-8 /bin/bash
```

4.登录mysql

```mysql
mysql -u root -p
```

5.远程登录mysql

连接宿主机IP，端口是宿主机的映射端口

**注意**：用docker运行mysql后远程连接mysql出现SQLYOG:2058/navicat:1251的错误。

**错误原因**: 客户端不支持caching_sha2_password的加密方式

**解决办法**:

1.进入mysql容器去执行 select user,host,plugin from mysql.user where user = 'root';查看一下

2.你会发现plugin是caching_sha2_password，我们把它换成mysql_native_password加密方式就完事了

执行如下两条sql:

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'mantou'; 
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'mantou';
3.别忘了执行命令flush privileges使权限配置项立即生效，问题轻松解决。

### 2.tomcat部署

1.拉取镜像

```docker
docker pull tomcat:7-jre7
```

2.创建容器

```docker
docker run -id --name=mytomcat -p 9000:8080 -v /usr/local/webapps:/usr/local/tomcat/webapps tomcat:7-jre7
```

### 3.Nginx部署

1.拉取镜像

```docker
docker pull nginx
```

2.创建nginx容器

```docker
docker run -id --name=mynginx -p 80:80 nginx
```

### 4.Redis部署

1.拉取镜像

```docker
docker pull redis
```

2.创建容器

```docker
docker run -id --name=myredis -p 6379:6379 redis
```

3.需要挂载配置文件的方式

```docker
docker run -d --privileged=true -p 6379:6379 -v /docker/redis/conf/redis.conf:/etc/redis/redis.conf -v /docker/redis/data:/data --name redis-test redis redis-server /etc/redis/redis.conf --appendonly yes
```

参数说明：

--privileged=true：容器内的root拥有真正root权限，否则容器内root只是外部普通用户权限

-v /docker/redis/conf/redis.conf:/etc/redis/redis.conf：映射配置文件

-v /docker/redis/data:/data：映射数据目录

redis-server /etc/redis/redis.conf：指定配置文件启动redis-server进程

--appendonly yes：开启数据持久化

## 6.迁移与备份

1.容器保存为镜像

```docker
docker commit 容器名称 镜像名称
```

2.镜像备份(保存为tar文件)

格式: docker save -o 压缩文件名 镜像名称     (-o表示output)

```docker
docker save -o mynginx.tar mynginx_i
```

3.镜像恢复与迁移

```docker
docker load -i 镜像压缩文件名
```

## 7.Dockerfile

***Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。***

### 1.常用命令

|                命令                |                             作用                             |
| :--------------------------------: | :----------------------------------------------------------: |
|        FROM image_name:tag         |              定义了使用哪个基础镜像启动构建流程              |
|        MAINTAINER user_name        |                       声明镜像的创建者                       |
|           ENV key value            |                   设置环境变量(可以写多条)                   |
|            RUN command             |             是Dockerfile的核心部分（可以写多条）             |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file |            和ADD相似，但是如果又压缩文件不能解压             |
|          WORKDIR path_dir          |                         设置工作目录                         |

### 2.使用脚本创建对象

1.创建目录

```linux
mkdir -p /usr/local/dockerjdk8
```

2.下载jdk-8u271-linux-x64.tar.gz并上传到服务器中的/usr/local/dockerjdk8目录

3.创建Dockerfile    vi  Dockerfile

```dockerfile
#依赖镜像的名称和ID
FROM ansible/centos7-ansible
#指定镜像创建者信息
MAINTAINER ManTou
#切换工作目录
WORKDIR /usr
RUN mkdir /usr/local/java
#ADD 是相对路径jar，把java添加到容器中
ADD jdk-8u271-linux-x64.tar.gz /usr/local/java/
#配置Java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_271
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSHOME $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
```

4.执行命令构建镜像

```docker
docker build -t='jdk1.8' .
```

5.查看镜像是否建立完成

```docker
docker images
```

## 8.Docker私有仓库

### 1.私有仓库搭建与配置

1.拉取私有仓库镜像

```docker
docker pull registry
```

2.启动私有仓库容器

```docker
docker run -id --name=registry -p 5000:5000 registry
```

3.打开浏览器输入地址: 宿主机IP:5000/v2/_catalog

​	返回{"repositories":{}} 表示私有仓库搭建成功并内容为空。

4.修改daemon.json

```linux
vi /etc/docker/daemon.json
```

​	添加一下内容，保存退出:

```txt
{"insecure-registries":["宿主机IP:5000"]}
```

​	用于让docker信任私有仓库地址

5.重启docker服务

```linux
systemctl restart docker
```

​	扩展:默认情况下重启docker，docker中的容器会自动停止，如果想容器重启自动运行则在创建容器的时候加上--restart=always参数。

```docker
docker container update --restart=always 容器名字
```

### 2.镜像上传到私有仓库

1.标记此镜像为私有仓库的镜像

```docker
docker tag jdk1.8 宿主机ip:5000/jdk1.8
```

2.上传标记的镜像

```docker
docker push 宿主机ip:5000/jdk1.8
```



