# 在qsub中使用Singularity镜像资源

Singularity是目前主流的镜像化技术之一，平台支持对Singularity 封装的工具，通过qsub或wdl 流程进行作业投递。

## 1. Singularity准备镜像

一般软件会发布Singularity版本的镜像，其发布形式为文件（simg或sif 后缀）。以`STAR-Fusion` 为例，其Singularity 镜像可以在其下载页面直接获取，链接如下：

https://data.broadinstitute.org/Trinity/CTAT_SINGULARITY/STAR-Fusion/

## 2. 作业脚本编写

`simg` 文件下载后，即可编写对应作业脚本，以下为示例脚本

```shell
STAR-Fusion        \
--left_fq /mnt/anna01/project/2023/zlj_ISMRNA20230301001/result/RNAseq-RNA-seq_v5.0.0-S2-0-RNAseq.sampleJobs-3-sample-S1-0-3-SampleWorkflow.qc-0-QC-I3-2-0-QC.Cutadapt/samples/MW1/lib_lib1--rg_lane1/cutadapt_MW1_S1_L001_R1_001.fastq.gz    \
--right_fq /mnt/anna01/project/2023/zlj_ISMRNA20230301001/result/RNAseq-RNA-seq_v5.0.0-S2-0-RNAseq.sampleJobs-3-sample-S1-0-3-SampleWorkflow.qc-0-QC-I3-2-0-QC.Cutadapt/samples/MW1/lib_lib1--rg_lane1/cutadapt_MW1_S1_L001_R2_001.fastq.gz   \
--genome_lib_dir  /mnt/alamo01/demo/Mouse_GRCm39_M31_CTAT_lib_Nov092022.plug-n-play/ctat_genome_lib_build_dir/  \
--CPU 20    \
-O /mnt/alamo01/demo/StarFusionOut2    \
--FusionInspector validate     \
--examine_coding_effect    \
--denovo_reconstruct
```

## 3. 脚本投递

投递命令行如下：

```bash
qsub --cwd \
--vol /mnt/anna01/ \
--vol /mnt/alamo01/demo \
-l cpu=20:mem=100G \
-si /mnt/alamo01/demo/star-fusion.v1.12.0.simg \
starfusion-singularity.sh
```

其中：
+ -si 为指定Singularity镜像的位置，由于是存储上的文件，因此需要写对应的 `simg` 文件的绝对路径
+ --vol 指定挂载目录，与使用 Docker 镜像类似。用户需确保其使用的数据目录已被挂载
+ -l 为指定计算所需的硬件资源
+ --cwd 为指定工作目录为当前目录

## 4. 结果查看

正常投递后，可通过`qstat -j <job id>` 查看运行状态。

```

======================================================================
job_number:                             415498
job_name:                               demo.paladin-task.QW9SdszH
job_type:                               SINGULARITY
state:                                  finished
state_reason:                           Container exited with status 0
exit_code:                              0
exit_code_msg:                          Container exited with status 0
exec_id:                                paladin-task.8e38e50f-ec59-4f64-96c2-f127bd6f0a32
submitted_at:                           2024-01-11T12:09:21+08:00
starved_at:                             -
executed_at:                            2024-01-11T12:09:22+08:00
terminated_at:                          2024-01-11T12:43:41+08:00
...
======================================================================

```

任务结束后，当前目录下的 `StarFusionOut2` 中生成了对应结果文件。
