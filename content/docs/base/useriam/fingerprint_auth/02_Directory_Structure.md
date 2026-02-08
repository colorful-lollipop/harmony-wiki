# 目录结构与模块职责

## 目的

本文档详细说明指纹认证组件的目录结构和各模块的职责，帮助开发者快速定位代码和理解代码组织。

## 适用范围

- 需要理解代码组织的开发者
- 组件维护者
- 新接触本项目的开发者

## 关键结论

1. **目录结构**：组件分为 5 个主要目录：`common/`（公共基础设施）、`services/`（核心服务）、`services_ex/`（扩展服务）、`sa_profile/`（SA 配置）、`test/`（测试，不包含）。

2. **模块职责**：
   - `common/`：提供日志、工具类、错误码定义等公共基础设施
   - `services/`：核心 System Ability 服务（SA 943）的实现
   - `services_ex/`：提供传感器照明等扩展 UI 功能
   - `sa_profile/`：System Ability 注册配置

3. **代码统计**：
   - 头文件：26 个 `.h` 文件
   - 源文件：20 个 `.cpp` 文件（不含测试）
   - 总代码量：约 1788 行（不含注释和空行）

---

## 完整目录树

```
fingerprint_auth/
├── README.md                      # 英文文档（空）
├── README_ZH.md                   # 中文文档
├── bundle.json                    # 组件元数据
├── fingerprint_auth.gni           # GN 构建配置
├── LICENSE                        # Apache 2.0 许可证
├── OAT.xml                       # OpenHarmony 合规
├── cfi_blocklist.txt              # CFI 异常列表
├── .clang-format                  # 代码格式化配置
│
├── figures/                       # 架构图和文档图片
│   └── fingerprintauth_architecture_ZH.png
│
├── common/                        # 公共基础设施
│   ├── inc/                       # 头文件
│   │   └── fingerprint_auth_defines.h
│   ├── logs/                      # 日志模块
│   │   └── iam_logger.h
│   └── utils/                     # 工具类
│       ├── iam_check.h
│       └── iam_ptr.h
│
├── services/                      # 核心指纹认证服务
│   ├── inc/                       # 服务头文件
│   │   ├── fingerprint_auth_all_in_one_executor_hdi.h
│   │   ├── fingerprint_auth_driver_hdi.h
│   │   ├── fingerprint_auth_executor_callback_hdi.h
│   │   ├── fingerprint_auth_hdi.h
│   │   ├── fingerprint_auth_interface_adapter.h
│   │   ├── fingerprint_auth_service.h
│   │   ├── isa_command_processor.h
│   │   ├── isensor_illumination_task.h
│   │   ├── memory_guard.h
│   │   ├── sa_command_manager.h
│   │   ├── sensor_illumination_manager.h
│   │   └── service_ex_manager.h
│   ├── src/                       # 服务实现
│   │   ├── fingerprint_auth_all_in_one_executor_hdi.cpp
│   │   ├── fingerprint_auth_driver_hdi.cpp
│   │   ├── fingerprint_auth_executor_callback_hdi.cpp
│   │   ├── fingerprint_auth_interface_adapter.cpp
│   │   ├── fingerprint_auth_service.cpp
│   │   ├── memory_guard.cpp
│   │   ├── sa_command_manager.cpp
│   │   ├── sensor_illumination_manager.cpp
│   │   └── service_ex_manager.cpp
│   └── BUILD.gn                  # 构建配置
│
├── services_ex/                   # 扩展服务（传感器照明）
│   ├── inc/                       # 扩展服务头文件
│   │   ├── screen_state_monitor.h
│   │   └── sensor_illumination_task.h
│   ├── src/                       # 扩展服务实现
│   │   ├── screen_state_monitor.cpp
│   │   └── sensor_illumination_task.cpp
│   └── BUILD.gn                  # 构建配置
│
├── sa_profile/                    # System Ability 配置
│   ├── 943.json                  # SA ID 943 配置
│   └── BUILD.gn                  # SA 配置构建
│
└── test/                          # 测试目录（本文档不覆盖）
    ├── unittest/                   # 单元测试
    └── fuzztest/                  # 模糊测试
```

---

## 模块详细说明

