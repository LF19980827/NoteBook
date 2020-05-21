

# 1、Docker简介

## 		1.1 Docker的定义

​	定义：解决了运行环境和配置问题软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。

​	比较Docker和传统虚拟化方式的不同之处:

- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程。而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。
- 每个容器之间互相隔离，每个容器有自己的文件系统，容器之间进程不会相互影响，能区分计算资源。

## 1.2 Docker的基本组成

Docker的架构图

![](G:\个人数据\笔记\NoteBook\Docker\img\1.1.png)

Docker三要素：镜像、容器、仓库

### 1.2.1 镜像（image）

​	Docker镜像(Image) 就是一-个只读的模板。镜像可以用来创建Docker容器，一 个镜像可以创建很多容器。容器与镜像的关系类似于类与对象

| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |

### 1.2.2 容器（container）

​	Docker利用容器(Container) 独立运行的一个或一组应用。 容器是用镜像创建的运行实例。

​	可以把容器看做是一个简易版的Linux环境(包括root用户权限、进程空间、用户空间和网络空间等)和运行在其中的应用程序。

### 1.2.3 仓库（repository）

​	仓库(Repository)是集中存放镜像文件的场所。I
​	仓库(Repository)和仓库注册服务器(Registry) 是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag) 。
​	仓库分为公开仓库(Public) 和私有仓库(Private) 两种形式。

### 1.2.4 总结

​	Docker本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成-一个可交付的运行环境，这个打包好的运行环境就似乎image镜像文件。只有通过这个镜像文件才能生成Docker容器。image文件可以看作是容器的模板。Docker根据image文件生成容器的实例。	同一个image文件，可以生成多个同时运行的容器实例。

​	image文件生成的容器实例，本身也是一个文件，称为镜像文件。

​	一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器至于仓储，就是放了一堆镜像的地方，我们可以把镜像发布到仓储中，需要的时候从仓储中拉下来就可以了。

# 2、Docker安装

## 2.1 deepin下安装

```shell
 #运行,这样可以把需要的包进行安装。
 sudo apt install docker*
 
 #查看版本
 docker version
```

 

## 2.2 配置阿里云镜像加速

1. https://dev.aliyun.com/search.html
2. 注册一个属于自己的阿里云账户(可复用淘宝账号)
3. 获得加速器地址连接（登陆阿里云开发者平台口，获取加速器地址口）
4. 配置本机Docker运行镜像加速器口
5. 重新启动Docker后台服务: service docker restart
6. Linux系统下配置完加速器需要检查是否生效

## 2.3 hello-world

```shell
docker run hello-world
```

run操作的流程

![](G:\个人数据\笔记\NoteBook\Docker\img\2.1.png)

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上，然后通过Socket连接 从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。

# 3、Docker常用命令

## 3.1 帮助命令

```shell
#查看版本
docker version

#查看详细信息
docker info

#帮助手册
docker --help
```

## 3.2 镜像命令

**1.docker images [options]** 

```shell
#列出本地主机上的镜像
docker images
```

（1）列表表头说明

- REPOSITORY：表示镜像的仓库源
- TAG：镜像的标签
- IMAGE ID：镜像ID
- CREATED：镜像创建时间
- SIZE：镜像大小

（2）options说明

- -a：列出本地所有的镜像(含中间映像层)
- -q：只显示镜像ID。
- --digests：显示镜像的摘要信息
- --no-trunc：显示完整的镜像信息

**2.docker search [options] xxx** 

```shell
#在docker hub上查找xxx的镜像信息，例如
docker search tomcat
```

(1)options说明

- --no-trunc：显示完整的镜像描述
- -s：列出收藏数不小于指定值的镜像
- --automated：只列出automated bulid类型的镜像

**3.docker pull xxx [:TAG]**

```shell
#下载镜像,TAG为版本，默认为lastest（最新）
docker pull tomcat
```

**4.docker rmi xxx [:TAG]**

```shell
#删除镜像，TAG与上相同
docker rmi tomcat

#删除多个镜像,-f为强制删除的意思
docker rmi -f tomcat hello-world

#删除全部镜像
docker rmi -f $(docker images -qa)
```

 

## 3.3 容器命令

### 3.3.1 基础命令

**1.新建并启动容器**

```shell
docker [OPTIONS] IMAGE[COMMAND][ARG]

#例1：启动id为d94da71c3a1f的镜像
docker run -it d94da71c3a1f

#例2：启动一个centos容器,名字为mycentos
docker run -it --name mycentos centos
```

(1)options说明

- --name="容器新名字"：为容器指定一个名称
- -d：后台运行容器，并返回容器ID， 也即启动守护式容器
- -i：以交互模式运行容器，通常与-t同时使用
- -t：为容器重新分配-一个伪输入终端，通常与-i同时使用
- -P：随机端口映射
- -p：指定端口映射，有以下四种格式
  - ip:hostPort:containerPort
  - ip::containerPort
  - hostPort:containerPort
  - containerPort

