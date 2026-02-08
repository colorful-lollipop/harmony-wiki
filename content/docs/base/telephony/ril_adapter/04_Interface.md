# 接口文档

> RIL Adapter 的 HDF 接口定义、请求/通知契约

## 1. 接口概述

RIL Adapter **不提供 N-API**，仅通过 **HDF (Hardware Driver Foundation)** 框架提供 HDI (Hardware Driver Interface) 接口与上层 telephony 服务通信。

### 1.1 通信模式

```
┌─────────────────────────────────────────────────────────┐
│                   telephony_core_service                 │
│                   (上层电话服务)                         │
└─────────────────────────────────────────────────────────┘
                          ↓ IPC
┌─────────────────────────────────────────────────────────┐
│                   HDF Framework                         │
│                   (驱动框架)                            │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              RIL Adapter (HDF Service)                  │
│                   services/hril_hdf/                    │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              HRIL Business Layer                       │
│                   services/hril/                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              Vendor Abstraction Layer                  │
│                   services/vendor/                      │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   Modem Hardware                        │
└─────────────────────────────────────────────────────────┘
```

## 2. HDF 接口定义

### 2.1 接口来源

**接口定义文件**：`drivers_interface` 仓库中的 HDI 接口

**接口版本**：V1.5

**主要接口**：
- `OHOS::HDI::Ril::V1_5::IRil` - 上层调用 RIL 的接口
- `OHOS::HDI::Ril::V1_5::IRilCallback` - RIL 回调上层的接口

### 2.2 IRil 接口方法

**调用方向**：上层服务 → RIL Adapter

| 方法 | 功能描述 | 参数类型 |
|------|----------|----------|
| `SetRadioPower` | 设置射频开关 | RequestInfo |
| `GetRadioPower` | 获取射频状态 | RequestInfo |
| `GetSignal` | 获取信号强度 | RequestInfo |
| `GetNetworkSearchInformation` | 搜索网络 | RequestInfo |
| `GetNetworkMode` | 获取网络模式 | RequestInfo |
| `SetNetworkMode` | 设置网络模式 | RequestInfo |
| ` ` | 设置首选网络 | RequestInfo |
| `Dial` | 拨打电话 | DialInfo |
| `HangUp` | 挂断电话 | RequestInfo |
| `AcceptCall` | 接听电话 | RequestInfo |
| `RejectCall` | 拒绝电话 | RequestInfo |
| `DeactivateDataCall` | 去激活数据连接 | RequestInfo |
| `ActivateDataCall` | 激活数据连接 | DataCallInfo |
| `SendSms` | 发送短信 | SmsInfo |
| `GetSmscAddr` | 获取短信中心地址 | RequestInfo |
| `SetSmscAddr` | 设置短信中心地址 | RequestInfo |
| `GetSimImsi` | 获取 IMSI | RequestInfo |
| `GetSimStatus` | 获取 SIM 状态 | RequestInfo |
| `EnterSimPin` | 输入 PIN 码 | PinInfo |
| ... | 其他方法 | ... |

### 2.3 IRilCallback 接口方法

**调用方向**：RIL Adapter → 上层服务

| 方法 | 功能描述 | 触发时机 |
|------|----------|----------|
| `CallStateUpdated` | 通话状态更新 | 异步事件 |
| `CallRingbackVoice` | 回铃音 | 异步事件 |
| `NetworkStateUpdated` | 网络状态更新 | 异步事件 |
| `NetworkSignalUpdated` | 信号强度更新 | 异步事件 |
| `DataConnectionStateUpdated` | 数据连接状态更新 | 异步事件 |
| `SimStateUpdated` | SIM 卡状态更新 | 异步事件 |
| `SmsMessageReceived` | 收到短信 | 异步事件 |
| `SmscAddrReport` | 短信中心地址报告 | 异步事件 |
| ... | 其他回调 | ... |

## 3. 内部接口定义

### 3.1 主 C API

**文件**：`interfaces/innerkits/include/hril.h`

