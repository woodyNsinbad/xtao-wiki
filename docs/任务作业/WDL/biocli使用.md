# biocli命令使用
Bioflow是流程化运行WDL脚本的服务，它具备很好的扩展性、性能和容错能力，是建议的生产化运行方式。biocli是它的命令行，用来管理流程和作业。

## 1. 流程管理

###  流程管理相关基本概念 
+ **用户**: Achelous 对流程的管理是基于用户的，在系统中，不同的用户之间的流程相对封闭。对于某流程而言，其所有者(owner) 是唯一的。只有流程所有者，才有权对流程进行修改或删除。
+ **用户组**： 与Linux 中组概念类似，同一组内的不同用户之间，可以复制组内其他成员的流程，复制后的流程可以进行修改和删除等操作。
+ **Pipeline-Name**：对于特定流程，流程名称是其唯一标识，用户可以通过使用流程名称，进行流程的修改操作。

###  添加 Pipeline

生物信息分析人员可以通过以下命令实现分析流程的添加：
```
biocli pipeline add [pipeline.json] -d [wdl-scripts-dir]
```
其中 pipeline.json 为流程信息文件，其基本格式为：
```json
{
    "Name":"Pipeline-Name", 
    "Type" :"WDL",
    "Description" : "Simple pipeline for Bioinformation analysis",
    "wdl" : {
    "WorkflowFile" : "main.wdl"
    }
}
```

其中-d 选项为需要添加的wdl脚本所在目录

> 注意：-d 所指示的路径应为相对路径，其下包含所有的WDL脚本文件，没有其它文件或者数据
{.is-warning}



### 列举可用的Pipeline
用户可通过以下命令获得系统中存在的pipeline 列表：
```
biocli pipeline list
```
例如：
```
xtao@biohpc003:~$ biocli pipeline list
Got 5 Pipelines:

Pipeline SIMPLE-PIPELINE1111:
   Type: WDL
   Items: 0
   State: InUse
   WorkDir: vol1@anna:bioflow-test/bioflow/wdl
   IgnoreDir:
   Parent: SIMPLE-PIPELINE1111
   LastVersion: 4
   Version: 5
   Description: A pipeline for simple calling
   Owner: root
   WorkflowFile: simple-workflow.wdl

Pipeline SIMPLE-PIPELINE1111-XTAO:
   Type: WDL
   Items: 0
   State: InUse
   WorkDir: anna01@anna01:bioflow-test/bioflow/wdl
   IgnoreDir:
   Parent: SIMPLE-PIPELINE1111-XTAO
   LastVersion: 5
   Version: 6
   Description: A pipeline for simple calling
   Owner: xtao
   WorkflowFile: simple-workflow.wdl

Pipeline SIMPLE-PIPELINE1111-XTAO1:
   Type: WDL
   Items: 0
   State: InUse
   WorkDir: anna01@anna01:bioflow-test/bioflow/wdl
   IgnoreDir:
   Parent:
   LastVersion:
   Version: 1
   Description: A pipeline for simple calling
   Owner: xtao
   WorkflowFile: simple-workflow.wdl

...

xtao@biohpc003:~$
```

其中显示了系统中所有的Piepeline，每个Piepeline包含如下重要信息：
- Owner： 谁创建了这个Piepeline
- Version： 这个Pipeline的版本号


###  复制其他用户Pipeline
对于不同用户的分析流程，可以通过以下命令实现

```
biocli pipeline clone [Original-Pipeline-Name] [Destination-Pipeline-Name]
```
其中重要的参数为：
- Original-Pipeline-Name 为需要被复制的流程名称；
- Destination-Pipeline-Name 为复制后流程的名称

> 注意：如果用户需要修改某一个Owner不是自己的流程，需要进行clone 操作
{.is-info}


### 删除 Pipeline
用户可以通过以下命令，实现删除已经存在的Pipeline

```
biocli pipeline delete [PIPELINE_NAME]
```

例如
```
xtao@biohpc003:~$ biocli pipeline delete SIMPLE-PIPELINE1111-XTAO
Succeed to delete pipeline SIMPLE-PIPELINE1111-XTAO
xtao@biohpc003:~$

```

> 注意：该操作只对Owner为目前用户的流程起效，非流程Owner无权删除该流程
{.is-info}


###  更新 Pipeline
对于已经存在的Pipeline 用户可以通过以下命令，对流程进行修改

```
biocli pipeline update [pipeline.json] -d [wdl-scripts-dir]
```
其中参数为：
- pipeline.json 为记录Pipeline 信息的json 文件；
- wdl-scripts-dir 为存放wdl 脚本的目录相对路径

> 注意：只有owner才能修改Pipeline，其它人只能clone然后修改clone过的Pipeline
{.is-info}


## 2 作业管理

###  作业管理的基本概念
+ job-id ： 对于多用户环境下，Achelous 会为每个投递的任务分配一个ID，其构成为32位字符。此id为作业的唯一标识。用户对于作业的状态查询，暂停，修改权限等操作，均需要以job-id为依据。注意：截取job-id的一部分，如果其唯一，则也可以进行相应的作用

+ job-name : 用户可为自己提交的作业任意命名，作业名称不具有查询效果

+ task-id : 由于job 中存在多个task 。因此，Achelous 中每个task 也有相应的ID 。但task-id 一般不能直接使用，需要通过结合job-id 进行查询。

+ 作业优先级 ： 作业优先级与Linux 中进程优先级类似。其取值为 0 - 10 ，数值越大，对应的优先级越高。在计算资源有限的情况下，优先级高的任务会先被执行。

