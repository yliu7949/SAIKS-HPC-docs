---
icon: material/checkbox-marked-circle-auto-outline
---

# 作业管理 {#slurm}

## sbatch (提交批处理作业)

`sbatch` 命令用于向 Slurm 作业调度系统提交一个作业脚本，以便在计算节点上执行。用户需将计算任务和资源申请需求编写在脚本文件中，然后通过 `sbatch` 提交。

 **为什么需要通过 `sbatch` 提交作业？**

高性能计算集群是由众多服务器组成的分布式系统。当您登录集群时，您所在的是**登录节点**。登录节点资源有限，仅用于文件管理、代码编辑和作业提交等轻量级任务。**严禁在登录节点上直接运行计算程序**。您必须通过 `sbatch` 等命令向作业调度系统申请计算资源（如 CPU核心、内存、GPU等），由系统将您的任务分配到合适的**计算节点**上运行。

---

### 使用示例

1.  **单节点单核心作业**

    申请一个 CPU 核心来执行任务。

    **job.sh**
    ```bash
    #!/bin/bash
    #SBATCH -J myFirstJob         # 作业名称
    #SBATCH -o job.%j.out         # 作业的標準输出文件
    #SBATCH -p compute            # 提交到名为 compute 的分区
    #SBATCH --qos=normal          # 指定服务质量 (QOS)
    #SBATCH -N 1                  # 申请 1 个节点
    #SBATCH -n 1                  # 申请 1 个核心

    # 打印当前节点的主机名
    hostname
    echo "Job finished!"
    ```

    假如以上脚本保存为 `job.sh`，通过以下命令提交：
    ```shell
    sbatch job.sh
    ```

    **脚本参数解析：**

    *   `-J myFirstJob`: 定义作业在调度系统中的名称为 `myFirstJob`。
    *   `-o job.%j.out`: 脚本的标准输出将被重定向到 `job.%j.out` 文件中，`%j` 会被系统自动替换为作业ID。
    *   `-p compute`: 指定作业提交到名为 `compute` 的分区。您需要根据集群的实际情况选择分区。
    *   `--qos=normal`: 指定作业的服务质量（QOS）。不同的 QOS 对应不同的优先级、资源限制和计费策略。
    *   `-N 1` 或 `--nodes=1`: 申请 1 个计算节点。对于非并行（如MPI）作业，此值通常为 1。
    *   `-n 1` 或 `--ntasks=1`: 申请 1 个任务（通常等同于 1 个 CPU 核心）。

    <br>

    **如何查看可用的分区和QOS？**

    *   **查看可用分区 (Partition):**
        ```shell
        sinfo -o "%P %G %m %c"
        # 或者使用以下命令查看您有权限使用的分区
        sacctmgr show ass user=`whoami` format=part | uniq
        ```
    *   **查看可用服务质量 (QOS):**
        ```shell
        sacctmgr show ass user=`whoami` format=user,part,qos
        ```

2.  **单节点 GPU 作业**

    如果您的程序需要使用 GPU，必须在脚本中明确申请 GPU 资源。

    **gpu_job.sh**
    ```bash
    #!/bin/bash
    #SBATCH -J myGPUJob
    #SBATCH -o gpu_job.%j.out
    #SBATCH -p gpu              # 提交到 GPU 分区
    #SBATCH --qos=gpu_normal
    #SBATCH -N 1
    #SBATCH -n 4                # 申请 4 个 CPU 核心
    #SBATCH --gres=gpu:1        # 申请 1 块 GPU 卡

    # 打印分配到的 GPU 信息
    nvidia-smi
    echo "GPU Job finished!"
    ```

    **关键参数说明：**

    *   `-p gpu`: 指定提交到拥有 GPU 资源的特定分区。
    *   `--gres=gpu:1`: `gres` (Generic RESource) 用于申请通用资源。`gpu:1` 表示申请 1 块 GPU 卡。如果需要 2 块，则使用 `--gres=gpu:2`。

    <br>

    **注意事项**

    *   并非所有节点都配备 GPU。请使用 `sinfo -o "%P %G"` 命令查看哪些分区包含 GPU 资源 (`%G` 列会显示 gpu 类型及数量)。
    *   在某些集群中，GPU 和 CPU 核心是按比例绑定的。例如，一个节点有 4 块 GPU 和 64 个 CPU 核心，那么申请 1 块 GPU 时，您最多可能只能使用 16 个 CPU 核心。超额申请 CPU 核心可能会导致作业排队时间变长或计费增加。