```cpp
// 厂商操作接口
struct RilInitOps {
    void (*)(void);                              // 初始化
    int32_t (*)(const char *, ResponseInfo);   // 通用请求
    // ...
};

// HRIL 报告回调
struct HRilReport {
    int32_t (*OnCallReport)(...);
    int32_t (*OnDataReport)(...);
    int32_t (*OnModemReport)(...);
    int32_t (*OnNetworkReport)(...);
    int32_t (*OnSimReport)(...);
    int32_t (*OnSmsReport)(...);
};
```

### 3.2 请求 ID 定义

**文件**：`interfaces/innerkits/include/hril_request.h`

| 前缀 | 类别 | 示例 |
|------|------|------|
| `HREQ_CALL_*` | 通话请求 | `HREQ_CALL_DIAL`, `HREQ_CALL_HANGUP` |
| `HREQ_DATA_*` | 数据请求 | `HREQ_DATA_ACTIVATE`, `HREQ_DATA_DEACTIVATE` |
| `HREQ_NETWORK_*` | 网络请求 | `HREQ_NETWORK_GET_SIGNAL` |
| `HREQ_SIM_*` | SIM 请求 | `HREQ_SIM_GET_STATUS`, `HREQ_SIM_ENTER_PIN` |
| `HREQ_SMS_*` | 短信请求 | `HREQ_SMS_SEND`, `HREQ_SMS_SET_SMSC` |
| `HREQ_MODEM_*` | Modem 请求 | `HREQ_MODEM_SET_POWER` |

**完整列表**：

```cpp
// 通话请求
HREQ_CALL_DIAL
HREQ_CALL_HANGUP
HREQ_CALL_REJECT
HREQ_CALL_ANSWER
HREQ_CALL_HOLD
HREQ_CALL_ACTIVE
HREQ_CALL_SWAP
HREQ_CALL_CONFERENCE
HREQ_CALL_DIVERT
HREQ_CALL_DTMF
HREQ_CALL_DTMF_START
HREQ_CALL_DTMF_END

// 数据请求
HREQ_DATA_ACTIVATE
HREQ_DATA_DEACTIVATE
HREQ_DATA_GET_LINK_CAPABILITY
HREQ_DATA_GET_HANDOVER_LINK_CAPABILITY
HREQ_DATA_SET_BAND_MODE
HREQ_DATA_CDMA_GET_BAND_MODE

// 网络请求
HREQ_NETWORK_GET_SIGNAL
HREQ_NETWORK_GET_REGISTRATION
HREQ_NETWORK_GET_OPERATOR
HREQ_NETWORK_GET_NEIGHBOR_CELL
HREQ_NETWORK_SET_PREFERRED_NETWORK
HREQ_NETWORK_GET_NETWORK_SELECTION_MODE
HREQ_NETWORK_SET_NETWORK_SELECTION_MODE
HREQ_NETWORK_SEARCH_NETWORKS
HREQ_NETWORK_GET_RRC_CONNECTION_STATE

// SIM 请求
HREQ_SIM_GET_STATUS
HREQ_SIM_GET_IMSI
HREQ_SIM_GET_ICCID
HREQ_SIM_ENTER_PIN
HREQ_SIM_UNLOCK_PIN
HREQ_SIM_CHANGE_PIN
HREQ_SIM_ENTER_PUK
HREQ_SIM_UNLOCK_PUK
HREQ_SIM_SET_FACILITY_LOCK
HREQ_SIM_GET_FACILITY_LOCK
HREQ_SIM_SIM_IO
HREQ_SIM_GET_SIM_LOCK_STATUS
HREQ_SIM_SIM_OPEN_LOGICAL_CHANNEL
HREQ_SIM_SIM_CLOSE_LOGICAL_CHANNEL
HREQ_SIM_SIM_TRANSMISSION_APDU
HREQ_SIM_SIM_AUTHENTICATION

// 短信请求
HREQ_SMS_SEND_GSM_SMS
HREQ_SMS_SEND_CDMA_SMS
HREQ_SMS_SET_SMSC
HREQ_SMS_GET_SMSC
HREQ_SMS_ADD_SIM_MESSAGE
HREQ_SMS_DEL_SIM_MESSAGE
HREQ_SMS_UPDATE_SIM_MESSAGE
HREQ_SMS_GET_SIM_MESSAGE
HREQ_SMS_GET_CB_CONFIG
HREQ_SMS_SET_CB_CONFIG
HREQ_SMS_SET_CDMA_CB_CONFIG
HREQ_SMS_GET_CDMA_CB_CONFIG
HREQ_SMS_SEND_SMS_ACK

// Modem 请求
HREQ_MODEM_SET_POWER
HREQ_MODEM_GET_POWER
HREQ_MODEM_GET_MEID
HREQ_MODEM_GET_IMEI
HREQ_MODEM_GET_IMEISV
HREQ_MODEM_GET_VENDOR
HREQ_MODEM_GET_BASEBAND
```

