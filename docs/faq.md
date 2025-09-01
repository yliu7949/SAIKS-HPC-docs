---
icon: material/frequently-asked-questions
---

# 常见问题 {#faq}

## 作业相关

Q1：为什么使用 `sinfo` 命令查看对应的分区有空闲节点，但是我的作业却还在排队？

A1：在 Slurm 作业调度系统中，调度器会为高优先级作业预留资源，以确保其顺利运行，同时兼顾其他优先级作业的调度。

Q2：在登录节点下载安装包时网络错误？如何访问互联网资源？

A2：执行 `ping sfd.org`，若 ping 通则说明已可访问外网，可直接下载安装包。若无法访问外网，请参考 <a href="https://lug.ustc.edu.cn/wiki/scripts/wlt/#%E4%BD%BF%E7%94%A8-curl" target="_blank" rel="noopener noreferrer">USTCLUG 网络通脚本</a> 编写 shell 脚本登录自己的网络通账号，执行脚本后即可正常访问互联网资源。
