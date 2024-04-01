# WDL 交互式运行
## 简介
平台提供了两种运行WDL脚本的方式：
- 命令行交互式：用户通过wdl命令行交互式运行WDL程序，方便学习和调试，不建议生产使用。
- 服务托管方式：通过biocli命令将WDL程序提交为pipeline，运行作业。服务托管方式能够自动容错，是推荐的生产环境使用方式。

wdl命令行支持用户通过本地二级制程序、执行docker和投递到集群运行三种方式。用户应采用第三种方式减少登录节点的负载，且能够利用集群的计算资源。

## WDL 命令行介绍
运行WDL脚本主要使用wdl run命令。可通过wdl run -h 命令参看可选参数。
```
[demo@Cc3Apc wdl-interactive]$ wdl run -h
usage: wdl run [<flags>] <wdlfile> <datafile>

Run the WDL file on the input data

Flags:
  -h, --help                 Show context-sensitive help (also try --help-long and --help-man).
  -t, --trace                Trace the execution of Wom Graph
  -c, --concurrent           Concurrent execution the Wom Graph
  -w, --workdir=WORKDIR      The work directory of the WDL program, default: /opt/workflow
  -m, --mode=MODE            execution mode of wdl task: docker | paladin
  -r, --resources=RESOURCES  Resources avaliable to the docker container, such as CPU, etc
  -q, --queue=QUEUE          The schedule domain used by the job
  -p, --priority="0"         The priority of the job

Args:
  <wdlfile>   The file of the WDL program
  <datafile>  The input data file of the WDL program
```

该命令执行的时候需要指定脚本的主workflow文件和参数说明文件。其中参数说明文件为JSON格式。

## 配置自动卷挂载
如果用户希望运行Docker或者集群作业，需要进行基本的配置。在用户的home目录下，编辑文件`.wdlconfig`如下： 

```json
{
   "volumes" : [
        "/home","/mnt/alamo01", "/mnt/anna01"
    ]
}
```
一般用户只需要在文件中配置volumes挂载路径，即默认每个容器需要挂载的卷目录。由于wdl命令行没有bioflow自动卷挂载功能，所有的卷可以在~/.wdlconfig中指定，或者在task中通过volumes属性指定。

**如果不指定volumes，但是需要创建~/.wdlconfig，编写一个空{}。**

## 例一：wdl 进行作业集群投递

### 1 作业提交
以下为使用wdl命令进行投递的命令行示例：
```
(base) [demo@Cc3Saturn wdl-interactive]$ wdl run -w ./work -m paladin src/demo-pipeline.wdl input.json 
Start execution remotely in Paladin
The resources configuration is: Resources(Cpu:-1.000000,Memory:-1.000000,Volumes:/home,NotifyServer:,PaladinServer:paladin-backend.servicemgr.saturn:1026)

Workflow Outputs: 
 output_bam: /home/demo/wdl-interactive/work/wxs-wxs.markdup/P59_N.dup.bam
 gvcf: /home/demo/wdl-interactive/work/wxs-wxs.HaplotypeCaller/P59_N.g.vcf.gz

Time cost:  4m17.274331754s
```
其中：

-w 指定工作目录，必须是一个共享存储的目录
-m paladin指定投递作业到Paladin（也就是在集群中运行任务）。-m不指定的话会在本地执行，不建议这样。
-p 可以指定作业的优先级0~10
-q 可以指定作业的队列或者调度域
在工作目录下生成一个wdl-engine.log,该文件记录作业的执行过程。