### 3.3 通知 ID 定义

**文件**：`interfaces/innerkits/include/hril_notification.h`

| 前缀 | 类别 | 示例 |
|------|------|------|
| `HNOTI_CALL_*` | 通话通知 | `HNOTI_CALL_STATE_UPDATED` |
| `HNOTI_DATA_*` | 数据通知 | `HNOTI_DATA_CONNECT_STATE` |
| `HNOTI_NETWORK_*` | 网络通知 | `HNOTI_NETWORK_STATE` |
| `HNOTI_SIM_*` | SIM 通知 | `HNOTI_SIM_STATE_CHANGED` |
| `HNOTI_SMS_*` | 短信通知 | `HNOTI_SMS_RECEIVED` |

**完整列表**：

```cpp
// 通话通知
HNOTI_CALL_STATE_UPDATED
HNOTI_CALL_RINGBACK_VOICE
HNOTI_CALL_END
HNOTI_CALL_HOLD
HNOTI_CALL_ACTIVE
HNOTI_CALL_WAITING
HNOTI_CALL_DISPLAY
HNOTI_CALL_CDMA_CALL_WAITING
HNOTI_CALL_MODE_UPDATED

// 数据通知
HNOTI_DATA_CONNECT_STATE
HNOTI_DATA_PDP_CONTEXT_LIST_UPDATED
HNOTI_DATA_HANDOVER_STATE
HNOTI_DATA_LINK_CAPABILITY_UPDATED
HNOTI_DATA_WIFI_PARA
HNOTI_DATA_SETUP_DATA_CALL_RESULT
HNOTI_DATA_RESUME_DATA_CALL_RESULT

// 网络通知
HNOTI_NETWORK_STATE
HNOTI_NETWORK_SIGNAL_UPDATED
HNOTI_NETWORK_RSSI_UPDATED
HNOTI_NETWORK_NR_SIGNAL_UPDATED
HNOTI_NETWORK_BER_UPDATED
HNOTI_NETWORK_TIME_UPDATED
HNOTI_NETWORK_TIME_ZONE_UPDATED
HNOTI_NETWORK_PS_REGISTRATION
HNOTI_NETWORK_CDMA_BSS
HNOTI_NETWORK_REG_STATUS_UPDATED
HNOTI_NETWORK_EMERGENCE_CAMP

// SIM 通知
HNOTI_SIM_STATE_CHANGED
HNOTI_SIM_REFRESH

// 短信通知
HNOTI_SMS_RECEIVED
HNOTI_SMS_CDMA_RECEIVED
HNOTI_SMS_STATUS_REPORT
HNOTI_SMS_SMSC_ADDR

// Modem 通知
HNOTI_MODEM_RADIO_STATE
HNOTI_MODEM_DTMF
HNOTI_MODEM_RTTY
HNOTI_MODEM_RESEND_INCOMING
HNOTI_MODEM_INCOMING_VOICE_RINGBACK
```

## 4. 公共数据结构

### 4.1 请求信息

**文件**：`interfaces/innerkits/include/hril_public_struct.h`

```cpp
// 请求数据信息
struct ReqDataInfo {
    int32_t slotId;           // 卡槽 ID
    int32_t serialId;         // 请求序列号
    int32_t requestId;        // 请求 ID (HREQ_*)
    void *data;               // 请求数据指针
    size_t dataLen;           // 数据长度
};
```

### 4.2 报告信息

