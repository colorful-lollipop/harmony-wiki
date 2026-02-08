# 内部 API 文档

## 文档信息

| 项目 | 内容 |
|------|------|
| **目的** | 描述 NFC 组件内部模块接口和依赖关系 |
| **适用范围** | 系统开发者 |
| **相关文档** | [架构设计](01_Architecture.md)、[目录结构](02_Directory_Structure.md) |

---

## 1. 接口层次

```
┌─────────────────────────────────────────────────────────────────┐
│                    客户端 API (Inner Kits)                       │
│  NfcController │ TagForeground │ HceService │ BasicTagSession  │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────────┐
│                    IPC 接口 (IDL)                                │
│  INfcController (idl) │ ITagSession (idl) │ IHceSession (idl)   │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────────┐
│                    NCI 接口 (Native)                             │
│  INciNfccInterface │ INciTagInterface │ INciCeInterface         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. IDL 接口

### 2.1 INfcController

**位置**: `interfaces/inner_api/controller/idl/INfcController.idl`

**方法**:
| IPC Code | 方法 | 说明 |
|----------|------|------|
| 101 | `GetState()` | 获取 NFC 状态 |
| 102 | `TurnOn()` | 开启 NFC |
| 103 | `TurnOff()` | 关闭 NFC |
| 105 | `RegisterNfcStatusCallBack()` | 注册状态回调 |
| 108 | `GetTagServiceIface()` | 获取标签服务接口 |
| 115 | `GetHceServiceIface()` | 获取 HCE 服务接口 |
| 118 | `RestartNfc()` | 重启 NFC |

### 2.2 ITagSession

**位置**: `interfaces/inner_api/tags/idl/ITagSession.idl`

**方法**:
| IPC Code | 方法 | 说明 |
|----------|------|------|
| 201 | `Connect()` | 连接标签 |
| 203 | `Disconnect()` | 断开标签 |
| 207 | `SendRawFrame()` | 发送原始帧 |
| 208 | `NdefRead()` | 读取 NDEF |
| 209 | `NdefWrite()` | 写入 NDEF |
| 109 | `RegForegroundDispatch()` | 注册前台分发 |

### 2.3 IHceSession

**位置**: `interfaces/inner_api/cardEmulation/idl/IHceSession.idl`

**方法**:
| IPC Code | 方法 | 说明 |
|----------|------|------|
| 301 | `StartHce()` | 启动 HCE |
| 302 | `StopHce()` | 停止 HCE |
| 303 | `RegHceCmdCallback()` | 注册 APDU 回调 |
| 305 | `SendRawFrame()` | 发送 APDU 响应 |

---

## 3. NCI 接口

### 3.1 INciNfccInterface

**位置**: `interfaces/inner_api/common/inci_nfcc_interface.h`

| 方法 | 说明 |
|------|------|
| `Initialize()` | 初始化 NFC 控制器 |
| `EnableDiscovery()` | 启用标签发现 |
| `DisableDiscovery()` | 禁用标签发现 |
| `SetScreenStatus()` | 设置屏幕状态 |

### 3.2 INciTagInterface

**位置**: `interfaces/inner_api/common/inci_tag_interface.h`

| 方法 | 说明 |
|------|------|
| `Connect()` | 连接标签 |
| `Disconnect()` | 断开标签 |
| `Transceive()` | 发送/接收数据 |
| `ReadNdef()` | 读取 NDEF |
| `WriteNdef()` | 写入 NDEF |

### 3.3 INciCeInterface

**位置**: `interfaces/inner_api/common/inci_ce_interface.h`

| 方法 | 说明 |
|------|------|
| `ComputeRoutingParams()` | 计算路由参数 |
| `CommitRouting()` | 提交路由配置 |
| `SendRawFrame()` | 发送 APDU |
| `AddAidRouting()` | 添加 AID 路由 |

---

## 4. 数据类型

### 4.1 TagInfo

**位置**: `interfaces/inner_api/common/taginfo.h`

```cpp
class TagInfo {
    int tagRfDiscId_;
    std::string tagUid_;
    std::vector<int> tagTechList_;
    std::vector<AppExecFwk::PacMap> tagTechExtrasData_;
};
```

### 4.2 NdefMessage

**位置**: `interfaces/inner_api/common/ndef_message.h`

```cpp
class NdefMessage {
    std::vector<std::shared_ptr<NdefRecord>> ndefRecordList_;
};

struct NdefRecord {
    short tnf_;
    std::string id_;
    std::string payload_;
    std::string tagRtdType_;
};
```

---

## 5. 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERR_NONE | 0 | 成功 |
| ERR_NO_PERMISSION | 201 | 权限不足 |
| ERR_NOT_SYSTEM_APP | 202 | 非系统应用 |
| ERR_NFC_BASE | 3100100 | NFC 状态错误基值 |
| ERR_TAG_BASE | 3100200 | 标签错误基值 |
| ERR_CE_BASE | 3100300 | 卡模拟错误基值 |

---

## 6. 模块依赖

### 6.1 服务层依赖

```
nfc_service
├── nfc_inner_kits_common
├── libnfc_hce_interface_stub
├── libnfc_controller_interface_stub
├── libnfc_tag_interface_stub
├── nfc_notification
├── nci_native_default (或 vendor)
└── [外部依赖]
    ├── ipc:ipc_core
    ├── access_token:libaccesstoken_sdk
    ├── safwk:system_ability_fwk
    └── ...
```

### 6.2 Inner API 依赖

```
nfc_inner_kits_controller
├── nfc_inner_kits_common
├── nfc_controller_interface
└── [外部依赖]

nfc_inner_kits_tags
├── nfc_inner_kits_common
├── nfc_inner_kits_controller
├── nfc_tag_interface
└── [外部依赖]
```

