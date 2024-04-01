# SGE模拟使用说明
Achelous 提供SGE模拟功能，可通过qsub、qstats、qdel 等命令，实现作业投递，取消，状态查看等功能。这些命令的输入输出尽可能兼容SGE原来命令格式。


基本使用方法：

```
### 一般使用方式，其中queue为需要投递到的队列名，all队列为不指定的默认值。

qsub  -l mem=2G:cpu=2 -N jobname -q queue work.sh 


qsub -V -l mem=2G:cpu=2 -N jobname -q centos work.sh 

###  或者不指定队列默认使用all，只包含centos节点

qsub -V -l mem=2G:cpu=2 -N jobname  work.sh
```
qsub命令的基本用法与原生qsub几乎一样。例如通过-l参数指定资源：
> 	cpu：通过cpu设置程序使用的cpu资源，可为小数。
> 	memory：通过mem来指定程序使用的最大内存。
> 	gpu设备：通过gpus和gpumem两项进行限定。分别限定GPU数量或最大显存。两项选填一项即可。一般应该使用gpu，如果需要使用gpumem需要咨询系统管理员。
> 	节点：通过h=<节点hostname>来进行指定。
{.is-info}

> 注意!!!
> 如果用户使用 docker 模式运行计算任务，则可以使用所有计算节点,对应 -q 默认值 `all`
{.is-warning}

例如：
> 	qsub -l cpu=1:mem=128M:gpus=1:h=biohpc005 mytest.sh 
>
> 	qsub -l cpu=1:mem=128M:gpumem=512M:h=biohpc005 mytest.sh 
{.is-info}

其它支持的高级功能包括（但不限于，具体可查看qsub -h）：
+ -V选项传递给任务登录节点的环境变量
+ PE选项支持smp
+ 支持Array Job
+ **运行Docker容器：-i**
+ **运行Singularity容器：-si**


注意：	cpu和mem两项预设值很小，一般生信程序投递几乎都需要对这两项进行设定

## 例一：qsub 投递命令行作业
### 1. 数据准备
用户可以在下载页面下载测试数据集

```
cd /mnt/vol1 
wget -c http://achelous.org/download/demo.dataset.tar.gz
cd demo-dataset 
ls ./
```
### 2. 安装bwa
用户需要自行在存储路径下安装bwa 。可使用conda 进行整体安装，也可以自行下载源码进行编译安装。

### 3. 作业脚本准备
用户需要自行编写需要投递的脚本。脚本内容如下

```
#! /usr/bin/sh  
/mnt/vol/bwa/bwa mem -t 5 /mnt/vol/demo-dataset/ref.fa /mnt/vol/demo-dataset/demo.r1.fq /mnt/vol/demo-dataset/demo.r2.fq > /mnt/vol/demo-dataset/mydemo.sam
```
### 4. 作业投递
用户在登录节点下才可以使用qsub进行作业投递，作业投递命令如下：

```
foo@local#: qsub my-first-sge-job.sh 
Job (id: 6228, name: nil) has been submitted
```
任务成功投递后会返回job-id，用户可以job-id 对作业状态进行查看。

### 5. 作业状态查看
用户可使用qstat 进行自身账户下作业运行状态查看,运行结果如下：

```
foo@local#: qstat -j 6228
==============================================================
job_number:                 6228
state:                      finished
exec_file:                  job_scripts/paladin-task.a8a9ec25-c430-4949-bff3-51d006d089d9
submission_time:            Thu Apr 29 14:31:11 CST 2021
deadline:                   Thu Apr 29 14:32:28 CST 2021
owner:                      foo
uid:                        10007
group:                      foo
gid:                        10007
sge_o_home:                 /home/foo
sge_o_log_name:             foo
sge_o_path:                 /usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/root/myriad/hadoop-2.7.6/bin:/home/grid/hbase-1.4.9/bin:/usr/local/maven/apache-maven-3.6.1/bin:/usr/local/slider/bin:/home/yawei/.local/bin:/home/yawei/bin
sge_o_shell:                /bin/bash
sge_o_workdir:              /home/foo
sge_o_host:                 Cc3Apc
execution_time:             Thu Apr 29 14:31:12 CST 2021
account:                    foo
stderr_path_list:           /home/foo/my-first-sge-job.sh.e6228Cc3Apc
mail_list:                  foo@Cc3Apc
job_name:                   foo.paladin-task.M533sw69
stdout_path_list:           /home/foo/my-first-sge-job.sh.o6228Cc3Apc
priority:                   1
env_list:                   TERM=NONE
script_file:                my-first-sge-job.sh
binding:                    NONE
job_type:                   NONE
usage         1:            cpu=0.100000, mem=64.00000 MB, io=00000 GB, vmem=N/A, maxvmem=N/A
binding       1:            NONE
scheduling info:            (Collecting of scheduler job information is turned off)

```

