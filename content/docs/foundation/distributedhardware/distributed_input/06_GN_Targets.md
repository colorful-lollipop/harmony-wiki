# GN 目标 (GN Targets)

## 目的

本文档描述 distributed_input 模块的 GN（Generate Ninja）构建目标，包括 targets 列表、类型、依赖关系、源文件和编译产物映射。

## 适用范围

本文档适用于以下场景：
- 理解模块的构建结构和依赖关系
- 进行编译定制和交叉编译
- 分析产物输出和安装路径
- 进行构建问题诊断

## 关键结论

1. **14 个共享库**: 模块生成 14 个 .so 共享库
2. **2 个 SA Profile**: Source (4809) 和 Sink (4810) System Ability 配置
3. **安全硬编译**: 所有库使用 CFI、UBSAN、栈保护等安全选项
4. **分层依赖**: 清晰的依赖层次（基础层 → 服务层 → SDK 层）
5. **统一配置**: 通过 distributedinput.gni 统一管理路径和编译选项

## GN 构建系统概述

### 构建入口

**位置**: [bundle.json](../bundle.json:55-77)

**sub_component**（17 个目标）:

```json
"sub_component": [
    "//foundation/distributedhardware/distributed_input/interfaces/inner_kits:libdinput_sdk",
    "//foundation/distributedhardware/distributed_input/sa_profile:distributed_input_source_sa_profile",
    "//foundation/distributedhardware/distributed_input/sa_profile:distributed_input_sink_sa_profile",
    "//foundation/distributedhardware/distributed_input/sa_profile:dinput.cfg",
    "//foundation/distributedhardware/distributed_input/services/source/sourcemanager:libdinput_source",
    "//foundation/distributedhardware/distributed_input/services/source/transport:libdinput_source_trans",
    "//foundation/distributedhardware/distributed_input/services/source/inputinject:libdinput_inject",
    "//foundation/distributedhardware/distributed_input/services/sink/sinkmanager:libdinput_sink",
    "//foundation/distributedhardware/distributed_input/services/sink/transport:libdinput_sink_trans",
    "//foundation/distributedhardware/distributed_input/services/sink/inputcollector:libdinput_collector",
    "//foundation/distributedhardware/distributed_input/services/transportbase:libdinput_trans_base",
    "//foundation/distributedhardware/distributed_input/services/state:libdinput_sink_state",
    "//foundation/distributedhardware/distributed_input/sourcehandler:libdinput_source_handler",
    "//foundation/distributedhardware/distributed_input/sinkhandler:libdinput_sink_handler",
    "//foundation/distributedhardware/distributed_input/inputdevicehandler:libdinput_handler",
    "//foundation/distributedhardware/distributed_input/dfx_utils:libdinput_dfx_utils",
    "//foundation/distributedhardware/distributed_input/utils:libdinput_utils"
]
```

### GNI 配置文件

**位置**: [distributedinput.gni](../distributedinput.gni:1-57)

**路径变量定义**:

| 变量 | 值 | 说明 |
|------|-----|------|
| `distributedinput_path` | `//foundation/distributedhardware/distributed_input` | 模块根路径 |
| `distributedhardwarefwk_path` | `//foundation/distributedhardware/distributed_hardware_fwk` | 分布式硬件框架路径 |
| `common_path` | `${distributedinput_path}/common` | 公共头文件路径 |
| `utils_path` | `${distributedinput_path}/utils` | 工具类路径 |
| `dfx_utils_path` | `${distributedinput_path}/dfx_utils` | DFX 工具路径 |
| `services_source_path` | `${distributedinput_path}/services/source` | Source 服务路径 |
| `services_sink_path` | `${distributedinput_path}/services/sink` | Sink 服务路径 |
| `services_state_path` | `${distributedinput_path}/services/state` | 状态管理路径 |
| `innerkits_path` | `${distributedinput_path}/interfaces/inner_kits` | Inner SDK 路径 |
| `ipc_path` | `${distributedinput_path}/interfaces/ipc` | IPC 路径 |
| `frameworks_path` | `${distributedinput_path}/frameworks` | 框架接口路径 |
| `service_common` | `${distributedinput_path}/services/common` | 服务公共路径 |
| `fwk_common_path` | `${distributedhardwarefwk_path}/common` | 框架公共路径 |