```shell
#对外暴露的端口是8888，内部对应端口8080。
#即开启后，访问localhost:8888会显示tomcat页面
docker run -it -p 8888:8080 tomcat
```

**2.列出所有正在运行的容器**

```shell
docker ps [OPTIONS]
```

(1)options说明

- -a：列出当前所有正在运行的容器+历史上运行过的
- -l：显示最近创建的容器。
- -n：显示最近n个创建的容器。
- -q：静默模式，只显示容器编号。
- --no-trunc ：不截断输出。

**3.退出容器**

```shell
#容器停止退出
exit

#容器不停止退出
ctrl+P+Q
```

**4.启动容器**

```shell
docker start 容器ID或容器名

#例
docker start mycentos
```

**5.重启容器**

```shell
docker restart 容器ID或容器名

#例
docker restart mycentos
```

**6.停止容器**

```shell
docker stop 容器ID或容器名

#例
docker stop mycentos
```

**7.强制停止容器**

```shell
docker kill 容器ID或容器名

#例
docker kill mycentos
```

**8.删除已停止容器**

```shell
#删除一个容器
docker rm -f 容器ID或容器名

#删除多个容器1
docker rm -f $(docker ps -a -q)

#删除多个容器2
docker ps -a -q | xargs docker rm
```

### 3.3.2 重要命令

**1.启动守护式容器**

```shell
#使用镜像centos:latest以后台模式启动一个容器
docker run -d 容器名
```

​	问题:使用docker ps -a进行查看,会发现容器已经退出

​	**Docker容器后台运行,就必须有一个前台进程**

​	容器运行的命令如果不是那些一直挂起的命令(比如运行top,tail)，就是会自动退出的。这个是docker的机制问题。所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行

**2.查看容器日志**

```shell
docker logs -f -t --tail 容器id
```

(1)options说明

- -t是加入时间戳
- -f跟随最新的日志打印
- --tail数字显示最后多少条

**3.查看容器内运行的进程**

```shell
docker top 容器id
```


**4.查看容器内部细节**

```shell
docker inspect 容器id
```


**5.进入正在运行的容器并以命令行交互**

```shell
#exec是在容器中打开新的终端，并且可以启动新的进程（直接进行想要的操作）
docker exec -it 容器id bashShell（操作）

#exec比attach强大,下面命令相当于attach（进入原来(#`O′未停止的容器）
docker exec -it 容器id /bin/bash

#attach直接进入容器启动命令的终端，不会启动新的进程（重新进入容器，然后执行想要的操作）
docker attach 容器id
```




**6.从容器内拷贝文件到主机上**

```shell
docker cp 容器id:容器内路径 目的主机路径
```



**7.提交容器副本使之称为新的镜像**

```shell
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
```



### 3.3.3 命令大全[手册]

![](G:\个人数据\笔记\NoteBook\Docker\img\3.1.png)

# 4、Docker镜像

​	定义：镜像是一种轻量级、可执行的独立软件包，**用来打包软件运行环境和基于运行环境开发的软件**，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

​	特点：Docker镜像都是只读的。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

## 4.1 Docker镜像原理

**1.UnionFS联合文件系统（镜像的底层实现）**

​	UnionFS (联合文件系统) : Union文件系统(UnionFS) 是一种分层、轻量级并且高性能的文件系统，**它支持对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下。Union 文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像。

​	特性: 一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

**2.Docker镜像加载原理**

​	docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。
​	botfs主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载botfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之 后整个内核就都在内存中了，此时内存的使用权己由bootfs转交给内核，此时系统也会卸载bootfs。
​	rootfs (root file system)，在bootfs之 上。 包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs 就是各种不同的操作系统发行版，比如Ubuntu, Centos 等等。

​	对于一个精简的OS，rootfs可 以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel,自己只需要提供rootfs就行了。由此可见对于不同的linux发行版, bootfs 基本是一致的，rootfs 会有差别，因此不同的发行版可以公用bootfs。

**3.采用分层结构的原因**

​	最大的一个好处就是——**共享资源**
​	比如:有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

# 5、Docker容器数据卷

## 5.1 容器数据卷概述

​	类似我们Redis里面的rdb和aof文件。

​	卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：

- 数据卷可在容器之间共享或重用数据
- 卷中的更改可以直接生效
- 数据卷中的更改不会包含在镜像的更新中
- 数据卷的生命周期一直持续到没有容器使用它为止

作用：实现容器的持久化、实现容器间继承+共享数据

## 5.2 添加数据卷

​	在容器里添加数据卷的两种方法：直接命令添加、DockerFile添加

### 1.直接命令添加

```shell
 #容器停止退出后，主机修改后数据依然同步
 docker run -it -v /宿主机绝对路径目录:/容器内目录  镜像名
 
 #加入只读权限（容器只读，主机可读可写）
 docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名
 
 #查看数据卷是否挂载成功
 docker inspect 容器ID
