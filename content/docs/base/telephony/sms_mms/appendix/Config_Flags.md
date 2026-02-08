# 配置标志附录

## 目的

本文档汇总短彩信模块的所有编译时配置标志（Feature Flags）、宏定义和关键常量，帮助构建工程师和开发者理解配置选项。

## 适用范围

本文档覆盖：
- GN 特性开关 (`smsmms.gni`)
- 编译宏定义 (BUILD.gn)
- 关键常量定义 (头文件)
- 默认值和影响范围

## GN 特性开关

### 开关总览

**文件**: `smsmms.gni:14-19`

```gn
declare_args() {
    sms_mms_dynamic_start = false      # 动态启动服务
    sms_mms_tel_power_mode = false      # 省电模式优化
    sms_mms_feature_support_mms = true  # MMS 功能支持
    sms_mms_satellite = false            # 卫星短信支持
}
```

### sms_mms_dynamic_start

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | 启用 SA 4008 的动态启动/停止功能 |

**影响**:
- 使用 `4008_dynamic.json` 替代 `4008.json` 作为 SA 配置
- 基于系统参数 `const.vendor.ril.dynamic_switch_modem` 控制启动
- 用于支持 modem 动态切换场景

**证据**: `sa_profile/4008_dynamic.json`

---

### sms_mms_tel_power_mode

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | 启用省电模式优化 |

**影响**:
- 添加宏定义: `BASE_POWER_IMPROVEMENT_FEATURE`
- 启用电源管理相关优化代码
- 可能影响短信发送的超时时间

**证据**: `smsmms.gni:23-25`

```gn
if (sms_mms_tel_power_mode) {
    global_defines += [ "BASE_POWER_IMPROVEMENT_FEATURE" ]
}
```

---

### sms_mms_feature_support_mms

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | 启用 MMS (多媒体短信) 功能 |

**影响**:

1. **添加的源文件** (`BUILD.gn:168-179`):
```gn
if (sms_mms_feature_support_mms) {
    sources += [
        "services/mms/data_request.cpp",
        "services/mms/mms_apn_info.cpp",
        "services/mms/mms_conn_callback_stub.cpp",
        "services/mms/mms_network_client.cpp",
        "services/mms/mms_network_manager.cpp",
        "services/mms/mms_persist_helper.cpp",
        "services/mms/mms_receive.cpp",
        "services/mms/mms_receive_manager.cpp",
        "services/mms/mms_send_manager.cpp",
        "services/mms/mms_sender.cpp",
    ]
}
```

2. **添加的头文件路径** (`BUILD.gn:29-32`):
```gn
if (sms_mms_feature_support_mms) {
    include_dirs += [ "services/mms/include" ]
    defines = [ "SMS_SUPPORT_MMS" ]
}
```

3. **代码中使用**:
```cpp
#ifdef SMS_SUPPORT_MMS
    // MMS 相关代码
    mmsSendManager_->SendMms(...);
#endif
```

**证据**: `BUILD.gn:28-33`

---

### sms_mms_satellite

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | 启用卫星短信功能 |

**影响**:

1. **添加的源文件** (`BUILD.gn:109-116`):
```gn
if (sms_mms_satellite) {
    sources += [
        "services/sms/satellite_service_interaction/src/satellite_sms_callback.cpp",
        "services/sms/satellite_service_interaction/src/satellite_sms_callback_stub.cpp",
        "services/sms/satellite_service_interaction/src/satellite_sms_client.cpp",
        "services/sms/satellite_service_interaction/src/satellite_sms_proxy.cpp",
    ]
}
```

2. **添加的宏定义** (`smsmms.gni:27-29`):
```gn
if (sms_mms_satellite) {
    global_defines += [ "SMS_MMS_SATELLITE" ]
}
```

3. **头文件路径** (`BUILD.gn:25`):
```gn
"services/sms/include/satellite",
```

**证据**: `BUILD.gn:109-116`, `smsmms.gni:27-29`

## 编译宏定义

### 日志相关宏

**文件**: `BUILD.gn:160-164`

```gn
defines = [
    "TELEPHONY_LOG_TAG = \"SmsMms\"",
    "LOG_DOMAIN = 0xD001F06",
    "LOG_TAG = \"SmsMms\"",
]
```

| 宏 | 值 | 说明 |
|----|----|------|
| TELEPHONY_LOG_TAG | "SmsMms" | HiLog 日志标签 |
| LOG_DOMAIN | 0xD001F06 | 日志域标识 |
| LOG_TAG | "SmsMms" | 日志标签 |

---

### 特性相关宏

