## 在SGE模拟中使用PBS脚本

> SGE模拟支持在PBS脚本中以`#PBS`或`#$`指定qsub命令的部分参数。支持的参数中分为有效参数和无效参数。
当指定有效参数时，效果和在qsub命令行指定该参数效果一致，而指定无效参数时，脚本可以正常解析但该参数没有任何效果。
指定不支持的参数时，在解析脚本时会返回错误。
{.is-info}

### 支持的有效参数列表
- `-l`:指定job申请的资源
    - example: `#PBS -l mem=1G:s_vmem=2G:h_vmem=2G:cpu=1:gpus=1(gpui=1/gpumem=2G):h=host1`
- `-q`:指定job绑定的队列（调度域）
    - example: `#$ -q test_queue`
- `-o`:指定job的stdout输出文件的绝对路径
    - example: `#PBS -o /mnt/test/stdout`
- `-e`:指定job的stderr输出文件的绝对路径
    - example: `#$ -e /mnt/test/stderr`
- `-j`:指定是否合并stderr到stdout文件中
    - example: `#PBS -j yes` 或 `#$ -j no`
- `-p`:指定job的优先级
    - example: `#PBS -p 3`
- `-v`:为job指定环境变量
    - example: `#$ -v env1=val1,env2=val2,env3=val3`
- `-N`:为job指定名字
    - example: `#PBS -N my_first_job`
- `-S`:为job指定shell解释器
    - example: `#$ -S /bin/zsh`
- `-V`:将当前环境的所有环境变量都添加到job的环境变量中
    - example: `#PBS -V`

### 支持的无效参数列表
- `-a`
- `-c`
- `-C`
- `-I`
- `-k`
- `-m`
- `-M`
- `-r`
- `-u`
- `-W`
- `-z`
- `-pe`
- `-cwd`