### 1. common/ - 公共基础设施

**目录结构**：
```
common/
├── inc/fingerprint_auth_defines.h
├── logs/iam_logger.h
└── utils/
    ├── iam_check.h
    └── iam_ptr.h
```

**职责**：提供跨模块的公共基础设施，包括日志、工具类和错误码定义。

#### 1.1 common/inc/fingerprint_auth_defines.h

**文件路径**：`common/inc/fingerprint_auth_defines.h`

**功能**：定义指纹认证服务的错误码枚举。

**关键内容**：
```cpp
namespace OHOS {
namespace UserIam {
namespace FingerprintAuth {
enum ResultCode {
    SUCCESS = 0,
    FAIL = 1,
    GENERAL_ERROR = 2,
    CANCELED = 3,
    TIMEOUT = 4,
    TYPE_NOT_SUPPORT = 5,
    TRUST_LEVEL_NOT_SUPPORT = 6,
    BUSY = 7,
    INVALID_PARAMETERS = 8,
    LOCKED = 9,
    NOT_ENROLLED = 10,
    OPERATION_NOT_SUPPORT = 11,
    VENDOR_RESULT_CODE_BEGIN = 10000
};
} // namespace FingerprintAuth
} // namespace UserIam
} // namespace OHOS
```

**证据**：
- 文件：`common/inc/fingerprint_auth_defines.h:24-77`
- 行数：82 行（含注释）

#### 1.2 common/logs/iam_logger.h

**文件路径**：`common/logs/iam_logger.h`

**功能**：提供统一的日志宏，基于 OpenHarmony Hilog 系统。

**关键宏**：
```cpp
#define IAM_LOGD(fmt, ...)  // Debug 日志
#define IAM_LOGI(fmt, ...)  // Info 日志
#define IAM_LOGW(fmt, ...)  // Warning 日志
#define IAM_LOGE(fmt, ...)  // Error 日志
```

**日志标签**：
- 服务层：`FINGERPRINT_AUTH_SA`
- 驱动层：`FINGERPRINT_AUTH_DRIVER`
- 回调层：`FINGERPRINT_AUTH_CALLBACK`

**证据**：
- 文件：`common/logs/iam_logger.h`

#### 1.3 common/utils/iam_check.h

**文件路径**：`common/utils/iam_check.h`

**功能**：提供参数验证宏，用于防御性编程。

**关键宏**：
```cpp
#define IF_FALSE_LOGE_AND_RETURN_VAL(condition, return_value)
#define IF_FALSE_LOGE_AND_RETURN(condition)
```

**用途**：在代码中快速进行参数检查和错误返回：
```cpp
IF_FALSE_LOGE_AND_RETURN_VAL(param == nullptr, ResultCode::INVALID_PARAMETERS);
```

**证据**：
- 文件：`common/utils/iam_check.h:25-39`

#### 1.4 common/utils/iam_ptr.h

**文件路径**：`common/utils/iam_ptr.h`

**功能**：提供智能指针工具函数。

**关键函数**：
```cpp
template<typename T, typename... Args>
std::shared_ptr<T> MakeShared(Args&&... args);

template<typename T, typename... Args>
std::unique_ptr<T> MakeUnique(Args&&... args);
```

**用途**：创建具有内存安全检查的智能指针。

---

### 2. services/ - 核心指纹认证服务

