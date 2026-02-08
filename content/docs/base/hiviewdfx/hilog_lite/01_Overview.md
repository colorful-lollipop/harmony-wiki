# Hilog Lite 组件概览

## 项目定位

Hilog_lite 是 OpenHarmony DFX（Debugging, Fault-tolerance, and eXperience）子系统的核心日志组件，提供统一的日志打印、过滤、缓冲和输出功能。

**核心职责**：
- 提供多级别日志输出（DEBUG/INFO/WARN/ERROR/FATAL）
- 支持隐私保护（%{public}/%{private} 标识）
- 日志限流与缓存机制
- 多系统适配（轻量系统/小型系统）

**组件名称**: `@ohos/hilog_lite`
**版本**: 4.0.2
**License**: Apache License 2.0

---

## 适用范围

### 适配系统类型

| 系统类型 | 内核 | 代码位置 | 编程语言 |
|----------|------|----------|----------|
| **轻量系统 (mini)** | LiteOS-M | `frameworks/mini/` | 标准 C |
| **小型系统 (small)** | LiteOS-A | `frameworks/featured/` | C/C++ |
| **JS/ACE Lite 应用** | LiteOS-M | `frameworks/js/` | C++ (JSI 绑定) |

### 资源占用

| 指标 | 大小 |
|------|------|
| ROM | ~500KB |
| RAM | ~500KB |

> 证据来源: bundle.json:45-46

---

## 核心能力

### 1. 日志打印能力

| 能力 | 描述 | 支持系统 |
|------|------|----------|
| 多级别日志 | DEBUG/INFO/WARN/ERROR/FATAL | 轻量 + 小型 |
| 格式化输出 | printf 风格，支持隐私标识 | 轻量 + 小型 |
| 模块化标签 | 64 个预定义模块 ID | 轻量 + 小型 |
| 隐私保护 | %{public}/%{private} 参数标记 | 小型系统 |
| 哈希打印 | 敏感信息哈希输出 | 轻量系统 |

### 2. 日志管理能力

| 能力 | 描述 |
|------|------|
| 日志限流 | 基于级别的流量控制 |
| 日志缓存 | 环形缓冲区管理 |
| 日志落盘 | 应用日志持久化存储 |
| 日志查看 | hilogcat 实时查看工具 |

### 3. 系统集成能力

| 能力 | 描述 |
|------|------|
| SAMgr 集成 | 与系统能力管理器集成 |
| 内核驱动 | 通过 ioctl 与内核日志驱动通信 |
| 文件输出 | 支持日志文件存储与压缩 |

---

## 关键概念

### 日志级别

| 级别 | 值 | 描述 | 轻量系统 | 小型系统 |
|------|-----|------|---------|----------|
| DEBUG | 3 | 调试信息 | ✅ | ✅ |
| INFO | 4 | 一般信息 | ✅ | ✅ |
| WARN | 5 | 警告信息 | ✅ | ✅ |
| ERROR | 6 | 错误信息 | ✅ | ✅ |
| FATAL | 7 | 致命错误 | ✅ | ✅ |

### 模块类型

系统预定义 64 个模块类型，常见模块包括：

| 模块 ID | 模块名 | 用途 |
|---------|--------|------|
| 0 | HIVIEW | DFX 子系统 |
| 1 | SAMGR | 系统能力管理器 |
| 3 | ACE | 轻量级应用框架 |
| 4 | GRAPHIC | 图形子系统 |
| 5 | APP | 第三方应用 |
| ... | ... | ... |

> 证据来源: hiview_log.h:90-131

### 日志类型（小型系统）

| 类型 | 值 | 描述 |
|------|-----|------|
| LOG_TYPE_MIN | 0 | 最小类型 |
| LOG_INIT | 1 | 仅初始化阶段使用 |
| LOG_CORE | 3 | 核心服务/框架使用 |

---

## 接口分类

### 对外接口（Kits）

