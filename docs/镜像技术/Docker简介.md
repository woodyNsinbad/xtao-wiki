# Docker基本概念
Docker是目前使用率最高的容器技术，详细技术介绍可以参考Docker官方网站（ https://docs.docker.com ） 或者中文社区（ https://www.docker.org.cn/index.html ）。对生信用户来说，Docker容器提供了独立的程序运行环境。不同用户通过使用不同的Docker镜像，拥有了独立的运行环境，包括使用自己熟悉的Linux发行版（centos、ubuntu等等），使用自己需要的软件版本等等。如果不使用容器，用户和系统管理员之间需要大量交互和协商。版本冲突也可能导致计算资源和特定软件绑定，造成资源浪费。Docker很好的解决了这个问题。

Docker包括下面基本技术概念：
- **镜像(image)**: 所谓镜像，是指一个静态的系统环境。用户可以创建、删除。镜像是一个系统的静态展现，其中包含了系统相关的文件以及软件。
- **容器(container)** : 容器是系统的一个实例，即运行着某一条或一些命令的系统。
- **Docker 服务**：Docker 镜像资源的使用依托于Docker 服务。Linux 用户可以通过系统管理员用户，安装docker 命令来获取Docker 服务。
- **宿主机** ：指运行Docker 服务的机器，此处一般指一台实体的计算机。
- **仓库** ： 保存镜像的存储服务，无论是从公网或者本地构建镜像后可以方便的在本地使用，当需要在其它服务器使用时需要集中存储、分发镜像的服务。本计算集群采用Docker Registry和harbor结合提供本地私有仓库服务。