| 宏 | 触发条件 | 说明 |
|----|---------|------|
| SMS_SUPPORT_MMS | sms_mms_feature_support_mms = true | MMS 功能支持 |
| SMS_MMS_SATELLITE | sms_mms_satellite = true | 卫星短信支持 |
| BASE_POWER_IMPROVEMENT_FEATURE | sms_mms_tel_power_mode = true | 省电优化 |
| ABILITY_POWER_SUPPORT | global_parts_info.powermgr_power_manager | 电源管理支持 |
| OHOS_BUILD_ENABLE_TELEPHONY_EXT | global_parts_info.telephony_telephony_enhanced | 扩展特性 |

**证据**: `BUILD.gn:182-206`

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.powermgr_power_manager) &&
    global_parts_info.powermgr_power_manager) {
    external_deps += [ "power_manager:powermgr_client" ]
    defines += [ "ABILITY_POWER_SUPPORT" ]
}

if (defined(global_parts_info) &&
    defined(global_parts_info.telephony_telephony_enhanced) &&
    global_parts_info.telephony_telephony_enhanced) {
    defines += [ "OHOS_BUILD_ENABLE_TELEPHONY_EXT" ]
}
```

## 关键常量

### 地址相关常量

**文件**: `interfaces/innerkits/sms_pdu_code_type.h` 和 `services/sms/include/gsm/gsm_pdu_code_type.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_ADDRESS_LEN | 21 | 短信地址最大长度（含'+'号） |
| SMS_MAX_ADDRESS_LEN | 21 | SMS 地址最大长度 |
| MAX_BCD_NUM_LEN | 20 | BCD 编码号码最大长度 |

**证据**: `interfaces/innerkits/sms_pdu_code_type.h:23`

```cpp
constexpr uint8_t SMS_MAX_ADDRESS_LEN = 21;
```

---

### 短信内容相关常量

**文件**: `services/sms/include/gsm/gsm_pdu_code_type.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_USER_DATA_LEN | 160 | 用户数据最大长度 (7-bit) |
| MAX_8BIT_DATA_LEN | 140 | 8-bit 数据最大长度 |
| MAX_UCS2_DATA_LEN | 70 | UCS2 编码最大长度 |
| MAX_SEGMENT_NUM | 15 | 短信分段最大数量 |
| MAX_UD_HEADER_NUM | 7 | 用户数据头最大数量 |
| MAX_GSM_7BIT_DATA_LEN | 160 | GSM 7-bit 最大长度 |
| MAX_GSM_8BIT_DATA_LEN | 140 | GSM 8-bit 最大长度 |
| MAX_GSM_MSG_LEN | 160 | GSM 短信最大长度 |

**证据**: `services/sms/include/gsm/gsm_pdu_code_type.h:26-32`

```cpp
constexpr int MAX_USER_DATA_LEN = 160;
constexpr int MAX_8BIT_DATA_LEN = 140;
constexpr int MAX_SEGMENT_NUM = 15;
constexpr int MAX_UD_HEADER_NUM = 7;
```

---

### PDU 相关常量

**文件**: `interfaces/innerkits/sms_pdu_code_type.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_TPDU_DATA_LEN | 255 | TPDU 最大数据长度 |
| MAX_CB_MSG_TEXT_LEN | 93 | 小区广播消息文本最大长度 |
| MAX_SMSC_LEN | 20 | 短信中心地址最大长度 |

**证据**: `interfaces/innerkits/sms_pdu_code_type.h:26-28`

```cpp
constexpr uint16_t MAX_TPDU_DATA_LEN = 255;
constexpr uint8_t MAX_CB_MSG_TEXT_LEN = 93;
constexpr uint8_t MAX_SMSC_LEN = 20;
```

---

### MMS 相关常量

**文件**: `interfaces/innerkits/sms_constants_utils.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_MMS_ATTACHMENT_LEN | 10 * 1024 * 1024 (10MB) | MMS 附件最大大小 |
| MMS_PDU_MAX_SIZE | 10 * 1024 * 1024 (10MB) | MMS PDU 最大大小 |

**证据**: `interfaces/innerkits/sms_constants_utils.h:23-24`

```cpp
static const int32_t MAX_MMS_ATTACHMENT_LEN = 10 * 1024 * 1024;
static const int32_t MMS_PDU_MAX_SIZE = 10 * 1024 * 1024;
```

---

### 发送相关常量

**文件**: `services/sms/include/sms_sender.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_SEND_RETRIES | 3 | 短信发送最大重试次数 |
| MAX_GSM_SMS_TPDU_LEN | 255 | GSM SMS TPDU 最大长度 |
| MAX_SEND_INTERVAL | 5000 (ms) | 发送重试间隔 |
| MAX_WAIT_ACK_TIME | 60000 (ms) | 等待 ACK 超时时间 |
| MAX_REPORT_LIST_LIMIT | 25 | 报告列表最大限制 |