```cpp
// 报告信息
struct ReportInfo {
    ReqDataInfo *requestInfo;           // 请求上下文
    int32_t notifyId;                   // 通知 ID (HNOTI_*)
    ReportType type;                    // HRIL_RESPONSE or HRIL_NOTIFICATION
    HRilErrNumber error;                // 错误码
    ModemReportErrorInfo modemErrInfo;  // Modem 错误信息
    HRilAckTypes ack;                   // ACK 类型
};
```

### 4.3 ReportType 枚举

```cpp
typedef enum {
    HRIL_RESPONSE = 0,     // 响应（请求的结果）
    HRIL_NOTIFICATION = 1, // 通知（异步事件）
} ReportType;
```

## 5. 错误码定义

**文件**：`interfaces/innerkits/include/hril_enum.h`

| 错误码 | 值 | 描述 | 处理建议 |
|--------|-----|------|----------|
| `HRIL_ERR_SUCCESS` | 0 | 成功 | - |
| `HRIL_ERR_NULL_POINT` | -1 | 空指针 | 检查参数有效性 |
| `HRIL_ERR_GENERIC_FAILURE` | 1 | 通用失败 | 查看详细日志 |
| `HRIL_ERR_INVALID_PARAMETER` | 2 | 无效参数 | 校验输入参数 |
| `HRIL_ERR_MEMORY_FULL` | 3 | 内存不足 | 检查内存泄漏 |
| `HRIL_ERR_CMD_SEND_FAILURE` | 4 | 命令发送失败 | 检查 AT 端口 |
| `HRIL_ERR_CMD_NO_CARRIER` | 5 | 无载波 | 检查 Modem 状态 |
| `HRIL_ERR_INVALID_RESPONSE` | 6 | 无效响应 | 检查 AT 响应解析 |
| `HRIL_ERR_REPEAT_STATUS` | 7 | 重复状态 | 忽略或上报 |
| `HRIL_ERR_HDF_IPC_FAILURE` | 65535 | HDF IPC 失败 | 检查 HDF 服务 |

## 6. 接口调用示例

### 6.1 拨打电话

```cpp
// 参数结构
struct DialInfo {
    int32_t slotId;
    std::string address;    // 电话号码
    int32_t clirMode;        // 来电显示设置
};

// 调用流程
IRil::Get()->Dial(serialId, dialInfo);

// 响应处理
void OnDialResponse(const ResponseInfo &response) {
    if (response.error == HRIL_ERR_SUCCESS) {
        // 通话已发起
    } else {
        // 拨打失败
    }
}
```

### 6.2 设置射频开关

```cpp
// 参数结构
struct RequestInfo {
    int32_t slotId;
    int32_t serialId;
    int32_t power;           // 0: 关闭, 1: 打开
};

// 调用流程
IRil::Get()->SetRadioPower(serialId, requestInfo);

// 通知处理
void OnRadioPowerUpdated(const ReportInfo &report) {
    // 射频状态已更新
}
```

### 6.3 发送短信

```cpp
// 参数结构
struct SmsInfo {
    int32_t slotId;
    std::string smsc;       // 短信中心号码
    std::string pdu;        // PDU 编码
    bool bEncrypt;          // 是否加密
};

// 调用流程
IRil::Get()->SendSms(serialId, smsInfo);

// 响应处理
void OnSendSmsResponse(const ResponseInfo &response) {
    // 发送结果
}
```

## 7. 接口安全注意事项

### 7.1 输入验证

| 检查点 | 说明 |
|--------|------|
| 参数非空 | 检查指针、字符串长度 |
| 数值范围 | 检查枚举值、索引 |
| 格式校验 | 电话号码、PDU 格式 |

### 7.2 错误处理

| 场景 | 处理方式 |
|------|----------|
| IPC 超时 | 重试 + 上报错误 |
| Modem 无响应 | 降级处理 |
| 权限不足 | 返回错误码 |

---

**相关文档**：
- [架构与数据流](02_Architecture.md) - 接口在架构中的位置
- [攻击面分析](05_AttackSurface.md) - 接口安全性分析
- [安全风险评估](06_SecurityReview.md) - 潜在漏洞点
