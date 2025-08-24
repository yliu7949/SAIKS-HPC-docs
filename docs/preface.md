---
icon: material/pac-man
---

# 前言 {#preface}

## 目标读者

本教程面向计算集群的新用户提供快速入门指导，涵盖从账号申请到作业提交的核心流程。**如果您已有集群使用经验，可以跳过基础章节，直接查阅[代码编译](./guideline/compile.md)和[作业提交](./guideline/slurm.md)等高级操作部分**。  

## 阅读指南

为便于查阅，本手册采用以下排版规范：

- **环境变量**：`PATH`  
- **命令操作**：`sinfo -al`  
- **配置文件内容**：  

    ```bash  
    # ~/.bashrc 示例  
    module load CMake/4.1.0  
    export OMP_NUM_THREADS=4
    ```

- **命令输出或普通文件内容**：

    ```shell
    PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
    A800*        up 3-00:00:00      5   idle gpu[1-5]
    RTX4090-7    up 3-00:00:00      1   idle gpu6
    RTX4090      up 3-00:00:00      2   idle gpu[7-8]
    L40          up 3-00:00:00      2   idle gpu[9-10]
    ```

- **完整的代码文件**：

    !!! note "示例代码文件：test.py"

        ```python
        import torch
        print(torch.cuda.is_available())
        print(torch.cuda.device_count())
        
        if torch.cuda.is_available():
            tensor_gpu = torch.rand(3, 3).cuda()
            print(tensor_gpu)
        ```

## 核心内容

在接下来的章节中，您将学习如何**高效安全地使用集群资源**，包括：

- **账号申请**：通过在线申请系统开通账户和计算集群账号，查看已使用的机时费
- **环境配置**：通过 module 命令加载 CUDA、Anaconda3 等基础环境
- **代码编译**：在用户登录节点使用用编译器正确编译 C/C++、CUDA 等代码
- **作业提交**：合理申请 CPU/GPU/内存资源，通过 Slurm 系统提交任务

## 重要守则

使用集群时请务必遵守：

1. **仅将计算资源用于科研或学习等用途**；
2. **不在用户登录节点裸跑任务**，始终通过 Slurm 作业调度系统申请资源和运行作业；
3. 提前预估资源需求，结束任务后及时释放资源。
