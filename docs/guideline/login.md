---
icon: material/information-box
---

# 集群登录与文件传输

## 集群登录

集群的用户登录节点运行着 64 位 Ubuntu 24.04.3 LTS 操作系统。获取集群用户账号和认证凭据后，您可以通过 SSH (Secure Shell) 协议登录集群。

### Windows 用户登录方式

Windows 用户可选择以下任一支持 SSH 协议的客户端软件进行登录：

1. **PuTTY** - 轻量级开源 SSH 客户端  
2. **Xshell** - 功能丰富的商业 SSH 客户端  
3. **CMD/PowerShell** - 内置 SSH 命令  

登录时需要提供以下信息：  

- 用户名
- 集群域名 / IP 地址
- 账号密码
- 2FA 动态验证码（双因素认证）

下面以 Xshell 软件为例，详细介绍使用图形界面客户端软件登录集群的步骤。 打开 Xshell 软件，点击软件左上角新建会话（或在打开软件时弹出的窗口中点击新建会话）

1. 在名称框中根据自己的习惯输入名称
2. 输入集群地址
3. 点击 "用户身份验证"，去输入账号信息
4. 进入 "用户身份验证" 输入用户名和密码
5. 方法栏勾选 "Password" 和 "Keyboard Interactive"
6. 点击连接，并在弹出的窗口中输入动态验证码

![图片](../images/xshell-1.png)
![图片](../images/xshell-2.png)

### macOS/Linux 用户登录方式

macOS 和 Linux 用户除了使用 Termius 等支持 SSH 协议的客户端软件外，还可以在终端进行 SSH 登录：

```shell
ssh username@cluster_address
```

首次连接时会提示确认主机密钥指纹，输入 yes 确认后继续。

Linux 操作系统的教程可以参考 [《Linux 101》在线讲义](https://101.lug.ustc.edu.cn/)。

## 文件传输

### 使用 SFTP 客户端传输文件

推荐工具：

- WinSCP (Windows)
- FileZilla (跨平台)
- Cyberduck (macOS)

### 命令行传输工具

(1) scp 命令：
```shell
# 上传文件到集群
scp /本地/文件 用户名@集群 IP:/远程/路径

# 从集群下载文件
scp 用户名@集群 IP:/远程/文件 /本地/路径
```

(2) rsync 命令：
```shell
# 同步本地目录到集群
rsync -avz /本地/目录/ 用户名@集群 IP:/远程/目录/

# 从集群同步到本地
rsync -avz 用户名@集群 IP:/远程/目录/ /本地/目录/
```

(3) rz/sz 命令：
```shell
# 上传文件
rz

# 下载文件
sz 文件名
```