**编译选项**:

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `check_same_account` | `true` | 是否检查同账号（条件依赖） |

## 核心库目标

### 1. libdinput_sdk（公共 SDK）

**BUILD.gn 位置**: [interfaces/inner_kits/BUILD.gn](../interfaces/inner_kits/BUILD.gn:1-135)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_sdk.so`

**源文件**（39 个）:

| 源文件 | 行数 | 说明 |
|---------|------|------|
| `distributed_input_kit.cpp` | - | DistributedInputKit 实现 |
| `distributed_input_client.cpp` | - | 客户端实现，管理 Source/Sink SA 连接 |
| `dinput_sa_manager.cpp` | - | SA 管理器，查找和加载 SA |
| IPC 回调 stub/proxy（18 个） | - | Prepare/Unprepare/Start/Stop 等回调的 Stub 和 Proxy |
| 公共工具 | - | `input_check_param.cpp`、`white_list_util.cpp` |

**公共包含目录**:

```gn
public_configs = [ ":input_sdk_public_config" ]

include_dirs = [
    "${common_path}/include",
    "${frameworks_path}/include",
    "${innerkits_path}/include",
    "${innerkits_path}/src",
    "${ipc_path}/include",
    "${ipc_path}/src",
    "${utils_path}/include",
]
```

**公共外部依赖**:

| 依赖 | 用途 |
|------|------|
| `distributed_hardware_fwk:distributed_av_receiver` | 音视频接收 |
| `distributed_hardware_fwk:distributed_av_sender` | 音视频发送 |
| `distributed_hardware_fwk:distributedhardwareutils` | 分布式硬件工具 |
| `distributed_hardware_fwk:libdhfwk_sdk` | 框架 SDK |
| `json:nlohmann_json_static` | JSON 解析 |

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `access_token:libaccesstoken_sdk` | 访问令牌 |
| `access_token:libtokenid_sdk` | Token ID |
| `c_utils:utils` | C 工具 |
| `config_policy:configpolicy_util` | 配置策略 |
| `dsoftbus:softbus_client` | 软总线客户端 |
| `eventhandler:libeventhandler` | 事件处理 |
| `hilog:libhilog` | 日志 |
| `ipc:ipc_core` | IPC 核心库 |
| `libevdev:libevdev` | 输入驱动库 |
| `safwk:system_ability_fwk` | 系统能力框架 |
| `samgr:samgr_proxy` | SA 管理器代理 |

**安全编译选项**:

```gn
sanitize = {
    boundary_sanitize = true
    integer_overflow = true
    ubsan = true
    cfi = true
    cfi_cross_dso = true
    debug = false
}
branch_protector_ret = "pac_ret"

cflags = [
    "-fstack-protector-strong",
    "-D_FORTIFY_SOURCE=2",
    "-O2",
]

ldflags = [
    "-fpie",
    "-Wl,-z,relro",
    "-Wl,-z,now",
]
```

**证据**: [interfaces/inner_kits/BUILD.gn:17-134](../interfaces/inner_kits/BUILD.gn:17-134)

---

### 2. libdinput_source（Source Manager）

**BUILD.gn 位置**: [services/source/sourcemanager/BUILD.gn](../services/source/sourcemanager/BUILD.gn:1-149)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_source.so`

