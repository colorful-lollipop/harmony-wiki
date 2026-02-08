# 01_Overview - 项目概览

## 1.1 项目定位

### 1.1.1 在 DFX 子系统中的角色

`blackbox_lite` 是 OpenHarmony **DFX（Design for X）子系统**的核心模块之一，负责**死机重启故障现场信息的抓取与保存**。

```
DFX 子系统
├── hiviewdfx_blackbox_lite    ← 本文档主题
├── hiviewdfx_hidumper_lite   # 系统信息转储
├── hiviewdfx_hilog_lite      # 日志系统
├── hiviewdfx_hievent_lite     # 事件系统
└── hiviewdfx_hiview_lite      # 故障视图
```

**证据**：`bundle.json:3-4`
> `"description": "blackbox_lite provides the software blackbox capability."`

### 1.1.2 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 故障信息获取 | 通过适配层获取系统异常信息 | `blackbox_core.c:252` |
| 故障日志保存 | 将故障数据写入持久化存储 | `blackbox_core.c:143` |
| 系统重启 | 触发可选的系统复位操作 | `blackbox_core.c:312` |
| 事件上报 | 通知 hiview 上报故障事件 | `blackbox_detector.c:18` |

### 1.1.3 适用场景

- **LiteOS 内核**：liteos_m、liteos_a
- **资源受限设备**：ROM ~10KB，RAM ~5KB
- **死机重启故障**：Panic、Watchdog、Hung Task 等

**证据**：`bundle.json:16-20`
```json
"adapted_system_type": ["mini"],
"rom": "10KB",
"ram": "~5KB"
```

## 1.2 关键概念

### 1.2.1 WEAK 适配器模式

blackbox_lite 采用 **WEAK 符号**实现平台无关代码，平台层通过重写 WEAK 函数提供具体实现。

```
┌─────────────────────────────────────────────────────┐
│                  blackbox_lite 核心代码             │
│  blackbox_core.c (不可修改)                         │
│         ↓ 调用                                       │
│  WEAK 函数接口 (blackbox_adapter.c)                 │
│         ↓ 由平台实现                                  │
│  device/soc/xxx/blackbox_adapter_impl.c             │
└─────────────────────────────────────────────────────┘
```

**证据**：`blackbox_adapter.c:25-73`
```c
WEAK void SystemModuleDump(const char *logDir, struct ErrorInfo *info)
{
    // 默认实现：打印错误信息
    BBOX_PRINT_ERR("Please implement the interface according to the platform!\n");
}
```

### 1.2.2 ErrorInfo 结构体

故障信息载体，记录事件、模块、描述三类信息。

**证据**：`blackbox.h:50-54`
```c
struct ErrorInfo {
    char event[EVENT_MAX_LEN];       // 32字节，事件类型
    char module[MODULE_MAX_LEN];      // 32字节，模块名
    char errorDesc[ERROR_DESC_MAX_LEN]; // 512字节，错误描述
};
```

### 1.2.3 ModuleOps 操作接口

模块向 blackbox 注册的操作集，包含 Dump/Reset/GetLastLogInfo/SaveLastLog 四个回调。

**证据**：`blackbox.h:56-62`
```c
struct ModuleOps {
    char module[MODULE_MAX_LEN];
    void (*Dump)(const char *logDir, struct ErrorInfo *info);
    void (*Reset)(struct ErrorInfo *info);
    int (*GetLastLogInfo)(struct ErrorInfo *info);
    int (*SaveLastLog)(const char *logDir, struct ErrorInfo *info);
};
```

## 1.3 目录结构

```
blackbox_lite/
├── blackbox_adapter.c           # WEAK 适配层实现（平台需重写）
├── blackbox_core.c             # 核心逻辑（不可修改）
├── blackbox_detector.c         # 事件上报（依赖 hiview_lite）
├── blackbox_detector.h         # 探测器头文件
├── interfaces/
│   └── native/
│       ├── innerkits/          # 内部接口（供子系统内使用）
│       │   ├── blackbox.h
│       │   └── blackbox_adapter.h
│       └── kits/               # 外部接口（与 innerkits 内容相同）
│           ├── blackbox.h
│           └── blackbox_adapter.h
├── BUILD.gn                    # GN 构建配置
├── bundle.json                 # 组件配置
└── README_zh.md               # 官方说明文档
```

## 1.4 运行环境

### 1.4.1 系统依赖

| 依赖组件 | 用途 | 来源 |
|----------|------|------|
| utils_lite | 链表、内存安全操作 | 系统基础库 |
| liteos_m | 内核 API (LOS_*) | LiteOS 内核 |
| hilog_lite | 日志打印 (HILOG_DEBUG) | DFX 子系统 |
| hiview_lite | 事件上报 (UploadEventByFile) | DFX 子系统 |

**证据**：`BUILD.gn:25-31`
```gn
include_dirs = [
    "//base/hiviewdfx/blackbox_lite",
    "//base/hiviewdfx/blackbox_lite/interfaces/native/kits",
    "//base/hiviewdfx/hiview_lite",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//commonlibrary/utils_lite/include",
]
```

### 1.4.2 外部符号依赖

以下符号由平台或内核提供：

| 符号 | 用途 | 提供方 |
|------|------|--------|
| LOS_BinarySemCreate | 信号量创建 | liteos_m |
| LOS_SemPend | 信号量等待 | liteos_m |
| LOS_SemPost | 信号量释放 | liteos_m |
| UploadEventByFile | 事件上报 | hiview_lite |

**证据**：`blackbox_adapter.h:42-44`
```c
extern unsigned int LOS_BinarySemCreate(unsigned short count, unsigned int *semHandle);
extern unsigned int LOS_SemPend(unsigned int semHandle, unsigned int timeout);
extern unsigned int LOS_SemPost(unsigned int semHandle);
```

## 1.5 快速上手

### 1.5.1 平台适配步骤

1. 创建 `blackbox_adapter_impl.c` 文件
2. 实现 `blackbox_adapter.c` 中的所有 WEAK 函数
3. 在构建系统中链接适配层实现

### 1.5.2 调用示例

```c
#include "blackbox.h"

// 注册模块操作
struct ModuleOps ops = {
    .module = "MY_MODULE",
    .Dump = MyModuleDump,
    .Reset = MyModuleReset,
    .GetLastLogInfo = MyModuleGetLog,
    .SaveLastLog = MyModuleSaveLog,
};

int ret = BBoxRegisterModuleOps(&ops);

// 触发故障通知
BBoxNotifyError("EVENT_PANIC", "MY_MODULE", "Unexpected error", 1);
```

---

## 参考文档

- [02_Architecture](02_Architecture.md) - 详细架构说明
- [03_API_Reference](03_API_Reference.md) - 接口参考
- [README_zh.md](../../README_zh.md) - 官方 README
