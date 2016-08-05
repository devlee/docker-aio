Docker On Ubuntu
====

本文主要介绍了[**Docker**](https://www.docker.com)在[**Ubuntu**](http://www.ubuntu.com/)上的主要应用，并详细的阐述了实现步骤、遇到的问题及问题的解决方案。新手可以先了解一下它们，老鸟可以直接跳过下面的若干步骤，废话不多说，开始吧~

###环境参数

> **NOTE:** 

> - **ubuntu** [16.04 LTS](http://www.ubuntu.com/download/desktop)

> - **docker** [1.12](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

> - **docker compose** [1.8.0-rc2](https://github.com/docker/compose/releases)

> - **docker machine** [0.8.0-rc2](https://github.com/docker/machine/releases)

###文档说明

> **名词解释:**

> - **主机 :** 安装**ubuntu**的机器，可以是物理机也可以是虚拟机

> - **docker0 :** **Docker**服务默认创建的网桥名

> **相关说明:**

> - 大多数命令需要root权限，可以通过以下命令来解决每次都需要sudo的步骤
> ```
> $ sudo -i
> ```

###更新APT资源列表
a. 在 **`主机`** 上使用 **`root`** 权限或 **`sudo`** 命令来执行以下命令
b. 打开一个 **`terminal`**
c. 更新包信息，确保 **`APT`** 使用 **`https`** 方式工作且安装了 **`CA`** 证书
```
$ sudo apt-get update

$ sudo apt-get install apt-transport-https ca-certificates
```

d. 添加新的 **`GPG`** key 
```
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
``` 
e. 使用你最喜欢的编辑器来打开以下文件，不存在则创建它
> /etc/apt/sources.list.d/docker.list

f. 清空所有已经存在的 **`entry`**，并添加一条新的 **`entry`**
> deb https://apt.dockerproject.org/repo ubuntu-xenial main

g. 保存并关闭该文件

**Tip:** `e` , `f` 和 `g` 这三个步骤可以通过以下命令实现
```
sh -c "echo deb https://apt.dockerproject.org/repo ubuntu-xenial main > /etc/apt/sources.list.d/docker.list"
```

h. 更新 **`APT`** 包索引
```
$ sudo apt-get update
```
i. 如果存在则先卸载旧有版本
```
$ sudo apt-get purge lxc-docker
```
j. 验证 **`APT`** 从正确的仓库中拉取
```
$ sudo apt-cache policy docker-engine
```
k. 执行以下命令，获取软件最新版本
```
$ sudo apt-get upgrade
```
**Tip:** `upgrade` 命令会更新所有软件包

###ubuntu安装docker之前的准备
a. 在 **`主机`** 上打开一个 **`terminal`**
b. 更新包管理器
```
$ sudo apt-get update
```
c. 安装推荐的包
```
$ sudo apt-get install linux-image-extra-$(uname -r)
```

###安装docker
a. 在 **`主机`** 上使用 **`sudo`** 
b. 更新 **`APT`** 包索引
```
$ sudo apt-get update
```
c. 安装 **`docker`**
```
$ sudo apt-get install docker-engine
```
d. 启动 **`docker`** daemon
```
$ sudo service docker start
```
e. 验证 **`docker`** 安装正确
```
$ sudo docker run hello-world
```
**Tip:** 这个命令会下载一个测试镜像并在容器中运行它，当容器运行时，它会在控制台打印出一条消息，随后便会结束退出。

###安装docker compose
a. 打开以下链接
> https://github.com/docker/compose/releases

b. 在页面中找到这样的命令并运行
```
curl -L https://github.com/docker/compose/releases/download/1.8.0-rc2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

###安装docker machine
a. 打开以下链接
> https://github.com/docker/machine/releases

b. 在页面中找到这样的命令并运行
```
curl -L https://github.com/docker/machine/releases/download/v0.8.0-rc2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine
chmod +x /usr/local/bin/docker-machine
```
###校验一下版本信息
> **NOTE:**
> a. 执行以下命令
> ```
> docker version
> ```
> 得到 **`docker`** 的版本信息，包括 **`客户端`** 和 **`服务端`**
> ```
> Client:
 Version:      1.12.0
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   8eab29e
 Built:        Thu Jul 28 22:11:10 2016
 OS/Arch:      linux/amd64
Server:
 Version:      1.12.0
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   8eab29e
 Built:        Thu Jul 28 22:11:10 2016
 OS/Arch:      linux/amd64
> ```
> b. 执行以下命令
> ```
> docker-compose version
> ```
> 得到 **`docker compose`** 当前版本信息，包括 **`py`** ,  **`ssl`** 等信息
> ```
> docker-compose version 1.8.0-rc2, build c72c966
docker-py version: 1.9.0-rc2
CPython version: 2.7.9
OpenSSL version: OpenSSL 1.0.1e 11 Feb 2013
> ```
> c. 执行以下命令
> ```
> docker-machine version
> ```
> 得到 **`docker machine`** 当前版本信息
> ```
> docker-machine version 0.8.0-rc2, build 4ca1b85
> ```

