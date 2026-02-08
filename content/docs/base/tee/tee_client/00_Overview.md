# TEE Client 概述

## 项目定位

TEE Client 是 OpenHarmony TEE（Trusted Execution Environment）子系统的客户端组件，负责向 REE（Rich Execution Environment）侧的 CA（Client Application）提供访问 TEE 的标准化接口。

**核心职责**：
- 实现 GlobalPlatform TEE Client API 规范
- 转发 CA 请求到 TEE 侧
- 管理 TEE 会话与共享内存
- 提供 TEE 日志打印能力

## 核心能力

| 能力 | 说明 | 位置 |
|------|------|------|
| TEE 上下文管理 | 初始化/销毁 TEE 连接 | libteec.so / libteec_vendor.so |
| 会话管理 | 打开/关闭与 TA 的会话 | libteec.so / libteec_vendor.so |
| 命令发送 | 向 TA 发送命令 | libteec.so / libteec_vendor.so |
| 共享内存 | 注册/分配/释放共享内存 | libteec.so / libteec_vendor.so |
| CA 请求转发 | 接收并转发 CA 请求到 TEE | cadaemon |
| TEE 代理服务 | 安全存储、日志等代理功能 | teecd |
| 日志打印 | TEE 日志读取与打印 | tlogcat |

## 运行环境

### 系统依赖

| 依赖组件 | 用途 |
|----------|------|
| OpenHarmony 内核 | 提供 TEE 设备节点 |
| tee_tzdriver | TEE 可信驱动 |
| ipc | 进程间通信框架 |
| safwk | 系统 ability 框架 |
| hilog | 日志系统 |

### 硬件要求

- 支持 TEE 的 SoC 芯片
- TEE 分区（通常为 /vendor）

### 运行时依赖文件

| 文件 | 类型 | 说明 |
|------|------|------|
| /dev/tee* | 设备节点 | TEE 驱动设备 |
| /system/lib/libteec.so | 动态库 | 系统组件 TEE API |
| /vendor/lib/libteec_vendor.so | 动态库 | 芯片组件 TEE API |
| /system/lib/libcadaemon.so | 动态库 | CA 守护进程 |
| /vendor/bin/teecd | 可执行文件 | TEE 代理服务 |
| /system/bin/tlogcat | 可执行文件 | 日志服务 |

## 关键概念

### CA (Client Application)

运行在 REE（Linux 内核层）的普通应用程序，通过调用 TEE Client API 访问 TEE 中的 TA。

### TA (Trusted Application)

运行在 TEE（可信执行环境）中的安全应用，提供密钥管理、安全存储、身份认证等安全功能。

### TEE Context

TEE 上下文，表示 CA 与特定 TEE 设备之间的逻辑连接。

### Session

会话，表示 CA 与特定 TA 之间的通信通道。

### Shared Memory

共享内存，用于 CA 与 TA 之间的高效数据传输。

## 模块概览

```
tee_client/
├── frameworks/
│   ├── libteec_client/      # libteec.so 实现（系统组件用）
│   └── libteec_vendor/      # libteec_vendor.so 实现（芯片组件用）
├── interfaces/
│   └── inner_api/           # 内部 API 头文件
└── services/
    ├── cadaemon/            # CA 请求转发守护进程
    ├── teecd/              # TEE 代理服务
    └── tlogcat/            # TEE 日志服务
```

## 版本信息

| 属性 | 值 |
|------|-----|
| 组件版本 | 1.0.0 |
| API 版本 | GlobalPlatform TEE Client API v1.0 |
| 许可证 | MulanPSL-2.0 |
