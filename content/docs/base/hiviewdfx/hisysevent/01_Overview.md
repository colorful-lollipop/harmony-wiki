# 项目概览

## 1.1 项目定位

### 一句话定义

**HiSysEvent**（Hi System Event）是 OpenHarmony DFX（Debug、Fault-tolerance、X-trace）子系统的核心事件日志服务，提供多语言编程接口（Native C/C++、C、Rust、JavaScript/ArkTS）用于记录系统关键进程运行事件，支持故障定位、性能分析和云端大数据处理。

### 核心能力

| 能力 | 说明 | 支持语言 |
|------|------|----------|
| **事件写入** | 记录系统运行事件（故障、统计、安全、行为） | C++、C、Rust、JS、ArkTS |
| **事件订阅** | 实时监听符合规则的系统事件 | C++、Rust |
| **事件查询** | 按条件查询历史事件记录 | C++、Rust、JS、ArkTS |
| **事件导出** | 将事件数据导出到文件进行分析 | C++ |
| **云端上报** | 支持事件数据上传至云端进行大数据分析 | 系统内置支持 |

### 项目边界

```
┌─────────────────────────────────────────────────────────────┐
│                      HiSysEvent 边界                        │
├─────────────────────────────────────────────────────────────┤
│  ✅ 提供的事件记录能力                                       │
│     - 事件定义（Domain + EventName）                        │
│     - 事件类型（FAULT/STATISTIC/SECURITY/BEHAVIOR）        │
│     - 事件参数（键值对形式，最多 128 个）                    │
│     - 事件传输（IPC + Socket）                              │
│                                                             │
│  ✅ 支持的编程接口                                           │
│     - Native C++ API（面向系统服务）                        │
│     - Native C API（面向 C 语言模块）                       │
│     - Rust FFI API（面向 Rust 子系统）                     │
│     - N-API（面向 JavaScript/ArkTS 应用）                  │
│     - ANI（面向 ArkTS Native Interface）                   │
│                                                             │
│  ❌ 不提供的能力                                             │
│     - 事件持久化存储（由 storage_service 提供）             │
│     - 事件解析与分析（由上层应用/云端处理）                  │
│     - 事件告警通知（由其他系统组件实现）                    │
└─────────────────────────────────────────────────────────────┘
```

### 代码位置

| 项目 | 路径 |
|------|------|
| **源码根目录** | `//base/hiviewdfx/hisysevent` |
| **适配层** | `//base/hiviewdfx/hisysevent/adapter` |
| **框架层** | `//base/hiviewdfx/hisysevent/frameworks` |
| **接口层** | `//base/hiviewdfx/hisysevent/interfaces` |

---

## 1.2 核心概念

### 事件域（Domain）

事件域用于标识事件所属的系统模块，HiSysEvent 预定义了 150+ 系统域，同时支持自定义域。

**预定义域示例**（`hisysevent.h:79-251`）：

```cpp
namespace HiSysEvent {
class Domain {
public:
    static constexpr const char* AAFWK = "AAFWK";      // 能力框架
    static constexpr const char* ACTS = "ACTS";        // 分布式调度
    static constexpr const char* APPEXECFWK = "APPEXECFWK"; // 应用框架
    static constexpr const char* BARRIER_FREE = "BARRIER_FREE"; // 无障碍
    static constexpr const char* BMS = "BMS";          // 包管理
    static constexpr const char* DISTRIBUTED_DATA = "DISTRIBUTED_DATA"; // 分布式数据
    // ... 更多预定义域
};
}
```

**自定义域约束**（`README.md:52`）：

| 约束 | 要求 |
|------|------|
| 最大长度 | 16 字符 |
| 字符集 | 数字（0-9）、大小写字母（a-z、A-Z）、下划线（_） |
| 首字符 | 必须为字母 |

### 事件类型（EventType）

| 枚举值 | 值 | 说明 | 典型用途 |
|--------|-----|------|----------|
| `FAULT` | 1 | 系统故障事件 | 进程崩溃、ANR、系统错误 |
| `STATISTIC` | 2 | 系统统计事件 | 性能指标、资源使用统计 |
| `SECURITY` | 3 | 安全相关事件 | 认证失败、权限变更、敏感操作 |
| `BEHAVIOR` | 4 | 系统行为事件 | 应用启动、页面跳转、用户操作 |

### 事件参数（Key-Values）

事件参数以键值对形式传递附加信息。

**参数约束**（`README.md:52`）：

| 约束 | 要求 |
|------|------|
| 单个参数值最大长度 | 48 字符 |
| 参数数量上限 | 128 个 |
| 字符集 | 数字（0-9）、大小写字母（a-z、A-Z）、下划线（_） |
| 首字符 | 必须为字母 |
| 支持类型 | 基本数据类型、`std::string`、`std::vector<基本类型>`、`std::vector<std::string>` |

---

## 1.3 系统能力声明

### SysCap 标识

HiSysEvent 通过 System Capability（系统能力）声明对外提供服务能力。

**证据来源**：`bundle.json:15-17`

```json
"syscap": [
  "SystemCapability.HiviewDFX.HiSysEvent"
]
```

**使用要求**：
- 开发者应用需要声明 `SystemCapability.HiviewDFX.HiSysEvent` 能力
- 系统服务默认具有该能力（无需额外声明）

