---
icon: material/frequently-asked-questions
---

# 常见问题 {#faq}

## 作业相关

Q1：为什么使用 `sinfo` 命令查看对应的分区有空闲节点，但是我的作业却还在排队？

在 Slurm 作业调度系统中，调度器会为高优先级作业预留资源，以确保其顺利运行，同时兼顾其他优先级作业的调度。
