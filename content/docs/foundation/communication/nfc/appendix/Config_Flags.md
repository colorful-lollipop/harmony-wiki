# 附录：配置标志说明

## 1. GN Feature 标志

### 1.1 nfc_use_vendor_nci_native

**类型**: `boolean`  
**默认值**: `false`  
**说明**: 使用厂商提供的 NCI native 实现  
**影响**:
- 加载 `libnci_native_vendor.z.so` 代替默认实现
- 定义宏 `USE_VENDOR_NCI_NATIVE`

**使用场景**:
- 芯片厂商需要提供自定义 NCI 实现
- 硬件特定的优化或扩展

**构建命令**:
```bash
./build.sh --product {product} --gn-args "nfc_use_vendor_nci_native=true"
```

### 1.2 nfc_service_feature_vendor_applications_enabled

**类型**: `boolean`  
**默认值**: `false`  
**说明**: 启用厂商应用扩展支持  
**影响**:
- 定义宏 `VENDOR_APPLICATIONS_ENABLED`
- 启用 `OnCardEmulationNotifyCb` 等扩展接口

**使用场景**:
- 厂商需要扩展 NFC 功能
- 特定应用场景定制

### 1.3 nfc_sim_feature

**类型**: `boolean`  
**默认值**: `false`  
**说明**: 启用 SIM 卡 NFC 功能  
**影响**:
- 定义宏 `NFC_SIM_FEATURE`
- 支持 UICC-based 卡模拟

**使用场景**:
- 运营商 SIM 卡支付
- 安全元件在 SIM 卡中

### 1.4 nfc_service_feature_ndef_wifi_enabled

**类型**: `boolean`  
**默认值**: `false` (自动检测)  
**说明**: 启用 NDEF WiFi 配对功能  
**影响**:
- 定义宏 `NDEF_WIFI_ENABLED`
- 编译 `wifi_connection_manager.cpp`
- 添加 WiFi SDK 依赖

**自动启用条件**:
```gn
if (defined(global_parts_info.communication_wifi)) {
  nfc_service_feature_ndef_wifi_enabled = true
}
```

### 1.5 nfc_service_feature_ndef_bt_enabled

**类型**: `boolean`  
**默认值**: `false` (自动检测)  
**说明**: 启用 NDEF 蓝牙配对功能  
**影响**:
- 定义宏 `NDEF_BT_ENABLED`
- 编译 `bt_connection_manager.cpp`
- 添加蓝牙 SDK 依赖

**自动启用条件**:
```gn
if (defined(global_parts_info.communication_bluetooth)) {
  nfc_service_feature_ndef_bt_enabled = true
}
```

### 1.6 nfc_vibrator_disabled

**类型**: `boolean`  
**默认值**: `false`  
**说明**: 禁用标签发现时的振动反馈  
**影响**:
- 定义宏 `NFC_VIBRATOR_DISABLED`
- 移除 vibrator 相关代码

**使用场景**:
- 节省电量
- 静音场景

### 1.7 nfc_handle_screen_lock

**类型**: `boolean`  
**默认值**: `false`  
**说明**: 处理屏幕锁定事件  
**影响**:
- 定义宏 `NFC_HANDLE_SCREEN_LOCK`
- 启用屏幕锁相关的 NFC 逻辑

## 2. C++ 宏定义

### 2.1 DEBUG

**来源**: services/BUILD.gn:21  
**说明**: Debug 构建标志  
**影响**: 启用调试日志和断言

### 2.2 DTFUZZ_TEST

**来源**: services/BUILD.gn:50-52  
**条件**: `is_asan || use_clang_coverage`  
**说明**: 模糊测试构建标志

### 2.3 NXP_EXTNS

**来源**: services/src/nci_adapter/nci_native_default/BUILD.gn  
**说明**: NXP NFC 芯片扩展支持  
**影响**: 启用 NXP 特定的 NCI 扩展

### 2.4 TAIHE_FWK

**来源**: frameworks/ets/taihe/*/BUILD.gn  
**说明**: Taihe 框架标志  
**影响**: 启用 Taihe (ArkTS) 特定实现

## 3. 系统参数

### 3.1 const.nfc.not_support

**类型**: `string`  
**默认值**: `"false"`  
**说明**: NFC 是否被标记为不支持  
**检查代码**:
```cpp
// nfc_api_control.cpp:31-48
static constexpr const char* NFC_NOT_SUPPORT_KEY = "const.nfc.not_support";
bool IsNfcNotSupported() {
    // 检查系统参数
    return nfcNotSupported == PARAM_TRUE;
}
```

### 3.2 const.nfc.hal_service.ready

**类型**: `string`  
**说明**: NFC HAL 服务就绪状态  
**用途**: SA 启动条件

### 3.3 const.nfc.state

**类型**: `string`  
**值**: `"on"` / `"off"`  
**说明**: NFC 开关状态  
**用途**: SA 停止条件

## 4. 编译期常量

### 4.1 超时配置 (nfc_service.h)

| 常量 | 值 | 说明 |
|------|-----|------|
| `WAIT_MS_INIT` | 90 * 1000 | 初始化等待时间（毫秒）|
| `WAIT_ROUTING_INIT` | 10 * 1000 | 路由初始化等待时间 |
| `MAX_RETRY_TIME` | 3 | 最大重试次数 |
| `SWITCH_OPER_WAIT_MS` | 200 | 开关操作等待时间 |

### 4.2 大小限制 (nfc_sdk_common.h)

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_APDU_DATA_BYTE` | 1024 * 5 | APDU 数据最大字节数 |
| `MAX_APDU_DATA_HEX_STR` | MAX_APDU_DATA_BYTE * 2 | APDU 数据最大十六进制字符串长度 |
| `MAX_AID_LIST_NUM_PER_APP` | 100 | 每应用 AID 最大数量 |
| `MAX_NDEFMSG_LEN` | 4096 | NDEF 消息最大长度 |
| `MAX_BYTES_LEN` | 10000 | 通用字节数组最大长度 |

### 4.3 SA ID (nfc_sdk_common.h)

| 常量 | 值 | 说明 |
|------|-----|------|
| `NFC_MANAGER_SYS_ABILITY_ID` | 1140 | NFC 服务 SA ID |

## 5. 配置组合建议

### 5.1 标准配置

```gn
# 标准功能，使用默认 NCI
nfc_use_vendor_nci_native = false
nfc_service_feature_vendor_applications_enabled = false
nfc_sim_feature = false
nfc_vibrator_disabled = false
nfc_handle_screen_lock = false
```

### 5.2 厂商定制配置

```gn
# 厂商定制，使用自有 NCI
nfc_use_vendor_nci_native = true
nfc_service_feature_vendor_applications_enabled = true
nfc_sim_feature = false
nfc_vibrator_disabled = true
nfc_handle_screen_lock = true
```

### 5.3 运营商配置

```gn
# 运营商配置，启用 SIM 卡 NFC
nfc_use_vendor_nci_native = false
nfc_service_feature_vendor_applications_enabled = false
nfc_sim_feature = true
nfc_vibrator_disabled = false
nfc_handle_screen_lock = false
```

