# 附录：配置标志

## 目的

本文档列出 Resource Schedule Service 的所有 Feature Flags 和编译配置选项。

---

## Feature Flags (bundle.json)

| Flag | 说明 | 默认状态 |
|------|------|----------|
| `resource_schedule_service_with_ffrt_enable` | FFRT 任务调度支持 | ✅ |
| `resource_schedule_service_with_ext_res_enable` | 扩展资源支持 | ❌ |
| `resource_schedule_service_socperf_executor_enable` | SocPerf 执行器 | ❌ |
| `resource_schedule_service_with_app_nap_enable` | AppNap 功能 | ❌ |
| `resource_schedule_service_cust_soc_perf_enable` | 自定义 SocPerf | ❌ |
| `resource_schedule_service_crown_power_key_enable` | 表冠按键支持 | ❌ |
| `resource_schedule_service_file_copy_soc_perf_enable` | 文件复制 SocPerf | ❌ |
| `resource_schedule_service_subscribe_click_recognize_enable` | 点击识别订阅 | ❌ |
| `resource_schedule_service_system_load_level_debug_feature_enable_for_2d` | 负载调试 | ❌ |
| `resource_schedule_service_has_sys_nice_enable` | 系统 nice 支持 | ❌ |
| `resource_schedule_service_depend_wm_enable` | 窗口管理依赖 | ❌ |

---

## GN 编译选项

### 条件编译宏

| 宏 | 说明 | 定义位置 |
|----|------|----------|
| `RESOURCE_SCHEDULE_SERVICE_WITH_FFRT_ENABLE` | 启用 FFRT | BUILD.gn |
| `RESOURCE_SCHEDULE_SERVICE_WITH_EXT_RES_ENABLE` | 启用扩展资源 | BUILD.gn |
| `RESSCHED_RESOURCESCHEDULE_SOC_PERF_ENABLE` | 启用 SocPerf | BUILD.gn |
| `RESSCHED_TELEPHONY_STATE_REGISTRY_ENABLE` | 启用电话监听 | BUILD.gn |
| `RESSCHED_COMMUNICATION_BLUETOOTH_ENABLE` | 启用蓝牙监听 | BUILD.gn |
| `RESSCHED_MULTIMEDIA_AV_SESSION_ENABLE` | 启用 AV 会话 | BUILD.gn |
| `RESSCHED_FRAME_AWARE_SCHED_ENABLE` | 启用帧感知 | BUILD.gn |
| `RSS_DEVICE_STANDBY_ENABLE` | 启用待机 | BUILD.gn |
| `POWER_MANAGER_ENABLE` | 启用电源管理 | BUILD.gn |
| `SET_SYSTEM_LOAD_LEVEL_2D_ENABLE` | 启用 2D 负载调试 | BUILD.gn |

---

## 运行时配置

### 插件开关

文件: `ressched/profile/res_sched_plugin_switch.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin_switch>
    <plugin name="libsocperf_plugin.z.so" switch="on" />
    <plugin name="libframe_aware_plugin.z.so" switch="on" />
    <plugin name="libcgroup_sched.z.so" switch="on" />
    <plugin name="libdevice_standby_plugin.z.so" switch="on" />
</plugin_switch>
```

### Cgroup 策略配置

文件: `ressched/plugins/cgroup_sched_plugin/profiles/cgroup_action_config.json`

```json
{
    "Cgroups": [
        {
            "controller": "cpu",
            "path": "/dev/cpuctl",
            "sched_policy": {
                "sp_default": "",
                "sp_background": "background",
                "sp_foreground": "foreground",
                "sp_system_background": "system-background",
                "sp_top_app": "top-app"
            }
        },
        {
            "controller": "cpuset",
            "path": "/dev/cpuset",
            "sched_policy": { ... }
        }
    ]
}
```

---

## 日志级别

| 级别 | 宏 | 说明 |
|------|-----|------|
| DEBUG | `RESSCHED_LOGD` | 调试信息 |
| INFO | `RESSCHED_LOGI` | 一般信息 |
| WARN | `RESSCHED_LOGW` | 警告信息 |
| ERROR | `RESSCHED_LOGE` | 错误信息 |

---

## 性能参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `WARN_TIME` | 1000 μs | 插件处理警告阈值 |
| `ERROR_TIME` | 10000 μs | 插件处理错误阈值 |
| `SINGLE_UID_REQUEST_LIMIT_COUNT` | 250 req/s | 单 UID 限流 |
| `ALL_UID_REQUEST_LIMIT_COUNT` | 650 req/s | 全系统限流 |
| `PAYLOAD_MAX_SIZE` | 4096 bytes | Payload 最大大小 |
| `CHECK_MUTEX_TIMEOUT` | 100 ms | 同步事件超时 |

---

## 代码证据

### Feature Flag 定义

```json
// 文件: bundle.json

"features": [
    "resource_schedule_service_with_ffrt_enable",
    "resource_schedule_service_with_ext_res_enable",
    "resource_schedule_service_socperf_executor_enable",
    "resource_schedule_service_with_app_nap_enable",
    ...
]
```

### 编译条件

```gn
# 文件: ressched/services/BUILD.gn

if (resource_schedule_service_with_ffrt_enable) {
    deps += [ "//foundation/commonlibrary/c_utils:utils" ]
    defines += [ "RESOURCE_SCHEDULE_SERVICE_WITH_FFRT_ENABLE" ]
}

if (ressched_socperf_enable) {
    deps += [ "//foundation/resourceschedule/soc_perf:socperf_client" ]
}
```

---

## 相关链接

- [GN 构建](../05_GN_Targets.md) - 构建配置
- [编译产物](../06_Build_Artifacts.md) - 输出文件