### 6. 作业结果查看

当作业运行结束后，在/mnt/vol/demo-dataset/目录下会有相应的结果文件生成，用户可进入目录进行查看：

```
foo@local#: cd /mnt/vol/demo-dataset/ 
foo@local#: ls ./
mydemo.sam
```

---

## 例二：qsub 运行Container 作业

### 1. 准备脚本
此应用比较典型的场景为：基因组建立比对所需索引。下面以bwa index为例，编写对应的脚本
```
#!/bin/bash
/bio/bwa/bwa index -a is  /autofs/vol6/qsub-test/chrM.fa.gz
```
需要注意

脚本第一行 #!/bin/bash 不能缺少
脚本中所有的路径均为绝对路径。若存在相对路径，需要在脚本中添加 cp 或 mv 将文件转移至存储路径下。
保存的脚本需要保存至存储路径下。

### 2. 任务投递
采用以下命令进行任务的投递
```
[user@Cc7Apc ~] qsub -l cpu=1:mem=512M:gpus=1 -i bwa:base --vol /var/log/:/var/test/log --vol /autofs/vol6/ /autofs/vol6/qsub-test/bwa-index.sh
Job (id: 47, name: nil) has been submitted
```
其中：

-l 为程序运行所需要的资源限制
-i 为程序运行所需的镜像名称
--vol 为目录的挂载位置；支持两种类型的挂载方式:

i) 指定挂载点形式，如：/var/log/:/var/test/log /var/log/ 为物理机目录；/var/test/log 为容器内挂载目录
ii) 默认挂载形式，如： /autofs/vol6/ 在此模式下，Achelous 系统会自动将指定目录挂载至容器的同名目录下。

/autofs/vol6/yawei/test/wdtest.sh 为所投递的脚本。
结果中id 为作业标识，后续用于查看作业状态。

### 3. 结果查看
用户可以通过以下命令查询作业运行状态
```
[user@Cc7Apc ~] qstat -j 47 ## 47 为作业编号

==============================================================
job_number:                 47
state:                      finished
state_reason:               Container exited with status 0
exec_file:                  job_scripts/paladin-task.715d5b71-54ce-43e9-815a-1b1ce221728c
submission_time:            Mon Jun  7 14:50:10 CST 2021
deadline:                   Mon Jun  7 14:50:13 CST 2021
owner:                      user
uid:                        10844
group:                      user
gid:                        10844
sge_o_home:                 /home/user
sge_o_log_name:             user
sge_o_path:                 /home/user/.cargo/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/var/lib/snapd/snap/bin:/usr/local/go/bin:/home/user/.local/bin:/home/user/bin
sge_o_shell:                /bin/bash
sge_o_workdir:              /home/user
sge_o_host:                 Cc6Apc
execution_time:             Mon Jun  7 14:50:12 CST 2021
account:                    user
stderr_path_list:           /home/user/bwa-index.sh.e47Cc1Apc
mail_list:                  user@Cc1Apc
job_name:                   user.paladin-task.qHBc2899
stdout_path_list:           /home/user/bwa-index.sh.o47Cc1Apc
priority:                   1
env_list:                   TERM=NONE
script_file:                /autofs/vol6/qsub-test/bwa-index.sh
binding:                    NONE
job_type:                   NONE
Starve Timeout:             1h 0m 0s
usage         1:            cpu=1.000000, mem=512.00000 MB, io=00000 GB, vmem=N/A, maxvmem=N/A
binding       1:            NONE
scheduling info:            (Collecting of scheduler job information is turned off)
==============================================================

[user@Cc1Apc ~]$ ll /autofs/vol6/qsub-test/
total 37
-rw-rw-r-- 1 user user    76 Jun  7 14:49 bwa-index.sh
-rwxr--r-- 1 user root    5537 Jun  7 14:47 chrM.fa.gz
-rw-r--r-- 1 user user    10 Jun  7 14:50 chrM.fa.gz.amb
-rw-r--r-- 1 user user    35 Jun  7 14:50 chrM.fa.gz.ann
-rw-r--r-- 1 user user 16648 Jun  7 14:50 chrM.fa.gz.bwt
-rw-r--r-- 1 user user  4144 Jun  7 14:50 chrM.fa.gz.pac
-rw-r--r-- 1 user user  8336 Jun  7 14:50 chrM.fa.gz.sa
```

