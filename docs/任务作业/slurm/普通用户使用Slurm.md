# 普通用户使用Slurm

## 1. 创建项目

用户在使用`Slurm`功能前，需要在Poros界面上创建对应项目。

![slurm普通用户使用1.png](/pic/slurm/slurm普通用户使用1.png)

![slurm普通用户使用2.png](/pic/slurm/slurm普通用户使用2.png)

![slurm普通用户使用3.png](/pic/slurm/slurm普通用户使用3.png)

![slurm普通用户使用4.png](/pic/slurm/slurm普通用户使用4.png)


## 2. 提交作业
普通用户使用Slurm的方式与标准用法一样，请参考（https://slurm.schedmd.com/overview.html）。

![如何提交任务到slurm.png](/pic/slurm/如何提交任务到slurm.png)

<!--
## 3. 管理账号
需要注意的是用户是否能够提交任务到某个账号需要由管理员进行配置，配置方式也是使用标准的sacctmgr命令。slurm中创建的账号将在集群中统一计费。

![slurm账号管理.png](/pic/slurm/slurm账号管理.png)
-->

## 3. salloc
申请计算节点，然后登录到申请到的计算节点上运行指令。
salloc的参数与sbatch相同，该部分介绍一个简单的使用案例。

![salloc运行例子.jpg](/pic/slurm/salloc运行例子.jpg)