| 接口类型 | 头文件位置 | 适用系统 |
|----------|------------|----------|
| 轻量系统 API | `interfaces/native/kits/hilog_lite/` | LiteOS-M |
| 小型系统 API | `interfaces/native/kits/hilog/` | LiteOS-A |
| JS 绑定 | `frameworks/js/builtin/` | ACE Lite |

### 内部接口（InnerKits）

| 接口类型 | 头文件位置 | 描述 |
|----------|------------|------|
| C++ 封装 | `interfaces/native/innerkits/hilog/hilog_cp.h` | HiLog 类封装 |
| 追踪 API | `interfaces/native/innerkits/hilog/hilog_trace.h` | 系统追踪 |

---

## 目录结构

```
hilog_lite/
├── interfaces/native/
│   ├── kits/
│   │   ├── hilog/           # 小型系统对外接口
│   │   └── hilog_lite/      # 轻量系统对外接口
│   └── innerkits/
│       └── hilog/           # 内部接口 (C++ 封装)
├── frameworks/
│   ├── mini/                # 轻量系统实现
│   ├── featured/            # 小型系统实现
│   └── js/                  # JS/ACE Lite 实现
├── services/
│   ├── hilogcat/           # 日志查看命令
│   └── apphilogcat/        # 应用日志落盘
├── command/                 # 轻量系统命令实现
└── wiki/                   # 工程文档
```

---

## 依赖关系

### 外部依赖

| 依赖组件 | 用途 | 必需性 |
|----------|------|--------|
| hiview_lite | DFX 基础框架 | 必需 |
| samgr_lite | 系统能力管理 | 必需 |
| utils_lite | 基础工具库 | 必需 |
| ace_engine_lite | ACE 框架 | JS 功能必需 |
| bounds_checking_function | 安全字符串函数 | 必需 |

### 被依赖关系

| 依赖方 | 依赖方式 |
|--------|----------|
| 整个 OpenHarmony 系统 | 日志打印基础设施 |
| DFX 子系统 | hiview、hievent 等组件 |

---

## 配置特性

### 编译时特性

| 特性名 | 描述 | 默认值 |
|--------|------|--------|
| `hilog_lite_disable_privacy_feature` | 禁用隐私保护 | false |
| `hilog_lite_disable_hilog_static` | 禁用静态库 | false |
| `hilog_lite_file_size` | 日志文件大小 | 8192 |
| `hilog_lite_disable_cache` | 禁用日志缓存 | false |
| `hilog_lite_limit_level_default` | 默认限流级别 | 30 |
| `hilog_lite_disable_print_limit` | 禁用打印限流 | false |
| `hilog_lite_disable_js_feature` | 禁用 JS 功能 | false |

> 证据来源: bundle.json:24-43

---

## 快速入门

### 轻量系统使用示例

```c
#include "log.h"

// 1. 定义模块（通常在模块初始化时）
extern int HiLogRegisterModule(uint16 id, const char *name);

// 2. 打印日志
HILOG_INFO(HILOG_MODULE_APP, "User login success, userId: %d", userId);
```

### 小型系统使用示例

```c
#include <hilog/log.h>

// 1. 定义领域和标签
#define LOG_DOMAIN 0x0D00
#define LOG_TAG "MyTag"

// 2. 打印日志
HILOG_INFO(LOG_APP, "Hello World");
// 带隐私保护
HILOG_WARN(LOG_APP, "Password: %{private}s", password);
```

### JS/ACE Lite 使用示例

```javascript
import hilog from '@ohos.hilog';

hilog.info(0, 'MyTag', 'Hello World');
hilog.isLoggable(0, 'MyTag', hilog.LogLevel.INFO);
```

---

## 相关文档

- [架构设计](02_Architecture.md)
- [Native API](03_Native_API.md)
- [GN 构建 Targets](05_GN_Targets.md)
- [安全评审](07_Security_Review.md)