3.  **多节点并行 (MPI) 作业**

    对于需要跨节点通信的 MPI (Message Passing Interface) 程序，您需要申请多个节点。

    **mpi_job.sh**
    ```bash
    #!/bin/bash
    #SBATCH -J myMPIJob
    #SBATCH -o mpi_job.%j.out
    #SBATCH -p compute
    #SBATCH --qos=normal
    #SBATCH -N 2                  # 申请 2 个节点
    #SBATCH --ntasks-per-node=32  # 每个节点运行 32 个任务/核心

    # 模块加载 (根据集群环境配置)
    module load intel-mpi
    module load vasp

    # 运行 MPI 程序 (共 2*32=64 个核心)
    mpirun -n 64 vasp_std > result.log
    ```

    **关键参数说明：**

    *   `-N 2`: 申请 2 个独立的计算节点。
    *   `--ntasks-per-node=32`: 指定在每个节点上运行 32 个任务。
    *   `mpirun -n 64`: 启动 MPI 程序，总共使用 64 个进程。这个数字通常应等于 `节点数 × 每节点任务数`。

---

### 常用参数

| 参数 (Parameter) | 简写 | 说明 |
| :--- | :--- | :--- |
| `--job-name=<jobname>` | `-J` | 指定作业的名称。 |
| `--output=<filename>` | `-o` | 指定标准输出文件。`%j` 代表作业ID, `%N` 代表节点名。 |
| `--error=<filename>` | `-e` | 指定标准错误文件。 |
| `--partition=<name>` | `-p` | 提交到指定分区。 |
| `--qos=<name>` | `-q` | 指定服务质量 (QOS)。 |
| `--nodes=<count>` | `-N` | 申请的节点数量。 |
| `--ntasks=<count>` | `-n` | 申请的总任务数（核心数）。 |
| `--ntasks-per-node=<count>` | | 指定每个节点上运行的任务数。 |
| `--cpus-per-task=<count>`| `-c` | 为每个任务分配的 CPU 核心数（用于多线程程序）。 |
| `--gres=<list>` | | 申请通用资源，如 GPU (`--gres=gpu:2`)。 |
| `--time=<time>` | `-t` | 作业运行的最大时间限制，格式为 `D-HH:MM:SS`。 |
| `--chdir=<directory>` | `-D` | 指定作业的工作目录。 |
| `--mail-user=<email>` | | 发送通知邮件的地址。 |
| `--mail-type=<type>` | | 邮件通知类型，如 `BEGIN`, `END`, `FAIL`, `ALL`。 |
| `--account=<account>` | `-A` | 指定计费账户（通常无需设置）。 |
| `--nodelist=<nodes>` | `-w` | 指定在哪些节点上运行。 |
| `--exclude=<nodes>` | `-x` | 指定排除哪些节点。 |

---

### 资源申请技巧

1.  **资源参数的优先级**

    `--nodes` (`-N`) 和 `--ntasks-per-node` 的组合通常**优先级高于** `--ntasks` (`-n`)。

    例如，以下配置：
    ```bash
    #SBATCH -N 2
    #SBATCH --ntasks-per-node=32
    #SBATCH -n 60  # 此行通常会被忽略
    ```
    系统会为您分配 `2 * 32 = 64` 个核心，而不是 60 个。因此，推荐使用 `-N` 和 `--ntasks-per-node` 的组合来精确控制资源。

2.  **核心与内存的关系**

    集群的计算节点有固定的 CPU 核心数和内存总量。您申请的核心数决定了您可以使用的内存量。

    *   使用 `sinfo -o "%P %c %m"` 可以查看各分区每个节点的 CPU 总数和内存总量。
    *   **估算可用内存**：`单个核心可用内存 ≈ 节点总内存 / 节点总核心数`。
    *   如果您的程序因内存不足（如 `OOM killed`）而失败，您可以：
        1.  在同一个节点上申请更多的核心 (`-n` 或 `--ntasks-per-node`)。
        2.  切换到一个内存更大的分区（如果可用）。
  