**目录结构**：
```
services/
├── inc/                          # 头文件
│   ├── fingerprint_auth_all_in_one_executor_hdi.h      # All-in-One 执行器
│   ├── fingerprint_auth_driver_hdi.h                  # HDI 驱动适配器
│   ├── fingerprint_auth_executor_callback_hdi.h          # 执行器回调
│   ├── fingerprint_auth_hdi.h                        # HDI 类型别名
│   ├── fingerprint_auth_interface_adapter.h             # HDI 接口适配器
│   ├── fingerprint_auth_service.h                     # 主服务类
│   ├── isa_command_processor.h                       # SA 命令处理器接口
│   ├── isensor_illumination_task.h                   # 传感器照明任务接口
│   ├── memory_guard.h                                # 内存保护 RAII
│   ├── sa_command_manager.h                          # SA 命令管理器
│   ├── sensor_illumination_manager.h                 # 传感器照明管理器
│   └── service_ex_manager.h                         # 扩展服务管理器
│
├── src/                          # 实现文件
│   ├── fingerprint_auth_all_in_one_executor_hdi.cpp     # 431 行
│   ├── fingerprint_auth_driver_hdi.cpp                 # 96 行
│   ├── fingerprint_auth_executor_callback_hdi.cpp         # 141 行
│   ├── fingerprint_auth_interface_adapter.cpp            # 31 行
│   ├── fingerprint_auth_service.cpp                    # 109 行
│   ├── memory_guard.cpp                               # 42 行
│   ├── sa_command_manager.cpp                          # 98 行
│   ├── sensor_illumination_manager.cpp                 # 206 行
│   └── service_ex_manager.cpp                         # 79 行
│
└── BUILD.gn                     # 构建配置
```

**职责**：实现核心指纹认证 System Ability 服务（SA 943）。

#### 2.1 核心类说明

| 类 | 头文件 | 实现文件 | 职责 |
|-----|---------|-----------|------|
| `FingerprintAuthService` | `fingerprint_auth_service.h` | `fingerprint_auth_service.cpp` | 主 System Ability 类，单例模式，管理服务生命周期 |
| `FingerprintAuthDriverHdi` | `fingerprint_auth_driver_hdi.h` | `fingerprint_auth_driver_hdi.cpp` | HDI 驱动适配器，管理执行器列表 |
| `FingerprintAllInOneExecutorHdi` | `fingerprint_auth_all_in_one_executor_hdi.h` | `fingerprint_auth_all_in_one_executor_hdi.cpp` | All-in-One 执行器实现，提供所有认证操作 |
| `FingerprintAuthExecutorCallbackHdi` | `fingerprint_auth_executor_callback_hdi.h` | `fingerprint_auth_executor_callback_hdi.cpp` | HDI 回调适配器，转换回调数据格式 |
| `FingerprintAuthInterfaceAdapter` | `fingerprint_auth_interface_adapter.h` | `fingerprint_auth_interface_adapter.cpp` | HDI 接口获取适配器 |
| `SaCommandManager` | `sa_command_manager.h` | `sa_command_manager.cpp` | SA 命令管理器，注册和分发命令处理器 |
| `SensorIlluminationManager` | `sensor_illumination_manager.h` | `sensor_illumination_manager.cpp` | 传感器照明管理器，协调照明状态 |
| `ServiceExManager` | `service_ex_manager.h` | `service_ex_manager.cpp` | 扩展服务管理器，动态加载 services_ex 库 |
| `MemoryGuard` | `memory_guard.h` | `memory_guard.cpp` | 内存保护 RAII 包装器 |

#### 2.2 接口说明

| 接口 | 头文件 | 用途 |
|------|---------|------|
| `ISaCommandProcessor` | `isa_command_processor.h` | SA 命令处理器接口，由 `SaCommandManager` 管理 |
| `ISensorIlluminationTask` | `isensor_illumination_task.h` | 传感器照明任务接口，由 `SensorIlluminationManager` 管理 |

#### 2.3 BUILD.gn 配置

**文件路径**：`services/BUILD.gn`

**主要 Target**：
- `fingerprintauthservice_source_set`：源码集合，编译核心服务
- `fingerprintauthservice`：共享库，输出 `libfingerprintauthservice.z.so`

**包含目录**：
```gn
include_dirs = [
    "inc",
    "../common/inc",
    "../common/logs",
    "../common/utils",
]
```

**外部依赖**：
- `drivers_interface_fingerprint_auth:libfingerprint_auth_proxy_2.0`
- `user_auth_framework:userauth_executors`
- `safwk:system_ability_fwk`
- `samgr:samgr_proxy`
- `hilog:libhilog`
- `ipc:ipc_core`
- `hdf_core:libhdf_utils`

**证据**：
- 文件：`services/BUILD.gn:14-99`

---

### 3. services_ex/ - 扩展服务

