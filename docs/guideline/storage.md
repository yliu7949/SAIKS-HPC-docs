---
icon: material/harddisk
---

# 集群存储 {#storage}

## 共享文件系统 {#shared-filesystem}

用户可在登录节点执行以下命令查看共享目录信息：

```bash
df -h | grep data
```

输出示例如下：

```bash
Filesystem      Size  Used Avail Use% Mounted on
data1:/opt     1006G   64G  892G   7% /opt
data1:/home      60T  1.8T   55T   4% /home
data1:/data1     59T     0   56T   0% /data1
data2:/data2    232T  139G  221T   1% /data2
data2:/backup   232T  1.0M  221T   1% /backup
```

集群配备 data1 和 data2 两台存储服务器。data1 提供 `/opt`、`/home` 和 `/data1` 三个共享目录，data2 提供 `/data2` 和 `/backup` 两个共享目录。各目录的详细用途与用户配额如下表所示：

| 挂载点       | 总容量     | 用户配额    | 用途          | 备注         |
|-----------|---------|---------|-------------|------------|
| `/opt`    | 1006 GB | 无       | 集群预装软件      | 普通用户只读     |
| `/home`   | 60 TB   | 120 GB  | 用户家目录       | 用户家目录下可读写  |
| `/data1`  | 59 TB   | 120 GB  | 供软件运行时使用    | 用户名文件夹下可读写 |
| `/data2`  | 232 TB  | 1024 GB | 大文件存储       | 用户名文件夹下可读写 |
| `/backup` | 232 TB  | 512 GB  | 归档结果和重要数据备份 | 用户名文件夹下可读写 |

使用 `/data1`、`/data2` 或 `/backup` 时，请进入对应用户组名下以你的用户名命名的子目录。例如，通过 `cd /data2/$(groups)/$(whoami)` 命令进入你在 `/data2` 中的个人目录。

使用 `quota -s` 命令可以查看共享目录配额和已使用的存储空间信息，例如：

```bash
Disk quotas for user user (uid 1000): 
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
   data2:/data2    139G   1000G   1024G             655       0       0        
  data2:/backup      4K    500G    512G               1       0       0
```

请根据实际需求合理使用各存储目录，并定期清理不再需要的文件，以免占用过多存储空间。如有特殊存储需求，请联系系统管理员。
