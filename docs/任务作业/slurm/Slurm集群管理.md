# Slurm集群管理

集群中的Slurm集群由Achelous动态扩展，配置和管理方式与标准的Slurm没有差别，唯一的区别在于系统管理员可以动态扩展。

## 通过Poros界面管理集群

管理员通过admin账号登录Poros界面，在框架页面，可以看到Slurm集群属于的调度框架。

![slurm框架.png](/server_usage/slurm框架.png)

点击进入框架，可以看到slurmd运行的节点。

![slurmd任务列表.png](/server_usage/slurmd任务列表.png)

后续界面上将提供集群的扩展。

## 通过命令行动态扩展和收缩Slurm集群
命令行fusioncli可以被用来查看，扩展和收缩slurm集群。注意该命令只能用来管理集群规模，其它管理命令和配置请参考标准的slurm配置文档（https://slurm.schedmd.com/overview.html）。
### 查看slurm集群
系统管理员root可以在沙箱中通过命令行fusioncli cluster status查看当前集群状态。

![fusion查看slurm状态.png](/slurm/fusion查看slurm状态.png)

注意：
- 该命令只显示动态扩展的slurm集群信息
- 输出中的ClusterSize指的是最小规模，如果slurmd掉线使当前集群规模小于该值，将自动拉起一个新的slurmd

另外下述命令可以显示slurm集群配置信息，其中的ScheduleDomain指明了slurmd启动的调度域。如果要在某个节点上动态启动slurmd，需要将节点加入到该调度域。

![fusion查看配置.png](/slurm/fusion查看配置.png)

### 创建新的slurm节点
root用户可以通过指定资源（CPU、内存、GPU）启动一个新的slurmd，相当于扩展了现有集群规模。

![扩展slurm集群.png](/slurm/扩展slurm集群.png)

注意slurmd只会在配置的队列所在的节点启动。

### 杀掉slurm节点
root用户可以杀掉某个slurmd，相当于收缩了slurm集群，将资源归还给系统。
1.  选择要kill的节点

![fusionci选择kill的任务.png](/slurm/fusionci选择kill的任务.png)

1.  kill掉该节点的slurmd

![fusioncli杀掉slurm节点.png](/slurm/fusioncli杀掉slurm节点.png)

1. 节点down后通过scontrol delete node=xxx将节点删除

![删除节点.png](/slurm/删除节点.png)
