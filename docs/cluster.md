---
icon: material/server
---

# 计算集群概况 {#cluster-overview}

集群试运行阶段：2025 年 8 月 25 日至 2025 年 9 月 30 日；正式运行阶段：自 2025 年 10 月 1 日起。

!!! info "计算集群已于 2025 年 10 月 1 日进入正式运行阶段。"

## 硬件环境 {#hardware}

### 节点配置

| 节点类别            | 节点名          | 主要规格                                                                     | 节点数 |
|-----------------|--------------|--------------------------------------------------------------------------|-----|
| GPU 节点（A800）    | gpu[1-5]     | 2 * Intel Xeon Platinum 8358P CPU @ 2.60GHz, 1TB, NVIDIA A800 8-GPU 80GB | 5   |
| GPU 节点（RTX4090） | gpu[6-8]     | 2 * Intel Xeon Gold 6426Y CPU @ 2.5GHz, 512GB, NVIDIA RTX4090 8-GPU 24GB | 3   |
| GPU 节点（L40）     | gpu[9-10]    | 2 * Intel Xeon Platinum 8360Y CPU @ 2.40GHz, 1TB, L40 8-GPU 45GB         | 2   |
| 登录节点 / 管理节点     | login/master | 2 * Intel Xeon Gold 6226R CPU @ 2.90GHz, 384GB                           | 1   |
| 存储节点 1          | data1        | 2 \* Intel Xeon Silver 4310, 128G, 36 \* 3.84TB SSD                      | 1   |
| 存储节点 2          | data2        | 2 \* Intel Xeon Silver 4310, 128G, 34 \* 16TB HDD                        | 1   |

### 节点性能

| 节点类别            | 单核主频    | 单节点核心数                                  | 单周期指令执行数 | 节点数    | 理论峰值/GFlops     |
|-----------------|---------|-----------------------------------------|----------|--------|-----------------|
| GPU 节点（A800）    | 2.60GHz | 3,456 Tensor cores + 55,296 CUDA cores  | NA       | 5      | 780,000         |
| GPU 节点（RTX4090） | 2.50GHz | 4,096 Tensor cores + 131,072 CUDA cores | NA       | 3      | 1,982,400       |
| GPU 节点（L40）     | 2.40GHz | 4,544 Tensor cores + 145,408 CUDA cores | NA       | 2      | 1,448,000       |
| 登录节点 / 管理节点     | 2.90GHz | 32                                      | 32       | 1      | 2,969.6         |
| 存储节点 1          | 2.10GHz | 48                                      | 32       | 1      | 3,225.6         |
| 存储节点 2          | 2.10GHz | 48                                      | 32       | 1      | 3,225.6         |
| **合计**          |         |                                         |          | **13** | **4,219,820.8** |

计算节点：10 个、登录/管理/存储节点：3 个、GPU 卡：80 张。理论计算峰值约 4.22PFLOPS。

### 存储配置

| 节点类别   | 主要规格            | 可用容量                |
|--------|-----------------|---------------------|
| 存储节点 1 | 36 * 3.84TB SSD | 配置硬件 RAID 后约为 120TB |
| 存储节点 2 | 34 * 16TB HHD   | 配置硬件 RAID 后约为 470TB |

存储容量合计：682.24TB，可用容量合计约 590TB。

## 分区设置 {#slurm-partition}

| 分区      | 节点列表      | 单节点规格                   | 数量 |                    备注                    |
|---------|-----------|-------------------------|----|:----------------------------------------:|
| A800    | gpu[1-5]  | 128 核，1TB 内存，8 张 GPU 卡  | 5  | 推荐每申请 1 张 GPU 卡搭配申请 16 核 CPU 和 128 GB 内存 |
| RTX4090 | gpu[6-8]  | 64 核，512GB 内存，8 张 GPU 卡 | 2  |  推荐每申请 1 张 GPU 卡搭配申请 8 核 CPU 和 64 GB 内存  |
| L40     | gpu[9-10] | 144 核，1TB 内存，8 张 GPU 卡  | 2  | 推荐每申请 1 张 GPU 卡搭配申请 18 核 CPU 和 128 GB 内存 |

当前资源配置策略：

- 每个计算任务仅支持在单一计算节点上运行；
- 在 A800 分区中运行的作业，必须至少申请 1 张 GPU 卡。

## QOS 设置 {#QOS}

| QOS 名称 |  MaxJobsPU  |  MaxSubmitPU  |
|--------|:-----------:|:-------------:|
| normal |      3      |       4       |

PU = per user（每用户）

- MaxJobsPU：每用户在此 QoS 下可同时运行的作业数上限：3。
- MaxSubmitPU：每用户在此 QoS 下运行 + 排队的作业数上限：4。例如 2 个运行 + 2 个排队。

## 计费标准 {#billing-standards}

为保障 GPU 资源的公平使用，我们实行**“等效占用”**的计费原则：若作业申请并使用了大量 CPU，即使未实际使用 GPU，
也需承担**相应比例的 GPU 成本**。该机制旨在防止变相独占稀缺资源，引导用户按需精确申请资源，从而提升整体资源利用率。

作业费用将根据**实际占用的加速卡与 CPU 资源量**计算，公式如下：

$$
\text{usageHours} \times \max \left\{ \text{gpusUsed}, \lceil \text{cpusAlloc} \times \frac{\text{gpusPerPartition}}{\text{coresPerPartition}} \rceil \right\} \times \text{unitPrice}.
$$

分区计费标准：

| 分区      | 单价（元） |                                                              费用计算公式                                                              |
|---------|:-----:|:--------------------------------------------------------------------------------------------------------------------------------:|
| A800    | 2.00  | $\text{usageHours} \times \max \left\{\text{gpusUsed},\;\lceil \text{cpusAlloc} \times \frac{8}{128} \rceil\right\} \times 2.00$ |
| RTX4090 | 0.50  | $\text{usageHours} \times \max \left\{\text{gpusUsed},\;\lceil \text{cpusAlloc} \times \frac{8}{64} \rceil\right\} \times 0.50$  |
| L40     | 1.00  | $\text{usageHours} \times \max \left\{\text{gpusUsed},\;\lceil \text{cpusAlloc} \times \frac{8}{144} \rceil\right\} \times 1.00$ |

!!! comment "作业费用计算示例"

    某作业被分配至 L40 分区，资源分配情况（AllocTRES）为 `cpu=64,mem=64G,node=1`，运行时长（Elapsed）为 `3-00:00:03`。根据费用计算公式：
    
    $$
    \frac{259203}{60 \times 60} \times \max\left\{0,\ \left\lceil 64 \times \frac{8}{144} \right\rceil \right\} \times 1.00 = 288.003 \text{ 元}.
    $$
    
    最终计算得出该作业费用为 288.003 元。

## 费用充值 {#recharge-method}

充值功能目前正在内部测试与流程拟定中，预计将于今年 11 月正式开放。