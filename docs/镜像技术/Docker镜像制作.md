# Docker镜像制作

## 如何获取Docker镜像
为了生物信息相关工作者，更方便的使用Docker 容器化技术。目前搭建了[本地私有仓库](https://10.61.51.7:5000/)极道科技为用户提供了的[极道社区 Docker Hub](http://www.xtaohub.com:8280/)， 用户可以轻松获取生物信息相关的分析软件的镜像资源。

用户可通过在宿主机上运行以下命令来获取镜像资源：

```
[xtao@biohpc003 sge]$ sudo docker pull centos

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for xtao:
Using default tag: latest



latest: Pulling from library/centos
a1d0c7532777: Pull complete
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
[xtao@biohpc003 sge]$
[xtao@biohpc003 sge]$

```

## 如何制作自己的Docker镜像

### 通过Dockerfile制作

用户也可以通过Dockerfile 生成镜像，以samtools 的镜像为例，下面为其对应的Dockerfile 中的内容
```
################## BASE IMAGE ######################

FROM biocontainers/biocontainers:latest

################## METADATA ######################
LABEL base.image="biocontainers:latest"
LABEL version="2"
LABEL software="Samtools"
LABEL software.version="1.3.1"
LABEL about.summary="Tools for manipulating next-generation sequencing data"
LABEL about.home="https://github.com/samtools/samtools"
LABEL about.documentation="https://github.com/samtools/samtools"
LABEL license="https://github.com/samtools/samtools"
LABEL about.tags="Genomics"

################## MAINTAINER ######################
MAINTAINER Saulo Alves Aflitos <sauloal@gmail.com>

RUN conda install samtools=1.3.1

WORKDIR /data/

CMD ["samtools"]
```

用户将上述内容保存为Dockerfile 后，执行以下代码可以生成对应的镜像

```
cp /path/to/Dockerfile ./
docker build ./
```

### Dockerfile 多阶段构建 

第一步： 编写Dockerfile 文件

```
FROM library/debian:latest as builder

RUN apt update -y 

RUN apt install r-base -y 

FROM library/debian:latest as prod
RUN apt update -y
RUN apt install  bwa -y 

```

第二步：构建镜像

```
docker build . -t debian.r-base.bwa:v1.0
```
例如: 当我们只想构建 builder 阶段的镜像时，增加 --target=builder 参数即可

```
$ docker build --target builder -t username/imagename:tag .
```