**证据**: `services/sms/include/sms_sender.h:111-123`

```cpp
static constexpr uint8_t MAX_SEND_RETRIES = 3;
static constexpr int32_t MAX_SEND_INTERVAL = 5000;
static constexpr int32_t MAX_WAIT_ACK_TIME = 60000;
static constexpr int32_t MAX_REPORT_LIST_LIMIT = 25;
```

---

### N-API 相关常量

**文件**: `frameworks/js/napi/include/napi_sms.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_TEXT_SHORT_MESSAGE_LENGTH | 4096 | 短消息文本最大长度 |
| DEFAULT_SIM_SLOT_ID | 0 | 默认 SIM 卡槽 ID |
| SIM_SLOT_COUNT | 2 | SIM 卡槽数量 |
| MAX_PORT | 0xFFFF | 最大端口值 |
| CB_RANGE_LIST_MAX_SIZE | 256 | CB 范围列表最大大小 |

**证据**: `frameworks/js/napi/include/napi_sms.h:38-51`

```cpp
constexpr int32_t MAX_TEXT_SHORT_MESSAGE_LENGTH = 4096;
constexpr int32_t DEFAULT_SIM_SLOT_ID = 0;
constexpr int32_t SIM_SLOT_COUNT = 2;
constexpr int32_t MAX_PORT = 0xFFFF;
constexpr int32_t CB_RANGE_LIST_MAX_SIZE = 256;
```

---

### 小区广播相关常量

**文件**: `services/sms/include/gsm/gsm_sms_cb_handler.h`

| 常量名 | 值 | 说明 |
|--------|----|------|
| MAX_CB_MSG_LEN | 4200 | 小区广播消息最大长度 |
| MAX_CB_MSG_LIST_SIZE | 10000 | CB 消息列表最大大小 |
| CB_MESSAGE_ID_MAX | 65535 | CB 消息 ID 最大值 |
| CB_CODE_SCHEME_MAX | 255 | CB 编码方案最大值 |

**证据**: `services/sms/include/gsm/gsm_sms_cb_handler.h:72-91`

```cpp
static const int32_t MAX_CB_MSG_LEN = 4200;
static const int32_t MAX_CB_MSG_LIST_SIZE = 10000;
static constexpr uint16_t CB_MESSAGE_ID_MAX = 65535;
```

---

### 其他常量

| 常量名 | 值 | 说明 | 文件 |
|--------|----|------|------|
| MAX_MSG_TEXT_LEN | 1530 | 消息文本最大长度 | sms_base_message.h:110 |
| MAX_SMS_PDU_LEN | 255 | SMS PDU 最大长度 | sms_base_message.h |
| RECONNECT_MAX_COUNT | 20 | 重连最大次数 | sms_receive_handler.h:93 |
| HTTP_TIME_MICRO_SECOND | 600000 | HTTP 超时时间 (600秒) | mms_network_client.cpp |

## 配置使用示例

### 启用 MMS 功能

```gn
# 在构建命令中设置
gn gen out/telephony --args='sms_mms_feature_support_mms=true'

# 或在 .gn 文件中设置
default_args = {
    sms_mms_feature_support_mms = true
}
```

### 启用卫星短信

```gn
# 在 smsmms.gni 中修改
sms_mms_satellite = true

# 或在构建命令中
ninja -C out/telephony tel_sms_mms sms_mms_satellite=true
```

### 禁用 MMS

```gn
# 对于不需要 MMS 的设备
gn gen out/telephony --args='sms_mms_feature_support_mms=false'
```

## 配置影响矩阵

| 开关 | MMS 代码 | 卫星代码 | 电源管理 | 扩展特性 |
|------|---------|---------|---------|---------|
| sms_mms_feature_support_mms | ✓ | - | - | - |
| sms_mms_satellite | - | ✓ | - | - |
| sms_mms_tel_power_mode | - | - | ✓ | - |
| powermgr_power_manager | - | - | ✓ | - |
| telephony_telephony_enhanced | - | - | - | ✓ |

## 相关跳转链接

- [GN 目标](../05_GN_Targets.md) - 了解完整构建配置
- [架构说明](../02_Architecture.md) - 了解模块架构
- [项目概览](../00_Overview.md) - 了解特性开关用途