**源文件**（47 个）:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_source_manager.cpp` | Source Manager 实现（含 SA 注册） |
| Event Handler 实现 | `distributed_input_source_event_handler.cpp`、`dinput_source_manager_event_handler.cpp` |
| Listener 实现 | `dinput_source_listener.cpp`、`distributed_input_source_sa_cli_mgr.cpp` |
| IPC stubs/proxies | IPC Source/Sink 接口（来自 IPC 层） |
| 公共工具 | `input_check_param.cpp`、`white_list_util.cpp` |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/services/transportbase:libdinput_trans_base",
    "${innerkits_path}:libdinput_sdk",
    "${services_source_path}/inputinject:libdinput_inject",
    "${services_source_path}/transport:libdinput_source_trans",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `access_token:libaccesstoken_sdk`, `access_token:libtokenid_sdk` | 权限验证 |
| `c_utils:utils` | C 工具 |
| `distributed_hardware_fwk:distributed_av_receiver`, `distributed_av_sender` | 音视频 |
| `distributed_hardware_fwk:distributedhardwareutils`, `libdhfwk_sdk` | 框架 |
| `dsoftbus:softbus_client` | 软总线 |
| `eventhandler:libeventhandler` | 事件处理 |
| `hicollie:libhicollie` | 崩溃检测 |
| `hilog:libhilog` | 日志 |
| `hisysevent:libhisysevent` | 系统事件 |
| `hitrace:hitrace_meter` | 性能跟踪 |
| `ipc:ipc_core` | IPC |
| `json:nlohmann_json_static` | JSON |
| `libevdev:libevdev` | 输入驱动 |
| `safwk:system_ability_fwk`, `samgr:samgr_proxy` | SA 框架 |

**包含目录**（12 个）:

```gn
include_dirs = [
    "include",
    "${frameworks_path}/include",
    "${innerkits_path}/include",
    "${innerkits_path}/include/ipc",
    "${innerkits_path}/src",
    "${ipc_path}/include",
    "${ipc_path}/src",
    "${common_path}/include",
    "${service_common}/include",
    "${services_source_path}/inputinject/include",
    "${services_source_path}/transport/include",
    "${distributedinput_path}/services/transportbase/include",
    "${dfx_utils_path}/include",
    "${utils_path}/include",
    "${distributedinput_path}/inputdevicehandler/include",
]
```

**证据**: [services/source/sourcemanager/BUILD.gn:27-43](../services/source/sourcemanager/BUILD.gn:27-43)

---

### 3. libdinput_source_trans（Source 传输）

**BUILD.gn 位置**: [services/source/transport/BUILD.gn](../services/source/transport/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_source_trans.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_source_transport.cpp` | Source Transport 实现 |
| IPC source/sink proxies | Source/Sink 通信代理 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/services/transportbase:libdinput_trans_base",
    "${innerkits_path}:libdinput_sdk",
    "${services_source_path}/inputinject:libdinput_inject",
    "${utils_path}:libdinput_utils",
]
```

**证据**: [services/source/transport/BUILD.gn](../services/source/transport/BUILD.gn)

---

### 4. libdinput_inject（输入注入）

**BUILD.gn 位置**: [services/source/inputinject/BUILD.gn](../services/source/inputinject/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_inject.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_inject.cpp` | 事件注入实现 |
| `distributed_input_node_manager.cpp` | 虚拟输入节点管理 |
| `virtual_device.cpp` | 虚拟设备创建和管理 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/services/state:libdinput_sink_state",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `libevdev:libevdev` | 输入驱动库（虚拟设备写入） |
| `openssl:libcrypto_shared` | 加密库 |

**证据**: [services/source/inputinject/BUILD.gn](../services/source/inputinject/BUILD.gn)

---

### 5. libdinput_sink（Sink Manager）

**BUILD.gn 位置**: [services/sink/sinkmanager/BUILD.gn](../services/sink/sinkmanager/BUILD.gn:1-114)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_sink.so`

**源文件**（4 个）:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_sink_manager.cpp` | Sink Manager 实现（含 SA 注册） |
| `distributed_input_sink_event_handler.cpp` | Sink 事件处理器 |
| IPC sink stub | DistributedInputSinkStub |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/services/state:libdinput_sink_state",
    "${distributedinput_path}/services/transportbase:libdinput_trans_base",
    "${innerkits_path}:libdinput_sdk",
    "${services_sink_path}/inputcollector:libdinput_collector",
    "${services_sink_path}/transport:libdinput_sink_trans",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `access_token:libaccesstoken_sdk`, `access_token:libtokenid_sdk` | 权限验证 |
