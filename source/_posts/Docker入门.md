---
layout: docker
title: Docker入门
date: 2018-04-13 18:22:35
tags: docker
---


查看有哪些镜像：

```shell
docker images
```

获取镜像：

```shell
docker pull ubuntu:16.04
```
<!-- more -->

等待下载完成，获取一个官方的`ubuntu 16.04`的官方镜像。

然后，使用docker run命令可以从一个镜像运行一个包含一个主进程的容器：

```shell
docker run -ti --name first ubuntu:16.04 bash
```



> 命令解释：
>
> * docker run ：从一个镜像运行一个容器。
> * -ti（terminal interactive）：进入容器的交互式终端。
> * --name：指定容器的名字，后面的first就是容器的命名。
> * ubuntu:16.04：指明从哪个镜像运行容器，ubuntu指仓库名，16.04是标签。
> * bash：指明使用bash终端。

现在，可以在容器里执行一些命令，就好像在一个全新的系统中运行指令一样，比如创建文件：

```shell
touch hello-docker.txt
```

现在，可以通过输入exit命令退出运行的容器，其实就是终止了容器。



如果将来，你希望每次运行一个新容器都包含刚刚创建的hello-docker.txt，可以把容器提交为一个镜像：

```
docker commit first my-image:v1.0
```

注意：这种创建镜像的方式并不推荐，应该避免通过这种方式创建镜像。



后台运行：docker run -d ...

删除容器：docker rm ...



#### 进入容器

* ssh登录
* attach和exec
* nesenter



在后台运行一个新的容器：

```shell
docker run -d -ti --name background ubuntu:16.04 bash
```

使用如下命令进入容器：

```shell
docker exec  -ti backgroud bash
```

第二种方式是使用attach命令：

```shell
docker attach background
```

然后按两次回车即可进入容器。



#### 终止容器

* 进入容器，然后输入命令：`exit`
* 通过kill命令



```shell
docker kill background
```



#### 查看容器

```shell
docker ps -a
```



#### 其他常用命令

* 查看容器详细信息

  ```shell
  docker inspect background
  ```

* 查看容器最近一个进程

  ```shell
  docker top background
  ```

* 停止一个正在运行的容器

  ```shell
  docker stop background
  ```

* 继续一个被停止的容器

  ```shell
  docker restart background
  ```

* 暂停一个一个容器

  ```shell
  docker pause background
  ```

* 取消暂停

  ```shell
  docker unpause background
  ```

* 终止一个容器

  ```shell
  docker kill background
  ```

  ​


#### Docker数据管理的方式

##### 数据卷（Data Volume）

数据卷的使用其实和Linux挂载文件目录是很相似的，数据卷就是一个可以供容器使用的特殊目录。

* **创建一个数据卷**

在使用`docker run`命令的时候使用-v参数为容器挂载一个数据卷：

```shell
docker run -ti --name hasVolume -v /myDir ubuntu:16.04 bash
```

然后可以看到我们的容器的目录里多了一个myDir的目录，这个就是数据卷：

