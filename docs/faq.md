---
icon: material/frequently-asked-questions
---

# 常见问题 {#faq}

## 作业与调度 {#job-scheduling}

Q1.1：为什么使用 `sinfo` 命令查看对应的分区有空闲节点，但是我的作业却还在排队？

A1.1：在 Slurm 作业调度系统中，调度器会为高优先级作业预留资源，以确保其顺利运行，同时兼顾其他优先级作业的调度。

## 集群访问 {#cluster-access}

Q2.1：校外是否可以访问计算集群？

A2.1：计算集群仅对校园网开放。教师用户可以使用[中国科学技术大学网络 OpenVPN 系统](https://openvpn.ustc.edu.cn/) 在校外访问计算集群。

## 网络 {#networking}

Q3.1：在登录节点下载安装包时网络错误？如何访问互联网资源？

A3.1：执行 `ping sfd.org`，若 ping 通则说明已可访问校外互联网，可直接下载安装包。若无法访问校外网络，请参考 <a href="https://lug.ustc.edu.cn/wiki/scripts/wlt/#%E4%BD%BF%E7%94%A8-curl" target="_blank" rel="noopener noreferrer">USTCLUG 网络通脚本</a> 编写 shell 脚本登录自己的网络通账号，执行脚本后即可正常访问互联网资源。

Q3.2：计算节点如何访问互联网资源？

A3.2：在登录节点登录自己的网络通账号后，计算节点同登录节点均可访问互联网资源。

## 软件使用 {#software-usage}

Q4.1：可否安装 CUDA 12.8？

A4.1：用户可通过 conda 在个人环境中安装特定版本的 CUDA Toolkit。请参考 <a href="https://201.ustclug.org/advanced/cuda/#cuda-conda" target="_blank" rel="noopener noreferrer">使用 Conda 安装 CUDA</a>。
