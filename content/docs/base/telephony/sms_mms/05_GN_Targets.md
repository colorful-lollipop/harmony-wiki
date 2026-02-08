# GN 目标与编译产物

## 目的

本文档详细说明短彩信模块的 GN 构建系统，包括所有 targets、编译配置、依赖关系和输出产物映射，帮助构建工程师理解和定制编译配置。

## 适用范围

本文档覆盖：
- 所有 BUILD.gn 和 .gni 文件分析
- 主要 targets 列表和分类
- target 依赖关系图
- 编译产物清单和安装路径
- 特性开关影响范围

不包含：
- 测试 targets（test/）见原文件
- 详细 GN 语法说明（见官方文档）

## GN 配置文件

### 根配置文件

#### smsmms.gni - 特性开关配置

**文件位置**: `smsmms.gni:14-29`

```gni
declare_args() {
    sms_mms_dynamic_start = false      # 动态启动服务
    sms_mms_tel_power_mode = false      # 省电模式优化
    sms_mms_feature_support_mms = true  # MMS 支持
    sms_mms_satellite = false            # 卫星短信支持
}

global_defines = []

if (sms_mms_tel_power_mode) {
    global_defines += [ "BASE_POWER_IMPROVEMENT_FEATURE" ]
}

if (sms_mms_satellite) {
    global_defines += [ "SMS_MMS_SATELLITE" ]
}
```

### bundle.json - 组件元数据

**文件位置**: `bundle.json:1-102`

```json
{
    "name": "@ohos/sms_mms",
    "version": "4.0",
    "component": {
        "name": "sms_mms",
        "subsystem": "telephony",
        "syscap": ["SystemCapability.Telephony.SmsMms"],
        "features": [
            "sms_mms_dynamic_start",
            "sms_mms_tel_power_mode",
            "sms_mms_feature_support_mms",
            "sms_mms_satellite"
        ],
        "build": {
            "sub_component": [
                "//base/telephony/sms_mms:tel_sms_mms",
                "//base/telephony/sms_mms/frameworks/native:tel_sms_mms_api",
                "//base/telephony/sms_mms/frameworks/cj:cj_sms_ffi",
                "//base/telephony/sms_mms/frameworks/js/napi/:sms",
                "//base/telephony/sms_mms/sa_profile:sms_mms_sa_profile",
                "//base/telephony/sms_mms/services/etc:sms_short_code_rules"
            ]
        }
    }
}
```

## 主 Build Targets

### 1. tel_sms_mms - 主共享库

**文件位置**: `BUILD.gn:36-210`

**target 类型**: `ohos_shared_library`

**输出**: `libtel_sms_mms.so`

**源文件**: 67+ C++ 文件