[![volume-1.png](https://i.loli.net/2018/04/14/5ad19e97dbef4.png)](https://i.loli.net/2018/04/14/5ad19e97dbef4.png)



* **删除一个数据卷**

数据卷的生命周期是独立于容器的，所以要删除数据卷，必须在删除容器的时候加上-v参数，

注意：如果你删除了挂载某个数据卷的全部容器，而没有使用-v清理数据卷，之后再想清除这个数据卷将会非常麻烦，所以，尽量在删除最后一个挂载该数据卷的容器时加上-v参数：

```shell
docker rm -v hasVolume
```

* **挂载一个主机目录到数据卷**

使用如下命令挂载一个主机目录到数据卷：

```shell
docker run -ti --name volume1 -v /home/xch/:/myShared ubuntu:16.04 bash
```

意思是，将宿主机上的/home/xch挂载到容器的myShared目录下，Docker挂载目录的默认权限是读写，可以通过`:ro`命令设置为只读：

```shell
docker run -ti --name volume2 -v /home/xch/:/myShared:ro ubuntu:16.04 bash
```



##### 数据卷容器

数据卷容器就是一个普通的容器，只不过这个容器是专门作为数据卷供其他容器挂载。

首先，运行容器时创建一个数据卷容器：

```shell
docker run -ti --name v0 -d -v /data-volume ubuntu:16.04
```

接着，创建一个新的容器挂载刚才创建的数据卷容器中的数据卷，使用--volumes-from参数：

```shell
docker run -ti --name v1 --volumes-from v0 ubuntu:16.04 bash
```

> 注意：
>
> * 数据卷容器被挂载时不必保证运行
> * 如果删除了v0，v1数据卷并不会被清理，应当运行docker rm -v命令



### Docker网络

#### 准备

由于ubuntu的官方镜像并没有默认安装ping，ifconfig等网络工具，所以需要创建一个具备一定网络功能的镜像（目前先使用容器提交的方式，后续再介绍dockerfile创建镜像）。

首先，从之前的的ubuntu:16.04创建一个容器：

```shell
docker run -ti --name network-example ubuntu:16.04 bash
```

然后依次执行如下命令：

```shell
apt-get update
apt-get install vim
apt-get install net-tools
apt install iputils-ping
apt install apache2
apt install apache2-utils
apt install openssh-server
apt install openssh-client
```

接着，使用vim修改：

```shell
vim /etc/ssh/sshd_config
```

将`PermitRootLogin`内容改为yes，保存退出。

输入：

```shell
passwd
```

输入root用户的新密码并确认。（密码要牢记！）

然后退出容器。然后基于这个容器提交新的镜像：

```shell
docker commit  -m 'my network example' network-example net:v1.0
```

> 说明：
>
> * -m参数是添加一个说明
> * 然后是容器名或id
> * 最后是新镜像名和标签



#### 端口暴露

使用`-p`参数进行端口映射，格式如下：

> -p hostPort：containerPort 映射所有ip地址上的指定端口到容器内部
>
> -p  ip:hostPort:containerPort 映射指定ip地址上的指定端口到容器内部
>
> -p ip::containerPort  映射指定ip上的任意端口到容器内部

比如：

```shell
docker run -ti --name web -p 80:80 net:v1.0 bash
```

这个命令启动一个容器，并映射宿主机所有ip的80端口到容器的80端口。

我们已经安装了apache服务器，启动它：

```shell
apache2ctl start
```

然后查看容器的ip地址：

```shell
ifconfig
```

[![net-work.png](https://i.loli.net/2018/04/14/5ad1b93b4e7e4.png)](https://i.loli.net/2018/04/14/5ad1b93b4e7e4.png)

这里我的是172.17.0.4，此时打开宿主机的浏览器，输入url:172.17.0.4，就可以访问到apache服务器的主页了。

端口暴露不仅可以把容器作为Web服务器使用，还可以通过网络让不同的容器相互通信。Docker默认使用tcp协议在容器间进行网络通信。如果要使用udp，可以使用如下命令：

```shell
docker run -ti --name web -p 80:80/ud net:v1.0 bash
```



#### 容器互联

容器互联可以不使用端口暴露就让容器之间可以相互通信。

首先，创建一个源容器：

```shell
docker run -ti -d --name source net:v1.0
```

然后运行另一个容器，使用--link参数连接第一个容器:

```shell
docker run -ti --name receiver --link source:sender net:v1.0 bash
```

这里的--link source:sender 的意思是把名为source的容器连接到别名sender，然后就可在第二个容器以sender这个名字和第一个容器通信，比如

```shell
ping sender
```

因为系统把这个别名添加到了`/etc/hosts`中。



#### SSH登录容器

首先运行一个容器：

```shell
docker run -ti --name ssh -p 6667:22 net:v1.0 bash
```

然后在容器里启动ssh服务：

```shell
service ssh start
```

查看ip地址：

```shell
ifconfig
```

[![ifconfig.png](https://i.loli.net/2018/04/14/5ad1be908a2d3.png)](https://i.loli.net/2018/04/14/5ad1be908a2d3.png)

然后在新的终端里运行：

```shell
ssh root@172.17.0.5
```

然后就可以顺利进入容器了。