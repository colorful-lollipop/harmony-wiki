# Hilog Lite Native API 参考

本文档描述 hilog_lite 组件对外提供的 Native API 接口。

## API 分类概览

| 系统类型 | 入口头文件 | 编程语言 |
|----------|------------|----------|
| 轻量系统 | `interfaces/native/kits/hilog_lite/hiview_log.h` | 标准 C |
| 小型系统 | `interfaces/native/kits/hilog/log.h` | C |
| JS/ACE Lite | `frameworks/js/builtin/include/` | JavaScript |

---

## 轻量系统 API (LiteOS-M)

### 头文件

```c
#include <log.h>
// 或
#include "log.h"
```

> 证据来源: interfaces/native/kits/hilog_lite/log.h:19

### 日志级别

```c
#define HILOG_LV_INVALID 0
#define HILOG_LV_DEBUG   1
#define HILOG_LV_INFO    2
#define HILOG_LV_WARN    3
#define HILOG_LV_ERROR   4
#define HILOG_LV_FATAL   5
#define HILOG_LV_MAX     6
```

> 证据来源: hiview_log.h:71-77

### 模块类型

```c
typedef enum {
    HILOG_MODULE_HIVIEW = 0,    // DFX
    HILOG_MODULE_SAMGR,         // System Ability Manager
    HILOG_MODULE_UPDATE,         // Update
    HILOG_MODULE_ACE,           // Ability Cross-platform Environment
    HILOG_MODULE_APP,           // Third-party applications
    HILOG_MODULE_AAFWK,         // Atomic Ability Framework
    HILOG_MODULE_GRAPHIC,       // Graphic
    HILOG_MODULE_MEDIA,         // Multimedia
    // ... 更多模块
    HILOG_MODULE_MAX = 64       // 最大模块数
} HiLogModuleType;
```

> 证据来源: hiview_log.h:90-131

### 核心 API

#### HiLogRegisterModule

```c
/**
 * @brief 注册日志模块
 *
 * @param id 模块 ID (0-63)
 * @param name 模块名称 (仅支持英文字母)
 * @return 注册成功返回 TRUE，失败返回 FALSE
 */
boolean HiLogRegisterModule(uint16 id, const char *name);
```

**参数校验** (hiview_log.c:81-95):
- `id` 必须小于 `HILOG_MODULE_MAX`
- `name` 不能为 NULL
- `name` 长度不超过 `LOG_MODULE_NAME_LEN - 1`
- `name` 仅支持大小写英文字母

> 证据来源: hiview_log.c:79-100

#### HiLogPrintf

```c
/**
 * @brief 输出日志
 *
 * @param module 模块 ID
 * @param level 日志级别
 * @param nums 参数数量标记 (使用 FUN_ARG_NUM 宏)
 * @param fmt 格式字符串
 * @param ... 可变参数
 */
void HiLogPrintf(uint8 module, uint8 level, const char *nums, const char *fmt, ...);
```

**参数校验** (hiview_log.c:121-124):
- 日志开关必须开启 (`logSwitch == HIVIEW_FEATURE_ON`)
- 参数必须有效 (`CheckParameters`)
- 模块必须已注册且在输出配置中

**限制**:
- 最多支持 6 个可变参数 (`LOG_MULTI_PARA_MAX`)
- 不支持 `%s` 格式说明符
- 单条日志最大长度 128 字节

> 证据来源: hiview_log.c:111-146

#### HiLogGetModuleName

```c
/**
 * @brief 获取模块名称
 *
 * @param id 模块 ID
 * @return 模块名称字符串，ID 无效返回空字符串
 */
const char *HiLogGetModuleName(uint8 id);
```

> 证据来源: hiview_log.c:102-109

### 日志宏

#### HILOG_DEBUG / INFO / WARN / ERROR / FATAL

```c
HILOG_DEBUG(mod, fmt, ...)
HILOG_INFO(mod, fmt, ...)
HILOG_WARN(mod, fmt, ...)
HILOG_ERROR(mod, fmt, ...)
HILOG_FATAL(mod, fmt, ...)
```

**使用示例**:

```c
// 包含头文件
#include "log.h"

// 使用日志宏
HILOG_INFO(HILOG_MODULE_APP, "User login success, userId: %d", userId);
HILOG_ERROR(HILOG_MODULE_HIVIEW, "Failed to open file: %{public}s", filePath);
```

