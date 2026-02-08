# Hilog Lite 内部模块与 API

本文档描述 hilog_lite 组件的内部模块结构、依赖关系和内部 API。

## 模块结构

```
frameworks/
├── mini/                          # 轻量系统实现
│   ├── hiview_log.c              # 日志核心
│   ├── hiview_log_limit.c        # 限流模块
│   ├── hiview_output_log.c       # 输出模块
│   ├── hiview_log_limit.h        # 限流头文件
│   ├── hiview_output_log.h       # 输出头文件
│   └── BUILD.gn                  # 构建配置
│
├── featured/                     # 小型系统实现
│   ├── hiview_log.c              # 日志核心
│   ├── hilog.cpp                 # C++ 封装
│   └── BUILD.gn                  # 构建配置
│
└── js/                          # JS/ACE Lite 实现
    └── builtin/
        ├── include/
        │   ├── hilog_module.h    # 模块接口
        │   ├── hilog_string.h   # 字符串处理
        │   ├── hilog_vector.h    # 向量容器
        │   ├── hilog_realloc.h   # 内存管理
        │   └── hilog_wrapper.h   # 宏包装
        ├── src/
        │   ├── hilog_module.cpp  # 模块实现
        │   ├── hilog_string.cpp
        │   ├── hilog_vector.cpp
        │   └── hilog_realloc.cpp
        └── BUILD.gn
```

---

## 轻量系统模块详情

### hiview_log.c

**职责**: 日志核心功能

**关键函数**:

| 函数 | 行号 | 描述 |
|------|------|------|
| `HiLogInit()` | 36 | 模块初始化 |
| `HiLogRegisterModule()` | 79 | 模块注册 |
| `HiLogPrintf()` | 111 | 日志输出 |
| `HiLogGetModuleName()` | 102 | 获取模块名 |
| `HILOG_HashPrintf()` | 148 | 哈希日志输出 |

**全局变量**:

| 变量 | 类型 | 描述 |
|------|------|------|
| `g_logModuleInfo` | `HiLogModuleInfo[HILOG_MODULE_MAX]` | 模块信息表 |
| `g_hiviewConfig` | `HiViewConfig` | 全局配置 |

> 证据来源: hiview_log.c:30-31

### hiview_log_limit.c

**职责**: 日志限流控制

**关键函数**:

| 函数 | 描述 |
|------|------|
| `InitLogLimit()` | 初始化限流配置 |
| `CheckLogLimit()` | 检查日志是否被限流 |

> 相关配置: `hilog_lite_limit_level_default`, `hilog_lite_disable_print_limit`

### hiview_output_log.c

**职责**: 日志输出管理

**关键函数**:

| 函数 | 描述 |
|------|------|
| `InitCoreLogOutput()` | 初始化核心日志输出 |
| `OutputLog()` | 输出日志到目标 |

---

## 小型系统模块详情

### hiview_log.c

**职责**: 小型系统日志核心实现

**特点**: 
- 支持更多日志类型 (LOG_INIT, LOG_CORE, LOG_APP)
- 支持 ioctl 与内核驱动通信
- 支持日志缓冲区

**关键常量**:

```c
#define LOG_BUF_SIZE (1024)
#define MAX_DOMAIN_TAG_SIZE 64
#define IOV_SIZE (3)
#define HILOG_IOV_SIZE 4
```

> 证据来源: frameworks/featured/hiview_log.c:99-102

### hilog.cpp

**职责**: C++ 封装层

**关键实现**:

```cpp
// HiLog 类方法实现
int HiLog::Debug(const HiLogLabel &label, const char *fmt, ...)
{
    // 调用 HiLogPrint
}
```

---

## JS/ACE Lite 模块详情

### hilog_module.cpp

**职责**: JS 模块绑定

**关键类**:

| 类 | 描述 |
|------|------|
| `HilogModule` | 日志模块类 |

**关键方法**:

| 方法 | 描述 |
|------|------|
| `HilogModule::Debug()` | DEBUG 日志 |
| `HilogModule::Info()` | INFO 日志 |
| `HilogModule::Error()` | ERROR 日志 |
| `HilogModule::Warn()` | WARN 日志 |
| `HilogModule::Fatal()` | FATAL 日志 |
| `HilogModule::IsLoggable()` | 检查日志级别 |

