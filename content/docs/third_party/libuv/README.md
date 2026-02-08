# libuv - OpenHarmony Wiki

## 库概览

**libuv** 是 OpenHarmony 的第三方异步 I/O 基础库，为系统各模块提供高性能的事件循环和线程池支持。

### 基本信息

| 属性 | 值 |
|-----|-----|
| **原始库** | libuv v1.48.0 |
| **上游地址** | https://github.com/libuv/libuv |
| **许可证** | MIT |
| **OH 组件名** | @ohos/libuv |
| **子系统** | thirdparty |
| **系统能力** | SystemCapability.ArkUI.ArkUI.Libuv |

### 核心功能

- ✅ 跨平台事件循环 (epoll/kqueue/IOCP)
- ✅ 异步 TCP/UDP 网络 I/O
- ✅ 异步文件系统操作
- ✅ 线程池与任务调度
- ✅ 定时器和信号处理
- ✅ 进程间通信 (IPC)

### OpenHarmony 特有增强

- 🔧 **FFRT 集成**: 支持 FFRT 任务调度框架，提供 QoS 分级
- 🔧 **HiLog 日志**: 集成 OHOS 统一日志系统
- 🔧 **HiTrace 跟踪**: 支持系统性能跟踪
- 🔧 **异步堆栈**: 支持异步调用堆栈跟踪
- 🔧 **DFX 诊断**: 集成 OHOS 诊断框架

---

## 文档导航

### 必读文档

| 文档 | 说明 | 优先级 |
|-----|------|-------|
| [01_Overview.md](./01_Overview.md) | 原始库简介和 OH 定位 | ⭐⭐⭐ |
| [02_Patches.md](./02_Patches.md) | Patch 分析（OH 定制化说明） | ⭐⭐⭐⭐⭐ |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 适配详解 | ⭐⭐⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景 | ⭐⭐⭐ |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异 | ⭐⭐ |
| [06_Security.md](./06_Security.md) | 安全风险分析 | ⭐⭐ |

### 阅读建议

**如果你是...**

- **系统开发者**: 阅读 01 → 04 → 02 → 03
- **库维护者**: 阅读 02 → 03 → 05 → 06
- **安全工程师**: 阅读 06 → 02 → 03
- **应用开发者**: 阅读 01 → 04

---

## 快速参考

### 关键宏定义

```c
// FFRT 集成
#define USE_FFRT

// OHOS 诊断框架
#define USE_OHOS_DFX

// 异步堆栈跟踪
#define ASYNC_STACKTRACE

// 中断支持
#define SUPPORT_INTERRUPT

// Worker 优先级
#define ENABLE_WORKER_PRIORITY
```

### 关键文件

```
third_party/libuv/
├── BUILD.gn                    # 构建配置
├── libuv.gni                   # GN 参数定义
├── src/unix/ohos/              # OH 特有实现
│   ├── trace_ohos.c           # HiTrace 集成
│   └── log_ohos.c             # 模拟器日志
├── src/dfx/async_stack/        # 异步堆栈
│   ├── libuv_async_stack.c
│   └── libuv_async_stack.h
└── src/uv_log.h               # 统一日志
```

### 关键 QoS 级别

```c
typedef enum {
  uv_qos_background = 0,        // 后台任务
  uv_qos_utility = 1,           // 实用工具
  uv_qos_default = 2,           // 默认
  uv_qos_user_initiated = 3,    // 用户发起
  uv_qos_reserved = 4,          // 保留
  uv_qos_user_interactive = 5,  // 用户交互
} uv_qos_t;
```

---

## 贡献与维护

- **维护者**: sunbingxin@huawei.com
- **子系统**: thirdparty
- **问题反馈**: OpenHarmony 社区

---

*最后更新: 2025-02-08*