## salloc (交互式作业)

`salloc` 命令用于实时申请计算资源，并为您创建一个交互式的会话。这允许您直接登录到计算节点上进行调试、软件测试或执行需要实时交互的命令。

**`salloc` 与 `sbatch` 的核心区别**

*   **`sbatch`**：**非交互式**。您将所有指令写入一个脚本，提交后由系统在后台自动执行，您只需等待结果。适用于成熟的、无需干预的计算任务。

*   **`salloc`**：**交互式**。您申请资源后，会得到一个在计算节点上的命令行 Shell。您可以像在本地终端一样逐条输入命令并立即看到反馈。适用于程序调试、环境配置和短时间测试。

---

### 使用示例

1.  **申请单节点多核心**

    假设您需要 6 个 CPU 核心来进行程序调试，最长需要 2 小时。

    ```shell
    # 申请资源，--pty bash 会在分配成功后直接为您启动一个 bash shell
    salloc -p compute -N 1 -n 6 --qos=normal -t 2:00:00 --pty bash

    # 命令执行后，若资源可用，您的命令提示符会发生变化，表明已在计算节点上
    # 例如：[user@compute-node-01 ~]$

    # 现在您可以执行您的调试任务了
    ./my_program

    # 调试完成后，退出 shell 即可释放资源
    exit
    ```

2.  **申请 GPU 资源**

    如果您需要调试一个 GPU 程序，可以申请一块 GPU 卡和相应的 CPU 核心。

    ```shell
    # 申请 1 个节点、4 个核心和 1 块 GPU，最长 24 小时
    salloc -p gpu -N 1 -n 4 --gres=gpu:1 --qos=gpu_normal -t 24:00:00 --pty bash

    # 进入交互式会话后，可以先确认 GPU 是否分配成功
    nvidia-smi

    # 然后运行您的 GPU 程序
    ./my_gpu_app

    # 任务完成，退出会话
    exit
    ```

3.  **申请多节点资源 (用于 MPI)**

    对于需要跨节点调试的 MPI 程序，您可以申请多个节点。

    ```shell
    # 申请 2 个节点，每个节点 12 个核心，共 24 个核心
    salloc -p compute -N 2 --ntasks-per-node=12 --qos=normal -t 2:00:00

    # salloc 成功后会返回一个作业ID，并在当前终端创建一个资源分配环境
    # Your job 123456 has been submitted
    # salloc: Granted job allocation 123456

    # 现在，您可以在这个终端内使用 srun 来启动您的 MPI 程序
    # srun 会自动利用您刚刚申请到的所有资源
    module load intel-mpi
    srun -n 24 hostname    # -n 24 表示使用全部 24 个核心

    # 调试完成后，释放资源
    exit
    ```

---

### 重要注意事项

*   **适用场景**：交互式任务主要用于**程序调试、编译和短时间测试**。对于长时间、稳定的计算任务，请务必使用 `sbatch` 提交作业。
*   **释放资源**：交互式会话结束后，**请务必确认资源已被释放**！最简单的方式是输入 `exit` 或关闭终端。您也可以在另一个终端窗口使用 `squeue -u $USER` 检查您的名下是否还有正在运行的作业。如果会话意外中断，可以使用 `scancel <JOB_ID>` 手动终止作业，避免不必要的资源占用和计费。
*   **参数通用性**：`salloc` 的资源申请参数（如 `-p`, `-N`, `-n`, `--gres` 等）与 `sbatch` **几乎完全相同**，您可以参考 `sbatch` 的说明来申请您需要的各种资源。

## sinfo (查看集群状态)

`sinfo` 命令用于实时查询集群中各个分区（Partition）和节点（Node）的状态信息。

---

### 使用示例

1.  **查看集群整体状态**

    直接运行 `sinfo` 可以快速了解整个集群的概览。

    ```shell
    sinfo
    ```

    输出结果中 `STATE` 列显示了节点的状态，常见的状态有：

    *   `idle`: 节点完全空闲，所有资源可用。

    *   `mix`: 节点部分被占用，仍有空闲核心可供使用。

    *   `alloc` / `allocated`: 节点资源已被完全占用。

    *   `down`: 节点故障或已关闭，不可用。

    *   `drain`: 节点正在排空，不再接受新作业，等待现有作业完成后下线维护。

