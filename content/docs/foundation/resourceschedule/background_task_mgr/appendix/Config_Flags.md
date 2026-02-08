# 附录 - 配置项与Feature Flags

本文档汇总后台任务管理模块的所有配置项和编译开关。

---

## 1. GN Feature Flags

### 1.1 主配置 (bgtaskmgr.gni)

| Flag | 类型 | 默认值 | 说明 | 代码位置 |
|------|------|--------|------|----------|
| `background_task_mgr_graphics` | bool | true | 启用图形支持 | bgtaskmgr.gni:27 |
| `background_task_mgr_jsstack` | bool | true | 启用JS栈支持 | bgtaskmgr.gni:28 |
| `background_task_mgr_device_enable` | bool | true | 设备使能开关 | bgtaskmgr.gni:29 |
| `has_os_account_part` | bool | auto | OS账号组件可用性 | bgtaskmgr.gni:32-37 |
| `distributed_notification_enable` | bool | auto | 分布式通知服务可用性 | bgtaskmgr.gni:39-44 |

**使用方式**:
```gn
if (background_task_mgr_device_enable) {
  deps = [ ... ]
}
```

### 1.2 服务编译定义

| Define | 说明 | 条件 |
|--------|------|------|
| `HAS_OS_ACCOUNT_PART` | 支持OS账号 | has_os_account_part |
| `DISTRIBUTED_NOTIFICATION_ENABLE` | 支持分布式通知 | distributed_notification_enable |
| `SUPPORT_GRAPHICS` | 支持图形 | background_task_mgr_graphics |
| `SUPPORT_AUTH` | 支持认证 | 手机/PC/平板产品 |
| `FEATURE_PRODUCT_PHONE` | 手机产品 | 产品定义 |
| `FEATURE_PRODUCT_WATCH` | 手表产品 | 产品定义 |
| `FEATURE_PRODUCT_PC` | PC产品 | 产品定义 |
| `FEATURE_PRODUCT_TABLET` | 平板产品 | 产品定义 |

---

## 2. 运行时配置

### 2.1 短时任务配额配置

**代码位置**: `services/common/include/bgtask_config.h`

| 配置项 | 默认值 | 说明 | 代码位置 |
|--------|--------|------|----------|
| `DEFAULT_DELAY_TIME` | 3分钟 | 单次默认延迟时间 | key_info.h:28 |
| `DEFAULT_QUOTA_PER_DAY` | 10分钟 | 每日默认配额 | decision_maker.cpp:60 |
| `MAX_REQUEST_ID` | INT32_MAX | 最大请求ID | bg_transient_task_mgr.cpp:45 |

**运行时修改** (通过Dump接口):
```cpp
// 设置新的配额
SetBgTaskConfig("{\"quota\":20}", CONFIG_TYPE_TRANSIENT_TASK);
```

### 2.2 长时任务配置

**代码位置**: `services/continuous_task/include/bg_continuous_task_mgr.h`

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `continuousTaskIdIndex_` | 0 | 任务ID起始值 |
| 通知文本数组 | 系统资源 | 9种模式的通知文本 |

**后台模式定义** (`interfaces/innerkits/include/background_mode.h`):
```cpp
enum class BackgroundMode : uint32_t {
    DATA_TRANSFER = 0,
    AUDIO_PLAYBACK = 1,
    AUDIO_RECORDING = 2,
    LOCATION = 3,
    BLUETOOTH_INTERACTION = 4,
    MULTI_DEVICE_CONNECTION = 5,
    WIFI_INTERACTION = 6,
    VOIP = 7,
    TASK_KEEPING = 8
};
```

### 2.3 能效资源配置

**代码位置**: `services/efficiency_resources/include/bg_efficiency_resources_mgr.h`

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `MAX_RESOURCE_MASK` | 0xFFFF | 资源类型位掩码上限 |

**资源类型定义** (`interfaces/innerkits/include/resource_type.h`):
```cpp
enum class ResourceType : uint32_t {
    CPU = 1,
    COMMON_EVENT = 2,
    TIMER = 4,
    WORK_SCHEDULER = 8,
    BLUETOOTH = 16,
    GPS = 32,
    AUDIO = 64,
    RUNNING_LOCK = 128,
    SENSOR = 256
};
```

**CPU级别定义** (`interfaces/innerkits/include/efficiency_resources_cpu_level.h`):
```cpp
enum class EfficiencyResourcesCpuLevel : int32_t {
    DEFAULT = 0,
    SMALL_CPU = 1,
    MEDIUM_CPU = 2,
    LARGE_CPU = 3
};
```

---

## 3. JSON配置文件

### 3.1 豁免应用配置格式

**文件位置**: 运行时加载的系统配置文件

