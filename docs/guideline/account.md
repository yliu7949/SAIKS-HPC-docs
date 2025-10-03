---
icon: material/account
---

# 账号申请 {#account}

本集群的用户需要通过 <a href="http://user.saids.hpc.gleamoe.com/" target="_blank" rel="noopener noreferrer">在线申请系统</a> 完成账户开通。学生作为申请人时与导师需分别完成注册和验证流程。

## 账户开通流程 {#account-opening-process}

### 用户注册阶段 {#user-registration-stage}

#### **导师注册**

导师注册是必需先行步骤。

  1. 访问 <a href="http://user.saids.hpc.gleamoe.com/" target="_blank" rel="noopener noreferrer">在线申请系统</a>，选择“导师申请”
  ![图片](https://cdn.gleamoe.com/saids/login/1.png%40same.webp)
  2. 填写申请信息
  ![图片](https://cdn.gleamoe.com/saids/login/2.png%40same.webp)
  3. 提交表单信息，等待系统管理员审核（≤1 个工作日）
  ![图片](https://cdn.gleamoe.com/saids/login/3.png%40same.webp)
  4. 账户申请成功（邮件通知）
  ![图片](https://cdn.gleamoe.com/saids/login/4.png%40same.webp)
  5. 导师登录系统审批学生申请流程：
    - 收到学生待审批邮件
  ![图片](https://cdn.gleamoe.com/saids/login/7.png%40same.webp)
    - 使用已注册的用户名及密码登录系统
  ![图片](https://cdn.gleamoe.com/saids/login/5.png%40same.webp)
    - 审批学生申请
  ![图片](https://cdn.gleamoe.com/saids/login/6.png%40same.webp)

#### **学生注册**

  1. 访问 <a href="http://user.saids.hpc.gleamoe.com/" target="_blank" rel="noopener noreferrer">在线申请系统</a>，选择“学生申请”
  ![图片](https://cdn.gleamoe.com/saids/login/8.png%40same.webp)
  2. 填写表单信息，提交后等待导师审批
  ![图片](https://cdn.gleamoe.com/saids/login/9.png%40same.webp)
  3. 收到导师审批通过邮件
  ![图片](https://cdn.gleamoe.com/saids/login/10.png%40same.webp)
  4. 等待系统管理员审核（≤1 个工作日），审核通过后账户申请成功
  ![图片](https://cdn.gleamoe.com/saids/login/11.png%40same.webp)

### 账户申请流程 {#account-application-process}

导师申请流程

```mermaid
sequenceDiagram
    participant T as 导师
    participant Sys as 申请系统
    participant M as 邮件系统
    participant Admin as 系统管理员

    T->>Sys: 提交导师注册申请
    note right of T: 填写表单并提交

    Sys->>Admin: 新增一条待审批的导师申请
    note over Sys,Admin: 等待系统管理员审核 (≤1 工作日)

    Admin->>Sys: 登录系统, 批准申请
    
    Sys->>M: 生成账户凭据邮件
    M->>T: 发送账户开通邮件 (含凭据和 2FA)
    note left of T: 导师账户申请成功
```

学生申请流程

```mermaid
sequenceDiagram
    participant S as 学生
    participant Sys as 申请系统
    participant M as 邮件系统
    participant T as 导师
    participant Admin as 系统管理员

    S->>Sys: 提交学生注册申请
    note right of S: 填写表单并提交

    Sys->>M: 向导师发送审批提醒
    M->>T: 发送学生待审批邮件
    note over S, T: 等待导师审批

    T->>Sys: 登录系统, 审批学生申请

    alt 导师批准
        Sys->>M: 向学生发送导师审批通过通知
        M->>S: 发送“导师已审批”邮件
        note right of S: 收到导师审批通过邮件

        Sys->>Admin: 新增一条待审批的学生申请
        note over S, Admin: 等待系统管理员最终审核 (≤1 工作日)

        Admin->>Sys: 登录系统, 最终批准申请

        Sys->>M: 生成账户凭据邮件
        M->>S: 发送账户开通邮件 (含凭据和 2FA)
        note right of S: 学生账户申请成功
    else 导师拒绝
        Sys->>M: 向学生发送申请被拒通知
        M->>S: 告知申请已被导师拒绝
    end
```

### 账户信息交付 {#account-information-delivery}

审批通过后将通过邮件发送包含：

- 登录节点 IP
- 初始凭证：
    - 用户名
    - 随机初始密码
    - 2FA 密钥