2.  **查看指定分区的状态**

    如果您只关心特定的几个分区，可以使用 `-p` 参数。

    ```shell
    # 查看单个分区
    sinfo -p gpu

    # 查看多个分区，用逗号隔开
    sinfo -p compute_a,compute_b
    ```

3.  **查看分区详细资源配置**

    列出每个分区的资源情况，如 GPU、CPU 核心数和内存。

    ```shell
    sinfo -o "%P %G %c %m"
    ```

    *   `%P`: 分区名称
    *   `%G`: GPU 类型及数量
    *   `%c`: 每个节点的 CPU 核心数
    *   `%m`: 每个节点的内存大小 (MB)

---

### 常用参数

| 参数 (Parameter) | 简写 | 说明 |
|:---|:---|:---|
| `--help` | | 显示 `sinfo` 命令的帮助信息。 |
| `--Node` | `-N` | 以节点为单位，每行列出一个节点，信息更详细。 |
| `--partition=<name>` | `-p` | 显示指定分区的信息，多个分区用逗号分隔。 |
| `--nodelist=<list>` | `-n` | 显示指定节点的信息，多个节点用逗号分隔。 |
| `--responding` | `-r` | 只显示当前响应正常的节点。 |
| `--dead` | `-d` | 只显示当前没有响应的节点。 |
| `--list-reasons` | `-R` | 显示节点离线或不正常工作的原因。 |
| `--iterate=<secs>` | `-i` | 每隔指定秒数自动刷新并显示信息，按 `Ctrl+C` 退出。 |

---

### 自定义格式化输出

`sinfo` 提供了强大的格式化输出功能，通过 `-o <format_string>` 或 `--format=<format_string>` 参数，您可以精确控制需要显示的内容。

格式化字符串由 `%` 加上特定字符组成，例如 `%.15P` 表示显示分区名，左对齐，宽度为15个字符。

**常用格式符列表：**

| 格式符 | 说明 |
|:---|:---|
| `%P` | 分区名 (默认分区会带 `*` 后缀)。 |
| `%N` | 节点列表。 |
| `%t` | 节点状态 (紧凑形式，如 `mix`, `idle`)。 |
| `%T` | 节点状态 (完整形式，如 `mixed`, `idle`)。 |
| `%c` | 每个节点的 CPU 核心数。 |
| `%C` | 核心数统计 (allocated/idle/other/total)。 |
| `%D` | 分区中的节点总数。 |
| `%m` | 每个节点的内存大小 (MB)。 |
| `%G` | 节点的通用资源信息 (常用于显示 GPU 类型和数量)。 |
| `%l` | 分区允许的最大作业运行时长。 |
| `%a` | 分区是否可用 (up/down)。 |
| `%E` | 节点不可用的原因。 |
| `%O` | 节点的 CPU 负载。 |

**格式化输出示例：**

以下命令将以自定义的格式显示分区名、可用状态、时间限制、节点数、节点状态和节点列表。

```shell
sinfo -o "%.15P %.5a %.10l %.6D %.6t %N"
```

这将生成一个非常清晰、对齐的表格，便于快速掌握集群资源状况。

## squeue (查看作业状态)

`squeue` 命令用于查看作业队列中的作业状态，是监控自己提交任务进展的核心工具。

---

### 使用示例

1.  **查看队列中的所有作业**

    直接运行 `squeue` 会列出当前调度系统中的所有作业，包括正在运行的（Running）和正在排队的（Pending）。

    ```shell
    squeue
    ```

    默认输出包含以下列：

    *   `JOBID`: 唯一的作业 ID。
    *   `PARTITION`: 作业所在的分区。
    *   `NAME`: 作业的名称。
    *   `USER`: 提交该作业的用户名。
    *   `ST`: 作业状态（`R` 表示正在运行, `PD` 表示正在排队）。
    *   `TIME`: 作业已经运行的时间。
    *   `NODES`: 作业占用的节点数。
    *   `NODELIST(REASON)`:
        *   对于 **正在运行** 的作业，显示其所在的节点列表。
        *   对于 **正在排队** 的作业，显示其等待的原因（如 `Resources`, `Priority` 等）。