示例中的input.json是一个JSON格式的文件，对应着作业的主workflow的输入参数赋值，如下所示：
```json
{
     "wxs.read1": "/home/demo/makeflow-demo/demo1.fq.gz",
     "wxs.read2": "/home/demo/makeflow-demo/demo2.fq.gz",
     "wxs.prefix" : "P59_N",
     "wxs.ref" : "/home/demo/makeflow-demo/chr22.fa"
}
```
其内容对应job的JSON中的WorkflowInput指向的部分：
```json
{
         "Name" : "demo-job",
         "Pipeline" : "GATK4-PART-PIPELINE",
         "InputDataSet" : {
               "WorkflowInput" : {
                       "wxs.read1": "/home/demo/makeflow-demo/demo1.fq.gz",
                       "wxs.read2": "/home/demo/makeflow-demo/demo2.fq.gz",
                       "wxs.prefix" : "P59_N",
                       "wxs.ref" : "/home/demo/makeflow-demo/chr22.fa"
                }
    },
         "Priority" : 7,
         "ScheduleDomains" : "all"
}
```
### 2 如何查看提交到Paladin的任务
wdl命令行产生的正在运行的任务提交到集群由Paladin调度，可以通过hermit qstat查看运行的任务：
```
(base) demo@Cc1Saturn:~$ 
(base) demo@Cc1Saturn:~$ hermit qstat
job-ID                                                        prior        name                        user        state                submit/start at                queue        slots        ja-task-ID
----------------------------------------------------------------------------------------------------------------------------------------------------------
paladin-task.10704ae6-b4cd-4fe9-973c-b77a10a16679        0        demo.wdlcmd.wxs-wxs.bwa_mem        demo        running        28 second(s) ago        -NA-        -NA-        -NA-
```
从上述结果可以看到，wdl命令行产生的task的name有明显的模式：USERID.wdlcmd.WDL-STAGE-NAME

如果需要查看作业的具体的信息，可以运行hermit qstat -i taskid或者qstat -lj taskid查看，例如：
```
[xtao@biohpc003 ~]$ qstat -lj paladin-task.e7d85548-5c1f-4938-85c1-a36e7cf36fd7

==============================================================
job_number:                 0
job_name:                   xtao.wdlcmd.wk-wk.s1
job_type:                   DOCKER
state:                      finished
state_reason:               Container exited with status 0
exit_code:                  0
exit_code_msg:              Container exited with status 0
exec_file:                  job_scripts/paladin-task.e7d85548-5c1f-4938-85c1-a36e7cf36fd7
submission_time:            2022-12-21T14:34:38+08:00
execution_time:             2022-12-21T14:34:40+08:00
deadline:                   2022-12-21T14:35:40+08:00
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
sge_o_workdir:              /home/xtao/wdl/simple/work/wk-wk.s1
sge_o_host:                 biohpc020
stdout_path_list:           NONE
stderr_path_list:           NONE
mail_list:                  xtao@biohpc003
priority:                   0
env_list:                   TERM=NONE
binding:                    NONE
container_image:            offloadmt:latest
container_network:          HOST
container_port_env:         CONTAINER_PORTS_AVAILABLE_HOST
port_binding:               NONE
volume_binding:             /home/xtao/wdl/simple/work/wk-wk.s1:/home/xtao/wdl/simple/work/wk-wk.s1, /mnt/anna01:/mnt/anna01, /mnt/anna02:/mnt/anna02
starve_timeout:             168h 0m 0s
running_timeout:            NONE
gpus_mode:                  GPU_MATCH_MODE_NO_LIMIT
memory_limit_ratio:         -
resource_request:           cpu=2.00, mem=3.91  G, gpus=0.00, gpui=0.00, gpumem=0.00  M
usage           0:          cpu=2.00, mem=3.91  G, gpus=0.00, gpui=0.00, gpumem=0.00  M
binding         0:          NONE
command_launch_type:        POSIX SHELL
command_content:            "/bin/sh -c echo \"Hello smoke3\" > somke3.txt\nsleep 60"
constraints:                []
schedule_domain:            [all]
labels:                     []
scheduling_info:            (Collecting of scheduler job information is turned off)
==============================================================

[xtao@biohpc003 ~]$

```

或者