注意 为了保证存储上数据的安全，用户在通过qsub 功能进行docker 作业提交时，只能对自己名下的目录或文件进行修改。

---

## qstat命令查看作业运行状态    

用户可通过极道qstat命令，查看自身用户下运行的特定作业的状态。
例如运行qstat可查看该用户所有的作业：
```
[xtao@biohpc003 sge]$
[xtao@biohpc003 sge]$ qsub -l cpu=0.5:mem=4096M --cwd -i offloadmt named.sge
Your job 13 ("named.sge") has been submitted
[xtao@biohpc003 sge]$
[xtao@biohpc003 sge]$ qstat
Total: 1
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
job-ID    prior  name                                                        user              state  submit at                  start at                   terminate at               queue                         ja-task-ID
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
13        1      xtao.paladin-task.NMtjWCLh                                  xtao              qw     2022-12-21T17:02:37+08:00  -                          -                          -                             -
[xtao@biohpc003 sge]$

```
通过-j选项可查看指定作业的信息：

```
[xtao@biohpc003 sge]$ qstat -j 13

==============================================================
job_number:                 13
job_name:                   xtao.paladin-task.NMtjWCLh
job_type:                   DOCKER
state:                      finished
state_reason:               Container exited with status 0
exit_code:                  0
exit_code_msg:              Container exited with status 0
exec_file:                  job_scripts/paladin-task.78eb0278-7148-46c8-83c9-dfe07c87cc80
submission_time:            2022-12-21T17:02:37+08:00
execution_time:             2022-12-21T17:02:43+08:00
deadline:                   2022-12-21T17:02:48+08:00
unreachable_time:           -
owner:                      xtao
account:                    xtao
uid:                        10004
gid:                        10004
groups:                     [10004(xtao)]
sge_o_home:                 /home/xtao
sge_o_log_name:             xtao
sge_o_path:                 /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/xtao/.local/bin:/home/xtao/bin
sge_o_shell:                /bin/bash
sge_o_workdir:              /home/xtao/sge
sge_o_host:                 biohpc004
stdout_path_list:           /home/xtao/sge/named.sge.o13biohpc003
stderr_path_list:           /home/xtao/sge/named.sge.e13biohpc003
mail_list:                  xtao@biohpc003
priority:                   1
env_list:                   ENVIRONMENT=BATCH,HOME=/home/xtao,REQUEST=named.sge,SGE_O_HOME=/home/xtao,SGE_O_HOST=biohpc003,SGE_O_WORKDIR=/home/xtao/sge,SGE_STDERR_PATH=/home/xtao/sge/named.sge.e13biohpc003,JOB_ID=13,SGE_STDOUT_PATH=/home/xtao/sge/named.sge.o13biohpc003,NQUEUES=1,SGE_CWD_PATH=/home/xtao/sge
script_file:                named.sge
binding:                    NONE
container_image:            offloadmt
container_network:          NONE
container_port_env:         NONE
port_binding:               NONE
volume_binding:             /home/xtao:/home/xtao, /home/xtao/sge:/home/xtao/sge
starve_timeout:             168h 0m 0s
running_timeout:            NONE
gpus_mode:                  GPU_MATCH_MODE_GPU_DEVICE_ONLY
memory_limit_ratio:         -
resource_request:           cpu=0.50, mem=4.00  G, gpus=0.00, gpui=0.00, gpumem=0.00  M
usage          13:          cpu=0.50, mem=4.00  G, gpus=0.00, gpui=0.00, gpumem=0.00  M
binding        13:          NONE
command_launch_type:        POSIX SHELL
command_content:            "/bin/bash -c /home/xtao/sge/named.sge"
constraints:                []
schedule_domain:            [all]
labels:                     []
scheduling_info:            (Collecting of scheduler job information is turned off)
==============================================================

[xtao@biohpc003 sge]$

```
这些信息显示了任务的运行情况、环境变量、运行时间等信息，与原生SGE兼容。如果投递的是容器作业，还会包括镜像、卷挂载等等。

