# 目录结构

> 源码组织与模块职责

---

## 顶层目录

```
/base/time/time_service/
├── bundle.json              # 项目元数据与构建配置
├── time.gni                 # GN 根配置（路径、feature flags）
├── README.md                # 项目文档（英文）
├── README_zh.md             # 项目文档（中文）
├── LICENSE                  # Apache 2.0
├── figures/                 # 架构图
├── framework/               # 多语言接口层
├── interfaces/              # 接口定义
├── services/                # 服务实现
├── utils/                   # 工具模块
└── test/                    # 测试（本文档不覆盖）
```

---

## framework/ - 多语言接口层

提供不同编程语言的 API 绑定。

### framework/js/napi/ - N-API 接口

**路径**：`framework/js/napi/`

| 目录 | 模块名 | 对应 JS 命名空间 |
|------|--------|------------------|
| `system_time/` | systemtime | `@ohos.systemTime` |
| `system_timer/` | systemtimer | `@ohos.systemTimer` |
| `system_date_time/` | systemdatetime | `@ohos.systemDateTime` |
| `common/` | - | NAPI 工具函数 |

**关键文件**：
- `system_time/src/js_systemtime.cpp:456` - NAPI 模块注册
- `system_timer/src/timer_init.cpp:54` - NAPI 模块注册
- `common/src/napi_utils.cpp` - 参数解析工具

### framework/js/taihe/ - Taihe 接口

**路径**：`framework/js/taihe/`

新框架接口（下一代 JS 绑定）：
- `system_datetime/` - 日期时间 Taihe 绑定
- `system_timer/` - 定时器 Taihe 绑定

### framework/cj/ - Cangjie FFI

**路径**：`framework/cj/`

Cangjie 编程语言 FFI 接口：
- `include/` - 头文件
- `src/` - 实现

### framework/js/ani/ - ANI 接口

**路径**：`framework/js/ani/`

ArkTS Native Interface：
- `ets/` - ArkTS 定义
- `src/` - 原生实现

---

## interfaces/ - 接口定义

### interfaces/inner_api/ - 内部 C++ API

**路径**：`interfaces/inner_api/`

**关键文件**：
| 文件 | 说明 |
|------|------|
| `include/time_service_client.h` | TimeServiceClient 类（客户端入口） |
| `include/itimer_info.h` | ITimerInfo 接口（定时器配置） |
| `src/time_service_client.cpp` | 客户端实现 |

**使用方式**：
```cpp
#include "time_service_client.h"
auto client = TimeServiceClient::GetInstance();
client->SetTime(1234567890);
```

### interfaces/kits/c/ - C NDK 接口

**路径**：`interfaces/kits/c/`

C 语言 NDK 接口：
- `include/time_service.h` - C API 头文件
- `src/time_service_capi.cpp` - C API 实现

---

## services/ - 服务实现

### services/time_system_ability.cpp - 主服务

**路径**：`services/time_system_ability.cpp`

SystemAbility 实现：
- SA_ID: 3702（定义在 `profile/3702.json`）
- 继承 `SystemAbility` + `TimeServiceStub`
- 实现 `OnStart()`, `OnStop()`, `SetTime()`, `CreateTimer()` 等

### services/time/ - 时间功能

**路径**：`services/time/`

| 文件 | 职责 |
|------|------|
| `src/ntp_update_time.cpp` | NTP 时间同步 |
| `src/time_zone_info.cpp` | 时区管理 |
| `src/time_tick_notify.cpp` | 时间滴答通知 |
| `src/event_manager.cpp` | 系统事件订阅 |

### services/timer/ - 定时器功能

**路径**：`services/timer/`

| 文件 | 职责 |
|------|------|
| `src/timer_manager.cpp` | 定时器核心管理 |
| `src/timer_handler.cpp` | 底层定时器硬件抽象 |
| `src/timer_proxy.cpp` | 定时器代理（后台冻结） |
| `src/batch.cpp` | 定时器批处理 |
| `src/timer_database.cpp` | 定时器持久化（RDB） |
| `src/cjson_helper.cpp` | JSON 配置解析 |

### services/etc/ - 配置文件

**路径**：`services/etc/`

| 文件 | 说明 |
|------|------|
| `init/timeservice.cfg` | 服务启动配置 |
| `time.para` | 系统参数 |
| `time.para.dac` | 权限配置 |

### services/profile/ - SA 配置

**路径**：`services/profile/`

| 文件 | 说明 |
|------|------|
| `3702.json` | TimeService SA 配置 |
| `BUILD.gn` | 构建配置 |

### services/dfx/ - 诊断功能

**路径**：`services/dfx/`

- `src/time_sysevent.cpp` - 系统事件上报
- `src/time_cmd_parse.cpp` - HIDumper 命令解析
- `src/time_cmd_dispatcher.cpp` - HIDumper 命令分发

---

## utils/ - 工具模块

**路径**：`utils/`

| 文件 | 职责 |
|------|------|
| `native/include/time_common.h` | 常量定义 |
| `native/include/time_file_utils.h` | 文件操作工具 |
| `native/include/time_xcollie.h` | HiCollie 监控 |

---

## 模块依赖关系

```
                    +------------------+
                    |   Applications   |
                    +--------+---------+
                             |
        +--------------------+--------------------+
        |                    |                    |
        v                    v                    v
+----------------+  +------------------+  +----------------+
|  JS/TS (NAPI)  |  |  C API (NDK)     |  |  Inner API     |
|  framework/js  |  |  interfaces/kits |  |  time_client   |
+--------+-------+  +---------+--------+  +--------+-------+
         |                    |                    |
         +--------------------+--------------------+
                              |
                              v
                    +------------------+
                    | IPC (Binder)     |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    | TimeSystemAbility|
                    | (SA_ID: 3702)    |
                    +--------+---------+
                             |
              +--------------+--------------+
              |                             |
              v                             v
     +------------------+         +------------------+
     |   TimerManager   |         |   TimeZoneInfo   |
     |   (定时器管理)    |         |   (时区管理)      |
     +--------+---------+         +------------------+
              |
     +--------+--------+
     |                 |
     v                 v
+-----------+  +--------------+
|TimerProxy |  |TimerHandler  |
|(后台代理)  |  |(硬件抽象)    |
+-----------+  +--------------+
```

---

## 相关链接

- [项目概览](./00_Overview.md)
- [架构说明](./02_Architecture.md)
- [N-API 参考](./03_NAPI_Reference.md)
