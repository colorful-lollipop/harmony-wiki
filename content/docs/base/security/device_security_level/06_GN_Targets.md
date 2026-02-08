# GN Targets 与编译产物

## 目的

本文档详细说明 DSLM 模块的 GN 构建系统，包括所有关键 targets、依赖关系和编译产物。

## 适用范围

- ✅ 所有关键 targets 列表（排除 test）
- ✅ target 依赖关系图
- ✅ 编译产物与安装路径
- ✅ 编译配置与特性开关

## GN 配置文件

### 根配置文件

#### dslm.gni

**路径**：`common/dslm.gni`

**定义的路径变量**：
```gn
dslm_hilog_path = "//base/hiviewdfx/hilog_lite/frameworks"
dslm_dsoftbus_path = "//foundation/communication/dsoftbus"
dslm_common_lib_path = "//commonlibrary/utils_lite"
dslm_ipc_path = "//foundation/communication/ipc"
dslm_samgr_path = "//foundation/systemabilitymgr"
dslm_lite_component_path = "//build/lite/config/component"
```

### Bundle.json

**路径**：`bundle.json`

**组件信息**：
- **名称**：`@ohos/device_security_level`
- **版本**：3.0.0
- **子系统**：security
- **ROM**：200KB
- **RAM**：2500KB

**编译组**：
- `fwk_group`：`interfaces/inner_api:fwk_group`
- `service_group`：`oem_property/ohos:dslm_service`, `profile:dslm_service.rc`

## 关键 Targets

### 1. dslm_sdk - 对外 SDK

| 属性 | 值 | 证据 |
|------|-----|------|
| **类型** | ohos_shared_library | `interfaces/inner_api/BUILD.gn:101` |
| **输出名** | libdslm_sdk.z.so | 推导出 |
| **源文件（Standard）** | 5 个 cpp 文件 | L112-118 |
| **源文件（Small）** | 2 个 c 文件 | L34-37 |
| **源文件（Mini）** | 2 个 c 文件 | L67-70 |
| **外部依赖（Standard）** | c_utils, hilog, ipc, samgr | L129-134 |
| **外部依赖（Lite）** | hilog, ipc, samgr | L55-58 |

### 2. dslm_service - SA 服务

| 属性 | 值 | 证据 |
|------|-----|------|
| **类型** | ohos_shared_library (shlib_type="sa") | `services/sa/BUILD.gn:148` |
| **输出名** | libdslm_service.z.so | 推导出 |
| **源文件（Standard）** | 4 个文件 | L162-167 |
| **源文件（Small）** | 4 个文件 | L54-59 |
| **源文件（Mini）** | 4 个文件 | L112-116 |
| **deps** | service_dslm_obj, service_msg_obj, dslm_oem_ext 等 | L177-184 |
| **外部依赖** | c_utils, hilog, ipc, safwk, samgr | L197-203 |

### 3. service_dslm_obj - DSLM 核心对象

| 属性 | 值 | 证据 |
|------|-----|------|
| **类型** | ohos_source_set | `services/dslm/BUILD.gn:36` |
| **源文件** | 7 个 C 文件 | L17-25 |
| **defines** | MAX_SEND_TIMES=5, SEND_MSG_TIMEOUT_LEN=40000 | L64-67 |

### 4. utils_static - 工具库

| 属性 | 值 | 证据 |
|------|-----|------|
| **类型** | ohos_static_library | `baselib/utils/BUILD.gn` |
| **源文件** | 9 个 C/C++ 文件 | 推断 |

### 5. messenger_static - 消息库

| 属性 | 值 | 证据 |
|------|-----|------|
| **类型** | ohos_static_library | `baselib/msglib/BUILD.gn` |
| **源文件** | 5 个 C/C++ 文件 | 推断 |

### 6. dslm_oem_ext - OEM 扩展

| 属性 | 值 | 证据 |
|------|-----|------|
| **类型** | ohos_source_set | `oem_property/BUILD.gn:20` |
| **源文件** | 1 个 C 文件 | L31 |

## 依赖关系图

