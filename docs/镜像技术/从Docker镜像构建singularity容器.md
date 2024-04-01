
## Singularity容器
Singularity没有镜像（image）的概念，只有容器（container），容器包含两种格式：SIF和Sandbox。
- SIF
  - a compressed read-only Singularity Image File (SIF) format, suitable for production (default)
- Sandbox
  - a writable (ch)root directory called a sandbox, for interactive development ( --sandbox option)


## 构建SIF容器
接下来将详细介绍在BioHPC集群中如何从已有的docker镜像构建SIF容器。

### 构建环境
在BioHPC集群，我们为大家创建了两个用于Docker和Singularity构建和测试的虚拟机, 见[测试节点](/en/注册登录/测试节点)
<!--
![docker-workstation.png](/docker-workstation.png)
- docker-workstation
  - 该虚拟机的OS为Centos，需要在Centos系统下测试可以使用此虚拟机，可以在Poros管理系统的虚拟机部分查找该虚拟机，点击虚拟机名称即可获取虚拟机信息。
  ![docker-workstation-centos.png](/docker-workstation-centos.png)
  
- docker-workstation-ubuntu
  - 该虚拟机的OS为Ubuntu，需要在Ubuntu系统下测试可以使用此虚拟机，在Poros管理系统的虚拟机部分查找该虚拟机，点击虚拟机名称即可获取虚拟机信息。
  ![docker-workstation-ubuntu.png](/docker-workstation-ubuntu.png)
-->
  
### 构建步骤
1. 登入虚拟机，本次示例使用demo用户登入docker-workstation-ubuntu
  - `ssh demo@10.10.6.51`并输入password登入
2. docker client登录docker registry
  - `docker login registry.servicemgr.biohpc:5000`
3. 从docker registry拉取所需image到本地，本示例选择`test/debian:stable`这个image
  - `docker pull registry.servicemgr.biohpc:5000/test/debian:stable`
4. 将刚刚拉取到本地的image保存为归档文件
  - `docker save -o archive/debian-stable.tar registry.servicemgr.biohpc:5000/test/debian:stable`
5. 从image的归档文件构建SIF容器
  - `singularity build /home/demo/singularity/archive/debian-stable.tar docker-archive:/home/demo/singularity/container/debian-stable.sif`
6. 使用构建好的SIF容器
  - `singularity shell /home/demo/singularity/container/debian-stable.sif`
  - `singularity run /home/demo/singularity/container/debian-stable.sif date`
![build-from-docker-archive-1.png](/build-from-docker-archive-1.png)
![build-from-docker-archive-2.png](/build-from-docker-archive-2.png)


