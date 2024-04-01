# GPU 资源使用指南

目前机器学习工具在生信领域得到了越来越多的应用。而因此在实践中，往往由于计算量较大，需要使用到GPU 设备，本文仅在于简述Achelous 平台上使用GPU的基本方法和相关原则。由于本平台目前的GPU 为 英伟达（Nvidia）公司产品，因此在下述中均指英伟达出品的GPU 。

## 1. GPU 使用概述

GPU使用主要取决于两个层次的因素：

+ **硬件驱动**，该部分主要于硬件版本有关。该部分属于系统管理层，一般用户无需考虑。
+ **应用层**，该部分主要指cuda库和其上的扩展库（例如：tensorflow、torch等）。

`cuda`（Compute Unified Device Architecture，意为“统一计算架构”）通过程序控制底层硬件GPU进行计算。其版本与GPU硬件型号直接相关，目前主流应用支持版本向下兼容（也就是低版本的cuda库上层应用，可以在高版本的GPU硬件上运行。反之则不行）

`tensorflow`和`Pytorch` 是目前两个主流的，可以使用GPU资源的机器学习框架。其版本与 `cuda`之间存在对应关系，因此在安装过程中，请注意版本是否合适。

## 2. 如何安装GPU 相关软件

### 2.1 使用 conda进行安装 

`conda` 自带的软件依赖解析功能，因此可直接进行安装相关的软件。例如：

```bash
### more informations plz see https://github.com/JackieHanLab/TOSICA ### 

conda create -n TOSICA python=3.8 scanpy
conda activate TOSICA
conda install pytorch=1.7.1 torchvision=0.8.2 torchaudio=0.7.2 cudatoolkit=10.1 -c pytorch
```

安装后，用户可以通过进程模式运行任务（如`qsub`进行任务提交）。

> 注意：由于`conda`安装过程相对自动，因此需要注意安装过程中依赖的`cuda`版本是否合适。若`cuda`版本过新，注意选择降低版本。
{.is-warning}


### 2.2 通过 Docker 使用 GPU 相关软件


目前一部分应用支持docker发布，例如`cell2location` 可通过下面的命令进行获取：

> 注意：请在镜像开发节点执行该命令 
{.is-warning}


```bash
docker pull quay.io/vitkl/cell2location
```

注意：需要确定docker中安装的cuda版本和系统硬件是否匹配

若用户需要自行编译安装特定版本的Tensorflow或 Pytorch， 可从英伟达提供的基础镜像开始进行安装。

```bash 
docker pull nvidia/cuda
```

> 具体细节，请仔细阅读对应[页面说明](https://hub.docker.com/r/nvidia/cuda)
{.is-info}

## 3. 如何使用GPU资源 

### 3.1 通过`qsub` 提交占用GPU资源的任务 

安装GPU的节点目前并不能普通用户直接登录，因此程序需要使用GPU资源，需要通过qsub或流程管理程序进行作业投递。qsub使用 

```bash
qsub -l cpu=1:mem=10G:gpus=1 ... ### 注意GPU资源对应参数为 gpus ！！！### 

```
### 3.2 通过WDL 脚本中指定GPU资源

用户采用wdl脚本编排流程，可在`task`中，通过`runtime` 属性指定使用GPU资源，例如：

```
task deepvariant_task{
    
    ... 
    ...

    runtime {
      docker:"demo/deepvariant:google-0.10.0-gpu"
      cpu:"5"
      gpu:"1"
      memory:"20G"
    }
}
```