**目录结构**：
```
services_ex/
├── inc/                          # 头文件
│   ├── screen_state_monitor.h                          # 屏幕状态监听器
│   └── sensor_illumination_task.h                   # 传感器照明任务
│
├── src/                          # 实现文件
│   ├── screen_state_monitor.cpp                        # 199 行
│   └── sensor_illumination_task.cpp                  # 356 行
│
└── BUILD.gn                     # 构建配置
```

**职责**：提供扩展功能，主要是传感器照明 UI。

#### 3.1 核心类说明

| 类 | 头文件 | 实现文件 | 职责 |
|-----|---------|-----------|------|
| `ScreenStateMonitor` | `screen_state_monitor.h` | `screen_state_monitor.cpp` | 监听屏幕开/关事件（`COMMON_EVENT_SCREEN_ON/OFF`） |
| `SensorIlluminationTask` | `sensor_illumination_task.h` | `sensor_illumination_task.cpp` | 实现传感器照明 UI，使用 Rosen 渲染服务 |

#### 3.2 BUILD.gn 配置

**文件路径**：`services_ex/BUILD.gn`

**主要 Target**：
- `fingerprintauthservice_ex_source_set`：源码集合
- `fingerprintauthservice_ex`：共享库，输出 `libfingerprintauthservice_ex.z.so`

**包含目录**：
```gn
include_dirs = [
    "inc",
    "../services/inc",
    "../common/inc",
    "../common/logs",
    "../common/utils",
]
```

**外部依赖**（图形相关）：
- `egl:libEGL`
- `graphic_2d:libcomposer`
- `graphic_2d:librender_service_base`
- `graphic_2d:librender_service_client`
- `graphic_surface:buffer_handle`
- `graphic_surface:surface_headers`
- `opengles:libGLES`
- `skia:skia_canvaskit`
- `window_manager:libdm`

**特性开关**：
```gn
if (use_display_manager_component) {
    external_deps += [ "display_manager:displaymgr" ]
    defines += [ "CONFIG_USE_DISPLAY_MANAGER_COMPONENT" ]
}

if (use_power_manager_component) {
    external_deps += [ "power_manager:powermgr_client" ]
    defines += [ "CONFIG_USE_POWER_MANAGER_COMPONENT" ]
}
```

**导出符号**：
- `GetSensorIlluminationTask*`：工厂函数，导出给 `services` 层使用

**证据**：
- 文件：`services_ex/BUILD.gn:14-113`

---

### 4. sa_profile/ - System Ability 配置

**目录结构**：
```
sa_profile/
├── 943.json                      # SA ID 943 配置
└── BUILD.gn                      # SA 配置构建
```

**职责**：定义 System Ability 的注册配置。

#### 4.1 943.json 配置

**文件路径**：`sa_profile/943.json`

**内容**：
```json
{
    "process": "useriam",
    "systemability": [
        {
            "name": 943,
            "libpath": "libfingerprintauthservice.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": ["libfingerprint_auth_proxy_2.0.z.so"]
        }
    ]
}
```

**配置说明**：

| 字段 | 值 | 说明 |
|------|-----|------|
| `process` | `useriam` | 运行的系统进程名称 |
| `name` | `943` | System Ability ID（唯一标识符） |
| `libpath` | `libfingerprintauthservice.z.so` | 加载的共享库文件名 |
| `run-on-create` | `true` | 系统启动时立即创建服务 |
| `distributed` | `false` | 不支持分布式 |
| `dump_level` | `1` | Dump 级别 |
| `min_hdi_proxy_version` | `libfingerprint_auth_proxy_2.0.z.so` | 最低 HDI 代理版本要求 |

**SA 注册代码**：
```cpp
// services/src/fingerprint_auth_service.cpp:51
const bool REGISTER_RESULT = SystemAbility::MakeAndRegisterAbility(
    FingerprintAuthService::GetInstance().get()
);
```

**SA 构造函数**：
```cpp
// services/src/fingerprint_auth_service.cpp:68
FingerprintAuthService::FingerprintAuthService()
    : SystemAbility(SUBSYS_USERIAM_SYS_ABILITY_FINGERPRINTAUTH, true)
{
}
```

**证据**：
- 文件：`sa_profile/943.json`
- 注册：`services/src/fingerprint_auth_service.cpp:51`
- 构造：`services/src/fingerprint_auth_service.cpp:68`

