# 构建系统

> **目的**: 详细说明 Intelligent Voice Framework 的 GN 构建配置和 Targets  
> **适用范围**: 构建工程师、模块开发者、持续集成  
> **最后更新**: 2026-02-06

---

## 1. 构建配置

### 1.1 根配置文件

| 文件 | 用途 |
|------|------|
| `intell_voice_service.gni` | Feature Flags 和构建变量 |
| `bundle.json` | 组件配置和子系统定义 |

### 1.2 Feature Flags

**文件**: `intell_voice_service.gni`

```gn
# 开关控制
inteligent_voice_framework_trigger_enable = true      # 触发模块开关
inteligent_voice_framework_engine_enable = true        # 引擎模块开关
inteligent_voice_framework_only_first_stage = false   # 仅第一阶段
inteligent_voice_framework_only_second_stage = false   # 仅第二阶段
inteligent_voice_framework_window_manager_enable = false # 窗口管理
inteligent_voice_framework_power_manager_enable = false # 电源管理
inteligent_voice_framework_first_stage_oneshot_enable = false # 第一阶段单次

# 条件编译
if (intelligent_voice_framework_power_manager_enable) {
    defines += [ "POWER_MANAGER_ENABLE" ]
}

if (intelligent_voice_framework_trigger_enable) {
    defines += [ "TRIGGER_ENABLE" ]
}

if (intelligent_voice_framework_engine_enable) {
    defines += [ "ENGINE_ENABLE" ]
}
```

**证据来源**: `intell_voice_service.gni`

---

## 2. Targets 清单

### 2.1 services/intell_voice_service/

| Target | 类型 | 描述 |
|--------|------|------|
| `server_source` | source_set | 服务源码集合 |
| `intell_voice_server` | shared_library | 主服务 (.so) |
| `intell_voice_server_test` | shared_library | 测试版本 |

**主要 sources**:
```
server/sa/intell_voice_service.cpp
server/sa/intell_voice_service_manager.cpp
server/sa/intell_voice_service_stub.cpp
server/sa/intell_voice_engine_registrar.cpp
server/sa/intell_voice_trigger_registrar.cpp
server/utils/system_event_observer.cpp
```

**关键 deps**:
```
deps: ["../../utils:intell_voice_utils"]
external_deps: [
    "ability_runtime:ability_manager",
    "access_token:libaccesstoken_sdk",
    "hilog:libhilog",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "ffrt:libffrt",
]
```

**证据来源**: `services/intell_voice_service/BUILD.gn`

---

### 2.2 services/intell_voice_engine/

| Target | 类型 | 描述 |
|--------|------|------|
| `engine_source` | source_set | 引擎源码集合 |
| `intelligentvoice_engine` | shared_library | 引擎库 (.so) |
| `intelligentvoice_engine_test` | shared_library | 测试版本 |

**条件编译**:
- `intelligent_voice_framework_engine_enable = true`: 完整引擎
- `intelligent_voice_framework_first_stage_oneshot_enable = true`: 仅第一阶段单次
- 否则: 虚拟引擎

**主要 sources** (完整引擎):
```
# 基础组件
server/base/engine_base.cpp
server/base/engine_factory.cpp
server/base/intell_voice_engine_stub.cpp

# 注册引擎
server/enroll/enroll_engine.cpp

# 唤醒引擎
server/wakeup/wakeup_engine.cpp
server/wakeup/wakeup_engine_impl.cpp

# 更新引擎
server/update/update_engine.cpp

# 管理器
server/manager/intell_voice_engine_manager.cpp
server/manager/intell_voice_engine_arbitration.cpp

# HDI 适配
server/hdi_adapter/engine_host_manager.cpp
```

**证据来源**: `services/intell_voice_engine/BUILD.gn`

---

### 2.3 services/intell_voice_trigger/

| Target | 类型 | 描述 |
|--------|------|------|
| `trigger_source` | source_set | 触发器源码集合 |
| `intelligentvoice_trigger` | shared_library | 触发器库 (.so) |
| `intelligentvoice_trigger_test` | shared_library | 测试版本 |

**主要 sources**:
```
server/trigger_service.cpp
server/trigger_manager.cpp
server/trigger_detector.cpp
server/trigger_connector.cpp
server/trigger_connector_mgr.cpp
server/trigger_db_helper.cpp
```

**条件编译**:
```gn
if (telephony_service_enable) {
    external_deps += ["call_manager:tel_call_manager_api"]
    defines += ["SUPPORT_TELEPHONY_SERVICE"]
}

if (intelligent_voice_framework_window_manager_enable) {
    external_deps += ["window_manager:libdm_lite"]
    defines += ["SUPPORT_WINDOW_MANAGER"]
}
```

