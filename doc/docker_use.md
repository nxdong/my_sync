## 参考文档
### https://yeasy.gitbooks.io/docker_practice/content/
### https://github.com/NVIDIA/nvidia-docker/wiki

## 修改 GRUB 配置文件
  - 修改 GRUB 的配置文件 `/etc/default/grub`，在 `GRUB_CMDLINE_LINUX` 中
  添加内核引导参数 `cgroup_enable=memory swapaccount=1`。
  - `sudo update-grub`
  - `sudo reboot`
## 使用脚本自动安装
  `curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -`
## 建立用户组，将当前用户加入用户组
  `sudo groupadd docker` （这一步可以省略，脚本会自动新建docker组）
  `sudo usermod -aG docker $USER`
## 启动docker 服务
  ` sudo service docker start`
## 镜像加速

## 获取NVIDIA Docker
### Install nvidia-docker and nvidia-docker-plugin
  下面的方法不好使，无法链接Amazon的服务器。从我这里拷贝
  `wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb`
  `sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb`
  执行`sudo dpkg -i nvidia-docker_1.0.1-1_amd64.deb` 安装NVIDIA docker
### Test nvidia-smi
`sudo nvidia-docker run nvidia/cuda nvidia-smi`
这个会下载一堆东西

## 打开docker容器bash
`docker exec -it dockername bash`
## 查看容器改动
`docker diff dockername`
## 将修改保存为镜像
`docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    dockername \
    仓库名:版本`
## 用 docker history 具体查看镜像内的历史记录
## 跑新镜像 docker run --name web2 -d -p 81:80 nginx:v2
--------------------------------------------------------------------------
# NVIDIA Docker
## 获取NVIDIA Docker
### Install nvidia-docker and nvidia-docker-plugin
  下面的方法不好使，无法链接Amazon的服务器。从我这里拷贝
  `wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb`
  `sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb`
  执行`sudo dpkg -i nvidia-docker_1.0.1-1_amd64.deb` 安装NVIDIA docker
## 下载镜像
`sudo nvidia-docker pull nvidia/cuda:8.0-devel-ubuntu14.04`
## 保存镜像
`docker save 8.0-devel-ubuntu14.04 | gzip > 8.0-devel-ubuntu14.04.tar.gz`
## 安装镜像
`docker load -i 8.0-devel-ubuntu14.04.tar.gz`
## 运行镜像 bash(交互运行，如果需要只运行一条命令则去掉`-i`)
`sudo nvidia-docker run -t -i nvidia/cuda:8.0-devel-ubuntu14.04 /bin/bash`
## docker stop 来终止一个运行中的容器
## nvidia-docker start 命令，直接将一个已经终止的容器启动运行
## docker restart 命令会将一个运行态的容器终止，然后再重新启动它
## 终止状态的容器可以用 docker ps -a 命令看到
## 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

数据卷可以在容器之间共享和重用
对数据卷的修改会立马生效
对数据卷的更新，不会影响镜像
数据卷默认会一直存在，即使容器被删除
*注意：数据卷的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷。*

创建一个数据卷

在用 docker run 命令的时候，使用 -v 标记来创建一个数据卷并挂载到容器里。在一次 run 中多次使用可以挂载多个数据卷。

下面创建一个名为 web 的容器，并加载一个数据卷到容器的 /webapp 目录。

$ sudo docker run -d -P --name web -v /webapp training/webapp python app.py
*注意：也可以在 Dockerfile 中使用 VOLUME 来添加一个或者多个新的卷到由该镜像创建的任意容器。*

删除数据卷

数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 docker rm -v 这个命令。无主的数据卷可能会占据很多空间，要清理会很麻烦。Docker官方正在试图解决这个问题，相关工作的进度可以查看这个PR。

挂载一个主机目录作为数据卷

使用 -v 标记也可以指定挂载一个本地主机的目录到容器中去。

$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
上面的命令加载主机的 /src/webapp 目录到容器的 /opt/webapp 目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，如果目录不存在 Docker 会自动为你创建它。

## 数据卷容器(本机目录:容器目录)
如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。

数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的。

首先，创建一个名为 dbdata 的数据卷容器：

$ sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres
然后，在其他容器中使用 --volumes-from 来挂载 dbdata 容器中的数据卷。

$ sudo docker run -d --volumes-from dbdata --name db1 training/postgres
$ sudo docker run -d --volumes-from dbdata --name db2 training/postgres
可以使用超过一个的 --volumes-from 参数来指定从多个容器挂载不同的数据卷。 也可以从其他已经挂载了数据卷的容器来级联挂载数据卷。

$ sudo docker run -d --name db3 --volumes-from db1 training/postgres
*注意：使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态。*

## 查看数据卷的具体信息
`docker inspect dockername
