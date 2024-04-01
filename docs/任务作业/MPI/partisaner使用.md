# Partisaner 使用

## MPI 框架简介绍
MPI (Message Passing Interface) 是经典的分布式并行编程接口。科学计算中常常基于MPI进行大规模的并行计算，突破单机算力极限。生物信息领域的MPI范例包括mpiBlast等，具备很好的加速效果。与qsub投递单个任务不同，Achelous平台的partisaner命令行支持在一组机器上启动相互交互的任务进行计算。用户可以通过partisaner投递分布式MPI、Tensorflow或者PyTorch程序进行科学计算或者分布式机器学习。Partisaner支持MPICH和OpenMPI等多种MPI实现。partisaner实质上替代了以前的mpiexec或者mpirun命令。

## 运行实例：MPI任务

### 1. MPI框架程序编写 
以计算圆周率分布式算法的程序的为例，展示如何运行MPI 框架作业。下面是程序代码。

```c
#include<stdio.h>
#include<mpi.h>
#include<math.h>
#include<stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]){
    int my_rank, num_procs;
    int i, n = 0;
    double sum, width, local, mypi, pi;
    double start = 0.0, stop = 0.0;
    int proc_len;
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD,  &num_procs);
    MPI_Comm_rank(MPI_COMM_WORLD,  &my_rank);
    MPI_Get_processor_name(processor_name,  &proc_len);
    printf("Process %d of %d\n", my_rank, num_procs);
    if(my_rank == 0){
        start = MPI_Wtime();
        n = atoi(argv[2]);
        printf("Process will compute PI in %d steps\n", n);
    }
    //  printf("Process %d of %d\n", my_rank, num_procs);
    MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);
    width = 1.0 / n;
    sum = 0.0;
    for(i = my_rank; i < n; i += num_procs){
        local = width * ((double)i + 0.5);
        sum += 4.0 / (1.0 + local * local);
    }
    mypi = width * sum;
    MPI_Reduce(&mypi,  &pi,  1,  MPI_DOUBLE,  MPI_SUM,  0,
               MPI_COMM_WORLD);
    sleep(60);
    if(my_rank == 0){
        printf("PI is %.20f\n", pi);
        stop = MPI_Wtime();
        printf("Time: %f on %s\n", stop-start, processor_name);
        fflush(stdout);
        FILE *f = fopen(argv[1], "w+");
        if(f != NULL) {
            printf("Write the step %d to %s\n", n, argv[1]);
            fprintf(f, "%d", n);
        }
        fclose(f);
    }
    MPI_Finalize();
    return 0;
}
```

此代码的功能为通过并行计算圆周率。对其编译后生成可执行文件 calculator-PI,并将其分装进镜像 partisaner_demo_mpich:latest 中。

### 2. 编写配置文件

配置文件以JSON格式编写，其中记录了任务相关信息，例如以下例子。
```json
{
        "Procs": 3,
        "Cpu": 1,
        "Mem": 1000,
        "WorkDir": "/tmp/",
        "Cmd": "/home/mpi/mpich-install/run/calculator-PI steps 50000",
        "Image": "partisaner_demo_mpich"
}
```

编写后保存为 demo-mpi-demo.json 后退出
### 3. 任务提交
任务提交采用Achelous 中 partisaner进行
```
mkdir my-mpi-demo-job 
cd my-mpi-demo-job 
partisaner -c demo-mpi-demo.json -t mpich
```
任务提交后，partisaner 会在任务提交目录(上述例子中为 my-mpi-demo-job )下，生成任务对应的数据库文件partisaner_db。

### 4. 结果查看
上述任务提交后，会提交目录下生成对应的数据库文件、标准输入、输出等信息。如下所示：
```
ls -al ./

drwxr-xr-x 2 root root  4096 May 27  2021 .
drwxr-xr-x 4 root root  4096 Sep 16  2020 ..
-rw-r--r-- 1 root root   162 May 25 10:59 cluster.json
-rw------- 1 root root 65536 May 27  2021 partisaner_db
-rw-r--r-- 1 root root     1 May 27  2021 .pid
-rw-r--r-- 1 root root     0 May 27  2021 stderr
-rw-r--r-- 1 root root     0 May 27  2021 stdin
-rw-r--r-- 1 root root   293 May 27  2021 stdout
```

由于上述程序的输出为标准输出，所以结果内容会输出至 stdout文件中。其内容如下：
```
cat stdout

mpiexec -n 3 -hosts Cc8Apc,Cc6Apc,Cc2Apc -launcher manual -wdir /tmp/ /home/mpi/mpich-install/run/calculator-PI steps 50000
Process 1 of 3
Process 2 of 3
Process 0 of 3
Process will compute PI in 50000 steps
PI is 3.14159265362312822845
Time: 60.008487 on Cc8Apc
Write the step 50000 to steps
```