2.  **只查看自己的作业**

    使用 `-u` 参数可以筛选特定用户的作业。

    ```shell
    # 使用环境变量 $USER 自动填充当前用户名
    squeue -u $USER

    # 或者直接指定用户名
    squeue -u your_username
    ```

3.  **使用别名简化命令**

    `squeue` 的默认输出信息有限。为了更方便地查看详细信息，您可以创建一个 shell 别名（alias）。

    首先，将以下行添加到您主目录下的 `.bashrc` 文件中：

    ```bash
    # ~/.bashrc
    alias sq='squeue -o "%.18i %.9P %.12j %.12u %.8T %.10M %.16l %.6D %R" -u $USER'
    ```

    然后，执行 `source ~/.bashrc` 或重新登录终端使别名生效。之后，您只需输入 `sq` 命令，就能看到格式化后的作业信息。

    ```shell
    sq
    ```

---

### 常用参数

| 参数 (Parameter) | 简写 | 说明 |
|:---|:---|:---|
| `--user=<user_list>` | `-u` | 显示指定用户的作业，多个用户用逗号分隔。 |
| `--jobs=<job_id_list>` | `-j` | 显示指定作业ID的信息，多个ID用逗号分隔。 |
| `--states=<state_list>`| `-t` | 显示处于特定状态的作业，如 `PENDING`, `RUNNING`。 |
| `--partition=<name>` | `-p` | 显示指定分区的作业。 |
| `--nodelist=<list>` | `-w` | 显示在指定节点上运行的作业。 |
| `--account=<list>` | `-A` | 显示指定计费账户下的作业。 |
| `--iterate=<secs>` | `-i` | 每隔指定秒数自动刷新并显示信息，按 `Ctrl+C` 退出。 |
| `--help` | | 显示 `squeue` 命令的帮助信息。 |

---

### 自定义格式化输出

通过 `-o <format_string>` 或 `--format=<format_string>` 参数，您可以完全自定义 `squeue` 的输出内容和格式。

**常用格式符列表：**

| 格式符 | 说明 |
|:---|:---|
| `%i` | 作业 ID (Job ID)。 |
| `%P` | 分区 (Partition)。 |
| `%j` | 作业名称 (Job Name)。 |
| `%u` | 用户名 (User)。 |
| `%T` | 作业状态 (State)，如 `RUNNING`, `PENDING`。 |
| `%M` | 已运行时间 (Time Used)。 |
| `%l` | 作业总运行时限 (Time Limit)。 |
| `%D` | 节点数 (Nodes)。 |
| `%C` | 请求的 CPU 核心数 (CPUs)。 |
| `%q` | 作业的服务质量或优先级 (QOS/Priority)。 |
| `%R` | 节点列表或排队原因 (Node List / Reason)。 |
| `%a` | 计费账户 (Account)。 |

**格式化输出示例：**

下面的命令会以一个清晰、对齐的表格形式显示作业的详细信息。

```shell
squeue -o "%.18i %.9P %.12j %.12u %.8T %.10M %.16l %.6D %R"
```

## 查看作业历史与详情 (`sacct` & `scontrol`)

需要查询作业的详细信息时，可以使用`sacct` 和 `scontrol show job` 两个工具。它们各有侧重：

*   `scontrol show job`: 提供**正在运行**或**刚结束**作业的**实时、详尽**快照。是调试问题的首选。
*   `sacct`: 用于查询**已完成**作业的**历史会计记录**，如资源使用情况、状态码等。是进行回顾性分析的工具。

---

### 实时查看作业详情 (`scontrol show job`)

`scontrol show job <job_id>` 命令可以显示一个作业极其全面的配置和状态信息，但其数据具有时效性，通常只能查询正在运行或在系统缓存中的近期作业。

**使用示例：**
```shell
# 查看作业 ID 为 7454119 的详细信息
scontrol show job 7454119
```

**能从中获取的关键信息包括：**