```

注意：Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个--privileged=true参数即可

### 2.DockerFile添加

（1）创建Dockerfile脚本，名为dockerfile（在根目录下）

```shell
# volume test
#可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished,--------success1"
CMD /bin/bash
```

（2）使用build生成新镜像，名为lfuser/centos
```shell
docker build -f dockerfile -t lfuser/centos:1.3 .
```


（3）启动容器
```shell
docker run -it lfuser/centos:1.3
```


（4）查看是否挂载成功，并查看主机文件默认存储位置
```shell
docker inspect 容器ID
```



## 5.3 数据卷容器

​	定义：命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器

```shell
#让容器dc02继承dc01
docker run -it --name dc02 --volumes-from dc01 lfuser/centos
```

​	**容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止**



# 6、DockerFile解析

## 6.1 DockerFile简介

​	定义：Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。	

​	构建步骤：

1. 手动编写一个dockerfile文件，当然，必须要符合file的规范
2. 直接docker build命令执行，获得一个自定义的的镜像
3. run

## 6.2 DockerFile构建过程解析

### 1.基础知识

- 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
- 指令按照从上到下，顺序执行
- #表示注释
- 每条指令都会创建一个新的镜像层，并对镜像进行提交

### 2.Docker执行Dockerfile的大致流程

1. docker从基础镜像运行一一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

### 3.总结

​	Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

![](G:\个人数据\笔记\NoteBook\Docker\img\6.1.png)

- Dockerfile：需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。
- Docker镜像：Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务
- Docker容器：容器是直接提供服务的

## 6.3 DockerFile体系结构(保留字指令)

- FROM：基础镜像，当前新镜像是基于哪个镜像的
- MAINTAINER：镜像维护者的姓名和邮箱地址
- RUN：容器构建时需要运行的命令
- EXPOSE：当前容器对外暴露出的端口
- WORKDIR：指定 在创建容器后，终端默认登陆的进来工作目录，一个落脚点
- ENV ：用来在构建镜像过程中设置环境变量【ENV MY_PATH /usr/mytest 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样也可以在其它指令中直接使用这些环境变量】
- ADD：将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
- COPY：将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。类似ADD，拷贝文件和目录到镜像中。【两种方式：1.COPY src dest  2.COPY ["src", "dest"]】
- VOLUME：容器数据卷，用于数据保存和持久化工作
- CMD：指定一个容器启动时要运行的命令。Dockerfile中可以有多个CMD指令，但**只有最后一个生效**，CMD 会被dockerrun之后的参数替换
- ENTRYPOINT：指定一个容器启动时要运行的命令。ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数，但是**ENTRYPOINT会追加命令，而不是只生效最后一个**
- ONBUILD：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

## 6.4总结

![](G:\个人数据\笔记\NoteBook\Docker\img\6.3.png)



# 7、Docker常用安装

1.使用MySQL镜像

```shell
docker run -p 12345:3306 --name mysql 
-v /lfuser/mysql/conf:/etc/mysql/conf.d 
-v /lfuser/mysql/logs:/logs -v /lfuser/mysql/data:/var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=123456 
-d mysql:5.6
```

命令说明：

```
-p 12345:3306：将主机的12345端口映射到docker容器的3306端口。
--name mysql：运行服务名字
-v /lfuser/mysql/conf:/etc/mysql/conf.d ：将主机/lfuser/mysql录下的conf/my.cnf 挂载到容器的 /etc/mysql/conf.d
-v /lfuser/mysql/logs:/logs：将主机/lfuser/mysql目录下的 logs 目录挂载到容器的 /logs。
-v /lfuser/mysql/data:/var/lib/mysql ：将主机/lfuser/mysql目录下的data目录挂载到容器的 /var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。
-d mysql:5.6 : 后台程序运行mysql5.6

```



2.使用Redis镜像

```shell
docker run -p 6379:6379 
-v /lfuser/myredis/data:/data 
-v /lfuser/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf  
-d redis:3.2 redis-server /usr/local/etc/redis/redis.conf 
--appendonly yes
##--appendonly yes是打开了持久化文件aof
```

 在主机/lfuser/myredis/conf/redis.conf目录下新建redis.conf文件	

```shell
vim /lfuser/myredis/conf/redis.conf/redis.conf
```

​	测试redis-cli连接

```shell
docker exec -it 运行着Rediis服务的容器ID redis-cli
```



# 8、本地镜像发布到阿里云

1.本地镜像发布到阿里云流程

![](G:\个人数据\笔记\NoteBook\Docker\img\8.1.png)

​	将本地镜像推送到阿里云：

1. 前往阿里云开发者平台https://dev.aliyun.com/search.html
2. 创建仓库镜像
3. 将镜像推送到registry

```shell
#将镜像推送到registry
sudo docker login --username=registry.cn-hangzhou.aliyuncs.com

sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/lfuser/mycentos:[镜像版本号]
sudo docker push registry.cn-hangzhou.aliyuncs.com/lfuser/mycentos:[镜像版本号]

```