| 类别 | 文件列表 | 说明 |
|------|---------|------|
| **GSM SMS** | `gsm_sms_sender.cpp`, `gsm_sms_receive_handler.cpp`, `gsm_sms_message.cpp`, `gsm_sms_param_codec.cpp`, `gsm_sms_tpdu_codec.cpp`, `gsm_sms_param_decode.cpp`, `gsm_sms_param_encode.cpp`, `gsm_user_data_encode.cpp`, `gsm_user_data_decode.cpp`, `gsm_user_data_pdu.cpp`, `gsm_cb_codec.cpp`, `gsm_cb_gsm_codec.cpp`, `gsm_cb_umts_codec.cpp`, `gsm_cb_pdu_decode_buffer.cpp`, `gsm_cb_handler.cpp`, `cb_start_ability.cpp`, `gsm_sms_common_utils.cpp` | GSM 网络短信处理（18 个文件）|
| **CDMA SMS** | `cdma_sms_sender.cpp`, `cdma_sms_receive_handler.cpp`, `cdma_sms_message.cpp`, `cdma_sms_parameter_record.cpp`, `cdma_sms_sub_parameter.cpp`, `cdma_sms_teleservice_message.cpp`, `cdma_sms_transport_message.cpp` | CDMA 网络短信处理（7 个文件）|
| **IMS 交互** | `ims_sms_client.cpp`, `ims_sms_proxy.cpp`, `ims_sms_callback_stub.cpp`, `ims_reg_state_callback_stub.cpp` | IMS 服务交互（4 个文件）|
| **卫星交互** | `satellite_sms_client.cpp`, `satellite_sms_proxy.cpp`, `satellite_sms_callback_stub.cpp` | 卫星服务交互（3 个文件，可选）|
| **WAP Push** | `sms_wap_push_handler.cpp`, `sms_wap_push_buffer.cpp`, `sms_wap_push_content_type.cpp` | WAP Push 处理（3 个文件）|
| **小区广播** | `sms_cell_broadcast_handler.cpp` | 小区广播处理（1 个文件）|
| **管理器** | `sms_interface_manager.cpp`, `sms_send_manager.cpp`, `sms_receive_manager.cpp`, `sms_misc_manager.cpp`, `sms_network_policy_manager.cpp` | 核心管理器（5 个文件）|
| **IPC Stub** | `sms_interface_stub.cpp`, `sms_send_short_message_proxy.cpp`, `sms_delivery_short_message_proxy.cpp` | IPC 实现（3 个文件）|
| **数据结构** | `sms_base_message.cpp`, `sms_send_indexer.cpp`, `sms_receive_indexer.cpp`, `sms_pdu_buffer.cpp`, `sms_state_handler.cpp`, `sms_state_observer.cpp`, `sms_broadcast_subscriber_receiver.cpp` | 数据结构和状态（7 个文件）|
| **服务入口** | `sms_service.cpp`, `sms_dump_helper.cpp` | 服务实现和 Dump 辅助（2 个文件）|
| **工具** | `sms_common_utils.cpp`, `sms_policy_utils.cpp`, `string_utils.cpp`, `text_coder.cpp`, `sms_hisysevent.cpp` | 通用工具（5 个文件）|

**依赖**:

| 类型 | 名称 | 用途 |
|------|------|------|
| internal deps | `frameworks/native:tel_sms_mms_api` | Native API 库 |
|  | `frameworks/native/mms:mms_native_source` | MMS 源代码集 |
| external deps | `ability_base:want`, `ability_base:zuri` | Ability 框架 |
|  | `ability_runtime:ability_manager` | Ability 管理器 |
|  | `access_token:libtokenidkit` | 访问令牌 |
|  | `bundle_framework:bundlefwk_core` | Bundle 框架 |
|  | `c_utils:utils` | C 工具库 |
|  | `common_event_service:cesfwk_innerkits` | 公共事件服务 |
|  | `core_service:libtel_common`, `core_service:tel_core_service_api` | 核心服务 |
|  | `curl:curl_shared` | HTTP 客户端（MMS）|
|  | `data_share:datashare_consumer` | 数据共享 |
|  | `eventhandler:libeventhandler` | 事件处理 |
|  | `ffrt:libffrt` | FFRT 线程库 |
|  | `hilog:libhilog` | 日志 |
|  | `hisysevent:libhisysevent` | 系统事件 |
|  | `icu:shared_icui18n`, `icu:shared_icuuc` | 国际化 |
|  | `ipc:ipc_single` | IPC 框架 |
|  | `libphonenumber:geocoding`, `libphonenumber:phonenumber_standard` | 电话号码 |
|  | `netmanager_base:net_conn_manager_if` | 网络管理 |
|  | `netstack:http_client` | HTTP 客户端 |
|  | `os_account:os_account_innerkits` | 账户 |
|  | `power_manager:power_ffrt` | 电源管理 |
|  | `protobuf:protobuf` | Protocol Buffers |
|  | `resource_management:global_resmgr` | 资源管理 |
|  | `safwk:system_ability_fwk` | 系统能力框架 |
|  | `samgr:samgr_proxy` | 系统能力管理器 |
|  | `telephony_data:tel_telephony_data` | 电话数据 |

**编译定义**:

```cpp
defines = [
    "TELEPHONY_LOG_TAG = \"SmsMms\"",
    "LOG_DOMAIN = 0xD001F06",
    "LOG_TAG = \"SmsMms\""
]
defines += global_defines
if (sms_mms_feature_support_mms) {
    defines += [ "SMS_SUPPORT_MMS" ]
}
```

**安全配置**:

