# appendix/Config_Flags.md - 关键配置与常量

## 概述

本文档汇总 ecological_rule_manager 模块中的关键配置、常量定义。

## System Ability 配置

### SA Profile

**文件**: `profile/6105.json`

```json
{
    "process": "foundation",
    "systemability": [
        {
            "name": 6105,
            "libpath": "libecologicalrulemgr_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": []
        }
    ]
}
```

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | 6105 | System Ability ID |
| `process` | foundation | 运行在 foundation 进程 |
| `libpath` | libecologicalrulemgr_service.z.so | SA 库路径 |
| `run-on-create` | true | 随进程创建时启动 |
| `distributed` | false | 不支持分布式 |
| `dump_level` | 1 | dump 级别 |

## 常量定义

### 接口令牌 (Interface Token)

**文件**: `services/manager/src/ecologic_rule_mgr_service_stub.cpp:29-30`

```cpp
static inline const std::u16string ERMS_INTERFACE_TOKEN =
    u"ohos.cloud.ecologicalrulemgrservice.IEcologicalRuleMgrService";
```

### Foundation UID

**文件**: `services/manager/src/ecologic_rule_mgr_service_stub.cpp:27`

```cpp
#define ERMS_FOUNDATION_UID 5523
```

### 大小限制

**文件**: `services/manager/include/ecological_rule_mgr_service_stub.h:74-75`

```cpp
const int32_t MAX_WANT_SIZE = 15;
const int32_t MAX_ABILITY_INFO_SIZE = 1000;
```

### 错误码

**文件**: `interfaces/innerkits/include/ecological_rule_mgr_service_interface.h:50-55`

```cpp
enum ErrCode {
    ERR_BASE = (-99),
    ERR_FAILED = (-1),
    ERR_PERMISSION_DENIED = (-2),
    ERR_OK = 0,
};
```

### AMS ExperienceRule 默认值

**文件**: `services/manager/src/ecologic_rule_mgr_service_stub.cpp:149`

```cpp
amsRule.resultCode = 10; // 10 is default value
```

## 日志配置

### HiLog Label

**文件**: `utils/include/ecological_rule_mgr_service_logger.h:21`

```cpp
static constexpr OHOS::HiviewDFX::HiLogLabel ERMS_LABEL = { LOG_CORE, 0xD0017D7, "ERMS" };
```

| 字段 | 值 | 说明 |
|------|-----|------|
| domain | LOG_CORE (0x0) | 日志域 |
| tag | 0xD0017D7 | 模块标识 |
| name | "ERMS" | 日志标签 |

### 日志宏

**文件**: `utils/include/ecological_rule_mgr_service_logger.h:28-32`

```cpp
#define LOG_DEBUG(fmt, ...) ERMS_LOG(Debug, TAG, fmt, ##__VA_ARGS__)
#define LOG_INFO(fmt, ...)  ERMS_LOG(Info, TAG, fmt, ##__VA_ARGS__)
#define LOG_WARN(fmt, ...)  ERMS_LOG(Warn, TAG, fmt, ##__VA_ARGS__)
#define LOG_ERROR(fmt, ...) ERMS_LOG(Error, TAG, fmt, ##__VA_ARGS__)
#define LOG_FATAL(fmt, ...) ERMS_LOG(Fatal, TAG, fmt, ##__VA_ARGS__)
```

### 日志 Tag

| 宏定义值 | 文件位置 | 用途 |
|----------|----------|------|
| `TAG` = "ERMS_MAIN" | `services/manager/src/ecologic_rule_mgr_service.cpp:28` | SA 主逻辑 |
| `TAG` = "ERMS_STUB" | `services/manager/src/ecologic_rule_mgr_service_stub.cpp:26` | IPC Stub |
| `TAG` = "ERMS_CLIENT" | `interfaces/innerkits/src/ecological_rule_mgr_service_client.cpp:28` | Client |
| `TAG` = "erms_proxy" | `interfaces/innerkits/src/ecological_rule_mgr_service_proxy.cpp:22` | Proxy |

## GN 配置变量

**文件**: `ecologicalrulemgrservice.gni`

```gn
ability_runtime_path = "//foundation/ability/ability_runtime"
ecologicalrulemgrservice_root_path =
    "//foundation/bundlemanager/ecological_rule_manager"
innerkits_path = "${ecologicalrulemgrservice_root_path}/interfaces/innerkits"
ecologicalrulemgrservice_path = "${ecologicalrulemgrservice_root_path}/services"
ecologicalrulemgrservice_utils_path =
    "${ecologicalrulemgrservice_root_path}/utils"
```

## CallerInfo 应用类型

**文件**: `interfaces/innerkits/include/ecological_rule_mgr_service_param.h:52-56`

```cpp
enum {
    TYPE_INVALID = 0,
    TYPE_HARMONY_APP,
    TYPE_ATOM_SERVICE,
};
```

### CallerInfo 链接类型

**文件**: `interfaces/innerkits/include/ecological_rule_mgr_service_param.h:58-63`

```cpp
enum {
    LINK_TYPE_INVALID = 0,
    LINK_TYPE_UNIVERSAL_LINK,
    LINK_TYPE_DEEP_LINK,
    LINK_TYPE_WEB_LINK,
    LINK_TYPE_ABILITY,
};
```

## 路径速查

| 配置项 | 文件路径 |
|--------|----------|
| SA 配置 | `profile/6105.json` |
| 接口定义 | `interfaces/innerkits/include/ecological_rule_mgr_service_interface.h` |
| 参数定义 | `interfaces/innerkits/include/ecological_rule_mgr_service_param.h` |
| Stub 头文件 | `services/manager/include/ecological_rule_mgr_service_stub.h` |
| GN 变量 | `ecologicalrulemgrservice.gni` |