| `c_utils:utils` | C 工具 |
| `distributed_hardware_fwk:distributed_av_receiver`, `distributed_av_sender` | 音视频 |
| `distributed_hardware_fwk:distributedhardwareutils`, `libdhfwk_sdk` | 框架 |
| `dsoftbus:softbus_client` | 软总线 |
| `eventhandler:libeventhandler` | 事件处理 |
| `graphic_2d:librender_service_base` | 渲染服务 |
| `graphic_surface:surface` | 图形表面 |
| `hilog:libhilog` | 日志 |
| `hisysevent:libhisysevent` | 系统事件 |
| `ipc:ipc_core` | IPC |
| `json:nlohmann_json_static` | JSON |
| `libevdev:libevdev` | 输入驱动 |
| `safwk:system_ability_fwk`, `samgr:samgr_proxy` | SA 框架 |
| `window_manager:libdm` | 窗口管理器 |

**证据**: [services/sink/sinkmanager/BUILD.gn:64-95](../services/sink/sinkmanager/BUILD.gn:64-95)

---

### 6. libdinput_sink_trans（Sink 传输）

**BUILD.gn 位置**: [services/sink/transport/BUILD.gn](../services/sink/transport/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_sink_trans.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_sink_transport.cpp` | Sink Transport 实现 |
| `distributed_input_sink_switch.cpp` | Sink 传输开关 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/services/transportbase:libdinput_trans_base",
    "${utils_path}:libdinput_utils",
]
```

**证据**: [services/sink/transport/BUILD.gn](../services/sink/transport/BUILD.gn)

---

### 7. libdinput_collector（输入采集）

**BUILD.gn 位置**: [services/sink/inputcollector/BUILD.gn](../services/sink/inputcollector/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_collector.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_collector.cpp` | 事件采集器实现 |

**内部依赖**:

```gn
deps = [
    "${distributedinput_path}/services/state:libdinput_sink_state",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `libevdev:libevdev` | 输入驱动库（读取本地输入设备） |
| `openssl:libcrypto_shared` | 加密库 |

**证据**: [services/sink/inputcollector/BUILD.gn](../services/sink/inputcollector/BUILD.gn)

---

### 8. libdinput_trans_base（传输基类）

**BUILD.gn 位置**: [services/transportbase/BUILD.gn](../services/transportbase/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_trans_base.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_transport_base.cpp` | Transport 基类实现（会话管理、消息发送/接收） |
| `softbus_permission_check.cpp` | SoftBus 权限检查 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `device_manager` | 设备管理和 ACL |
| `dsoftbus:softbus_client` | 软总线 |
| `os_account` | OS 账号 |
| `c_utils:utils` | C 工具 |

**证据**: [services/transportbase/BUILD.gn](../services/transportbase/BUILD.gn)

---

### 9. libdinput_sink_state（状态管理）

**BUILD.gn 位置**: [services/state/BUILD.gn](../services/state/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_sink_state.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `dinput_sink_state.cpp` | 状态管理器实现 |
| `touchpad_event_fragment_mgr.cpp` | 触摸板事件片段管理 |
| `touchpad_event_fragment.cpp` | 触摸板事件片段定义 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/services/sink/inputcollector:libdinput_collector",
    "${distributedinput_path}/services/sink/transport:libdinput_sink_trans",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `libevdev:libevdev` | 输入驱动库 |
| `dsoftbus:softbus_client` | 软总线 |

**证据**: [services/state/BUILD.gn](../services/state/BUILD.gn)

---

### 10. libdinput_source_handler（Source Handler）

**BUILD.gn 位置**: [sourcehandler/BUILD.gn](../sourcehandler/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_source_handler.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_source_handler.cpp` | Source Handler 实现（框架集成） |
| `load_d_input_source_callback.cpp` | SA 加载回调 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/interfaces/ipc:libdinput_sdk",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `c_utils:utils` | C 工具 |
| `distributed_hardware_fwk:distributedhardwareutils`, `libdhfwk_sdk` | 框架 |
| `dsoftbus:softbus_client` | 软总线 |
| `eventhandler:libeventhandler` | 事件处理 |
| `hilog:libhilog` | 日志 |
| `ipc:ipc_core` | IPC |
| `libevdev:libevdev` | 输入驱动 |
| `safwk:system_ability_fwk`, `samgr:samgr_proxy` | SA 框架 |

**证据**: [sourcehandler/BUILD.gn](../sourcehandler/BUILD.gn)

---

### 11. libdinput_sink_handler（Sink Handler）

**BUILD.gn 位置**: [sinkhandler/BUILD.gn](../sinkhandler/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_sink_handler.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_sink_handler.cpp` | Sink Handler 实现（框架集成） |
| `load_d_input_sink_callback.cpp` | SA 加载回调 |

**内部依赖**:

```gn
deps = [
    "${dfx_utils_path}:libdinput_dfx_utils",
    "${distributedinput_path}/interfaces/ipc:libdinput_sdk",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `c_utils:utils` | C 工具 |
| `distributed_hardware_fwk:distributedhardwareutils`, `libdhfwk_sdk` | 框架 |
| `dsoftbus:softbus_client` | 软总线 |
| `eventhandler:libeventhandler` | 事件处理 |
| `hilog:libhilog` | 日志 |
| `ipc:ipc_core` | IPC |
| `libevdev:libevdev` | 输入驱动 |
| `safwk:system_ability_fwk`, `samgr:samgr_proxy` | SA 框架 |

**证据**: [sinkhandler/BUILD.gn](../sinkhandler/BUILD.gn)

---

### 12. libdinput_handler（设备能力查询）

**BUILD.gn 位置**: [inputdevicehandler/BUILD.gn](../inputdevicehandler/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_handler.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `distributed_input_handler.cpp` | 设备能力查询实现 |

**内部依赖**:

```gn
deps = [
    "${distributedinput_path}/services/state:libdinput_sink_state",
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `dsoftbus:softbus_client` | 软总线 |
| `libevdev:libevdev` | 输入驱动 |

**证据**: [inputdevicehandler/BUILD.gn](../inputdevicehandler/BUILD.gn)

---

### 13. libdinput_dfx_utils（DFX 工具）

**BUILD.gn 位置**: [dfx_utils/BUILD.gn](../dfx_utils/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_dfx_utils.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `hidumper.cpp` | HiDumper 实现（dump 信息） |
| `hisysevent_util.cpp` | HiSysEvent 工具实现 |

**内部依赖**:

```gn
deps = [
    "${utils_path}:libdinput_utils",
]
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `c_utils:utils` | C 工具 |
| `hisysevent:libhisysevent` | 系统事件 |

**证据**: [dfx_utils/BUILD.gn](../dfx_utils/BUILD.gn)

---

### 14. libdinput_utils（基础工具）

**BUILD.gn 位置**: [utils/BUILD.gn](../utils/BUILD.gn)

**目标类型**: `ohos_shared_library`

**输出文件**: `libdinput_utils.so`

**源文件**:

| 源文件 | 说明 |
|---------|------|
| `dinput_utils_tool.cpp` | 工具函数实现（设备信息、UUID、JSON 校验等） |

**内部依赖**:

```gn
deps = []  # 无内部依赖，仅依赖外部
```

**外部依赖**:

| 依赖 | 用途 |
|------|------|
| `c_utils:utils` | C 工具 |
| `dsoftbus:softbus_client` | 软总线 |
| `distributed_hardware_fwk:distributedhardwareutils` | 框架 |
| `libevdev:libevdev` | 输入驱动 |
| `openssl:libcrypto_shared` | 加密库 |

**证据**: [utils/BUILD.gn](../utils/BUILD.gn)

---

## SA Profile 目标

### 15. distributed_input_source_sa_profile（Source SA Profile）

**BUILD.gn 位置**: [sa_profile/BUILD.gn:17-21](../sa_profile/BUILD.gn:17-21)

**目标类型**: `ohos_sa_profile`

**输出**: SA Profile

**源文件**: `4809.json`

**内容**:

```json
{
    "process": "dinput",
    "systemability": [{
        "name": 4809,
        "libpath": "libdinput_source.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1,
        "min_hdi_proxy_version": []
    }]
}
```

**证据**: [sa_profile/4809.json](../sa_profile/4809.json:1-13)

---

### 16. distributed_input_sink_sa_profile（Sink SA Profile）

**BUILD.gn 位置**: [sa_profile/BUILD.gn:23-27](../sa_profile/BUILD.gn:23-27)

**目标类型**: `ohos_sa_profile`

**输出**: SA Profile

**源文件**: `4810.json`

**内容**:

```json
{
    "process": "dinput",
    "systemability": [{
        "name": 4810,
        "libpath": "libdinput_sink.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1,
        "min_hdi_proxy_version": []
    }]
}
```

**证据**: [sa_profile/4810.json](../sa_profile/4810.json:1-13)

---

### 17. dinput.cfg（配置文件）

**BUILD.gn 位置**: [sa_profile/BUILD.gn:29-34](../sa_profile/BUILD.gn:29-34)

**目标类型**: `ohos_prebuilt_etc`

**输出**: 配置文件

**源文件**: `dinput.cfg`

**内容**:

```ini
{
    "services" : [{
        "name" : "4809-4810",
        "path" : ["system/lib/libdinput_source.z.so", "system/lib/libdinput_sink.z.so"]
    }],
    "permission": [
        "ohos.permission.DISTRIBUTED_DATASYNC",
        "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE"
    ]
}
```

**安装路径**: `/system/etc/init/dinput.cfg`

**证据**: [sa_profile/dinput.cfg](../sa_profile/dinput.cfg:1-17)

---

## 目标依赖层次

### 依赖树

```
libdinput_sdk (公共 API)
├── libdinput_utils
└── 外部依赖 (access_token, dsoftbus, eventhandler, hilog, ipc, libevdev, safwk, samgr)

libdinput_source (Source Manager)
├── libdinput_dfx_utils
├── libdinput_trans_base
│   └── libdinput_utils
├── libdinput_sdk
├── libdinput_inject
│   └── libdinput_sink_state
│       └── libdinput_utils
└── libdinput_source_trans
    └── libdinput_utils
└── 外部依赖 (access_token, dsoftbus, distributed_hardware_fwk, eventhandler, hicollie, hilog, hisysevent, hitrace, ipc, json, libevdev, safwk, samgr)

libdinput_sink (Sink Manager)
├── libdinput_dfx_utils
├── libdinput_sink_state
│   ├── libdinput_dfx_utils
│   ├── libdinput_collector
│   │   └── libdinput_utils
│   └── libdinput_sink_trans
│       └── libdinput_utils
└── libdinput_trans_base
    └── libdinput_utils
├── libdinput_sdk
├── libdinput_collector
│   └── libdinput_sink_state
│       └── libdinput_utils
└── libdinput_sink_trans
    └── libdinput_utils
└── 外部依赖 (access_token, c_utils, distributed_hardware_fwk, dsoftbus, eventhandler, graphic_2d, graphic_surface, hilog, hisysevent, ipc, json, libevdev, safwk, samgr, window_manager)

libdinput_source_handler (Source Handler)
├── libdinput_dfx_utils
├── libdinput_sdk
└── libdinput_utils
└── 外部依赖 (c_utils, distributed_hardware_fwk, dsoftbus, eventhandler, hilog, ipc, libevdev, safwk, samgr)

libdinput_sink_handler (Sink Handler)
├── libdinput_dfx_utils
├── libdinput_sdk
└── libdinput_utils
└── 外部依赖 (c_utils, distributed_hardware_fwk, dsoftbus, eventhandler, hilog, ipc, libevdev, safwk, samgr)

libdinput_handler (设备能力查询)
├── libdinput_sink_state
└── libdinput_utils
└── 外部依赖 (dsoftbus, libevdev)

libdinput_dfx_utils (DFX 工具)
└── libdinput_utils
└── 外部依赖 (c_utils, hisysevent)

libdinput_utils (基础工具)
└── 外部依赖 (c_utils, dsoftbus, distributed_hardware_fwk, libevdev, openssl)
```

---

## 构建配置总结

### 共享安全配置

所有共享库（libdinput_*.so）使用相同的安全配置：

| 配置项 | 值 | 说明 |
|---------|-----|------|
| `boundary_sanitize` | `true` | 边界安全 |
| `integer_overflow` | `true` | 整数溢出检查 |
| `ubsan` | `true` | 未定义行为 Sanitizer |
| `cfi` | `true` | 控制流完整性 |
| `cfi_cross_dso` | `true` | 跨 DSO CFI |
| `debug` | `false` | 非调试版本 |
| `branch_protector_ret` | `"pac_ret"` | 返回地址保护 |

**证据**: [interfaces/inner_kits/BUILD.gn:18-36](../interfaces/inner_kits/BUILD.gn:18-36)

### 编译标志

| 编译器标志 | 值 | 说明 |
|-----------|-----|------|
| `-fstack-protector-strong` | - | 强栈保护 |
| `-D_FORTIFY_SOURCE=2` | - | 源码强化 |
| `-O2` | - | 优化级别 2 |

### 链接器标志

| 链接器标志 | 值 | 说明 |
|-----------|-----|------|
| `-fpie` | - | 位置无关可执行 |
| `-Wl,-z,relro` | - | 只读重定位 |
| `-Wl,-z,now` | - | 立即绑定 |

---

## 构建流程

### 1. 完整构建流程

```
[执行 gn gen]
      │
      ▼
[生成 ninja 构建文件]
      │
      ├───────────────────┐
      ▼                   │
[编译 14 个共享库]
      │
      ├──────────────────────┐
      ▼                    │
[生成 libdinput_*.so]
      │
      ├──────────────────────┐
      ▼                    │
[链接外部依赖]
      │
      ├──────────────────────┐
      ▼                    │
[生成最终 .so 文件]
      │
      └──────────────────────┘
```

### 2. 增量编译示例

```bash
# 编译整个模块
hb build -f distributedhardware/distributed_input

# 仅编译特定库（如修改了 Source Manager）
hb build -f distributedhardware/distributed_input --build-target libdinput_source

# 清理后重新编译
hb clean && hb build -f distributedhardware/distributed_input
```

---

## 编译产物汇总

| 产物类型 | 数量 | 产物列表 |
|----------|------|----------|
| 共享库（.so） | 14 | libdinput_sdk.so, libdinput_source.so, libdinput_sink.so, libdinput_source_trans.so, libdinput_inject.so, libdinput_sink_trans.so, libdinput_collector.so, libdinput_trans_base.so, libdinput_sink_state.so, libdinput_source_handler.so, libdinput_sink_handler.so, libdinput_handler.so, libdinput_dfx_utils.so, libdinput_utils.so |
| SA Profile | 2 | 4809.json, 4810.json |
| 配置文件 | 1 | dinput.cfg |

---

## 相关跳转

- [编译产物](07_Build_Artifacts.md) - 产物详细说明和安装路径
- [目录结构](02_Directory_Structure.md) - BUILD.gn 文件位置
- [公共 API](04_Public_API.md) - Inner SDK 接口

---

*更新时间: 2026-02-06 15:08:55*
