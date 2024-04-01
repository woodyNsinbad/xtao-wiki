# Docker 镜像管理功能

## 1. web管理：
用户可通过 https://10.61.51.7:5000/ 的 Harbor 镜像管理页面，对镜像进行管理和查看。也可以新建私有仓库，推荐使用library仓库。
平台用户可通过该地址，对自己有使用权限的镜像进行查看。登录用户名及密码为平台通用密码。


## 2. registry服务，镜像的push和pull。

### 2.1 平台私有仓库使用

> 例如：需要使用debian最新的stable版本，可以直接从docker hub获取。
> 登陆节点不提供docker run权限，如需拉取公共镜像，需在[测试节点](/en/注册登录/测试节点)进行操作。
{.is-success}


 第一步：登陆本地私有仓库：registry.servicemgr.apc:5000
 ```
 docker login registry.servicemgr.apc:5000
 ```
 第二步： 从公共仓库来取镜像或自己制作镜像。
 ```
 docker pull debian:stable
 ```
 第三步：将公共镜像的名称转换为私有镜像的名称
 ```
 docker tag debian:stable registry.servicemgr.apc:5000/library/debian:stable
 ```
 > 注意：集群前缀为 registry.servicemgr.apc:5000/，用户需区分私有镜像和公共镜像，私有镜像自己管理，公共镜像只能使用，不能修改。例如registry.servicemgr.apc:5000/libray/myimage:v1 
{.is-warning}

 第四步：将构建好的私有镜像保存在私有仓库中。
 ```
 docker push registry.servicemgr.apc:5000/library/debian:stable
 ```
 经过上述操作后，镜像即被保存。用户可在今后集群环境内，使用该镜像。

> 用户在使用时，指定集群前缀，只需要指明镜像名称部分即可
{.is-info}


 
>  备注：办公网使用地址：registry.servicemgr.apc:5000
{.is-info}

  
 如需使用私有镜像，可通过qsub直接使用。

 例如：

```sh
qsub -cwd -i test/debian:stable -l cpu=2:mem=5g --vol /mnt/alamo01/users/xxx 1.sh


```
>  该qsub 命令中 -i 指向 私有仓库下的 test/debian:stable 镜像。
>  或写完整：registry.servicemgr.apc:5000/test/debian:stable
{.is-info}

平台中公共生信工具镜像，存放在library项目下。用户在使用时，需要在镜像前加上对应项目前缀。

例如：

```sh
qsub -cwd -i systools/rstudio-4.2.2 -l cpu=2:mem=5g --vol /mnt/alamo01/ 1.sh

## 该qsub 命令中 -i 指向 library 项目下的 rstudio-4.2.2 镜像。
```