*   `JobState`: 作业当前状态。
*   `WorkDir`: 作业的工作目录。
*   `SubmitTime`, `StartTime`, `RunTime`: 提交、开始及已运行时间。
*   `NodeList`: 正在使用的节点。
*   `NumCPUs`, `NumNodes`: 申请的 CPU 核心数和节点数。
*   `StdOut`, `StdErr`: 标准输出和标准错误文件的路径。
*   `Command`: 提交的命令或脚本路径。

**注意**
此命令直接从 Slurm 控制器的内存中读取数据，因此无法查询很久以前的历史作业。如果查询失败，请使用 `sacct` 命令。

---

### 查询历史作业记录 (`sacct`)

`sacct` 命令通过查询 Slurm 的会计数据库来获取作业信息，因此它可以检索到所有**已完成**的历史作业记录。

**使用示例：**
```shell
# 查询作业 ID 为 7454119 的历史记录
sacct -j 7454119
```

默认输出会展示作业的主要步骤和信息，包括：

*   `JobID`: 作业ID。
*   `JobName`: 作业名。
*   `Partition`: 分区。
*   `Account`: 计费账户。
*   `AllocCPUS`: 分配的CPU核心数。
*   `State`: 最终状态 (如 `COMPLETED`, `FAILED`, `CANCELLED`)。
*   `ExitCode`: 退出码 ( `0:0` 表示成功, 非零表示有错误)。

**自定义输出格式：**

与 `sinfo` 和 `squeue` 类似，`sacct` 也支持强大的格式化输出，这对于提取特定信息非常有用。

**使用示例：**
```shell
# 定义一个格式化字符串
FORMAT="jobid,jobname,partition,nodelist,alloccpus,state,end"

# 使用该格式查询作业
sacct --format=$FORMAT -j 7454119
```

**常用格式化字段：**

| 字段 (Field) | 说明 |
|:---|:---|
| `JobID` | 作业 ID |
| `JobName` | 作业名称 |
| `Partition` | 分区 |
| `User` | 用户名 |
| `Account` | 计费账户 |
| `AllocCPUs` | 分配的 CPU 核心数 |
| `ReqMem` | 请求的内存量 |
| `MaxRSS` | 作业使用的峰值内存 |
| `State` | 最终状态 |
| `ExitCode` | 退出码 |
| `Submit` | 提交时间 |
| `Start` | 开始时间 |
| `End` | 结束时间 |
| `Elapsed` | 总运行时长 |
| `NodeList` | 运行节点列表 |

## scancel (取消作业)

`scancel` 命令用于取消您已经提交到队列中的作业，无论是正在排队的（Pending）还是正在运行的（Running）。

**重要：** 您只能取消属于您自己账户下的作业。

-----

### 使用示例

1.  **取消单个作业**

    只需提供要取消的作业ID。

    ```shell
    # 取消作业 ID 为 123 的作业
    scancel 123
    ```

2.  **取消自己的所有作业**

    该命令会取消提交的**所有**作业，包括正在运行和排队的。

    > **警告：** 这是一个危险操作，请在执行前务必确认无误！

    ```shell
    # 使用环境变量 $USER 自动填充当前用户名
    scancel -u $USER
    ```

3.  **按状态取消作业**

    此命令会取消所有处于 `PENDING`（排队中）状态的作业，而不会影响正在运行的作业。

    ```shell
    scancel -t PENDING -u $USER
    ```

    同理，也可以使用 `-t RUNNING` 来取消所有正在运行的作业。

-----

### 常用参数

| 参数 (Parameter) | 简写 | 说明 |
|:---|:---|:---|
| `--user=<user_name>` | `-u` | 取消指定用户的作业。 |
| `--state=<state_name>` | `-t` | 取消处于特定状态的作业，如 `PENDING`, `RUNNING`。 |
| `--name=<job_name>` | `-n` | 取消指定作业名称的作业。 |
| `--partition=<name>` | `-p` | 取消提交到指定分区的作业。 |
| `--qos=<qos>` | `-q` | 取消属于指定 QOS 的作业。 |
| `--account=<account>` | `-A` | 取消指定计费账户下的作业。 |
| `--help` | | 显示 `scancel` 命令的帮助信息。 |
