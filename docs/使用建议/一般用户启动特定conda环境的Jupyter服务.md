# 用户使用自有conda环境的jupyter服务

目前由于公共服务中jupyter服务对应python版本固定，因此对于用户自主安装的conda环境无法保证完美切换。针对这种情况，用户可在自有conda下，启动jupyter服务，使用其中对应的扩展包。具体方法如下：

> 注意：该方法只针对conda环境中已经安装了`jupyter`的情况。
{.is-warning}


## 1. 编写脚本

在终端切换到conda环境下，编辑一个shell 脚本。

例如将其命名为`jupyter-conda.sh`,内容如下：

```bash
#!/bin/bash
### 注意 5699 端口号仅为示例，请根据个人具体情况进行替换
jupyter-notebook --port 5699 --ip 0.0.0.0 
```

> 由于小编号端口为系统占用，因此请自觉将端口编号设置为5000以上、9000以下
{.is-warning}


## 2. 服务启动提交

qsub 投递该脚本

```bash
qsub -V -l cpu=2:mem=2G jupyter-conda.sh
```

> 注意：用户可根据实际情况对计算资源进行申请，cpu=2:mem=2G 仅为示例
{.is-warning}


投递成功后，返回信息中会有对应链接

```
    To access the notebook, open this file in a browser:
        file:///home/wd/.local/share/jupyter/runtime/nbserver-61-open.html
    Or copy and paste one of these URLs:
        http://biohpc026:5699/?token=6fee8b5860dbf61b9f96e4b4f7a564f96d61ff88acc2c265
     or http://127.0.0.1:5699/?token=6fee8b5860dbf61b9f96e4b4f7a564f96d61ff88acc2c265
```

直接访问对应链接（需要换到可访问的IP）。

> 注意：上述链接需要用户自主转换为可访问的IP地址。例如 http://biohpc026:5699/ 应转化为 http://10.10.4.126:5699/
{.is-warning}

## 3. 终止服务

用户可通过`qdel` 停止服务。例如qsub返回的jupyter服务任务编号为1234，则停止方式为：

```
qdel 1234
```

> 注意：用户资源使用情况会影响账户计费，因此请自觉停止长期不用服务，以防止过度计费。
{.is-warning}