**证据来源**: `services/intell_voice_trigger/BUILD.gn`

---

### 2.4 frameworks/js/

| Target | 类型 | 描述 |
|--------|------|------|
| `intelligentvoice_js` | js_declaration | JS 声明 |
| `intelligentvoice` | shared_library | N-API 模块 (.so) |

**sources**:
```
napi/intell_voice_manager_napi.cpp
napi/enroll_intell_voice_engine_napi.cpp
napi/wakeup_intell_voice_engine_napi.cpp
napi/wakeup_manager_napi.cpp
napi/engine_event_callback_napi.cpp
```

**关键 deps**:
```
deps: [
    "../../frameworks/native:intellvoice_native",
    "../../services:intell_voice_proxy",
]
external_deps: [
    "napi:ace_napi",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
]
```

**证据来源**: `frameworks/js/BUILD.gn`

---

### 2.5 frameworks/native/

| Target | 类型 | 描述 |
|--------|------|------|
| `intellvoice_native` | shared_library | Native API 库 (.so) |
| `innerapi_tags` | ["ndk"] | NDK 导出 |

**sources**:
```
enroll_intell_voice_engine.cpp
intell_voice_manager.cpp
wakeup_intell_voice_engine.cpp
```

**证据来源**: `frameworks/native/BUILD.gn`

---

### 2.6 utils/

| Target | 类型 | 描述 |
|--------|------|------|
| `intell_voice_utils` | shared_library | 公共工具库 (.so) |

**sources** (条件编译):
```
# 基础工具
base_thread.cpp
message_queue.cpp
task_executor.cpp

# 业务工具 (条件编译)
if (intelligent_voice_framework_engine_enable) {
    sources += ["huks_aes_adapter.cpp"]
    external_deps += ["huks:libhukssdk"]
}
```

**关键 external_deps**:
```
external_deps: [
    "ffrt:libffrt",
    "hilog:libhilog",
    "ipc:ipc_core",
    "access_token:libaccesstoken_sdk",
    "kv_store:distributeddata_inner",
]
```

**证据来源**: `utils/BUILD.gn`

---

## 3. 组件配置 (bundle.json)

```json
{
    "name": "@ohos/intelligent_voice_framework",
    "component": {
        "name": "intelligent_voice_framework",
        "subsystem": "ai",
        "syscap": ["SystemCapability.AI.IntelligentVoice.Core"],
        "sub_component": [
            "//foundation/ai/intelligent_voice_framework/services/intell_voice_service:intell_voice_server",
            "//foundation/ai/intelligent_voice_framework/services/intell_voice_trigger:intelligentvoice_trigger",
            "//foundation/ai/intelligent_voice_framework/services/intell_voice_engine:intelligentvoice_engine",
            "//foundation/ai/intelligent_voice_framework/frameworks/js:intelligentvoice",
            "//foundation/ai/intelligent_voice_framework/frameworks/native:intellvoice_native",
            "//foundation/ai/intelligent_voice_framework/utils:intell_voice_utils"
        ],
        "inner_kits": [
            {
                "name": "//foundation/ai/intelligent_voice_framework/frameworks/js:intelligentvoice_js",
                "header": {
                    "header_files": [
                        "intell_voice_manager_napi.h",
                        "intell_voice_engine_napi.h",
                        "enroll_intell_voice_engine_napi.h"
                    ],
                    "header_base": "//foundation/ai/intelligent_voice_framework/frameworks/js/napi/include"
                }
            },
            {
                "name": "//foundation/ai/intelligent_voice_framework/frameworks/native:intellvoice_native",
                "header": {
                    "header_files": [
                        "intell_voice_manager.h",
                        "i_headset_wakeup.h",
                        "wakeup_intell_voice_engine.h"
                    ],
                    "header_base": "//foundation/ai/intelligent_voice_framework/interfaces/inner_api/native"
                }
            }
        ]
    }
}
```

**证据来源**: `bundle.json`

---

## 4. 安全编译选项

所有主要 targets 启用了以下安全特性：

```gn
sanitize = {
    cfi = true              # 控制流完整性
    cfi_cross_dso = true   # 跨 DSO CFI
    cfi_vcall_icall_only = true  # 仅虚函数调用检查
    debug = false
}
branch_protector_ret = "pac_ret"  # 返回地址保护
```

**证据来源**: 各 `BUILD.gn` 文件

---

## 5. 相关文档

| 文档 | 说明 |
|------|------|
| [编译产物](./07_Artifacts.md) | 产物说明 |
| [配置开关](./appendix/Config_Flags.md) | Feature Flags |
| [目录结构](./03_Directory_Structure.md) | 模块组织 |
