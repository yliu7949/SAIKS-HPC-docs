---
icon: material/checkbox-marked-circle-auto-outline
---

# 作业提交 {#slurm}

## sbatch

将整个计算过程，写到脚本中，通过 sbatch 指令提交到计算节点上执行。

### 使用案例

#### 单节点单核心案例

```bash
#!/bin/bash
#SBATCH -o job.%j.out
#SBATCH -p C032M0128G
#SBATCH --qos=low
#SBATCH -J myFirstJob
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1

hostname
```

提交命令：

```bash
sbatch job.sh
```

#### 单节点 GPU 案例

```bash
#!/bin/bash
#SBATCH -o job.%j.out
#SBATCH --partition=GPU
#SBATCH --qos=low
#SBATCH -J myFirstGPUJob
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=6
#SBATCH --gres=gpu:1

nvidia-smi
```

#### 多节点多核心案例

```bash
#!/bin/bash
#SBATCH -o job.%j.out
#SBATCH --partition=C032M0128G
#SBATCH --qos=low
#SBATCH -J myFirstMPIJob
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=32

module load intel/2017.1
module load vasp/5.4.4-intel-2017.1

mpirun -n 64 vasp_std > log
```

### 其他使用技巧

#### 申请资源量参数

* `-n, --ntasks=<number>`：最多运行几个任务
* `-N, --nodes=<number>`：申请几个节点
* `--ntasks-per-node=<ntasks>`：每个节点运行几个任务
* `-c, --cpus-per-task=<ncpus>`：每个任务使用几个核心，默认每个任务 1 个核心

#### 关于核心与内存

通过以下指令可以查询现有分区，以及每个分区可以申请多少个 CPU 核心和多少内存：

```bash
sinfo -o "%P %m %c"
```

### 申请 GPU 注意事项

申请 GPU 时需要注意，GPU 分区上的 GPU 卡和 CPU 核心数量是绑定的，例如未名二号的 GPU40G 分区，单节点是 4 块 GPU 卡 64 个 CPU 核心，那么申请一个 GPU 卡的同时，最多能申请 16 个 CPU 核心，如果申请 17 个 CPU 核心，会按照两块 GPU 卡进行收费。

## sinfo

`sinfo` 命令用于查询各分区节点的状态。

### 使用示例

#### 查看节点空闲状态

显示集群的所有分区节点的空闲状态：

```bash
sinfo
````

* `idle`：空闲
* `mix`：节点部分核心可以使用
* `alloc`：已被占用

#### 查看指定分区空闲状态

指定显示 `C032M0128G` 和 `C032M0256G` 分区的空闲状态：

```bash
sinfo -p C032M0128G,C032M0256G
```

#### 查看分区的 GPU 卡和 CPU 核心数

查看哪些分区有 GPU 卡、分区中每个节点有多少块卡、多少核心、多少内存：

```bash
sinfo -o "%P %G %c %m"
```

### 常见参数

* `--help`：显示 `sinfo` 命令的使用帮助信息；
* `-d`：查看集群中没有响应的节点；
* `-i <seconds>`：每隔相应的秒数，对输出的分区节点信息进行刷新；
* `-n <name_list>`：显示指定节点的信息，如果指定多个节点的话用逗号隔开；
* `-N`：按每个节点一行的格式来显示信息；
* `-p <partition>`：显示指定分区的信息，如果指定多个分区的话用逗号隔开；
* `-r`：只显示响应的节点；
* `-R`：显示节点不正常工作的原因。

### 按照固定格式输出

* `-o <output_format>`：显示指定的输出信息，指定的方式为 `%[[.]size]type`。

  * `.`：表示右对齐，不加的话表示左对齐；
  * `size`：表示输出项的显示长度；
  * `type`：需要显示的信息。

常见的输出格式类型：

* `%a`：是否可用状态；
* `%A`：以 "allocated/idle" 的格式来显示节点数，不要和 `%t` 或 `%T` 一起使用；
* `%c`：节点的核心数；
* `%C`：`allocated/idle/other/total` 格式显示核心总数；
* `%D`：节点总数；
* `%E`：节点不可用的原因；
* `%m`：每个节点的内存大小（单位为 MB）；
* `%N`：节点名；
* `%O`：CPU 负载；
* `%P`：分区名，作业默认分区带 `*`；
* `%r`：只有 root 可以提交作业（yes/no）；
* `%R`：分区名；
* `%t`：节点状态（紧凑形式）；
* `%T`：节点的状态（扩展形式）。

例如：

```bash
sinfo -o "%.15P %.5a %.10l %.6D %.6t %N"
```
