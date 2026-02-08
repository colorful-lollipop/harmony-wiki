# 附录：配置标志

> 关键宏与 Feature Flags 详细说明

---

## time.gni 配置

**文件位置**: `//base/time/time_service/time.gni`

### 路径变量

| 变量 | 值 | 说明 |
|------|-----|------|
| `time_root_path` | `//base/time/time_service` | 项目根路径 |
| `api_path` | `${time_root_path}/interfaces/inner_api` | 内部 API 路径 |
| `time_capi_path` | `${time_root_path}/interfaces/kits/c` | C API 路径 |
| `time_service_path` | `${time_root_path}/services` | 服务实现路径 |
| `time_utils_path` | `${time_root_path}/utils` | 工具路径 |
| `time_sanitize_debug` | `false` | 调试标志 |

---

## Feature Flags

### device_standby

| 属性 | 值 |
|------|-----|
| **默认值** | `true` |
| **控制条件** | `global_parts_info.resourceschedule_device_standby` |
| **作用** | 启用设备待机服务支持 |

**相关代码**:
```cpp
// time_system_ability.cpp:198
AddSystemAbilityListener(DEVICE_STANDBY_SERVICE_SYSTEM_ABILITY_ID);
```

---

### time_service_debug_able

| 属性 | 值 |
|------|-----|
| **默认值** | `true` |
| **作用** | 启用调试日志输出 |

**使用示例**:
```cpp
#ifdef TIME_DEBUG
    TIME_HILOGD(TIME_MODULE_SERVICE, "debug message");
#endif
```

---

### time_service_hicollie_able

| 属性 | 值 |
|------|-----|
| **默认值** | `true` |
| **作用** | 启用 HiCollie 卡顿检测 |

**相关代码**:
```cpp
// time_xcollie.h
class TimeXCollie {
public:
    explicit TimeXCollie(const std::string &name);
    ~TimeXCollie();
};

// 使用示例（time_system_ability.cpp:373）
TimeXCollie timeXCollie("TimeService::CreateTimer");
```

---

### time_service_hidumper_able

| 属性 | 值 |
|------|-----|
| **默认值** | `true` |
| **作用** | 启用 HIDumper 诊断命令 |

**相关代码**:
```cpp
#ifdef HIDUMPER_ENABLE
void TimeSystemAbility::InitDumpCmd() {
    auto cmdTime = std::make_shared<TimeCmdParse>(...);
    TimeCmdDispatcher::GetInstance().RegisterCommand(cmdTime);
    // ...
}
#endif
```

---

### time_service_set_auto_reboot

| 属性 | 值 |
|------|-----|
| **默认值** | `false` |
| **作用** | 启用自动重启相关功能 |

**相关代码**:
```cpp
#ifdef SET_AUTO_REBOOT_ENABLE
    AddSystemAbilityListener(POWER_MANAGER_SERVICE_ID);
#endif
```

---

### time_service_multi_account

| 属性 | 值 |
|------|-----|
| **默认值** | `true` |
| **作用** | 启用多账号支持 |

**相关代码**:
```cpp
#ifdef MULTI_ACCOUNT_ENABLE
void TimeSystemAbility::RegisterOsAccountSubscriber() {
    AccountSA::OsAccountSubscribeInfo subscribeInfo(...);
    // ...
}
#endif
```

---

### time_service_rdb_enable

| 属性 | 值 |
|------|-----|
| **默认值** | `true` |
| **作用** | 启用 RDB 数据库存储 |

**影响**:
- `true`: 使用 `TimeDatabase` (RDB) 存储定时器
- `false`: 使用 `CjsonHelper` (JSON 文件) 存储定时器

**相关代码**:
```cpp
#ifdef RDB_ENABLE
    TimeDatabase::GetInstance().ClearDropOnReboot();
#else
    CjsonHelper::GetInstance().Clear(DROP_ON_REBOOT);
#endif
```

