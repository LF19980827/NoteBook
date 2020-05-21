# 1、Nginx概述

1.nginx简介

​	Nginx (" engine x")是一个高性能的HTTP和反向代理服务器,特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好。Nginx专为性能优化而开发，性能是其最重要的考量，实现上非常注重效率，能经受高负载的考验，有报告表明能支持高达50, 000个并发连接数。+

2.反向代理

- 正向代理：在客户端(浏览器)配置代理服务器，通过代理服务器进行互联网访问。
- 反向代理：我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。

3.负载均衡

​	单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡.

​	负载均衡策略

- 轮询(默认)：每个请求按时间顺序逐一分配到不同的后端服务器
- weight：weight代表权重默认为1,权重越高被分配的客户端越多
- ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器
- fair (第三方) ：按后端服务器的响应时间来分配请求，响应时间短的优先分配

4.动静分离

​	为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

5.高可用

​	Keepalived+Nginx实现高可用。

​	Keepalived是一个高可用解决方案，主要是用来防止服务器单点发生故障，可以通过和Nginx配合来实现Web服务的高可用。（其实，Keepalived不仅仅可以和Nginx配合，还可以和很多其他服务配合）

# 2、Nginx的安装

## 2.1 deepin系统安装

1. Nginx编译相关依赖库的安装

```shell
#安装gcc g++的依赖库
sudo apt-get install build-essential
sudo apt-get install libtool

#安装pcre依赖库（http://www.pcre.org/）
sudo apt-get install libpcre3 libpcre3-dev

#安装zlib依赖库（http://www.zlib.net）
sudo apt-get install zlib1g-dev

#安装SSL依赖库
sudo apt-get install openssl
```

2.Nginx下载编译安装

```shell
# 下载nginx-1.16.0.tar.gz源码包
wget http://nginx.org/download/nginx-1.16.0.tar.gz

# 解压nginx-1.16.0.tar.gz源码包
tar zxvf nginx-1.16.0.tar.gz

# 切换到nginx-1.16.0.tar.gz解压后的nginx-1.16.0/目录
cd nginx-1.16.0/

# 设置nginx的编译参数，可通过./configure --help命令查看nginx有哪些编译参数
./configure --prefix=/usr/local/nginx

# 编译
make

# 编译安装
sudo make install
```

3.验证是否安装成功

```shell
# 使用默认配置文件启动nginx
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

# 查看nginx的版本号
sudo /usr/local/nginx/sbin/nginx -v
```

 浏览器访问localhost，默认80端口 ,出现欢迎页则成功

## 2.2使用Docker安装

```shell
#查看nginx可用镜像
docker search nginx

#拉取最新版镜像
docker pull nginx:latest

#查看镜像
docker images

#运行容器
docker run --name nginx-test -p 8080:80 -d nginx

#--name nginx-test：容器名称。
#-p 8080:80： 端口进行映射，将本地 8080 端口映射到容器内部的 80 端口。
#-d nginx： 设置容器在在后台一直运行。
```



# 3、Nginx的常用命令和配置

## 3.1常用命令

```shell
#使用Nginx命令的前提，进入nginx的目录 /usr/local/nginx/sbin

#1.查看版本
./nginx -v

#2.启动nginx
./nginx

#3.关闭nginx
./nginx -s stop

#4.重新加载nginx
./nginx -s reload
```

## 3.2 配置文件

nginx常见配置文件：

| 配置文件     | 作用                              |
| ------------ | --------------------------------- |
| nginx.conf   | nginx的基本配置文件（主配置文件） |
| mime.types   | MIME类型关联的扩展文件            |
| fastcgi.conf | 与fastcgi相关的配置文件           |
| proxy.conf   | 与proxy相关的配置                 |
| sites.conf   | 配置nginx提供的网站，包括虚拟主机 |

nginx.conf配置文件详细解析可参考https://blog.csdn.net/tjcyjd/article/details/50695922

nginx配置文件位置：/usr/local/nginx/conf/nginx.conf

配置文件的组成

- main配置段：全局配置段，其中main配置段中可能包含event配置段。从配置文件开始到events 块之间的内容,主要会设置一些影响 nginx服务器整体运行的配置指令
- event{}：定义event模型工作特性，主要影响，nginx服务器与用户的网络连接
- http{}：定义http协议相关配置。
  - http全局块：http全局块配置的指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求数上限等。
  - server块：这块和虚拟主机有密切关系,虚拟主机从用户角度看,和一台独立的硬件主机是完全一样的,该技术的产生是为了节省互联网服务器硬件成本。