```
dslm_sdk (对外 SDK)
    │
    ├── ipc:ipc_core
    ├── samgr:samgr_proxy
    ├── hilog:libhilog
    └── c_utils:utils
        │
        ├── utils_static
        └── messenger_static
            │
            └── hilog:libhilog

dslm_service (SA 服务)
    │
    ├── service_dslm_obj
    │   └── utils_static
    ├── service_msg_obj
    │   ├── messenger_static
    │   └── utils_static
    ├── dslm_oem_ext
    │   ├── dslm_ohos_cred_obj
    │   │   └── utils_static
    │   └── oem_common_obj
    │       └── utils_static
    ├── service_common_obj
    │   └── utils_static
    └── dslm_extension_dfx
        └── utils_static
    │
    ├── safwk:system_ability_fwk
    ├── ipc:ipc_core
    ├── samgr:samgr_proxy
    ├── hilog:libhilog
    └── c_utils:utils
```

## 编译产物

### 主要产物文件

| Target | 产物文件 | 安装路径 | 说明 |
|--------|----------|----------|------|
| **dslm_sdk** | libdslm_sdk.z.so | /system/lib64/ 或 /usr/lib/ | 对外共享库 |
| **dslm_service** | libdslm_service.z.so | /system/lib64/ 或 /system/lib/ | SA 共享库 |
| **dslm_server** | dslm_server (可执行) | /system/bin/ 或 /usr/bin/ | Lite 系统服务进程 |
| **utils_static** | libdslm_utils.a | 内部静态库 | 不安装 |
| **messenger_static** | libdslm_messenger.a | 内部静态库 | 不安装 |

### 配置文件产物

| 文件 | 产物 | 安装路径 |
|------|------|----------|
| **dslm_service.xml** | XML 配置 | /system/profile/ |
| **dslm_service.json** | JSON 配置 | /system/profile/ |
| **dslm_service.cfg** | 权限配置 | /etc/permissions/ |
| **dslm_service.rc** | 启动脚本 | /etc/init/ |

## 运行时加载关系

```
系统启动
    │
    ▼
init 进程（根据 dslm_service.rc）
    │
    ▼
SAMgr（系统服务管理器）
    │ 读取 dslm_service.xml/dslm_service.json
    │
    ▼
加载 libdslm_service.z.so（SA 3511）
    │
    ▼
┌─────────────────────────────────────────────┐
│      DslmService (OnStart)            │
│  - 初始化独立线程                           │
│  - 加载插件（PLUGIN_SO_PATH）              │
│  - 发布服务（Publish）                    │
│  - 设置自动卸载定时器                      │
└─────────────────────────────────────────────┘
    │
    ▼
等待 IPC 请求（10秒无请求则自动卸载）
    │
    ▼
应用进程链接 libdslm_sdk.z.so
    │
    ▼
┌─────────────────────────────────────────────┐
│  Client SDK (通过 IPC Proxy)              │
└─────────────────────────────────────────────┘
```

## 编译配置与特性开关

### 编译配置

#### 安全加固（Standard 版本）

```gn
sanitize = {
  integer_overflow = true
  ubsan = true
  boundary_sanitize = true
  cfi = true
  cfi_cross_dso = true
  blocklist = "../../cfi_blocklist.txt"
}
branch_protector_ret = "pac_ret"
```

**证据**：`interfaces/inner_api/BUILD.gn:102-109`, `services/sa/BUILD.gn:152-159`

#### 特性开关（Feature Flags）

| 特性 | 默认值 | 说明 | 声明位置 |
|------|---------|------|----------|
| **device_security_level_feature_cred_level** | 1 | 凭据安全等级（1-5） | `oem_property/ohos/standard/BUILD.gn` |
| **device_security_level_feature_plugin_path** | "" | 插件 SO 路径 | `services/sa/BUILD.gn` |
| **device_security_level_feature_secondary_session_name** | "" | 次级会话名称 | `services/msg/BUILD.gn` |

## 关键结论

1. **主要 targets**：dslm_sdk（对外）、dslm_service（SA）、service_dslm_obj（核心）、utils_static（工具）
2. **编译产物**：libdslm_sdk.z.so、libdslm_service.z.so（以及 Lite 系统的可执行文件）
3. **依赖关系**：清晰分层，无循环依赖
4. **安全加固**：Standard 版本启用 CFI、UBSan、Integer Overflow Sanitize
5. **SA ID 3511**：作为系统服务运行，支持自动卸载和插件加载

## 相关跳转

- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [00_Overview.md](./00_Overview.md) - 项目概览
- [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 特性开关详解