---

## 编译期宏定义

### 在 BUILD.gn 中定义的宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `TIME_SERVICE_EXPORT` | `visibility.h` | API 导出标志 |
| `RDB_ENABLE` | `services/BUILD.gn` | RDB 支持 |
| `HIDUMPER_ENABLE` | `services/BUILD.gn` | HIDumper 支持 |
| `SET_AUTO_REBOOT_ENABLE` | `services/BUILD.gn` | 自动重启功能 |
| `MULTI_ACCOUNT_ENABLE` | `services/BUILD.gn` | 多账号支持 |
| `TIME_DEBUG` | `utils/BUILD.gn` | 调试模式 |

### 在 time.gni 中定义的宏

```gn
defines = [
  "TIME_SERVICE_ROOT_PATH=\"${time_root_path}\"",
  "API_PATH=\"${api_path}\"",
  "TIME_SERVICE_PATH=\"${time_service_path}\"",
]
```

---

## 运行时参数

### 系统参数（time.para）

**文件位置**: `services/etc/time.para`

| 参数名 | 默认值 | 说明 |
|--------|--------|------|
| `persist.time.auto_time` | `ON`/`OFF` | 自动时间同步开关 |

**读取方式**:
```cpp
#include "parameters.h"
std::string autoTime = system::GetParameter("persist.time.auto_time", "OFF");
```

---

### 启动参数（bootevent）

| 参数名 | 说明 |
|--------|------|
| `bootevent.boot.completed` | 系统启动完成标志 |

**使用位置**:
```cpp
// time_system_ability.cpp:187
std::string bootCompleted = system::GetParameter("bootevent.boot.completed", "");
if (bootCompleted != "true") {
    // 清理重启时应丢弃的定时器
}
```

---

## 配置文件

### SA 配置文件 (3702.json)

**文件位置**: `services/profile/3702.json`

```json
{
  "process": "timeservice",
  "systemability": [
    {
      "name": 3702,
      "libpath": "libtime_system_ability.z.so",
      "run-on-create": true,
      "auto-restart": true,
      "distributed": false,
      "dump-level": 1
    }
  ]
}
```

### 服务启动配置 (timeservice.cfg)

**文件位置**: `services/etc/init/timeservice.cfg`

```json
{
  "services": [{
    "name": "timeservice",
    "path": ["/system/bin/sa_main", "3702"],
    "uid": "system",
    "gid": ["system", "shell"],
    "secon": "u:r:time_service:s0",
    "permission": ["ohos.permission.SET_TIME"]
  }]
}
```

---

## 数据库表结构

### RDB 表 (time_service_rdb_enable=true)

**表名**: `hold_on_reboot`, `drop_on_reboot`

| 字段 | 类型 | 说明 |
|------|------|------|
| timerId | INTEGER | 定时器 ID |
| type | INTEGER | 定时器类型 |
| flag | INTEGER | 标志位 |
| windowLength | INTEGER | 窗口长度 |
| interval | INTEGER | 重复间隔 |
| uid | INTEGER | 用户 ID |
| bundleName | TEXT | 应用包名 |
| wantAgent | TEXT | WantAgent 序列化 |
| state | INTEGER | 状态 |
| triggerTime | INTEGER | 触发时间 |
| pid | INTEGER | 进程 ID |
| name | TEXT | 定时器名称 |

### JSON 文件 (time_service_rdb_enable=false)

**文件位置**: `/data/service/el1/public/database/time/time.json`

**结构**:
```json
{
  "hold_on_reboot": [
    {"timerId": "123", "type": 3, "flag": 0, ...}
  ],
  "drop_on_reboot": []
}
```

---

## 相关链接

- [GN 构建系统](./../05_GN_Targets.md) - 构建配置
- [目录结构](./../01_Directory_Structure.md) - 源码组织
- [架构说明](./../02_Architecture.md) - 组件关系