**内部方法**:

| 方法 | 描述 |
|------|------|
| `HilogModule::HilogImpl()` | 通用日志实现 |
| `HilogModule::ParseLogContent()` | 解析日志内容 |
| `HilogModule::ParseNapiValue()` | 解析 N-API 值 |

> 证据来源: hilog_module.cpp:44-316

### 辅助模块

| 模块 | 文件 | 描述 |
|------|------|------|
| HilogString | hilog_string.cpp | 字符串处理 |
| HilogVector | hilog_vector.cpp | 向量容器 |
| HilogRealloc | hilog_realloc.cpp | 内存分配 |

---

## 内部数据结构

### HiLogModuleInfo

```c
typedef struct {
    const char *name;  // 模块名称
    uint16 id;         // 模块 ID
} HiLogModuleInfo;
```

> 证据来源: hiview_log.h:144-150

### HiLogContent

```c
#pragma pack(1)
typedef struct {
    uint8 head;           // 日志头
    uint8 module;        // 模块 ID
    uint8 level : 4;     // 日志级别
    uint8 valueNumber : 4;// 参数数量
    uint8 task;          // 任务 ID
    uint32 time;         // 时间戳 (秒)
    uint16 milli;        // 毫秒
} HiLogCommon;
```

> 证据来源: hiview_log.h:143-153

### HiLogLabel (C++)

```cpp
struct HiLogLabel {
    LogType type;
    unsigned int domain;
    const char *tag;
};
```

> 证据来源: hilog_cp.h:25-29

---

## 依赖方向

```
                    ┌─────────────────────┐
                    │     应用层           │
                    │  (C/C++/JS App)     │
                    └──────────┬──────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    接口层 (Kits)                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │ kits/hilog/  │  │kits/hilog_lite│  │ frameworks/js/   │   │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘   │
└─────────┼──────────────────┼───────────────────┼─────────────┘
          │                  │                   │
          ▼                  ▼                   ▼
┌──────────────────────────────────────────────────────────────┐
│                    框架层 (Frameworks)                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              mini / featured                         │    │
│  │  hiview_log.c → hiview_log_limit.c → output_*.c    │    │
│  └─────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    服务层 (Services)                          │
│  ┌──────────────────┐  ┌──────────────────────────────────┐ │
│  │ hilogcat         │  │ apphilogcat                      │ │
│  │ (日志查看)       │  │ (日志落盘)                       │ │
│  └──────────────────┘  └──────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    系统层                                     │
│  Kernel Driver (ioctl)  │  UART/Console  │  File System      │
└──────────────────────────────────────────────────────────────┘
```

---

## 稳定性标注

### 稳定接口

| 接口 | 位置 | 稳定性 |
|------|------|--------|
| `HiLogPrint()` | log.h | 稳定 (v1.0+) |
| `HILOG_*` 宏 | log.h | 稳定 (v1.0+) |
| `HiLog` 类 | hilog_cp.h | 稳定 |

### 内部接口

| 接口 | 位置 | 稳定性 |
|------|------|--------|
| `HiLogInit()` | hiview_log.c:36 | 内部使用 |
| `HiLogRegisterModule()` | hiview_log.c:79 | 内部使用 |
| `OutputLog()` | hiview_output_log.c | 内部使用 |

---

## 资源生命周期

### 模块初始化顺序

1. **HiLogInit()** (编译时常量优先级)
   - 注册预定义模块
   - 初始化输出模块
   - 初始化限流模块

> 证据来源: hiview_log.c:36-64

```c
#ifndef DISABLE_HILOG_LITE_CORE_INIT
CORE_INIT_PRI(HiLogInit, 0);
#endif
```

### 日志输出流程

```
HiLogPrint/HILOG_* macro
        │
        ▼
CheckParameters (级别/模块校验)
        │
        ▼
CheckLogLimit (限流检查)
        │
        ▼
FormatLogContent (格式化)
        │
        ▼
OutputLog (输出到目标)
        │
        ├──▶ Kernel Driver (ioctl)
        ├──▶ UART/Console
        └──▶ Ring Buffer
```

---

## 相关文档

- [概览](01_Overview.md)
- [架构设计](02_Architecture.md)
- [Native API](03_Native_API.md)
- [GN Targets](05_GN_Targets.md)