```
[xtao@biohpc003 ~]$ hermit qstat -i paladin-task.e7d85548-5c1f-4938-85c1-a36e7cf36fd7

==============================================================
ID:                    paladin-task.e7d85548-5c1f-4938-85c1-a36e7cf36fd7
Job ID:                wdlcmd
Name:                  xtao.wdlcmd.wk-wk.s1
Host name:             biohpc020
Host IP:               172.168.78.20
Priority:              0
Resource limit:
  Memory limit ratio:  -
  Original:
    CPUS:              2
    Memory:            3.91 G
    GPUS:              -
    GPU Instance:      -
    GPU memory:        -
  Current:
    CPUS:              2
    Memory:            3.91 G
    GPUS:              -
    GPU memory:        -
Image:                 offloadmt:latest
ExecMode:              DOCKER
DisableOOMKill         false
Constraints:           []
Schedule domains:      [all]
Used schedule domain:  all
Environment variables: map[PALADIN_JOBID:paladin-task.e7d85548-5c1f-4938-85c1-a36e7cf36fd7 SRM_TASK_ID:paladin-task.e7d85548-5c1f-4938-85c1-a36e7cf36fd7]
Status:                TASK_FINISHED
Message:               Container exited with status 0
Exit code:             0
Exit message:          Container exited with status 0
Command launch type:   POSIX SHELL
Command content:       /bin/sh -c echo "Hello smoke3" > somke3.txt
sleep 60
Init time:             2022-12-21 14:34:38
Start time:            2022-12-21 14:34:40
Terminated time:       2022-12-21 14:35:40
Starved Time:          -
Starve Timeout:        168h 0m 0s
==============================================================

[xtao@biohpc003 ~]$

```

## 例二：wdl 进行作业本地运行


### 1. 准备示例脚本
在示例中 demo.fastqtobam.wdl 中为bwa 比对和samtools 比对结果格式转化。其内容如下：
```
### Achelous Demo pipeline2 ###
### contact us e-mail: di.wu@xtaotech.com ####
workflow fastqtobam{
    File fastq1
    File fastq2
    File Ref

    call bwa_mem { input: fastq1 = fastq1, fastq2 = fastq2, Ref = Ref }
    call samtobam { input: sam = bwa_mem.sam }
    output {
        File bam_file = samtobam.bam
    }
}

task bwa_mem{
    File fastq1
    File fastq2
    File Ref
    command{
        /bio/bwa mem ${Ref} -t 5  ${fastq1} ${fastq2} > toy.sam
    }
    output {
        File sam = "toy.sam"
    }
}

task samtobam{
    File sam
    command {
        /bio/samtools view -bS ${sam} -o toy.bam
    }
    output {
        File bam = "toy.bam"
    }
}
```
### 准备输入参数文件

input.json 文件中记录的程序运行相关参数，在此示例中，input.json 内容如下：
```json
{
 "fastqtobam.fastq1" : "/Bioinformatics-pipeline/demo-dataset/demo-reads/demo.r1.fq",
 "fastqtobam.fastq2" : "/Bioinformatics-pipeline/demo-dataset/demo-reads/demo.r2.fq",
 "fastqtobam.Ref" : "/Bioinformatics-pipeline/demo-dataset/example_ref/example_genome.fa"
}
```
注意：

由于该wdl 脚本的运行环境为单机，因此使用的软件（如：bwa、samtools）需要预先由用户自行安装。
流程运行于本地环境，因此于集群环境不同，无需设置wdl 语法中task 的 runtime
其中 -w 参数指定输出结果目录，如果没有指定，wdl 程序会自动输出至系统 /opt/workflow 下

### 3. 运行作业

```
wdl run demo.fastqtobam.wdl input.json -w /opt/my_output/path
```

### 4. 结果查看
任务完成后，用户可进入目录对结果进行查看
```
foo@local#:cd /output/path 
foo@local#:ls -R ./ 
boltstore.db  fastqtobam-fastqtobam.bwa_mem  fastqtobam-fastqtobam.samtobam

./fastqtobam-fastqtobam.bwa_mem:
stderr  stdout  toy.sam

./fastqtobam-fastqtobam.samtobam:
stderr  stdout  toy.bam
```