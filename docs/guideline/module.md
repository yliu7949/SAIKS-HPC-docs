---
icon: material/apps-box
---

# 设置编译及运行环境 {#module}

本集群安装了多种编译环境及应用软件。为方便用户使用，系统采用 [Environment Modules](https://modules.sourceforge.net) 工具进行封装，用户可以通过 `module` 命令来加载、卸载和查看环境模块。

## 基本命令

- `module avail`  
  列出所有可用的模块。运行结果如下所示：

    ```shell
    --------------------------------------------------------------------- /opt/Modules/app ----------------------------------------------------------------------
    Anaconda3/2025.06  CMake/4.1.0  code-server/4.104.1
    
    ------------------------------------------------------------------- /opt/Modules/compiler -------------------------------------------------------------------
    cuda/12.9.1  cuda/13.0
    
    -------------------------------------------------------------------- /opt/Modules/intel ---------------------------------------------------------------------
    advisor/2025.2                compiler-rt/2025.2.0  debugger/2025.2.0       dpct/2025.2.0             intel_ippcp_intel64/2025.2  mpi/2021.16  vtune/2025.4  
    advisor/latest                compiler-rt/latest    debugger/latest         dpct/latest               intel_ippcp_intel64/latest  mpi/latest   vtune/latest  
    ccl/2021.16.0                 compiler/2025.2.0     dev-utilities/2025.2.0  dpl/2022.9                ishmem/1.3.0                tbb/2022.2   
    ccl/latest                    compiler/latest       dev-utilities/latest    dpl/latest                ishmem/latest               tbb/latest   
    compiler-intel-llvm/2025.2.0  dal/2025.6            dnnl/3.8.1              intel_ipp_intel64/2022.2  mkl/2025.2                  umf/0.11.0   
    compiler-intel-llvm/latest    dal/latest            dnnl/latest             intel_ipp_intel64/latest  mkl/latest                  umf/latest
    
    Key:
    modulepath
    ```

- `module load <name>`  
    加载指定模块，例如：`module load Anaconda3/2025.06`。

- `module unload <name>`  
  卸载指定模块。

- `module list`  
  显示当前已加载的模块。

- `module purge`  
  卸载所有已加载的模块。

- `module help <name>`  
  查看指定模块的说明。

## 简写命令

系统还提供了更简便的命令 `ml`，它是 `module` 的别名：

- `ml avail` → 等同于 `module avail`
- `ml load <name>` → 等同于 `module load <name>`
- `ml unload <name>` → 等同于 `module unload <name>`
- `ml list` → 等同于 `module list`
- `ml purge` → 等同于 `module purge`

例如：

```bash
ml Anaconda3/2025.06
ml list
```

这样可以更快速地管理环境模块。

## 提示

- 建议新手用户主要掌握 `avail`、`load`、`unload`、`list`、`purge` 即可。  
- 进阶用户可以参考 `man module` 获取更详细的功能说明。  