###  提交作业
用户可通过下面的命令进行作业提交。
```
biocli job submit [job.json]
```
其中

job.json 为作业文件，其中记录了作业相关信息，包括作业名称、作业使用的Pipeline 、作业描述、作业相关输入及参数、优先级等信息，其格式如下：
```json
{
    "Name" : "My_first_WDL_job",
    "Pipeline" : "demo_pipeline",
    "InputDataSet" : {
        "WorkflowInput" : {
                "fastqtobam.fastq1" : "",
                "fastqtobam.fastq2" : "",
                "fastqtobam.ref" : ""
        }
    },
    "Priority" : 7
}
```
作业正确提交后会返回作业的job-id，用于后续对此作业的操作。例如：
```
[xtao01@Cc1Xtcls]$ biocli job submit  my.demo.job.json
The job added success, job ID is: d0a35ca2-192a-4b31-7082-9df4bd49cf8c
```
结果中 d0a35ca2-192a-4b31-7082-9df4bd49cf8c 为该任务的 job-id

###  查看作业状态
用户可通过以下命令对已经成功提交的作业进行状态查看。
```
biocli job status [job-id] [-d]
```
其中：

job-id 为提交作业的作业id
-d 为开关选项，添加 -d 可列出任务详细信息。
例如：

```
[xtao01@Cc1Xtcls]$ biocli job status d0a35ca2-192a-4b31-7082-9df4bd49cf8c -d

Status of Job d0a35ca2-192a-4b31-7082-9df4bd49cf8c:
 Name: MY_DEMO_JOB
 Pipeline: FILTER
 State: RUNNING
 Owner: xtao01
 WorkDir: vol1@xtao:/Demo_data/batch-runs/job-FILTER-d0a35ca2-192a-4b31-7082-9df4bd49cf8c
 PausedState: N/A
 Created: 2021-05-20T12:14:25Z
 Finished: N/A
 RetryLimit: 3
 RunCount: 1
 UserStageCount: 0
 StageQuota: -1
 Priority: 7
 FailReason:
 GraphBuildStatus: Completed
 DoneStages: No stage done
 RunningStages: 23
```

###  取消作业
用户可以通过以下命令，取消作业。

```
biocli job cancel [job-id]
```
其中：
- job-id： 需要取消的作业id
例如：
```
[xtao01@Cc1Xtcls]$ biocli job cancel d0a35ca2 ## 截取部分job-id 也可以作为作业唯一标识
Successfully cancel job d0a35ca2-192a-4b31-7082-9df4bd49cf8c
```

### 列举所有作业
用户通过biocli job list可以查看用户当前正在运行的作业，通过biocli job list -o可以查看用户历史作业。例如：
```
xtao@biohpc003:~$
xtao@biohpc003:~$ biocli job list
No jobs found
xtao@biohpc003:~$
xtao@biohpc003:~$
xtao@biohpc003:~$ biocli job list -o
2 history jobs:
 Job 1:
    ID: Q0-9537a16b-6906-4e16-753f-881aa3098498
    Name: simple-job
    Pipeline: SIMPLE-PIPELINE1111-XTAO (Version: 6)
    Created: 2022-12-21T14:28:14Z
    Finished: 2022-12-21T14:31:16Z
    State: FINISHED
    Owner: xtao
    Priority: 9

 Job 2:
    ID: Q0-7c6f845b-3abb-48f1-684b-e0e1a3617724
    Name: simple-job
    Pipeline: SIMPLE-PIPELINE1111-XTAO1 (Version: 1)
    Created: 2022-12-21T19:41:43Z
    Finished: 2022-12-21T19:43:55Z
    State: FINISHED
    Owner: xtao
    Priority: 9

xtao@biohpc003:~$

```

> 注意：biocli job list只能看到属于该用户的作业，root用户可以看到所有作业。
{.is-info}


### 暂停作业
用户可以通过以下命令暂停作业。
```
biocli job pause [job-id]
```
其中：
- job-id： 需要暂停的作业的job-id

暂停后的作业并没有完全结束,如果作业中存在正在运行的任务（task），那么这些任务并不会停止，而是继续执行，但作业中未被投递的任务，将暂时不再投递。

例如：
```
[xtao01@Cc1Xtcls]$ biocli job pause b5d0d361
Successfully pause job b5d0d361-04e2-4ecf-6274-594f7e9052c5
```

### 恢复暂停状态中的作业
用户可以通过以下命令恢复暂停状态的作业。
```
biocli job resume [job-id]
```
其中job-id为需要恢复的作业的ID。例如：

```
[uec@Cc1Xtcls vcf-filter]$ biocli job resume b5d0d361
Successfully resume job b5d0d361-04e2-4ecf-6274-594f7e9052c5
```

### 恢复失败状态的作业
如果作业运行失败，显示状态为”PSEDONE“。用户找到错误，修改Pipeline之后，可以恢复失败的作业。命令如下：
```
biocli job recover -i [job-id]
```

> 注意：recover作业不会从头开始执行，会从上一次失败的位置继续执行。
{.is-info}


### 重新设定优先级
用户可以通过以下命令调整作业的优先级
```
biocli job setpri [job-id] [priority]
```
其中：
- job-id： 需要调整优先级的作业的ID
- priority： 为优先级数值，取值范围为 0 到 10之间的整数。数值越高，优先级越高

例如：
```
[xtao01@Cc1Xtcls]$ biocli job setpri b5d0d361  8
Successfully update job b5d0d361-04e2-4ecf-6274-594f7e9052c5 priority to 8
```