### Feature 开关

| Feature 名称 | 默认值 | 作用 | 证据来源 |
|--------------|--------|------|----------|
| `hisysevent_feature_support_usr_symlink` | 待确认 | 支持用户空间符号链接 | `bundle.json:19` |

---

## 1.4 运行环境

### 依赖的系统服务

| 服务名 | 用途 | 依赖类型 |
|--------|------|----------|
| **samgr** | 系统服务注册与发现 | 强依赖 |
| **safwk** | System Ability 框架 | 强依赖 |
| **ipc** | 进程间通信机制 | 强依赖 |
| **access_token** | 权限管理服务 | 强依赖 |
| **storage_service** | 持久化存储服务 | 强依赖 |
| **hilog** | 系统日志服务 | 强依赖 |
| **hitrace** | 性能追踪服务 | 弱依赖 |
| **jsoncpp** | JSON 序列化库 | 强依赖 |

**证据来源**：`bundle.json:26-40`

### 运行特权要求

| 特权 | 说明 | 证据来源 |
|------|------|----------|
| `ohos.permission.ACCESS_SYSTEM_SERVICE` | 访问系统服务权限 | `def.h` |

### 支持的系统类型

| 系统类型 | 支持状态 | 证据来源 |
|----------|----------|----------|
| **standard** | ✅ 支持 | `bundle.json:22-23` |
| **lite** | ❌ 不支持 | - |
| **mini** | ❌ 不支持 | - |

---

## 1.5 快速上手

### C++ 集成示例

**步骤 1：添加头文件**

```cpp
#include "hisysevent.h"
```

**步骤 2：添加编译依赖**

```gn
# BUILD.gn
external_deps = [ "hisysevent:libhisysevent" ]
```

**步骤 3：写入事件**

```cpp
#include "hisysevent.h"

void LogAppStart(const std::string& appName) {
    HiSysEvent::Write(
        HiSysEvent::Domain::APPEXECFWK,
        "start_app",
        HiSysEvent::EventType::BEHAVIOR,
        "app_name", appName.c_str(),
        "pid", getpid()
    );
}
```

### JavaScript/ArkTS 集成示例

```typescript
import hiSysEvent from '@ohos.hiSysEvent';

@Entry
@Component
struct Index {
  build() {
    Button('Start Event')
      .onClick(() => {
        hiSysEvent.write({
          domain: 'APPEXECFWK',
          name: 'start_app',
          type: hiSysEvent.EventType.BEHAVIOR,
          params: {
            app_name: 'com.example.app',
            pid: 12345
          }
        });
      })
  }
}
```

### Rust 集成示例

```rust
use hisysevent::{write, EventType};

fn log_app_start(app_name: &str) {
    write(
        "APPEXECFWK",
        "start_app",
        EventType::BEHAVIOR,
        &[("app_name", app_name)],
    ).expect("Failed to write HiSysEvent");
}
```

---

## 1.6 常见使用场景

### 场景一：应用启动监控

```cpp
// 记录应用启动事件
HiSysEvent::Write(
    HiSysEvent::Domain::APPEXECFWK,
    "app_start",
    HiSysEvent::EventType::BEHAVIOR,
    "app_name", appName,
    "bundle_path", bundlePath,
    "start_time", GetCurrentTimeMs()
);
```

### 场景二：故障事件上报

```cpp
// 记录进程崩溃事件
HiSysEvent::Write(
    HiSysEvent::Domain::AAFWK,
    "process_crash",
    HiSysEvent::EventType::FAULT,
    "pid", pid,
    "reason", crashReason,
    "stack_trace", stackTrace
);
```

### 场景三：安全审计

```cpp
// 记录敏感操作事件
HiSysEvent::Write(
    HiSysEvent::Domain::SECURITY,
    "sensitive_access",
    HiSysEvent::EventType::SECURITY,
    "operation", "file_access",
    "resource", filePath,
    "result", "success"
);
```

---

## 1.7 版本信息

| 版本 | 发布日期 | 主要变更 |
|------|----------|----------|
| 3.1 | 待确认 | 当前版本 |
| 3.0 | - | 初始版本，支持多语言接口 |

---

## 1.8 相关资源

### 代码资源

| 资源 | 路径 |
|------|------|
| **C++ API 头文件** | `interfaces/native/innerkits/hisysevent/include/hisysevent.h` |
| **C API 头文件** | `interfaces/native/innerkits/hisysevent/include/hisysevent_c.h` |
| **Easy C API** | `interfaces/native/innerkits/hisysevent_easy/include/hisysevent_easy.h` |
| **N-API 实现** | `interfaces/js/kits/napi/src/napi_hisysevent_js.cpp` |
| **Rust API** | `interfaces/rust/innerkits/src/lib.rs` |
| **IPC 接口定义** | `adapter/native/idl/ISysEventService.idl` |

### 文档资源

| 资源 | 链接 |
|------|------|
| **OpenHarmony DFX 子系统** | [官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md) |
| **N-API 开发指南** | [官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/Readme-CN.md) |
| **权限管理** | [access_token 文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/Access_Token.md) |

---

*文档版本：1.0*
*创建时间：2026-02-07*
*最后更新：2026-02-07*