> 证据来源: hiview_log.h:314-404

#### HILOG_*_HASH 变体

```c
HILOG_DEBUG_HASH(mod, hash, ...)
HILOG_INFO_HASH(mod, hash, ...)
HILOG_WARN_HASH(mod, hash, ...)
HILOG_ERROR_HASH(mod, hash, ...)
HILOG_FATAL_HASH(mod, hash, ...)
```

用于输出哈希值而非敏感数据。

> 证据来源: hiview_log.h:426-489

---

## 小型系统 API (LiteOS-A)

### 头文件

```c
#include <hilog/log.h>
```

> 证据来源: log.h:32

### 日志类型

```c
typedef enum {
    LOG_TYPE_MIN = 0,   // 最小类型
    LOG_INIT = 1,       // 仅初始化阶段使用
    LOG_CORE = 3,       // 核心服务/框架使用
    LOG_TYPE_MAX
} LogType;
```

> 证据来源: log.h:128-135

### 日志级别

```c
typedef enum {
    LOG_DEBUG = 3,
    LOG_INFO = 4,
    LOG_WARN = 5,
    LOG_ERROR = 6,
    LOG_FATAL = 7,
} LogLevel;
```

> 证据来源: log.h:154-165

### 模块类型

```c
typedef enum {
    HILOG_MODULE_HIVIEW = 0,  // DFX
    HILOG_MODULE_SAMGR,      // System Ability Manager
    HILOG_MODULE_ACE,        // GRAPHIC
    HILOG_MODULE_GRAPHIC,    // GRAPHIC
    HILOG_MODULE_APP,        // Third-party applications
    HILOG_MODULE_MAX
} HiLogModuleType;
```

> 证据来源: log.h:78-91

### 核心 API

#### HiLogPrint

```c
/**
 * @brief 输出日志
 *
 * @param type 日志类型 (LOG_INIT/LOG_CORE/LOG_APP)
 * @param level 日志级别 (LOG_DEBUG/INFO/WARN/ERROR/FATAL)
 * @param domain 服务域 (0x0 - 0xFFFFF)
 * @param tag 日志标签
 * @param fmt 格式字符串
 * @param ... 可变参数
 * @return 成功返回 >=0，失败返回 <0
 */
int HiLogPrint(LogType type, LogLevel level, unsigned int domain,
               const char* tag, const char* fmt, ...);
```

**格式字符串**:
- 支持 printf 风格格式化
- 支持隐私标识: `%{public}` / `%{private}`
- 参数类型: `%d`, `%s`, `%x`, `%f` 等

**使用示例**:

```c
#include <hilog/log.h>

// 定义域和标签
#define LOG_DOMAIN 0x00201  // 子系统:模块 = 002:01
#define LOG_TAG "MY_TAG"

// 输出日志
HILOG_WARN(LOG_APP, "Failed to visit %{private}s, reason:%{public}d.", url, errno);
```

> 证据来源: log.h:191-296

### 日志宏

```c
HILOG_DEBUG(type, ...)
HILOG_INFO(type, ...)
HILOG_WARN(type, ...)
HILOG_ERROR(type, ...)
HILOG_FATAL(type, ...)
```

**使用示例**:

```c
// 调试日志 (通常在 Release 版禁用)
HILOG_DEBUG(LOG_APP, "Variable value: %d", value);

// 信息日志
HILOG_INFO(LOG_CORE, "Service started successfully");

// 警告日志
HILOG_WARN(LOG_APP, "Disk space low: %{public}d%%", percent);

// 错误日志
HILOG_ERROR(LOG_CORE, "Failed to open file: %{public}s", filename);

// 致命错误
HILOG_FATAL(LOG_APP, "Critical error occurred");
```

> 证据来源: log.h:216-296

### C++ 封装类

#### HiLogLabel

```cpp
struct HiLogLabel {
    LogType type;
    unsigned int domain;
    const char *tag;
};
```

> 证据来源: hilog_cp.h:25-29

#### HiLog 类

