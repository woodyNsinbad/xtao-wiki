# Conda 使用说明
Anaconda是一个用于科学计算的Python发行版，支持 Linux、Mac、 Windows系统以及 Python、R等科学计算语言，提供了包（Package）管理与环境（Environment）管理的功能，可以很方便地解决多版本多环境并存的问题。用户可以为某项具体的任务创建单独的环境，环境之间相互隔离。这样可以避免同一环境中各类软件相互冲突的问题。Anaconda 利用conda命令来进行包和环境的管理，并且已经包含了Python和相关的配套工具。

> conda 可以支持多虚拟环境，用户在使用Anaconda 或 docker 来管理和使用各类应用时，复杂流程在本集群建议优先考虑docker与WDL或CWL结合使用。如同时使用conda管理R和python相关模块，一定要区分安装路径，使用时加载正确的路径，才能在集群正常使用。
{.is-warning}

## conda 安装
### 1. 集群已经安装最新版本conda，conda 共享安装路径：
```
/mnt/anna01/software/share/conda3/bin/conda
```
可以通过配置环境变量使用，即在.bahsrc中添加路径即可。
```
vim ~/.bashrc
```
编辑路径，在脚本末尾添加：
```
export PATH=/mnt/anna01/software/share/conda3/bin:$PATH
```
然后运行命令:
```
source ~/.bashrc
```
使配置的环境变量生效。
### 2. 用户也可以自行安装conda在自己的home目录。
> 如果在使用conda时，遇到没有权限写入等错误，则需要在自己路径下安装Miniconda。
{.is-warning}


Miniconda路径：
```
/mnt/anna01/app/anaconda3/Miniconda3-latest-Linux-x86_64.sh
```
将该安装包复制到自己路径下，然后输入如下命令进行安装，安装完成后即可使用。
```
cp /mnt/anna01/app/anaconda3/Miniconda3-latest-Linux-x86_64.sh . 
```
```
./Miniconda3-latest-Linux-x86_64.sh
```
安装完成后删除安装包。

```
rm Miniconda3-latest-Linux-x86_64.sh
```
## conda 激活
```
/mnt/anna01/software/share/conda3/bin/conda init
```

## conda 取消自动激活 conda base 默认环境
```
conda config --set auto_activate_base False
```
## 如何添加conda源
Anaconda默认的软件源在国外，速度比较慢，可以将其更换为阿里云或清华源，详见[阿里云](https://developer.aliyun.com/mirror/anaconda/)或[清华大学](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)开源软件镜像站的说明。

- 添加阿里云国内源：
```
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/main/
conda config --add channels https://mirrors.aliyun.com/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.aliyun.com/anaconda/cloud/bioconda/

```

## 创建conda虚拟新环境
```
conda create --name <env_name> <package_names>
```
<env_name>即创建的环境名。建议以英文命名，且不加空格，名称两边不加尖括号“<>”。

<package_names>即安装在环境中的包名。名称两边不加尖括号“<>”。如果要在新创建的环境中创建多个包，则直接在<package_names>后以空格隔开，添加多个包名即可。例如，创建一个名为py311的环境，环境中安装版本为3.11的python，同时也安装了numpy和pandas：
```
conda create --name py3.11 python=3.11 numpy pandas
```
新的环境以及环境内的包会被安装到/home/username/.conda/envs/目录下。

## 激活或切换 mytestenv conda 虚拟环境
```
conda activate mytestenv
```

## 取消或退出 mytestenv conda 虚拟环境
```
conda deactivate
```

## 使用conda安装软件包到个人存储目录
创建并**激活**自己的conda 虚拟环境后，可直接使用conda install 安装相关软件。
使用命令如下，例如：
1.  将包scipy安装到当前活跃环境中
```
conda install scipy
```

2.  将多个软件包安装到当前活跃环境中
```
conda install -n myenv curl wheel scipy
```

3.  将特定版本的python安装到某个环境
```
conda install -p /path/to/myenv  python=3.7.13
```


### 在指定环境中安装包
```
conda install --name <env_name> <package_name>
```

> 注意：
> 
> <env_name> 即将包安装的指定环境名。环境名两边不加尖括号“<>”。
> 
> <package_name> 即要安装的包名。包名两边不加尖括号“<>”。
> 
> 不加-name <env_name>，则安装到当前所在的环境。
{.is-info}

## 复制环境
```
conda create --name <new_env_name> --clone <old_env_name>
```
<new_env_name>为复制的新环境名称，<new_env_name>为原有的环境名称。环境名两边不加尖括号“<>”。 由于conda不支持重命名环境，如果要重命名，可以通过先复制一个新环境，再删除原来环境。

## 显示环境
```
conda info --envs
```

## 删除环境
```
conda remove --name <env_name> --all
```
注意：<env_name>为被删除环境的名称。环境名两边不加尖括号“<>”。

## 包管理
获取当前环境中已安装的包信息
```
conda list
```



## 卸载包
```
conda remove -n <env_name> <package_name>
```
## pip
相比Anaconda，pip可以安装的包更多。用户可以先切换到所需环境，再在环境中执行
```
pip install <package_name>
```



## 如何在qsub中使用conda环境

> 1. 一种conda 环境不需要conda run，只需在登陆节点先激活conda 环境，例如： conda activate xxx
> 2. 然后qsub 加上-V 参数即可。

> 3. 多种conda 环境，需要conda run。 
{.is-success}


例如编写下面的脚本demo.sh:
```
#!/bin/bash

bwa

```
然后通过qsub投递这个脚本：
```
qsub -l cpu=10:mem=12G --cwd -V demo.sh
```

### 如何在qsub中切换不同的conda环境
在写pipeline时可能需要使用多个版本的python，则需要建不同的环境，在qsub中可通过conda run 执行。

```
#!/bin/bash

which conda
echo $PATH
conda run -n test bwa 
## print the python version
conda run -n test3.7 python -V 

conda run -n test3.11 python -V

```
然后通过qsub投递这个脚本：
```
qsub -l cpu=10:mem=12G --cwd -V demo.sh
```