---

## 代码统计（不含测试）

| 类型 | 数量 | 总行数 |
|------|------|--------|
| 头文件 (.h) | 26 | ~1000 |
| 源文件 (.cpp) | 20 | 1788 |
| BUILD.gn 文件 | 4 | ~240 |
| 配置文件 | 3 | ~100 |
| **总计** | **53** | **~3128** |

**源文件详细行数**：
```
     431  services/src/fingerprint_auth_all_in_one_executor_hdi.cpp
      96  services/src/fingerprint_auth_driver_hdi.cpp
     141  services/src/fingerprint_auth_executor_callback_hdi.cpp
      31  services/src/fingerprint_auth_interface_adapter.cpp
     109  services/src/fingerprint_auth_service.cpp
      42  services/src/memory_guard.cpp
      98  services/src/sa_command_manager.cpp
     206  services/src/sensor_illumination_manager.cpp
      79  services/src/service_ex_manager.cpp
     199  services_ex/src/screen_state_monitor.cpp
     356  services_ex/src/sensor_illumination_task.cpp
   ─────
    1788  总计
```

---

## 模块依赖关系

### 依赖图

```
┌─────────────────────────────────────────────┐
│          FingerprintAuthService            │
│         (fingerprint_auth_service)        │
│         SA 943, 单例模式                │
└──────────────┬──────────────────────────┘
               │
      ┌────────┴────────┐
      │                 │
┌─────▼─────┐   ┌───▼──────────────────────┐
│ DriverHdi  │   │ SaCommandManager        │
│            │   │                      │
│ All-in-One │   │ SensorIlluminationMgr  │
│ Executor   │   │                      │
└─────┬─────┘   └───┬──────────────────────┘
      │              │
      │              │
      │         ┌────▼─────────┐
      │         │ ServiceExMgr  │
      │         │ (dlopen ex)  │
      │         └────┬─────────┘
      │              │
      │         ┌────▼──────────────────────┐
      │         │ SensorIlluminationTask   │
      │         │ (Rosen 渲染)          │
      │         └─────────────────────────┘
      │
┌─────▼──────────────────────────────────┐
│         HDF Driver Interface          │
│  (drivers_interface_fingerprint_auth) │
└─────────────────────────────────────────┘
```

### 跨目录依赖

| 源目录 | 依赖目录 | 依赖内容 |
|---------|---------|---------|
| `services/` | `common/` | 日志、工具类、错误码定义 |
| `services_ex/` | `common/` | 日志、工具类、错误码定义 |
| `services_ex/` | `services/` | `FingerprintAuthService`、`SensorIlluminationManager` |
| `services/` | `services_ex/` | 通过 `ServiceExManager` 动态加载 |

---

## 文件命名规范

### 头文件命名

| 命名模式 | 示例 | 说明 |
|----------|------|------|
| `fingerprint_auth_*.h` | `fingerprint_auth_service.h` | 服务核心组件 |
| `fingerprint_auth_*_hdi.h` | `fingerprint_auth_driver_hdi.h` | HDI 相关类 |
| `*_manager.h` | `sensor_illumination_manager.h` | 管理器类 |
| `*_task.h` | `sensor_illumination_task.h` | 任务类 |
| `*_monitor.h` | `screen_state_monitor.h` | 监听器类 |
| `i*_*.h` | `isa_command_processor.h` | 接口类（Interface） |

### 源文件命名

- 源文件与头文件一一对应，同名但扩展名为 `.cpp`
- 接口类 `ISaCommandProcessor` 的实现通常在包含它的管理器中

### BUILD.gn 命名

| Target 类型 | 命名模式 | 示例 |
|------------|-----------|------|
| source_set | `{module}_source_set` | `fingerprintauthservice_source_set` |
| shared_library | `{module}` | `fingerprintauthservice` |
| sa_profile | `{module}_sa_profile` | `fingerprintauth_sa_profile` |

---

## 相关跳转

- [01_Overview.md](./01_Overview.md) - 组件全貌
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [05_Internal_APIs.md](./05_Internal_APIs.md) - 内部 API 详解
- [06_GN_Targets.md](./06_GN_Targets.md) - 构建配置详解
