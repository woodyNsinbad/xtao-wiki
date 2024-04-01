
# WDL语言介绍-2

本篇主要介绍在实际开发流程时，开发人员可能遇到的问题和对应解决建议。建议初学者首先阅读`WDL语言介绍-1`后，再进一步阅读本文档。

---

## 1. WDL 编写环境构建

目前WDL作为主流的流程开发语言，已有不少编程界面支持其语法高亮等功能，流程开发人员可根据实际情况加以选择。

### 1.1 vim环境
vim的WDL插件，项目地址 https://github.com/broadinstitute/vim-wdl 

![vim-ide.png](/pic/wdl/vim-ide.png)

### 1.2 VScode环境

可直接搜索WDL后安装对应模块即可
![vscode-ide.png](/pic/wdl/vscode-ide.png)

### 1.3 语法校验工具

用户在非集群环境，可通过WOMtool进行wdl脚本语法校验，流程图示生成等操作。具体内容可参考[WOMtool介绍页面](https://cromwell.readthedocs.io/en/stable/WOMtool/)

> 该部分功能已在`biocli`或`wdlrun`中实现，上述内容仅为在非集群环境上调试脚本使用。
{.is-warning}


## 2. 流程脚本编排

与其他编程语言类似，wdl脚本内，可以引用其他wdl脚本中的内容，包括：

+ 数据类型：如定义的struct 数据结构
+ task：可以重复使用已有的task，避免重复编写
+ workflow ： 支持用户以引用子流程（subworkflow）的形式将其他流程引入程序中 

### 2.1 脚本的引用

wdl脚本支持`import`和`as` 关键字实现脚本的引用。`import + "./other_wdl_script.wdl"` 的形式即可引入其他脚本的内容，而`as`则定义了在脚本中引用外部脚本的对象名称。可通过`.`操作对其中内容进行定义。


```
import "./data_struct.wdl"
import "./rna-seq-fastq-qc.wdl" as fastq_qc
import "./align_assemble.wdl" as align
import "./rna-seq-bam-qc.wdl" as bam_qc

workflow base_rna_seq {
	Array[Sample_Reads] raw_samples
	Ref_Files ref

  scatter (sample in raw_samples) {
      call fastq_qc.fastqqc { 
                                input:
                                        read1 = sample.read1,
                                        read2 = sample.read2,
                                        prefix = sample.prefix
         }
  }
        Array[Sample_Reads] sample_clean_reads = fastqqc.clean_reads

...
}

```

上述脚本中，前四行均为引入外部脚本。

> 注意，由于平台内流程管理模式，请用户在引用脚本时，尽量将被引用的脚本或脚本所在目录，和主脚本放置于同一个目录下。路径写法为相对路径。目前`biocli`和`wdlrun`暂不支持http协议的脚本引用。
{.is-success}


### 2.2 子流程

WDL支持对子流程的引用，因此在代码编排过程中。引用的方式，与引用task类似。但需要注意的是，一个wdl文件里最多只能有一个workflow模块。因此在开发前，最好对流程的分配进行合理规划，并且对于一个流程的不同部分可以实现多人协同开发。

![sub-workflow1.png](/pic/wdl/sub-workflow1.png)

![sub-workflow1.png](/pic/wdl/sub-workflow2.png)

## 3. task资源优化

由于WDL中对每个task都有资源要求限制，因此用户可能会面临通用流程资源使用不均衡的情况出现。以下是针对资源优化的一些建议：

### 3.1 通过平台资源监控查看资源分配是否合理

对于部分运行行为不太清楚的程序，建议用户可以先通过sge模拟功能进行一定的测试后，通过平台资源监控。对运行的命令执行情况有大致了解后，进行资源的优化。

![资源监控示例图.png](/pic/wdl/资源监控示例图.png)

### 3.2 通过WDL内置函数，合理分配资源

据作者经验，部分程序由于代码优化做的比较出色，因此在内存使用上可以相对宽松，比如常用的比对软件`bwa` \ `bowtie` \ `samtools`的部分功能。计算资源（cpu、内存）不充足只会造成运行速度较慢。但部分软件，代码优化不充分，因此经常因为读入较大文件而造成内存使用超出上限。

WDL语言的内置函数`size()`可用于动态调节task占用资源的大小。

```
task bam_sort {
    File bam
    Int MEMORY = ceil(size(bam,"MB"))
  command <<<
    samtools sort ~{bam} ...
  >>>
  runtime {
    memory: ~{MEMORY}
  }
}
```