# 使用镜像
Docker 运行容器前需要本地存在对应的镜像，如果本地镜像不存在，Docker 会从镜像仓库下载镜像。
## 获取镜像
1. 从公共镜像站点获取，例如: [Docker Hub](https://hub.docker.com/) 或从本地私有仓库获取，例如：[本地私有仓库](https://10.61.51.7:5000/) 从 Docker 镜像仓库获取镜像的命令是 docker pull。其命令格式为：
```
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```
具体的选项可以通过 docker pull --help 命令看到，这里我们说一下镜像名称的格式。

1. Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub(docker.io)。

1. 仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。
举个例子：
### 从docker Hub 获取公共镜像：
```
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
92dc2a97ff99: Pull complete
be13a9d27eb8: Pull complete
c8299583700a: Pull complete
Digest: sha256:4bc3ae6596938cb0d9e5ac51a1152ec9dcac2a1c50829c74abd9c4361e321b26
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```
上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub （docker.io）获取镜像。而镜像名称是 ubuntu:18.04，因此将会获取官方镜像 library/ubuntu 仓库中标签为 18.04 的镜像。docker pull 命令的输出结果最后一行给出了镜像的完整名称，即： docker.io/library/ubuntu:18.04。

### 从本地私有仓库获取公共镜像或者私有镜像
1. 在biohpc 集群获取镜像, 只需要有docker pull权限，即可从本地library项目获取镜像。详细请参考harbor本地仓库使用。
```
$ docker pull registry.servicemgr.apc:5000/library/trimmomatic:0.36--6
0.36--6: Pulling from library/trimmomatic
4f4fb700ef54: Already exists 
b0dc45cd432d: Already exists 
9466b3513669: Already exists 
ddd482ea7b54: Already exists 
4d69f833b9d8: Already exists 
e7c454e5167d: Already exists 
e38092b005c0: Already exists 
f879b42dfe2b: Already exists 
762b04cf8d24: Pull complete 
Digest: sha256:207435c165117b41f9568b7b189d0877ac467fb6f7a01df0e63974497a9c6eb8
Status: Downloaded newer image for registry.servicemgr.apc:5000/library/trimmomatic:0.36--6
```
2. 在办公网络或外网访问，后续上线。


## 列出镜像
获取镜像之后，如何查看本地镜像，可以使用docker image ls命令：
```
$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
redis                latest              5f515359c7f8        5 days ago          183 MB
nginx                latest              05a60462f8ba        5 days ago          181 MB
mongo                3.2                 fe9198c04d62        5 days ago          342 MB
<none>               <none>              00285df0df87        5 days ago          342 MB
ubuntu               18.04               329ed837d508        3 days ago          63.3MB
ubuntu               bionic              329ed837d508        3 days ago          63.3MB
```
列表包含了 仓库名、标签、镜像 ID、创建时间 以及 所占用的空间。
其中仓库名、标签在之前的基础概念章节已经介绍过了。镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个 标签。因此，在上面的例子中，我们可以看到 ubuntu:18.04 和 ubuntu:bionic 拥有相同的 ID，因为它们对应的是同一个镜像。

## 删除本地镜像
在删除本地镜像之后需确保镜像已经保存，或者不再使用，可以使用 docker iamge rm 命令删除镜像，其格式为：
```
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```
比如我们有这么一些镜像：
```
$ docker image ls
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
centos                      latest              0584b3d2cf6d        3 weeks ago         196.5 MB
redis                       alpine              501ad78535f0        3 weeks ago         21.03 MB
docker                      latest              cf693ec9b5c7        3 weeks ago         105.1 MB
nginx                       latest              e43d811ce2f4        5 weeks ago         181.5 MB
```
我们可以用镜像的完整 ID，也称为 长 ID，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用 短 ID 来删除镜像。docker image ls 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。

比如这里，如果我们要删除 redis:alpine 镜像，可以执行：

```
$ docker image rm 501
Untagged: redis:alpine
Untagged: redis@sha256:f1ed3708f538b537eb9c2a7dd50dc90a706f7debd7e1196c9264edeea521a86d
Deleted: sha256:501ad78535f015d88872e13fa87a828425117e3d28075d0c117932b05bf189b7
Deleted: sha256:96167737e29ca8e9d74982ef2a0dda76ed7b430da55e321c071f0dbff8c2899b
Deleted: sha256:32770d1dcf835f192cafd6b9263b7b597a1778a403a109e2cc2ee866f74adf23
Deleted: sha256:127227698ad74a5846ff5153475e03439d96d4b1c7f2a449c7a826ef74a2d2fa
Deleted: sha256:1333ecc582459bac54e1437335c0816bc17634e131ea0cc48daa27d32c75eab3
Deleted: sha256:4fc455b921edf9c4aea207c51ab39b10b06540c8b4825ba57b3feed1668fa7c7
```
我们也可以用镜像名，也就是 <仓库名>:<标签>，来删除镜像。

```
$ docker image rm centos
Untagged: centos:latest
Untagged: centos@sha256:b2f9d1c0ff5f87a4743104d099a3d561002ac500db1b9bfa02a783a46e0d366c
Deleted: sha256:0584b3d2cf6d235ee310cf14b54667d889887b838d3f3d3033acd70fc3c48b8a
Deleted: sha256:97ca462ad9eeae25941546209454496e1d66749d53dfa2ee32bf1faabd239d38
```
当然，更精确的是使用 镜像摘要 删除镜像。
```
$ docker image ls --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```

虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除。

```
$ docker image prune
```

## 制作镜像
推荐使用Dockerfile定制镜像，不建议使用docker commit 制作镜像。镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，这个脚本就是 Dockerfile。Dockerfile定制请参考 [如何使用docker封装容器R](/en/软件安装/如何使用docker封装容器R)。dockerfile命令参考[dockerfile命令详解](https://yeasy.gitbook.io/docker_practice/image/dockerfile)


## 保存镜像

对所有保存在本地仓库的容器镜像，需在wiki提供dockerfile文件内容和使用帮助文档，否则管理员有权删除该镜像。详细请参考镜像管理[镜像管理](/en/容器使用/镜像管理)。**防止部分用户镜像过大。**



## 操作容器
## 在qsub 中指定集群中镜像

在通过qsub投递容器任务时，用户可直接引用镜像名称即可，无需指定指定前缀。例如：

```
docker pull registry.servicemgr.apc:5000/library/r-base.sva:4.2.2
qsub -i registry.servicemgr.apc:5000/library/r-base.sva:4.2.2 -cwd -l mem=2G:cpu=2 --vol /home/zhuwenjie/test/ demo.docker.sh
```

## 在wdl脚本中指定集群中镜像

在wdl脚本中，可通过runtime 指定镜像，同样，无需指定前缀，即可使用

```wdl
## wdl version 1.1

task my_task1 {
	...
  runtime{
  	docker :"registry.servicemgr.apc:5000/library/r-base.sva:4.2.2"
    ...
  }

}
```

## 镜像的使用
### 运行一个程序
用户可以通过以下命令运行一个docker 任务

```
docker run -it [image_name:tag] [/path/to/program]
```

特别地，用户可以通过

```
docker run -it [image_name:tag] bash
```

进入镜像，运行程序

### 进入一个存在的容器
对于一个已经存在的容器，用户可以通过以下命令进入容器查看运行状态

```
docker exec -it [container-id] bash
```

### 如何映射和访问存储卷
在实际使用上，Docker 必然需要与宿主机存在数据交互。在使用时需要用户启动容器时，增加挂载位置信息，其具体命令如下
```
docker run -v /path/to/local:/path/to/container [image_name:tag] [program]
```







