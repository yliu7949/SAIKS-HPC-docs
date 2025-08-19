---
icon: material/apps-box
---

# 设置编译及运行环境 {#module}

本集群安装了多种编译环境及应用软件。为方便用户使用，系统采用 [Environment Modules](https://modules.sourceforge.net) 工具进行封装，用户可以通过 `module` 命令来加载、卸载和查看环境模块。

## 基本命令

- `module avail`  
  列出所有可用的模块。

- `module load <name>`  
  加载指定模块，例如：
  ```bash
  module load intel/2020
  ```

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
ml intel/2020
ml list
```

这样可以更快速地管理环境模块。

## 提示

- 建议新手用户主要掌握 `avail`、`load`、`unload`、`list`、`purge` 即可。  
- 进阶用户可以参考 `man module` 获取更详细的功能说明。  
