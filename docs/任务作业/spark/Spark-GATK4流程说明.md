# GATK-Spark 使用说明

`Spark`是GATK4使用的一款用于进行多线程处理的软件，多线程是一种并行化形式，可以使计算机（或计算机集群）更快地完成任务。`Spark`软件库是一款开源框架，它在计算机行业中被广泛使用，是加速分析流程执行的最有前途的技术之一。本说明旨在介绍基础的`Spark` 程序使用基础情况，具体分析任务，可联系极道方面人员进行支持。

---

## 1. 镜像准备

流程中涉及的生信软件均以Docker 镜像形式进行部署，具体情况如下：

|镜像名称|说明|
|:-|:-|
|demo/xt-gatk4:4.beta.4|该镜像主要封装了GATK4的jar包及其他组件，由gatk官方提供|
|demo/datamover:latest|datamover是极道提供的一个数据拷贝工具。可以实现快速的将本地文件系统数据传输到hdfs上，尤其是对目录下存在大量小文件的情况，尤其适用。|
|demo/hdfstool|该镜提供了`hdfs`命令的镜像封装。|

--- 

## 2. `hdfs`存储系统说明

程序中，参考文件需要上传至hdfs存储上，目前集群上已经部署了hdfs测试环境。用户可通过centos沙箱中的 `hdfs` 命令进行文件操作，具体示例如下：

```bash

## hdfs 文件系统查看
(base) [demo@biohpc001 ~]$ hdfs ls hdfs://biohpc001:9000/

## hdfs 文件系统内新建目录
(base) [demo@biohpc001 ~]$ hdfs mkdir -p hdfs://biohpc001:9000/reference/hg38/

## 数据上传
(base) [demo@biohpc001 ~]$ hdfs put reference/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz hdfs://biohpc001:9000/reference/hg38

```

--- 

## 3. 流程说明

示例流程采用wdl进行编写，不同于以往`Spark`框架的流程，用户可以直接在同一个流程中实现一般计算和Spark计算的混合编程。

--- 

## 4. 作业投递及结果查看

用户可通过`biocli`进行作业投递。需要编写job 文件，可参考下述例子：
```json
{
    "Name" : "gatk4-beta-wdl-job",
    "Pipeline" : "gatk4-beta-call-pipeline",
    "InputDataSet" : {
        "WorkflowInput" : {
            "Gatk4VarationCall.readPairs" : [
                   ["/mnt/alamo01/Biodata/data/project_data/ID/001/201710008347_L1_1_clean.fq.gz",
                    "/mnt/alamo01/Biodata/data/project_data/ID/001/201710008347_L1_2_clean.fq.gz"]],
            "Gatk4VarationCall.reference" : "hdfs://biohpc001:9000/reference/hg38/Homo_sapiens_assembly38.2bit",
            "Gatk4VarationCall.faReference" : "/mnt/anna01/reference/human/hg38/gatk/reference/Homo_sapiens_assembly38.fasta",
            "Gatk4VarationCall.bwaIndexFile" : "/mnt/anna01/reference/human/hg38/gatk/reference/Homo_sapiens_assembly38.fasta.bwamemindexImage",
            "Gatk4VarationCall.siteFile1" : "hdfs://biohpc001:9000/reference/hg38/Homo_sapiens_assembly38.dbsnp138.vcf.gz",
            "Gatk4VarationCall.siteFile2" : "hdfs://biohpc001:9000/reference/hg38/1000G_phase1.snps.high_confidence.hg38.vcf.gz",
            "Gatk4VarationCall.siteFile3" : "hdfs://biohpc001:9000/reference/hg38/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz",
            "Gatk4VarationCall.sample" : "demo_sample1",
            "Gatk4VarationCall.SparkExecutorURI" : "hdfs://biohpc001:9000/xtao-internal/spark/spark-2.2.0-bin-hadoop2.7.tgz",
            "Gatk4VarationCall.hdfsWorkDir" : "hdfs://biohpc001:9000/jobs/gatk4"
        }
    },
    "ScheduleDomains" : "all",
    "Priority" : 10
}

```

上述文件中：
+ `Gatk4VarationCall.readPairs` 为输入fastq文件
+ `Gatk4VarationCall.sample` 为样本名称。用户可以根据不同人物进行替换。
+ `ScheduleDomains` 项为调度域，请根据用户自身情况进行相应设置。

其余参数均为人类参考基因组（hg38版本）的固定参数。上述`json`文件内容保存为 job-gatk4-beta-call.json 后，即可进行任务投递。

```console 
(base) [demo@biohpc001 ~]$  biocli job submit job-gatk4-beta-call.json
```