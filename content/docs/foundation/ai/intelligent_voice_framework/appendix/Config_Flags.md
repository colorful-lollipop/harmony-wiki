# 配置开关

> **目的**: 记录 Intelligent Voice Framework 的所有 Feature Flags 和编译配置  
> **适用范围**: 性能优化、定制开发、问题调试  
> **最后更新**: 2026-02-06

---

## 1. 根配置 (intell_voice_service.gni)

### 1.1 模块开关

| 变量 | 默认值 | 类型 | 描述 |
|------|--------|------|------|
| `telephony_service_enable` | `false` | bool | 电话服务集成开关 |
| `intelligent_voice_framework_trigger_enable` | `true` | bool | 触发器模块开关 |
| `intelligent_voice_framework_engine_enable` | `true` | bool | 引擎模块开关 |
| `intelligent_voice_framework_only_first_stage` | `false` | bool | 仅第一阶段唤醒 |
| `intelligent_voice_framework_only_second_stage` | `false` | bool | 仅第二阶段唤醒 |
| `intelligent_voice_framework_window_manager_enable` | `false` | bool | 窗口管理集成 |
| `intelligent_voice_framework_power_manager_enable` | `false` | bool | 电源管理集成 |
| `intelligent_voice_framework_first_stage_oneshot_enable` | `false` | bool | 第一阶段单次唤醒 |

### 1.2 配置组合

| 场景 | trigger_enable | engine_enable | first_stage | second_stage | oneshot |
|------|---------------|---------------|-------------|--------------|---------|
| 完整功能 | true | true | false | false | false |
| 仅 DSP | true | false | - | - | - |
| 仅 AP (唤醒) | false | true | true | false | false |
| 仅 AP (单次) | false | true | true | false | true |
| 仅 AP (完整) | false | true | false | true | - |
| 虚拟引擎 | false | false | - | - | - |

---

## 2. 编译宏定义

### 2.1 条件编译宏

| 宏定义 | 条件 | 用途 |
|--------|------|------|
| `TRIGGER_ENABLE` | `trigger_enable = true` | 触发器代码 |
| `ENGINE_ENABLE` | `engine_enable = true` | 引擎代码 |
| `ONLY_FIRST_STAGE` | `only_first_stage = true` | 第一阶段唤醒 |
| `ONLY_SECOND_STAGE` | `only_second_stage = true` | 第二阶段唤醒 |
| `FIRST_STAGE_ONESHOT_ENABLE` | `oneshot_enable = true` | 单次唤醒 |
| `POWER_MANAGER_ENABLE` | `power_manager_enable = true` | 电源管理 |
| `SUPPORT_TELEPHONY_SERVICE` | `telephony_service_enable = true` | 电话服务 |
| `SUPPORT_WINDOW_MANAGER` | `window_manager_enable = true` | 窗口管理 |
| `INTELL_VOICE_BUILD_VARIANT_ROOT` | `build_variant == "root"` | root 版本 |
| `USE_FFRT` | 全局 | FFRT 任务调度 |

### 2.2 调试宏

| 宏定义 | 默认值 | 用途 |
|--------|--------|------|
| `HILOG_ENABLE` | true | HiLog 日志 |
| `ENABLE_DEBUG` | true | 调试日志 |

---

## 3. 服务配置 (intell_voice_service.cfg)

### 3.1 权限配置

```json
{
    "permission": [
        "ohos.permission.MANAGE_INTELLIGENT_VOICE",
        "ohos.permission.MICROPHONE",
        "ohos.permission.GET_TELEPHONY_STATE",
        "ohos.permission.READ_CALL_LOG",
        "ohos.permission.START_ABILITIES_FROM_BACKGROUND",
        "ohos.permission.WAKEUP_VOICE",
        "ohos.permission.WAKEUP_VISION",
        "ohos.permission.PERMISSION_USED_STATS"
    ]
}
```

### 3.2 SELinux 配置

```json
{
    "process": "intell_voice_service",
    "selinux": "u:r:intell_voice_service:s0"
}
```

---

## 4. SA 配置 (intell_voice_service.json)

```json
{
    "process": "intell_voice_service",
    "systemability": [
        {
            "name": 312,
            "libpath": "libintell_voice_server.z.so",
            "run-on-create": false,
            "auto-restart": true,
            "distributed": false,
            "dump_level": 1,
            "start-on-demand": {
                "commonevent": [
                    {
                        "name": "usual.event.BOOT_COMPLETED"
                    },
                    {
                        "name": "usual.event.POWER_SAVE_MODE_CHANGED",
                        "value": "600"
                    }
                ]
            }
        }
    ]
}
```

### 4.1 配置项说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | 312 | SA ID |
| libpath | libintell_voice_server.z.so | 服务库路径 |
| run-on-create | false | 按需启动 |
| auto-restart | true | 崩溃自动重启 |
| distributed | false | 分布式支持（关闭） |
| dump_level | 1 | 转储级别 |
| commonevent | BOOT_COMPLETED, POWER_SAVE_MODE_CHANGED | 监听事件 |

---

## 5. 构建配置 (BUILD.gn)

### 5.1 安全配置

所有主要 targets 启用以下安全特性：

```gn
sanitize = {
    cfi = true                    # 控制流完整性
    cfi_cross_dso = true         # 跨 DSO CFI
    cfi_vcall_icall_only = true  # 仅虚函数调用检查
    debug = false
}
branch_protector_ret = "pac_ret"  # PAC 返回地址保护
```

### 5.2 NDK 导出配置

```gn
ohos_shared_library("intellvoice_native") {
    # ...
    innerapi_tags = ["ndk"]  # 导出为 NDK
}
```

---

## 6. 引擎类型配置

### 6.1 引擎类型枚举

| 类型 | 值 | 说明 |
|------|------|------|
| ENROLL_ENGINE_TYPE | 0 | 注册引擎 |
| WAKEUP_ENGINE_TYPE | 1 | 唤醒引擎 |
| UPDATE_ENGINE_TYPE | 2 | 更新引擎 |

### 6.2 灵敏度配置

| 灵敏度 | 值 | 说明 |
|--------|------|------|
| LOW_SENSIBILITY | 1 | 低灵敏度 |
| MIDDLE_SENSIBILITY | 2 | 中灵敏度 |
| HIGH_SENSIBILITY | 3 | 高灵敏度 |

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [构建系统](../06_Build_System.md) | 构建配置 |
| [编译产物](../07_Artifacts.md) | 产物说明 |
| [项目概览](../01_Overview.md) | 基本配置 |
