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