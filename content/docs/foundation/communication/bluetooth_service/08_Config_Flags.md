# Feature Flags 与配置

## 文档信息

- **目的**: 列出所有 Feature flags 和编译配置选项
- **适用范围**: 开发者配置编译选项、产品经理了解功能模块
- **关键结论**:
  1. 使用 Feature flags 控制模块编译
  2. 部分功能依赖其他组件
  3. 默认配置已优化
- **相关文档**: [00_Overview](00_Overview.md), [04_GN_Targets](04_GN_Targets.md)

---

## Feature Flags 定义

### 定义文件

`bluetooth.gni:14-32`

### 完整列表

| Flag | 类型 | 默认值 | 说明 | 依赖 |
|-------|------|----------|------|
| `bluetooth_service_a2dp_sink_feature` | bool | `false` | A2DP Sink Profile | audio_framework |
| `bluetooth_service_a2dp_source_feature` | bool | `true` | A2DP Source Profile | audio_framework |
| `bluetooth_service_avrcp_ct_feature` | bool | `true` | AVRCP Controller Target | av_session |
| `bluetooth_service_avrcp_tg_feature` | bool | `true` | AVRCP Target | av_session |
| `bluetooth_service_hfp_ag_feature` | bool | **条件** | HFP Audio Gateway | call_manager, core_service, state_registry |
| `bluetooth_service_hfp_hf_feature` | bool | `false` | HFP Hands-Free | - |
| `bluetooth_service_hid_host_feature` | bool | `true` | HID Host Profile | - |
| `bluetooth_service_pan_feature` | bool | `false` | PAN Profile | - |
| `bluetooth_service_avrcp_avsession` | bool | `true` | AVRCP AVSession 集成 | av_session, image_framework, input |

**证据**: `bluetooth.gni:14-32`

---

## Feature Flags 详解

### A2DP Source

**Flag**: `bluetooth_service_a2dp_source_feature`

**默认值**: `true`

**启用功能**:
- A2DP 音频源端（手机播放音乐）
- A2DP 协议实现（AVDTP、SDP）
- SBC/AAC 音频编解码

**编译影响**:

| 添加文件 | 宏定义 |
|---------|--------|
| `services/bluetooth/server/src/bluetooth_a2dp_source_server.cpp` | `BLUETOOTH_A2DP_SRC_FEATURE` |
| `services/bluetooth/service/src/a2dp_src/a2dp_src_service.cpp` | - |
| `services/bluetooth/service/src/gavdp/*.cpp` (22 文件) | - |
| `services/bluetooth/ipc/src/bluetooth_a2dp_src_*.cpp` | - |

**依赖**:
- `audio_framework:audio_client` - 音频框架

**证据**:
- `bluetooth.gni:16`
- `services/bluetooth/server/BUILD.gn:58-62`
- `services/bluetooth/service/BUILD.gn:239-241`

---

### A2DP Sink

**Flag**: `bluetooth_service_a2dp_sink_feature`

**默认值**: `false`

**启用功能**:
- A2DP 音频接收端（音箱/耳机）
- A2DP 协议实现

**依赖**:
- `audio_framework:audio_client` - 音频框架

---

### AVRCP CT

**Flag**: `bluetooth_service_avrcp_ct_feature`

**默认值**: `true`

**启用功能**:
- AVRCP 控制器（手机控制耳机）
- AVCTP 协议实现

**编译影响**: 添加 12 个源文件

---

### AVRCP TG

**Flag**: `bluetooth_service_avrcp_tg_feature`

**默认值**: `true`

**启用功能**:
- AVRCP 目标（耳机响应手机）
- AVCTP 协议实现

**编译影响**: 添加 12 个源文件

---

### HFP AG

**Flag**: `bluetooth_service_hfp_ag_feature`

**默认值**: **条件**

**条件判断**:
```gn
if ((defined(global_parts_info) &&
     !defined(global_parts_info.telephony_call_manager)) ||
    !defined(global_parts_info.telephony_core_service) ||
    !defined(global_parts_info.telephony_state_registry)) {
  bluetooth_service_hfp_ag_feature = false
} else {
  bluetooth_service_hfp_ag_feature = true
}
```

**依赖**:
- `call_manager:tel_call_manager_api`
- `core_service:tel_core_service_api`
- `state_registry:tel_state_registry_api`

**说明**: 仅在 telephony 组件存在时启用

**证据**: `bluetooth.gni:20-27`

---

### HFP HF

**Flag**: `bluetooth_service_hfp_hf_feature`

**默认值**: `false`

**启用功能**:
- HFP 免提设备（手机端）

---

### HID Host

**Flag**: `bluetooth_service_hid_host_feature`

**默认值**: `true`

**启用功能**:
- HID Host Profile
- HID 设备支持（键盘、鼠标）
- HOGP 协议

**编译影响**: 添加 6 个源文件

---

### PAN

**Flag**: `bluetooth_service_pan_feature`

**默认值**: `false`

**启用功能**:
- PAN Profile
- 网络共享
- BNEP 协议

**编译影响**: 添加 5 个源文件

---

### AVRCP AVSession

**Flag**: `bluetooth_service_avrcp_avsession`

**默认值**: `true`

**定义位置**: `services/bluetooth/service/BUILD.gn:21-23`

**启用功能**:
- AVRCP 与 AVSession 集成
- 媒体元数据同步