## qhost命令查看集群队列
用户或管理员用户可根据qhost命令查看集群负载状态，例如：
```
[xtao@biohpc003 sge]$ qhost
Cluster: biohpc
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
HOSTNAME                ARCH      ACTIVE  NCPU     CPUSUSE  NSOC NCOR NTHR LOAD  MEMTOT   MEMUSE   GPUSTOT  GPUSUSE  GPUISTOT  GPUISUSE  GPUMEMTOT  GPUMEMUSE  SWAPTO  SWAPUS
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
global                  -         -       -        -        -    -    -    -     -        -        -        -        -         -         -          -          -       -
biohpc003               lx-amd64  true    22.00    19.60    4    4    4    0.01  117.7G   66.0G    -        -        -         -         -          -          0.0     0.0
biohpc004               lx-amd64  true    155.00   0.00     4    4    4    0.01  2.0T     0.0M     -        -        -         -         -          -          0.0     0.0
biohpc005               lx-amd64  true    155.00   0.00     4    4    4    0.01  1.7T     0.0M     -        -        -         -         -          -          0.0     0.0
biohpc006               lx-amd64  true    155.00   0.00     4    4    4    0.01  2.0T     0.0M     -        -        -         -         -          -          0.0     0.0
biohpc007               lx-amd64  true    155.00   0.00     4    4    4    0.01  2.0T     0.0M     -        -        -         -         -          -          0.0     0.0
biohpc008               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc009               lx-amd64  true    22.00    5.00     4    4    4    0.01  123.7G   10.0G    -        -        -         -         -          -          0.0     0.0
biohpc010               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc011               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc012               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc013               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc014               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc015               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc016               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc017               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc018               lx-amd64  true    22.00    5.00     4    4    4    0.01  123.7G   10.0G    -        -        -         -         -          -          0.0     0.0
biohpc019               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc020               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc021               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc022               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc023               lx-amd64  true    10.00    0.00     4    4    4    0.01  60.7G    0.0M     -        -        -         -         -          -          0.0     0.0
biohpc024               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc025               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc026               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc027               lx-amd64  true    22.00    0.00     4    4    4    0.01  123.7G   0.0M     -        -        -         -         -          -          0.0     0.0
biohpc028               lx-amd64  true    62.00    0.00     4    4    4    0.01  1005.6G  0.0M     -        -        -         -         -          -          0.0     0.0
[xtao@biohpc003 sge]$

```


