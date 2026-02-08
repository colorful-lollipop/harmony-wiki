# 术语表

## A

| 术语 | 英文 | 说明 |
|------|------|------|
| A-core | Application Processor Core | 应用处理器核，通常指 Cortex-A 系列，支持完整操作系统 |

## B

| 术语 | 英文 | 说明 |
|------|------|------|
| Binder | Binder | Android/Linux 进程间通信机制 |
| Bootstrap | Bootstrap | 系统引导服务，负责初始化系统服务 |

## C

| 术语 | 英文 | 说明 |
|------|------|------|
| CMSIS | Cortex Microcontroller Software Interface Standard | ARM Cortex-M 微控制器软件接口标准 |
| Consumer | Consumer | 事件消费者，订阅并接收广播通知 |

## D

| 术语 | 英文 | 说明 |
|------|------|------|
| DBinder | DBinder | 轻量级 Binder，适用于资源受限设备 |

## F

| 术语 | 英文 | 说明 |
|------|------|------|
| Feature | Feature | 服务的子功能模块，可独立提供 API |

## G

| 术语 | 英文 | 说明 |
|------|------|------|
| GN | Generate Ninja | Google 开发的构建系统生成工具 |

## I

| 术语 | 英文 | 说明 |
|------|------|------|
| Identity | Identity | 服务或 Feature 的唯一身份标识 |
| IpcIo | IPC I/O | IPC 消息序列化/反序列化接口 |
| IPC | Inter-Process Communication | 进程间通信 |
| IUnknown | IUnknown | COM 风格的通用接口基类 |

## M

| 术语 | 英文 | 说明 |
|------|------|------|
| M-core | Microcontroller Core | 微控制器核，通常指 Cortex-M 系列，资源受限 |
| Message | Message | 消息，请求/响应通信的基本单位 |

## P

| 术语 | 英文 | 说明 |
|------|------|------|
| POSIX | Portable Operating System Interface | 可移植操作系统接口标准 |
| Provider | Provider | 事件发布者，发布广播主题 |
| Pub/Sub | Publish/Subscribe | 发布/订阅模式 |

## R

| 术语 | 英文 | 说明 |
|------|------|------|
| RPC | Remote Procedure Call | 远程过程调用 |
| Request | Request | 请求消息 |

## S

| 术语 | 英文 | 说明 |
|------|------|------|
| SA | System Ability | 系统能力，系统提供的功能单元 |
| Samgr | Service Manager | 服务管理器 |
| Service | Service | 服务，系统功能的基本单元 |
| SOA | Service-Oriented Architecture | 面向服务的架构 |

## T

| 术语 | 英文 | 说明 |
|------|------|------|
| Task | Task | 任务，线程的执行单元 |
| Topic | Topic | 主题，广播事件类型标识 |

## V

| 术语 | 英文 | 说明 |
|------|------|------|
| Vector | Vector | 动态数组容器 |

## 其他术语

| 术语 | 说明 |
|------|------|
| DEFAULT_VERSION | IUnknown 接口默认版本号 (0x20) |
| CLIENT_PROXY_VER | IPC 客户端代理版本号 |
| SERVER_PROXY_VER | IPC 服务端代理版本号 |
| SHARED_TASK | 共享任务类型 |
| SINGLE_TASK | 独占任务类型 |
| SYSEX_SERVICE_INIT | 服务初始化宏 |
| SYSEX_FEATURE_INIT | Feature 初始化宏 |

## 目录结构术语

| 目录 | 说明 |
|------|------|
| interfaces/kits | 对外 API 头文件 |
| services/samgr_lite/samgr | Samgr 核心服务实现 |
| services/samgr_lite/samgr_client | IPC 客户端 |
| services/samgr_lite/samgr_server | IPC 服务端 |
| services/samgr_lite/samgr_endpoint | IPC 通信层 |
| services/samgr_lite/communication | 广播服务 |
| samgr/adapter | 平台适配层 |
