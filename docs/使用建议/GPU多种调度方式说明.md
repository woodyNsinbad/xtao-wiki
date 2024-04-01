# GPU的多种调度方式说明
为满足不同的GPU使用场景，Achelous提供了多种不同的GPU资源分配方式，包括按物理GPU卡分配、按GPU显存分配、将GPU卡进行MIG切分后按GPU instance分配。
> 这三种GPU资源的使用方式是彼此互斥的，只能以一种方式来为任务指定GPU资源的申请
{.is-warning}


## 1. 按物理GPU卡分配
这是最简单和最直接的GPU分配方式，任务以独占的方式使用物理GPU设备，一个任务可以分配多个物理GPU设备，在该任务运行期间其他任务没法使用这些GPU。这种方式要求用户提交任务时必须指定整数个GPU卡资源。
**注意**：通过指定GPUs Mode我们也可以将MIG切分后的GPU instance视为GPU卡来分配，目前仅SGE模拟客户端支持。

> example 1：使用SGE模拟客户端提交使用GPU卡的任务
该命令提交一个名为`test-gpu-device`的任务，并请求分配100GiB内存，20cpu，2个gpu卡（将被这个任务独占），并且只使用真实的GPU卡而不会使用MIG切分后的GPU instance
```
qsub -N test-gpu-device -l mem=100G:cpu=20:gpus=2 --gpus_mode=gpus_only /mnt/vo1/test-gpu.sh
```

> example 2：使用SGE模拟客户端提交使用GPU卡的任务
该命令提交一个名为`test-gpu-device`的任务，并请求分配100GiB内存，20cpu，2个gpu卡（将被这个任务独占），优先使用真实的GPU卡，如果真实GPU卡资源不满足则会使用MIG切分后的GPU instance资源
```
qsub -N test-gpu-device -l mem=100G:cpu=20:gpus=2 --gpus_mode=gpus_first /mnt/vo1/test-gpu.sh
```

> example 3: 使用SGE模拟客户端提交使用GPU卡的任务
该命令提交一个名为`test-gpu-device`的任务，并请求分配100GiB内存，20cpu，2个gpu卡（将被这个任务独占），优先使用MIG切分后的GPU instance，如果GPU instance资源不满足则会使用真实的GPU卡资源
```
qsub -N test-gpu-device -l mem=100G:cpu=20:gpus=2 --gpus_mode=gpui_first /mnt/vo1/test-gpu.sh
```

> example 4: 使用bioflow提交使用GPU卡的WDL任务
我们在task的runtime中请求分配5cpu，50GiB内存，2个gpu卡（将被这个任务独占），并指定使用的容器镜像，然后通过biocli客户端提交让任务到bioflow
```
  task gpu_test {
    input { 
      ... 
    }
    
    command <<<
      ... 
    >>>
    
    output {
      ...
    }
    
    runtime {
      docker: "image name",
      cpu: 5,
      memory: "50G",
      gpu: 2,
    }
  }
```
 
```
biocli job submit gpu-test-workflow.json
```

## 2. 按显存分配
当任务独占GPU设备却又无法充分利用显存时，可以将一个GPU设备按照显存分配给多个任务，提高GPU设备利用率。
> 注意：需要确保同一个GPU设备上任务的显存使用上限不超过设备的显存最大值
{.is-warning}

> 示例一：使用SGE模拟客户端提交使用显存（GPU Memory）的任务
该命令提交一个名为`test-gpu-mem`的任务，并请求分配100GiB内存，20cpu，2GiB显存
```
qsub -N test-gpu-mem -l mem=100G:cpu=20:gpumem=2G /mnt/vol1/test-gpu.sh
```

> 示例二：使用bioflow提交使用GPU Memory的WDL任务
我们在task的runtime中请求分配5cpu，50GiB内存，2048MiB显存，指定使用的容器镜像，然后通过biocli客户端提交让任务到bioflow
```
  task gpu_test {
    input { 
      ... 
    }
    
    command <<<
      ... 
    >>>
    
    output {
      ...
    }
    
    runtime {
      docker: "image name",
      cpu: 5,
      memory: "50G",
      gpumemory: 2048,
    }
  }
```
 
```
biocli job submit gpu-test-workflow.json
```


## 3. 通过 MIG 指定资源
如果GPU设备支持MIG（Multi-Instance GPU）切分，则可以通过启用该功能将一个物理GPU卡切分为多个独立并隔离的GPU实例，每个实例拥有自己的GPU核心、显存和高速缓存。这样可以更高效的利用GPU设备。
在Achelous平台上，MIG切分后的实例作为`gpui`（gpu instance）来分配给任务使用，但我们也支持将`gpui`视为独立的gpu卡来分配。
> 示例一：使用SGE模拟客户端提交使用GPU instance的任务
该命令提交一个名为`test-gpu-instance`的任务，并请求分配100GiB内存，20cpu，2个GPU instance
```
qsub -N test-gpu-mem -l mem=100G:cpu=20:gpui=2 /mnt/vol1/test-gpu.sh
```

> 示例二：使用bioflow提交使用GPU Memory的WDL任务
我们在task的runtime中请求分配5cpu，50GiB内存，2个GPU instance，指定使用的容器镜像，然后通过biocli客户端提交让任务到bioflow
```
  task gpu_test {
    input { 
      ... 
    }
    
    command <<<
      ... 
    >>>
    
    output {
      ...
    }
    
    runtime {
      docker: "image name",
      cpu: 5,
      memory: "50G",
      gpui: 2,
    }
  }
```
 
```
biocli job submit gpu-test-workflow.json
```