```json
{
    "transientTaskExemptedQuatoList": {
        "com.example.app1": {
            "signature": "SHA256_HASH_VALUE",
            "reason": "System app exemption"
        }
    },
    "taskKeepingExemptedQuatoList": {
        "com.example.app2": {
            "signature": "SHA256_HASH_VALUE"
        }
    },
    "maliciousAppBlocklist": {
        "com.malicious.app": {
            "signature": "SHA256_HASH_VALUE"
        }
    },
    "allowApplyCpuBundleInfoList": {
        "com.example.app3": {
            "appId": "com.example.app3_BQ1bKlbLSQGvfN7...",
            "maxCpuLevel": 3
        }
    },
    "specialExemptedQuatoList": {
        "com.special.app": {
            "signature": "SHA256_HASH_VALUE"
        }
    }
}
```

**解析代码**: `services/common/src/bgtask_config.cpp:184-320`

---

## 4. SA配置

### 4.1 SA ID 1903配置

**文件**: `sa_profile/1903.json`

```json
{
    "process": "resource_schedule_service",
    "systemability": [
        {
            "name": 1903,
            "libpath": "libbgtaskmgr_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "extension": ["backup", "restore"]
        }
    ]
}
```

**字段说明**:
| 字段 | 值 | 说明 |
|------|-----|------|
| `process` | resource_schedule_service | 所属进程 |
| `name` | 1903 | SA ID |
| `libpath` | libbgtaskmgr_service.z.so | 动态库路径 |
| `run-on-create` | true | 创建时启动 |
| `distributed` | false | 不支持分布式 |
| `dump_level` | 1 | Dump级别 |
| `extension` | ["backup", "restore"] | 支持的扩展 |

---

## 5. Bundle配置

### 5.1 组件配置 (bundle.json)

**SystemCapability声明**:
```json
{
    "component": {
        "syscap": [
            "SystemCapability.ResourceSchedule.BackgroundTaskManager.ContinuousTask",
            "SystemCapability.ResourceSchedule.BackgroundTaskManager.TransientTask",
            "SystemCapability.ResourceSchedule.BackgroundTaskManager.EfficiencyResourcesApply"
        ],
        "features": [
            "background_task_mgr_graphics",
            "background_task_mgr_jsstack",
            "background_task_mgr_device_enable"
        ]
    }
}
```

### 5.2 权限声明

**使用的权限**:
| 权限 | 用途 | 级别 |
|------|------|------|
| `ohos.permission.KEEP_BACKGROUND_RUNNING` | 长时任务申请 | normal |
| `ohos.permission.SET_BACKGROUND_TASK_STATE` | 设置任务状态 | system_basic |
| `ohos.permission.GET_BACKGROUND_TASK_INFO` | 获取任务信息 | system_basic |
| `ohos.permission.DUMP` | Dump调试 | system_core |

---

## 6. 日志配置

### 6.1 日志标签

| 标签 | 用途 |
|------|------|
| `background_task_mgr` | 主服务日志 |
| `BgTransientTaskMgr` | 短时任务管理器 |
| `BgContinuousTaskMgr` | 长时任务管理器 |
| `BgEfficiencyResourcesMgr` | 能效资源管理器 |

### 6.2 日志级别

**代码定义**: `frameworks/common/include/bgtaskmgr_log_wrapper.h`

```cpp
#define BGTASK_LOGF(...) HILOG_FATAL(LOG_CORE, __VA_ARGS__)
#define BGTASK_LOGE(...) HILOG_ERROR(LOG_CORE, __VA_ARGS__)
#define BGTASK_LOGW(...) HILOG_WARN(LOG_CORE, __VA_ARGS__)
#define BGTASK_LOGI(...) HILOG_INFO(LOG_CORE, __VA_ARGS__)
#define BGTASK_LOGD(...) HILOG_DEBUG(LOG_CORE, __VA_ARGS__)
```

---

## 7. Hisysevent事件

### 7.1 配置 (hisysevent.yaml)

```yaml
background_task_mgr:
  __BASE: {type: statistic, level: MINOR}
  transient_task_start: {type: behavior, level: MINOR}
  transient_task_cancel: {type: behavior, level: MINOR}
  continuous_task_start: {type: behavior, level: MINOR}
  continuous_task_cancel: {type: behavior, level: MINOR}
  efficiency_resources_apply: {type: behavior, level: MINOR}
  efficiency_resources_reset: {type: behavior, level: MINOR}
```

### 7.2 上报代码

**位置**: `services/common/src/report_hisysevent_data.cpp`

---

## 8. 编译安全选项

### 8.1 加固配置

**位置**: 各BUILD.gn

```gn
# CFI (Control Flow Integrity)
sanitize = {
  cfi = true
  cfi_cross_dso = true
}

# PAC-RET (Pointer Authentication)
branch_protector_ret = "pac_ret"

# Stack Protection
cflags_cc = ["-fstack-protector-strong"]

# Symbol Visibility
cflags_cc += ["-fvisibility=hidden"]
```

---

## 相关文档

- [GN构建](../05_GN_Build.md) - 构建配置详细说明
- [目录结构](../01_Directory_Structure.md) - 配置文件位置
- [安全风险](../06_Security.md) - 配置安全分析
