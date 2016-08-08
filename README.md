Docker On Ubuntu
====

本文主要介绍了[**Docker**](https://www.docker.com)在[**Ubuntu**](http://www.ubuntu.com/)上的主要应用，并详细的阐述了实现步骤、遇到的问题及问题的解决方案。新手可以先了解一下它们，老鸟可以直接跳过下面的若干步骤，废话不多说，开始吧~

### 我们要做什么
明确目标很关键，我们的主要目标就是**基于Docker**来完成以下工作
> 建立私有的gitlab

> 建立私有的docker registry

> 通过gitlab-ci来自动化build，test，push，deploy我们的项目

> 利用集群部署我们的服务

### 环境参数

> **系统软件:**

> - **ubuntu** [16.04 LTS](http://www.ubuntu.com/download/desktop)

> - **docker** [1.12](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

> - **docker compose** [1.8.0-rc2](https://github.com/docker/compose/releases)

> - **docker machine** [0.8.0-rc2](https://github.com/docker/machine/releases)

### 文档说明

> **名词解释:**

> - **主机 :** 安装**ubuntu**的机器，可以是物理机也可以是虚拟机

> - **主机ip :** **`主机`** 的ip，可通过 **`ifconfig`** 命令获取

> - **其他主机 :** 安装有**Docker**的其他 **`主机`**

> - **docker0 :** **Docker**服务默认创建的网桥名

> - **&lt;username&gt; :** 你的用户名，我这里使用自己的用户名**devlee**

> - **工作目录 :** `主机`上进行本文描述的相关工作的工作目录

> - **我的项目 :** 我把这篇文章中涉及的代码总结于此项目

> **相关说明:**

> - 大多数命令需要root权限，可以通过以下命令来解决每次都需要sudo的步骤

> 	```
> 	$ sudo -i
> 	```

> **工作目录:**

> - /work/&lt;username&gt;

> **我的项目:**

> - [docker-aio](https://github.com/devlee/docker-aio)

### 准备工作
1. 创建好 **`工作目录`**

	```
	$ sudo mkdir /work/devlee
	```

	> **Tip:** 准备工作到第一步就可以了，因为后续步骤会创建若干目录及文件，如果你想偷懒，不愿意一步一步创建这些目录及文件，可以继续准备工作的后续几步。

2. 进入**`工作目录`**并拉取**`我的项目`**，该项目中拥有一些我们下面操作中需要的文件

	```
	$ sudo cd /work/devlee && git clone https://github.com/devlee/docker-aio.git
	```

3. 将项目中src中的所有文件夹拷贝至**`工作目录`**下

	```
	$ sudo cp -r /work/devlee/docker-aio/src /work/devlee
	```

4. 此时查看**`工作目录`**，应该拥有 **`gitlab`**，**`registry`**，**`machine`**等文件夹


### 更新APT资源列表
1. 在 **`主机`** 上使用 **`root`** 权限或 **`sudo`** 命令来执行以下命令

2. 打开一个 **`terminal`**

3. 更新包信息，确保 **`APT`** 使用 **`https`** 方式工作且安装了 **`CA`** 证书

	```
	$ sudo apt-get update
	```

	```
	$ sudo apt-get install apt-transport-https ca-certificates
	```

4. 添加新的 **`GPG`** key

	```
	$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
	```

5. 使用你最喜欢的编辑器来打开以下文件，不存在则创建它
> /etc/apt/sources.list.d/docker.list

6. 清空所有已经存在的 **`entry`**，并添加一条新的 **`entry`**
> deb https://apt.dockerproject.org/repo ubuntu-xenial main

7. 保存并关闭该文件
> **Tip:** `5` , `6` 和 `7` 这三个步骤可以通过以下命令实现
	```
	sh -c "echo deb https://apt.dockerproject.org/repo ubuntu-xenial main > /etc/apt/sources.list.d/docker.list"
	```

8. 更新 **`APT`** 包索引

	```
	$ sudo apt-get update
	```

9. 如果存在则先卸载旧有版本

	```
	$ sudo apt-get purge lxc-docker
	```

10. 验证 **`APT`** 从正确的仓库中拉取

	```
	$ sudo apt-cache policy docker-engine
	```

11. 执行以下命令，获取软件最新版本

	```
	$ sudo apt-get upgrade
	```

	> **Tip:** `upgrade` 命令会更新所有软件包

### ubuntu安装docker之前的准备
1. 在 **`主机`** 上打开一个 **`terminal`**

2. 更新包管理器

	```
	$ sudo apt-get update
	```

3. 安装推荐的包

	```
	$ sudo apt-get install linux-image-extra-$(uname -r)
	```

### 安装docker
1. 在 **`主机`** 上使用 **`sudo`**

2. 更新 **`APT`** 包索引

	```
	$ sudo apt-get update
	```

3. 安装 **`docker`**

	```
	$ sudo apt-get install docker-engine
	```

4. 启动 **`docker`** daemon

	```
	$ sudo service docker start
	```

5. 验证 **`docker`** 安装正确

	```
	$ sudo docker run hello-world
	```

	> **Tip:** 这个命令会下载一个测试镜像并在容器中运行它，当容器运行时，它会在控制台打印出一条消息，随后便会结束退出。

### 安装docker compose
1. 打开以下链接
> https://github.com/docker/compose/releases

2. 在页面中找到这样的命令并运行

	```
	curl -L https://github.com/docker/compose/releases/download/1.8.0-rc2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
	```

	```
	chmod +x /usr/local/bin/docker-compose
	```

### 安装docker machine
1. 打开以下链接
> https://github.com/docker/machine/releases

2. 在页面中找到这样的命令并运行
	```
	curl -L https://github.com/docker/machine/releases/download/v0.8.0-rc2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine
	```
	```
	chmod +x /usr/local/bin/docker-machine
	```

### 校验一下版本信息
1. 执行以下命令

	```
	docker version
	```

	得到 **`docker`** 的版本信息，包括 **`客户端`** 和 **`服务端`**

	```
	Client:
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
	```

2. 执行以下命令

	```
	docker-compose version
	```

	得到 **`docker compose`** 当前版本信息，包括 **`py`** ,  **`ssl`** 等信息

	```
	docker-compose version 1.8.0-rc2, build c72c966
	docker-py version: 1.9.0-rc2
	CPython version: 2.7.9
	OpenSSL version: OpenSSL 1.0.1e 11 Feb 2013
	```

3. 执行以下命令

	```
	docker-machine version
	```

	得到 **`docker machine`** 当前版本信息

	```
	docker-machine version 0.8.0-rc2, build 4ca1b85
	```

### 搭建私有的gitlab
1. 进入**`工作目录`**，创建文件夹**`gitlab`**，并进入该目录

	```
	$ sudo cd /work/devlee && mkdir gitlab && cd gitlab
	```

2. 创建docker-compose.yml，并将以下链接中的代码拷贝至该文件中
> https://raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml
> **Tip:** 与我的项目中的文件内容有些许差异，主要是**镜像版本号**和**容器** **`container_name`** 的定义

3. 执行以下命令

	```
	$ sudo docker-compose up -d
	```

4. 搭建完成，访问以下地址进行初始化并重置 **`root`** 密码
> http://localhost:10080

### 搭建私有的docker registry
> **Tip:**

> 1. 这里的 **`registry`** 版本为 **`v2`** ，隶属于 **`docker distribution`** 项目，项目地址为：https://github.com/docker/distribution

> 2. 我们同时会利用 **`nginx`** 来实现 **`https`** 仓库

> 3. 我们会使用 **`docker-registry-frontend`** 来作为web前端展示

1. 进入**`工作目录`**，创建文件夹**`registry`**，并进入该目录

	```
	$ sudo cd /work/devlee && mkdir registry && cd registry
	```

2. 创建docker-compose.yml文件，并写入以下代码

	```
	version: '2'
	services:
	  nginx:
	    restart: always
	    image: 'nginx:latest'
	    ports:
	      - 443:443
	    links:
	      - registry:registry
	    volumes:
	      - ./nginx/:/etc/nginx/conf.d
	    container_name: 'docker-registry-proxy'
	  registry:
	    restart: always
	    image: 'registry:2.4.1'
	    ports:
	      - 5000:5000
	    environment:
	      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data
	    volumes:
	      - ./data:/data
	    container_name: 'docker-registry'
	  frontend:
	    restart: always
	    image: 'konradkleine/docker-registry-frontend:v2'
	    ports:
	      - 8080:80
	    environment:
	      - ENV_DOCKER_REGISTRY_HOST=nginx
	      - ENV_DOCKER_REGISTRY_PORT=443
	      - ENV_DOCKER_REGISTRY_USE_SSL=1
	    links:
	      - nginx:nginx
	    container_name: 'docker-registry-frontend'
	```

3. 创建文件夹data和nginx，并进入nginx文件夹

	```
	$ sudo mkdir data && mkdir nginx && cd nginx
	```

4. 创建registry.conf文件，并写入以下代码

	```
    upstream docker-hub {
      server registry:5000;
    }

    server {
      listen 443;
      server_name localhost;

      # SSL
      ssl on;
      ssl_certificate /etc/nginx/conf.d/docker-registry.crt;
      ssl_certificate_key /etc/nginx/conf.d/docker-registry.key;

      # disable any limits to avoid HTTP 413 for large image uploads
      client_max_body_size 0;

      # required to avoid HTTP 411: see Issue #1486 (https://github.com/docker/docker/issues/1486)
      chunked_transfer_encoding on;

      location /v2/ {
        # Do not allow connections from docker 1.5 and earlier
        # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
        if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
        return 404;
        }

        # To add basic authentication to v2 use auth_basic setting plus add_header
        auth_basic "registry.localhost";
        auth_basic_user_file /etc/nginx/conf.d/docker-registry.htpasswd;
        add_header 'Docker-Distribution-Api-Version' 'registry/2.4.1' always;

        proxy_pass http://docker-hub;
        proxy_set_header Host $http_host; # required for docker client's sake
        proxy_set_header X-Real-IP $remote_addr; # pass on real client's IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 900;
      }
    }
	```

5. 生成 **`ssl`** 证书

	```
	$ sudo openssl req -x509 -nodes -newkey rsa:2048 -keyout /work/devlee/registry/nginx/docker-registry.key -out /work/devlee/registry/nginx/docker-registry.crt
	```

	> **Tip:** 生成证书的时候需要格外注意，所有选型除了 **`CN`** 都可以选择默认，这个选项需要输入仓库的域名，由于我们是本地内网所以输入 **`docker-registry`** ，记得要在其他主机的 **`/etc/hosts`** 中添加解析，通过 **`ifconfig`** 命令可以查看 **`主机`** 的 **`ip`**

6. 安装 **`htpasswd`**

	```
	$ sudo apt-get install apache2-utils
	```

7. 创建用户名密码，这里用户名是 **`devlee`** ，第一个用户需要添加 **`-c`** 参数

	```
	$ sudo htpasswd -c /work/devlee/registry/nginx/docker-registry.htpasswd devlee
	```

8. 启动服务

	```
	$ sudo cd /work/devlee/registry && docker-compose up -d
	```

9. 搭建完成，访问以下地址进行验证，登录后会返回 **`{}`**

	```
	https://localhost:443/v2
	```

	> **Tip:** 目前所搭建的仓库是在主机上进行访问的，可以直接通过 **`localhost`** 访问，要在 **`其他主机`** 上访问仓库，需要 **`其他主机`** 配置有证书 **`/work/devlee/registry/nginx/docker-registry.crt`** ，且需要设置 **`/etc/hosts`** (别名访问)，通过域名或别名进行访问，**`https://docker-registry:443/v2`**

10. 在 **`其他主机`** 上配置证书 （内网配置，如果仓库在外网，不用配置hosts）
	a. 配置 **`/etc/hosts`** ，假设 **`主机ip`** 为 **`10.12.30.52`** ，写入

	> 10.12.30.52 docker-registry

	b. 启动**Docker**服务

	```
	$ sudo service docker start
	```

	c. 创建扩展证书存放目录

	```
	$ sudo mkdir /usr/share/ca-certificates/extra
	```

	d. 拷贝 **`主机`** 中的 **`docker-registry.crt`** 文件至上述目录

    e. 注册证书

	```
	$ sudo dpkg-reconfigure ca-certificates
	```

11. 在 **`其他主机`** 上配置证书（方法2）
	a. 配置 **`/etc/hosts`** ，假设 **`主机ip`** 为 **`10.12.30.52`** ，写入

	> 10.12.30.52 docker-registry

	b. 创建目录

	```
	$ sudo cd /etc/docker && mkdir cert.d && cd cert.d && mkdir docker-registry:443
	```

	c. 拷贝 **`主机`** 中的 **`docker-registry.crt`** 文件至上述最后一个创建的目录

    d. 重启**Docker**服务

	```
	$ sudo service docker restart
	```

12. 推送测试

	```
	$ sudo docker tag hello-world docker-registry:443/hello
	$ sudo docker push docker-registry:443/hello
	```

### 搭建gitlab-runner
1. 向 **`/work/devlee/gitlab/docker-compose.yml`** 添加以下代码

		runner-docker-in-docker:
            restart: always
            image: sameersbn/gitlab-ci-multi-runner:latest
            depends_on:
            	- gitlab
            volumes:
            	- /srv/docker/gitlab-runner/docker-in-docker:/home/gitlab_ci_multi_runner/data
            links:
            	- gitlab:gitlab
            environment:
                - CI_SERVER_URL=http://<gitlab-ip>:10080/ci
                - RUNNER_TOKEN=<gitlab-token>
                - RUNNER_DESCRIPTION=docker-in-docker-runner
                - RUNNER_EXECUTOR=docker
                - DOCKER_IMAGE=docker:latest
                - DOCKER_PRIVILEGED=true
                - DOCKER_EXTRA_HOSTS=localhost:<registry-ip>
            container_name: gitlab-runner-docker-in-docker

        runner-docker-socket-binding:
            restart: always
            image: sameersbn/gitlab-ci-multi-runner:latest
            depends_on:
            	- gitlab
            volumes:
                - /srv/docker/gitlab-runner/docker-socket-binding:/home/gitlab_ci_multi_runner/data
                - /var/run/docker.sock:/var/run/docker.sock
                - /git/devlee/registry/nginx/volumes:/etc/docker/cert.d
            links:
            	- gitlab:gitlab
            environment:
                - CI_SERVER_URL=http://<gitlab-ip>:10080/ci
                - RUNNER_TOKEN=<gitlab-token>
                - RUNNER_DESCRIPTION=docker-socket-binding-runner
                - RUNNER_EXECUTOR=docker
                - DOCKER_IMAGE=docker:latest
                - DOCKER_EXTRA_HOSTS=localhost:<registry-ip>
                - DOCKER_VOLUMES=/git/devlee/registry/nginx/volumes:/etc/docker/cert.d
            container_name: gitlab-runner-docker-socket-binding
            extra_hosts:
                - "localhost:172.17.0.1"
                - "docker-registry:172.17.0.1"

        runner-shell:
            restart: always
            image: sameersbn/gitlab-ci-multi-runner:latest
            depends_on:
            	- gitlab
            volumes:
            	- /srv/docker/gitlab-runner/shell:/home/gitlab_ci_multi_runner/data
            links:
            	- gitlab:gitlab
            environment:
                - CI_SERVER_URL=http://<gitlab-ip>:10080/ci
                - RUNNER_TOKEN=<gitlab-token>
                - RUNNER_DESCRIPTION=shell-runner
                - RUNNER_EXECUTOR=shell
            container_name: gitlab-runner-shell

	>**Tip:**
	> **&lt;gitlab-ip&gt;**: 私有的gitlab服务ip，由于我们是搭建在容器里的，所以这里取 **`主机`** 的 **`docker0`** 的 **`ip`**
	> **&lt;gitlab-token&gt;**: 这里的token可以填gitlab上某个项目的 **`special token`** ，也可以是 **`shared token`** ，我这里用的是 **`shared token`** ，使用 **`admin`** 账户登录gitlab，访问 **`loalhost:10080/admin/runners`** 就可以看到token
	> **&lt;registry-ip&gt;**: 私有的registry服务ip，由于我们是搭建在容器里的，所以这里取 **`主机`** 的 **`docker0`** 的 **`ip`**

2. 启动服务

		$ sudo cd /work/devlee/gitlab && docker-compose up -d

3. 打开gitlab，使用admin账户登录，在runners页面中给这三个runner添加tag，分别为 **`docker`** , **`sock`** , **`shell`**

### 新建项目koa-app于私有的gitlab
1. 用 **`admin`** 账户在 **`gitlab`** 上新建一个 **`koa-app`** 项目

2. 进入 **`工作目录`** ，创建文件夹 **`dev`** ，并进入该目录

3. 拷贝我们的项目至本地

	```
	$ sudo git clone http://localhost:10080/root/koa-app.git
	```

4. 进入我们的项目目录

	```
    $ sudo cd koa-app
    ```

5. 安装 **`npm`**

	```
    $ sudo apt-get install python-software-properties
    $ sudo add-apt-repository ppa:gias-kay-lee/npm
    $ sudo apt-get update
    $ sudo apt-get install npm
    ```

6. 创建 **`package.json`**

	```
    $ sudo npm init
    ```

7. 安装 **`koa2`**

    ```
    $ sudo npm install koa@2 --save
    ```

8. 创建 **`index.js`** ，代码如下

		const Koa = require('koa')

		const app = new Koa

		app.use(ctx => {
    		ctx.body = 'Hello, Docker'
    	})

    	app.listen(3232)

9. 创建 **`.dockerignore`** ，内容如下

	> .git
	> .gitlab-ci.yml
	> README.md

10. 创建 **`.gitignore`** ，内容如下

	> node_modules

11. 创建 **`.gitlab-ci.yml`** ，内容如下

		image: docker:latest

    	before_script:
    		- docker version

        build:image
        	stage: build
            script:
            	- docker login -u devlee -p xxxx localhost:443
            	- docker build -t localhost:443/devlee/koa-app:latest .
            	- docker push localhost:443/devlee/koa-app:latest
            tags:
            	- sock

	> **Tip:** **xxxx**是创建用户 **`devlee`** 时输入的密码

12. 创建 **`Dockerfile`** ，内容如下

		FROM localhost:443/devlee/node:latest
        COPY . /devlee/
        WORKDIR /devlee/
        RUN npm install
        CMD ["node", "index"]
        EXPOSE 3232

13. 提交代码到gitlab

    	$ sudo git add --all
        $ sudo git commit -m "xxxx"
        $ sudo git fetch
        $ sudo git rebase
        $ sudo git push

	> **Tip:** **xxxx**是提交的修改注释

14. 由于提交的代码中包含了.gitlab-ci.yml文件，会自动触发ci相关任务。

### 集群部署服务koa-app
1. 我们利用虚拟机当集群节点，所以需要安装 **`virtualbox`**

		$ sudo sh -c "echo deb http://download.virtualbox.org/virtualbox/debian xenial contrib > /etc/apt/sources.list"
        $ sudo wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
        $ sudo wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
        $ sudo apt-get update
        $ sudo apt-get install virtualbox-5.1

2. 执行以下命令获得 **`<token>`**

		$ sudo docker run --rm swarm create

3. 创建swarm master

		$ sudo docker-machine create -d virtualbox --swarm --swarm-master --swarm-discovery token://<token> swarm-master

4. 创建swarm node 01

		$ sudo docker-machine create -d virtualbox --swarm --swarm-discovery token://<token> swarm-node-01

5. 创建swarm node 02

		$ sudo docker-machine create -d virtualbox --swarm --swarm-discovery token://<token> swarm-node-02