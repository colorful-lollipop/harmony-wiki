# AI Engine 概览

## 项目定位

AI Engine 是 OpenHarmony AI 子系统的核心框架，提供统一的 AI 算法引擎能力。其设计目标包括：

- **插件化架构**：实现 AI 算法插件的快速集成与热插拔
- **分布式能力**：支持设备间 AI 能力的分布式调用
- **统一接口**：标准化 AI 算法 API，提供统一的推理接口
- **轻量级设计**：适用于 OpenHarmony 小型设备（ROM 130KB，RAM ~337KB）

**证据**：`bundle.json:13-19` 定义组件信息，`README.md:12` 描述框架目标

## 核心能力

### 主要功能

| 能力 | 说明 |
|------|------|
| **插件管理** | 动态加载/卸载 AI 算法插件，支持生命周期管理 |
| **算法推理** | 支持同步和异步两种推理模式 |
| **客户端 SDK** | 提供面向应用的轻量级 SDK 接口 |
| **IPC 通信** | 基于 SAMGR 的进程间通信，支持共享内存优化 |
| **线程调度** | 内置线程池和任务队列，支持高并发 |

### 内置算法插件

| 插件 | 类型 | 功能 |
|------|------|------|
| Keyword Spotting (KWS) | ASR | 语音关键词唤醒 |
| Image Classification (IC) | CV | 图像分类识别 |

**证据**：`README.md:33-37` 目录结构描述插件类型

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **操作系统** | OpenHarmony |
| **系统类型** | small (小型设备) |
| **编程语言** | C/C++ (C++14) |
| **依赖组件** | hilog, utils_base, ipc, samgr_lite |
| **第三方依赖** | bounds_checking_function |

### 外部依赖

| 依赖 | 用途 |
|------|------|
| SAMGR Lite | 系统能力管理器，服务注册与发现 |
| IPC | 进程间通信 |
| hilog | 日志输出 |
| bounds_checking_function | 安全函数 |

**证据**：`bundle.json:20-30` 组件依赖配置

## 关键概念

### 客户端与服务器

AI Engine 采用 C/S 架构设计：

- **客户端 (Client)**：调用 AI 能力的应用进程，通过 SDK 接口发起请求
- **服务器 (Server)**：运行 AI 引擎服务的系统进程，管理插件和执行推理

### 插件与引擎

- **插件 (Plugin)**：实现 `IPlugin` 接口的算法实现单元，可动态加载
- **引擎 (Engine)**：服务器端的推理执行器，管理插件实例和任务队列

### 同步与异步

- **同步模式 (Sync)**：阻塞式推理调用，立即返回结果
- **异步模式 (Async)**：非阻塞式推理，通过回调返回结果

### UID 与权限

AI Engine 基于 Linux UID 进行权限控制：

- **clientUid**：客户端进程的 UID
- **serverUid**：服务端进程的 UID
- **共享内存**：通过 UID 控制共享内存的访问权限

**证据**：`aie_info_define.h:25-35` ClientInfo 结构体定义

## 版本信息

| 版本 | 日期 | 说明 |
|------|------|------|
| 3.1 | 2021 | 当前版本，AI Engine 框架 |

**证据**：`bundle.json:4` 组件版本

## 相关文档

- [AI 子系统概览](https://gitee.com/openharmony/docs/blob/master/en/readme/ai.md)
- [AI Engine Framework 开发指南](https://gitee.com/openharmony/docs/blob/master/en/device-dev/subsystems/subsys-ai-aiframework-devguide.md)
- [SAMGR Lite](https://gitee.com/openharmony/distributedschedule_samgr_lite/blob/master/README.md)