```cpp
namespace OHOS {
namespace HiviewDFX {
class HiLog final {
public:
    static int Debug(const HiLogLabel &label, const char *fmt, ...)
        __attribute__((__format__(printf, 2, 3)));
    static int Info(const HiLogLabel &label, const char *fmt, ...)
        __attribute__((__format__(printf, 2, 3)));
    static int Warn(const HiLogLabel &label, const char *fmt, ...)
        __attribute__((__format__(printf, 2, 3)));
    static int Error(const HiLogLabel &label, const char *fmt, ...)
        __attribute__((__format__(printf, 2, 3)));
    static int Fatal(const HiLogLabel &label, const char *fmt, ...)
        __attribute__((__format__(printf, 2, 3)));
};
} // namespace HiviewDFX
} // namespace OHOS
```

**使用示例**:

```cpp
#include <hilog/hilog_cp.h>

using namespace OHOS::HiviewDFX;

static constexpr HiLogLabel LABEL = { LOG_CORE, 0x00201, "MY_TAG" };

HiLog::Info(LABEL, "Hello World");
HiLog::Warn(LABEL, "Warning: %{private}s", sensitiveData);
```

> 证据来源: hilog_cp.h:31-108

---

## JS/ACE Lite API

### 导入方式

```javascript
import hilog from '@ohos.hilog';
```

### API 列表

| 方法 | 描述 | 参数 |
|------|------|------|
| `debug(domain, tag, fmt, ...)` | DEBUG 级别日志 | domain: number, tag: string, ... |
| `info(domain, tag, fmt, ...)` | INFO 级别日志 | domain: number, tag: string, ... |
| `warn(domain, tag, fmt, ...)` | WARN 级别日志 | domain: number, tag: string, ... |
| `error(domain, tag, fmt, ...)` | ERROR 级别日志 | domain: number, tag: string, ... |
| `fatal(domain, tag, fmt, ...)` | FATAL 级别日志 | domain: number, tag: string, ... |
| `isLoggable(domain, tag, level)` | 检查日志级别 | domain: number, tag: string, level: number |

### 日志级别常量

```javascript
hilog.LogLevel.DEBUG  // 3
hilog.LogLevel.INFO   // 4
hilog.LogLevel.WARN   // 5
hilog.LogLevel.ERROR  // 6
hilog.LogLevel.FATAL  // 7
```

### 使用示例

```javascript
import hilog from '@ohos.hilog';

// 输出日志
hilog.info(0x00201, 'MyApp', 'Application started');
hilog.warn(0x00201, 'MyApp', 'Warning: %{private}s', sensitiveInfo);

// 检查日志级别
if (hilog.isLoggable(0x00201, 'MyApp', hilog.LogLevel.INFO)) {
    hilog.info(0x00201, 'MyApp', 'Debug information');
}
```

### 注册入口

```cpp
// frameworks/js/builtin/src/hilog_module.cpp
void InitHilogModule(JSIValue exports)
{
    InitLogLevelType(exports);
    JSI::SetModuleAPI(exports, "debug", HilogModule::Debug);
    JSI::SetModuleAPI(exports, "info", HilogModule::Info);
    JSI::SetModuleAPI(exports, "error", HilogModule::Error);
    JSI::SetModuleAPI(exports, "warn", HilogModule::Warn);
    JSI::SetModuleAPI(exports, "fatal", HilogModule::Fatal);
    JSI::SetModuleAPI(exports, "isLoggable", HilogModule::IsLoggable);
}
```

> 证据来源: hilog_module.cpp:329-342

---

## 参数校验规则

### 通用校验

| 参数 | 校验规则 |
|------|----------|
| 模块 ID | 必须在 0 到 HILOG_MODULE_MAX-1 之间 |
| 日志级别 | 必须在有效范围内 (DEBUG-FATAL) |
| 域值 (domain) | 0x0 - 0xFFFFF |
| 标签 (tag) | 最大长度 32 字节 |
| 格式字符串 | 最大长度 1024 字节 |

### 隐私标识处理

| 标识 | 行为 |
|------|------|
| `%{private}` | 参数值替换为 `<private>` |
| `%{public}` | 参数值保持原样显示 |
| 无标识 | 参数默认隐私保护 |

---

## 错误码

| 错误码 | 含义 |
|--------|------|
| >= 0 | 成功 |
| < 0 | 失败 |

---

## 相关文档

- [概览](01_Overview.md)
- [架构设计](02_Architecture.md)
- [内部模块](04_Internal_API.md)
- [配置项](appendix/Config_Flags.md)