## qdel 取消正在运行的作业
当用户希望终止一个已经投递的任务，可以采用qdel进行操作。例如：
```
[xtao@biohpc003 sge]$
[xtao@biohpc003 sge]$ qsub -l cpu=0.5:mem=4096M --cwd -i offloadmt named.sge
Your job 14 ("named.sge") has been submitted
[xtao@biohpc003 sge]$
[xtao@biohpc003 sge]$ qdel 14
accept to deleted job 14
[xtao@biohpc003 sge]$
[xtao@biohpc003 sge]$ qstat
[xtao@biohpc003 sge]$

```
## qsub 命令：
qsub 命令参数简介：
   打印帮助信息:
   `--sgehelp, --hl                                print sge help (default: false)`  
  作业开始时间:
   ` --date_time value, -a value                    request a start time`  
 
   `--context_list value, --ac value               add context variable(s)` 
   
   `--fname value, --Ap value                      add a new parallel environment from file`

   `--binary value, -b value                       add context variable(s)`
   
   `--ckpt_selector value, -c value                define type of checkpointing for job`
   
   `--directive_prefix value, -C value             define command prefix for job script`
   `--simple_context_list value, --dc value        delete context variable(s)`
   `--listname_list value, --dul value             delete userset list(s) completely`
   `--start_now, --now                             start job immediately or not at all (default: false)`
   `--context_lists value, --sc value              set job context (replaces old context)`
   `--soft, --sof                                  consider following requests as soft (default: false)`
   `--rqs_list value, --srqs value                 show resource quota set(s)`
   `--sync, --sy                                   wait for job to end and return exit code (default: false)`
   `--hard, --hr                                   consider following requests hard (default: false)`
   `--verify, --vf                                 do not submit, just verify (default: false)`
   指定作业名称，否则会随机生成作业名称：
   `--name value, -N value                         specify job name. e.g. '-N my_job'`
   `--wc_pe_name value, --pe value                 request slot range for parallel jobs`
   `--terse, --ts                                  causes qsub to display only the job-id of the job being submitted (default: false)`
   `--task_id_range value, -t value                create a job-array with these tasks`
   在当前目录下提交作业，如果程序未指定输出路径及未指定log日志文件，则log日志和结果在当前目录保存。
   `--workdir, --cwd                               use current working directory (default: false)`
   `--working_directory value, --wd value          use working_directory`
   job的标准输出文件，可写绝对路径或相对路径。
   `--out_path_list value, -o value                specify standard output stream path(s)`
   job的报错信息输出文件，科协绝对路径或相对路径、
   `--err_path_list value, -e value                specify standard error stream path(s)`
   
   `--merge_stdout_and_stderr value, -j value      merge stdout and stderr stream of job, specified 'yes' or 'y' will merge stderr to stdout, specified 'no' or 'n' will forbidden combined output`
   `--job_identifier_list value, --hold_jid value  define jobnet interdependencies`
   指定作业所需资源，按实际需求申请，h=Cc1Apc 则表示在此节点运行，不推荐。内存溢出会报错，CPU允许超限使用，超额会变慢。
   `--resource_list value, -l value                request the given resources, unit of memory and gpumem support M,G,T. e.g. '-l cpu=1:mem=128M:gpus=1(gpui=1/gpumem=512M):h=Cc1Apc'`
   
   `--alternate_resource value                     after the waiting time, the task is still unable to run due to insufficient resources, and the alternated resources will be enabled one by one, unit of memory and gpumem support M,G,T, unit of wait time support h, m, s, like: 35s means 35second, current not support multiple units to be compounded, like:1h17m33s. e.g. '--alternate_resource cpu=2:mem=1.5G:gpus=1(gpui=1/gpumem=2G):wt=1h'`
   `--gpus_mode value                              specifiy gpus mode, include:'gpus_first','gpus_only','gpui_first', defaul is 'gpus_only'. e.g. --gpus_mode gpus_first`
   `--mem_limit_ratio value                        specifiy ration of memory limit, ration value should be greater than or equal to 1`
   `--revocable_resource                           task will use revocable resources first (default: false)`
   自动调节CPU和内存的使用量，如果程序允许慢或内存溢出，可尝试使用该参数优化资源。
   `--auto_adjust_resource value, --aar value      auto adjust resource of task instead of OOM, support 'up', 'down', 'auto'. e.g. --aar auto`
   指定需要使用的队列，默认队列all.
   `--wc_queue value, -q value                     bind job to queue. e.g. '-q queue_test'`
   指定项目名称，便于统计及计量计费.
   `--project value                                bing job to project. e.g. '--project my_project'`
   指定优先级，10为最高优先级。
   `--priority value, -p value                     define job's relative priority, range:0-10`
   将个人特定环境变量带入计算节点，例如：PYTHONPATH:/path/to/your/python
   `--variable_list value, -v value                export these environment variables. e.g. '-v k1:v1'`
   将当前个人所有环境变量传入计算节点，当conda有多个环境变量时，只传入当前已经激活的环境变量。
   `--env, -V                                      export all environment variables (default: false)`
   `--disable_oom_kill                             instead of kill the task when the task memory exceeds the hard limit, keep task hang. Note: works for container mode. (default: false)`
   `--running_timeout value                        If task is not terminated within the running timeout period, the task will be killed. Support integer units of time: h, m, s, like: 35s means 35second, 19m means 19minute, 1h means 1hour, current not support multiple units to be compounded, like:1h17m33s`
   `--starve_timeout value, --st value             If task is not successfully scheduled within the starve timeout period, the task will be terminated. Support integer units of time: h, m, s, like: 35s means 35second, 19m means 19minute, 1h means 1hour, current not support multiple units to be compounded, like:1h17m33s`
  如果你所使用的软件或脚本在docker镜像中，使用-i 参数可将docker镜像传入计算节点计算，可解决软件依赖问题，需确保镜像在本地镜像仓库已保存。
 `--image value, -i value                        docker image name. e.g. '-i my_docker_image'`
 同-i 参数，支持singularity镜像。
 `--singularity_image value, --si value          singularity/docker image name. e.g. '-si my_singularity_image.sif' or '-si my_docker_image'`
 使用-i 参数时需增加--vol参数，将需要计算的数据和脚本目录传入计算节点。


   `--volume value, --vol value                    set volumes binding of container(docker or singularity), support '--vol [host_path]:[container_path]' format and '--vol [host_path]' format. e.g. '--vol /host/my_app:/container/my_app --vol /host/my_dir'`
   `--port value                                   set ports binding of container(docker or singularity), support '--port [host_port]:[container_port]' format, '--port [host_port]' format and '--port [container_port]' format`
   `--port_num value                               specify number of ports for container, port generate with random, only use for 'HOST' network. e.g. --port_num 3 will random generate three port for this container`
   如计算需要访问外网时，可以设置为--net HOST。
   `--network value, --net value                   specify network for container, support 'BRIDGE','HOST','NONE', default is 'NONE'. e.g. --net HOST`
   `--shell_interpreter value, --shell value       specify shell interpreter. e.g. '--shell /usr/bin/bash'`
   `--callback_addr value, --callback value        specify callback address to be used for notifications. e.g. '--callback http://localhost:8080/callback'`
   `--help, -h                                     show help (default: false)`