```cpp
sanitize = {
    cfi = true                     // 控制流完整性
    cfi_cross_dso = true          // 跨 DSO CFI
    debug = false
}
branch_protector_ret = "pac_ret"  // 指针认证返回保护
cflags_cc = [
    "-D_FORTIFY_SOURCE=2",     // Fortify 源
    "-Os",                       // 优化大小
]
```

### 2. tel_sms_mms_api - API 库

**文件位置**: `frameworks/native/BUILD.gn`

**target 类型**: `ohos_shared_library`

**输出**: `libtel_sms_mms_api.so`

**源文件**:

| 目录 | 源文件 |
|------|-------|
| `sms/` | `short_message.cpp` (8 个文件）|
| `mms/` | 17 个 MMS 编解码文件 |

**依赖**:

| 类型 | 名称 | 说明 |
|------|------|------|
| internal deps | `sms_native_source` | SMS 源代码集 |
|  | `mms_native_source` | MMS 源代码集 |
| external deps | `c_utils:utils` | C 工具 |
|  | `hilog:libhilog` | 日志 |
|  | `icu:shared_icui18n` | 国际化 |
|  | `libphonenumber:phonenumber_standard` | 电话号码 |

**版本脚本**: `tel_sms_mms_api.versionscript`

用于控制符号导出可见性。

### 3. sms_native_source - SMS 源代码集

**文件位置**: `frameworks/native/sms/BUILD.gn`

**target 类型**: `ohos_source_set`

**源文件**: 8 个文件

| 文件 | 说明 |
|------|------|
| `short_message.cpp` | ShortMessage 类实现 |
| `sms_service_proxy.cpp` | SMS 服务客户端代理 |
| `sms_delivery_short_message_proxy.cpp` | 送达回调代理 |
| `send_short_message_callback_ipc_interface_code.cpp` | 发送回调 IPC 码 |
| `delivery_short_message_callback_ipc_interface_code.cpp` | 送达回调 IPC 码 |
| `sms_send_short_message_proxy.cpp` | 发送短信代理 |
| `sms_service_manager_client.cpp` | SMS 服务管理器客户端 |

### 4. mms_native_source - MMS 源代码集

**文件位置**: `frameworks/native/mms/BUILD.gn`

**target 类型**: `ohos_source_set`

**源文件**: 17 个文件

| 文件 | 说明 |
|------|------|
| `mms_msg.cpp` | MMS 消息编解码 |
| `mms_header.cpp` | MMS 头部处理 |
| `mms_body.cpp` | MMS 主体处理 |
| `mms_address.cpp` | MMS 地址处理 |
| `mms_attachment.cpp` | MMS 附件处理 |
| `mms_codec_type.cpp` | MMS 编解码类型 |

### 5. sms - N-API 模块

**文件位置**: `frameworks/js/napi/BUILD.gn`

**target 类型**: `ohos_shared_library`

**输出**: `libsms.so`

**安装路径**: `module/telephony/`

**源文件**: 8 个文件

| 文件 | 说明 |
|------|------|
| `napi_sms.cpp` | SMS N-API 绑定（主模块）|
| `napi_mms.cpp` | MMS 编解码 N-API |
| `napi_mms_pdu.cpp` | MMS PDU 数据库 N-API |
| `napi_send_recv_mms.cpp` | MMS 发送/接收 N-API |
| `napi_sms_util.cpp` | N-API 工具函数 |
| `send_callback.cpp` | 发送回调实现 |
| `delivery_callback.cpp` | 送达回调实现 |

**依赖**:

| 类型 | 名称 | 说明 |
|------|------|------|
| internal deps | `:sms_native_source` | Native SMS 源代码 |
| external deps | `napi:ace_napi.z.so` | N-API 框架 |
|  | `hilog:libhilog` | 日志 |
|  | `c_utils:utils` | C 工具 |

### 6. cj_sms_ffi - Cangjie 绑定

**文件位置**: `frameworks/cj/BUILD.gn`

**target 类型**: `ohos_shared_library`

**输出**: `libcj_sms_ffi.so`

**依赖**: Cangjie FFI 框架

### 7. telephony_sms_taihe - Taihe 绑定

**文件位置**: `frameworks/ets/taihe/BUILD.gn`

**target 类型**: `group`

**包含 targets**:

| target | 类型 | 输出 |
|--------|------|------|
| `telephony_sms_taihe_native` | `taihe_shared_library` | `.so` 文件 |
| `telephony_sms_etc` | `ohos_prebuilt_etc` | `.abc` 文件 |

**安装路径**: `/system/framework/`

### 8. sms_mms_sa_profile - SA 配置

**文件位置**: `sa_profile/BUILD.gn`

**target 类型**: `ohos_sa_profile`

**源文件**: `4008.json` 或 `4008_dynamic.json`

**SA ID**: 4008 (TELEPHONY_SMS_MMS_SYS_ABILITY_ID)

**依赖**: SA 4010 (core_service)

### 9. sms_short_code_rules - 短信码规则

**文件位置**: `services/etc/BUILD.gn`

**target 类型**: `group` 包含 `ohos_prebuilt_etc`

**输出**: `sms_short_code_rules.json`

**安装路径**: `/system/etc/telephony/`

## 依赖关系图

### 内部依赖

```
tel_sms_mms (主库）
    ├── tel_sms_mms_api (API 库）
    │       ├── sms_native_source
    │       └── mms_native_source
    │
    ├── sms (N-API 模块）
    │       └── sms_native_source
    │
    └── (其他直接依赖）
```

### 外部依赖层次

```
sms_mms
    ├── core_service (基础依赖）
    │       └── (提供 RIL 接口）
    │
    ├── safwk / samgr (系统框架）
    │       └── (提供 SA 框架）
    │
    ├── napi (JS 框架）
    │       └── (提供 N-API 绑定）
    │
    ├── hilog / hisysevent (日志/事件）
    │       └── (提供日志和事件）
    │
    ├── curl / netstack (网络）
    │       └── (提供 HTTP 客户端，MMS 使用）
    │
    ├── data_share (数据共享）
    │       └── (提供数据库访问）
    │
    ├── access_token (权限）
    │       └── (提供权限检查）
    │
    └── (其他支持库）
```

## 编译产物清单

### 共享库产物

| target | 输出文件名 | 安装路径 | 说明 |
|--------|-----------|----------|------|
| `tel_sms_mms` | `libtel_sms_mms.z.so` | `/system/lib64/` | 主服务库 |
| `tel_sms_mms_api` | `libtel_sms_mms_api.z.so` | `/system/lib64/` | API 导出库 |
| `sms` | `libsms.z.so` | `/system/lib64/module/telephony/` | N-API 模块 |
| `cj_sms_ffi` | `libcj_sms_ffi.z.so` | `/system/lib64/` | Cangjie 绑定 |
| `telephony_sms_taihe_native` | `.so` | `/system/framework/` | Taihe 绑定 |

### 配置文件产物

| target | 输出文件名 | 安装路径 | 说明 |
|--------|-----------|----------|------|
| `sms_mms_sa_profile` | `4008.json` | `/system/profile/` | SA 启动配置 |
| `sms_short_code_rules` | `sms_short_code_rules.json` | `/system/etc/telephony/` | 短信码规则 |
| `telephony_sms_taihe_etc` | `.abc` | `/system/framework/` | Taihe ABC 文件 |

### Taihe 产物

| 文件 | 类型 | 说明 |
|------|------|------|
| `ohos.telephony.sms.abc` | ArkTS 编译产物 | Taihe 框架运行时文件 |

## 特性开关影响

### SMS_SUPPORT_MMS - MMS 支持

**默认**: `true`

**影响范围**:

- **添加的源文件** (BUILD.gn:168-179):
  - `mms_data_request.cpp`
  - `mms_apn_info.cpp`
  - `mms_conn_callback_stub.cpp`
  - `mms_network_client.cpp`
  - `mms_persist_helper.cpp`
  - `mms_receive.cpp`
  - `mms_receive_manager.cpp`
  - `mms_send_manager.cpp`
  - `mms_sender.cpp`

- **添加的头文件** (smsmms.gni:29-32):
  - `"services/mms/include"`

- **添加的宏定义**:
  - `SMS_SUPPORT_MMS`

**移除的组件**:
- 无移除，MMS 是可选特性，默认启用

### SMS_MMS_SATELLITE - 卫星短信支持

**默认**: `false`

**影响范围**:

- **添加的源文件** (BUILD.gn:109-116):
  - `satellite_sms_client.cpp`
  - `satellite_sms_proxy.cpp`
  - `satellite_sms_callback_stub.cpp`
  - `satellite_sms_callback.cpp`

- **添加的头文件**:
  - `"services/sms/include/satellite"`

- **添加的宏定义**:
  - `SMS_MMS_SATELLITE`

### BASE_POWER_IMPROVEMENT_FEATURE - 省电优化

**默认**: `false`

**影响范围**:

- **添加的依赖**:
  - `power_manager:powermgr_client`

- **添加的宏定义**:
  - `ABILITY_POWER_SUPPORT`

### ABILITY_POWER_SUPPORT - 电源管理支持

**默认**: `false`

**触发条件**: 存在 `global_parts_info.powermgr_power_manager`

**影响范围**:

- **添加的功能**:
  - 电源状态监听
  - 休眠/唤醒管理

### OHOS_BUILD_ENABLE_TELEPHONY_EXT - 扩展特性

**默认**: `false`

**触发条件**: 存在 `global_parts_info.telephony_telephony_enhanced`

**影响范围**:

- **添加的组件**:
  - `telephony_ext_wrapper`

## 编译配置

### 日志配置

所有组件使用统一日志配置：

```cpp
#define TELEPHONY_LOG_TAG "SmsMms"
#define LOG_DOMAIN 0xD001F06
#define LOG_TAG "SmsMms"
```

### 安全编译选项

所有 targets 默认启用以下安全特性：

| 选项 | 说明 | 适用 targets |
|------|------|-----------|
| **CFI** | 控制流完整性 | 所有共享库 |
| **Cross-DSO CFI** | 跨 DSO CFI | 所有共享库 |
| **PAC-RET** | 指针认证返回保护 | 所有共享库 |
| **Fortify Source** | 源码强化 | 所有共享库 |

### 优化选项

```cpp
cflags_cc = [
    "-Os"  # 优化代码大小
]
```

## 构建命令示例

### 完整构建

```bash
# 设置特性
export sms_mms_feature_support_mms=true
export sms_mms_satellite=false

# 生成构建文件
gn gen out/sms_mms --args="target_os=ohos"

# 编译
ninja -C out/sms_mms tel_sms_mms
ninja -C out/sms_mms tel_sms_mms_api
ninja -C out/sms_mms sms
```

### 单独编译组件

```bash
# 只编译主服务
ninja -C out/sms_mms tel_sms_mms

# 只编译 N-API 模块
ninja -C out/sms_mms sms

# 只编译 MMS 组件
ninja -C out/sms_mms mms_native_source
```

## 安装和运行时加载

### 系统启动顺序

1. **SAMGR** 启动，加载所有 SA 配置
2. **SAMGR** 检测到 SA 4008 (sms_mms)
3. **SAMGR** 启动 telephony 进程，加载 `libtel_sms_mms.z.so`
4. **SmsService** 构造函数执行，注册为 SA 4008
5. **SmsService::OnStart()** 调用：
   - 创建 `SmsInterfaceStub`
   - 初始化 `SmsInterfaceManager`
   - 等待 core_service (SA 4010) 初始化
6. **SmsService::Publish()** 发布服务
7. **N-API 模块加载** (`libsms.z.so`)，注册 `telephony.sms`

### 运行时依赖加载

| 库名 | 依赖的运行时 |
|------|-------------|
| `libtel_sms_mms.z.so` | `libz.so`, `libhilog.z.so`, `libffrt.so`, ...（25+ 依赖）|
| `libsms.z.so` | `libtel_sms_mms_api.z.so`, `libace_napi.z.so` |
| `libtel_sms_mms_api.z.so` | `libhilog.z.so`, `libphonenumber_standard.z.so` |

## 相关跳转链接

- [目录结构](01_Directory_Structure.md) - 了解源文件组织
- [架构说明](02_Architecture.md) - 了解模块依赖关系
- [安全评审](06_Security_Review.md) - 了解编译安全特性