**依赖**:
- `av_session:avsession_client`
- `image_framework:image_native`
- `input:libmmi-client`

**证据**: `services/bluetooth/service/BUILD.gn:380-386`

---

## 编译配置

### 启用/禁用 Feature

#### 方法 1: 修改 bluetooth.gni

```gn
# bluetooth.gni
bluetooth_service_a2dp_source_feature = true
bluetooth_service_hfp_ag_feature = true
```

#### 方法 2: 命令行参数

```bash
./build.sh --gn-args='bluetooth_service_pan_feature=true'
```

---

### 检查配置

#### 查看编译日志

```bash
grep BLUETOOTH_A2DP_SRC_FEATURE build.log
```

#### 查看宏定义

```bash
# 编译后检查宏
nm libbluetooth_server.z.so | grep BLUETOOTH
```

---

### 配置组合建议

#### 最小配置 (嵌入式)

```gn
bluetooth_service_a2dp_sink_feature = false
bluetooth_service_a2dp_source_feature = true
bluetooth_service_avrcp_ct_feature = true
bluetooth_service_avrcp_tg_feature = true
bluetooth_service_hfp_ag_feature = false
bluetooth_service_hfp_hf_feature = false
bluetooth_service_hid_host_feature = false
bluetooth_service_pan_feature = false
```

**估计 ROM**: ~3MB
**估计 RAM**: ~4MB

#### 标准配置 (手机)

```gn
bluetooth_service_a2dp_sink_feature = false
bluetooth_service_a2dp_source_feature = true
bluetooth_service_avrcp_ct_feature = true
bluetooth_service_avrcp_tg_feature = true
bluetooth_service_hfp_ag_feature = true  # 需 telephony
bluetooth_service_hfp_hf_feature = false
bluetooth_service_hid_host_feature = true
bluetooth_service_pan_feature = false
```

**估计 ROM**: ~4.5MB
**估计 RAM**: ~7.5MB

#### 完整配置 (车机/平板)

```gn
bluetooth_service_a2dp_sink_feature = true
bluetooth_service_a2dp_source_feature = true
bluetooth_service_avrcp_ct_feature = true
bluetooth_service_avrcp_tg_feature = true
bluetooth_service_hfp_ag_feature = true
bluetooth_service_hfp_hf_feature = true
bluetooth_service_hid_host_feature = true
bluetooth_service_pan_feature = true
```

**估计 ROM**: ~6MB
**估计 RAM**: ~10MB

---

## 运行时配置

### 配置文件位置

**推断路径**: `/system/etc/bluetooth/` 或 `/data/bluetooth/`

**支持的配置类型**:
- XML 配置 (OBEX)
- 状态存储 (配对信息)

---

### 配置文件示例

**TODO(需确认)**: 具体配置文件格式待确认

---

## HiSysEvent 配置

### 事件定义文件

`hisysevent.yaml`

### 关键事件

| 事件 | 用途 | 配置 |
|------|------|------|
| `BR_SWITCH_STATE` | Classic 开关状态 | PID, UID, STATE |
| `BLE_SWITCH_STATE` | BLE 开关状态 | PID, UID, STATE |
| `DISCOVERY_STATE` | 设备发现状态 | PID, UID, STATE |
| `A2DP_CONNECTED_STATE` | A2DP 连接状态 | STATE |
| `BLE_SCAN_START/STOP` | BLE 扫描控制 | PID, UID, TYPE |
| `BLE_SCAN_DUTY_CYCLE` | BLE 扫描占空比 | WINDOW, INTERVAL, TYPE |
| `GATT_CONNECT_STATE` | GATT 连接 | ADDRESS, STATE, ROLE |
| `GATT_APP_REGISTER` | GATT 应用注册 | ACTION, SIDE, ADDRESS, PID, UID, APPID |

**证据**: `hisysevent.yaml:14-84`

---

## 编译标志

### C++ 编译选项

**文件**: `services/bluetooth/service/BUILD.gn:183-208`

**主要选项**:
- `-fPIC` - 位置无关代码
- `-fexceptions` - 启用异常
- `-Wunused-*` - 警告未使用变量
- `-Wdelete-non-abstract-non-virtual-dtor` - 虚析构函数警告

**安全选项**:
- `stack_protector_ret = true` - 栈保护

**证据**:
- `services/bluetooth/server/BUILD.gn:26`
- `services/bluetooth/service/BUILD.gn:183-208`

---

## 依赖配置

### 外部组件依赖

见 `bundle.json:60-93`

### 关键依赖

| 依赖 | 用途 | Feature Flag |
|------|------|-----------|
| `audio_framework` | A2DP 音频 | a2dp_*_feature |
| `av_session` | AVRCP 集成 | avrcp_avsession |
| `call_manager` | HFP AG | hfp_ag_feature |
| `input` | AVRCP 按键 | avrcp_avsession |
| `image_framework` | 媒体封面 | avrcp_avsession |

---

## 总结

**Feature Flags 作用**:
1. 控制模块编译（减少 ROM/RAM）
2. 适配不同产品需求
3. 管理组件依赖

**配置原则**:
- 根据产品需求选择 Feature
- 考虑 ROM/RAM 限制
- 确认依赖组件可用

**相关文档**:
- GN Targets: [04_GN_Targets](04_GN_Targets.md)
- 编译产物: [05_Build_Artifacts](05_Build_Artifacts.md)
- 概览: [00_Overview](00_Overview.md)
