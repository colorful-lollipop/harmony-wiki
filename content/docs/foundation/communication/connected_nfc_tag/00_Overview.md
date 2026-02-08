# Connected NFC Tag - 项目概览

## 1. 项目定位

### 1.1 组件定位

**Connected NFC Tag** 是 OpenHarmony 通信子系统的近场通信（NFC）标签读写组件，为具有**连接型 NFC 标签芯片**的设备提供标签读写能力。

```
┌─────────────────────────────────────────────────────────────────┐
│                    Connected NFC Tag 定位                        │
├─────────────────────────────────────────────────────────────────┤
│  应用层                                                           │
│    ↓ JS API 调用                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │               N-API 接口层 (connectedtag)                   │  │
│  │     //foundation/communication/connected_nfc_tag/         │  │
│  │     //frameworks/js/napi/                                 │  │
│  └───────────────────────────────────────────────────────────┘  │
│    ↓ IPC 调用                                                     │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              服务层 (nfc_tag_service)                       │  │
│  │     SA ID: 1148                                           │  │
│  │     //foundation/communication/connected_nfc_tag/services │  │
│  └───────────────────────────────────────────────────────────┘  │
│    ↓ HDI 调用                                                     │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              HDI 适配层 (nfc_tag_hdi_adapter)              │  │
│  │     //foundation/communication/connected_nfc_tag/         │  │
│  │     //services/src/hdi/                                   │  │
│  └───────────────────────────────────────────────────────────┘  │
│    ↓ 厂商驱动                                                      │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              HDI 驱动接口 (厂商实现)                        │  │
│  │     //drivers/interface/connected_nfc_tag/                │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 核心能力

| 能力 | 描述 | 支持模式 |
|------|------|----------|
| **NDEF 读写** | 读取/写入 NFC 标签的 NDEF（NFC Data Exchange Format）数据 | 字符串/二进制 |
| **事件监听** | 监听 NFC RF（射频场）状态变化（进入/离开） | 异步回调 |
| **初始化管理** | NFC 硬件的初始化/反初始化 | 同步 |

### 1.3 运行环境要求

| 要求 | 详情 |
|------|------|
| **硬件要求** | 设备必须具有连接型 NFC 标签芯片 |
| **系统要求** | OpenHarmony Standard 系统 |
| **权限要求** | `ohos.permission.NFC_TAG`（系统权限） |
| **系统能力** | `SystemCapability.Communication.ConnectedTag` |

---

## 2. 目录结构

```
/foundation/communication/connected_nfc_tag/
├── interfaces/                        # 接口层
│   └── inner_api/                     # 系统内部 API
│       ├── include/                   # 头文件
│       │   ├── nfc_tag_client.h       # 客户端接口
│       │   ├── nfc_tag_proxy.h        # IPC 代理
│       │   ├── nfc_tag_callback_stub.h# 回调存根
│       │   ├── infc_tag_service.h    # 服务接口
│       │   ├── infc_tag_callback.h    # 回调接口
│       │   ├── nfc_tag_errcode.h      # 错误码
│       │   └── nfc_tag_log.h          # 日志
│       └── src/                       # 实现
│           ├── nfc_tag_client.cpp
│           ├── nfc_tag_proxy.cpp
│           └── nfc_tag_callback_stub.cpp
├── frameworks/                        # 框架层
│   └── js/                            # JS API
│       └── napi/                      # N-API 实现
│           ├── nfc_napi_entry.cpp     # 模块注册
│           ├── nfc_napi_adapter.cpp   # API 实现
│           ├── nfc_napi_event.cpp     # 事件处理
│           ├── nfc_napi_utils.cpp     # 工具函数
│           └── BUILD.gn
├── services/                          # 服务层
│   ├── include/                       # 服务头文件
│   │   ├── nfc_tag_service.h          # 服务主类
│   │   ├── nfc_tag_stub.h            # IPC 存根
│   │   ├── nfc_tag_callback_proxy.h   # 回调代理
│   │   ├── nfc_tag_utils.h           # 工具类
│   │   └── nfc_tag_sys_perm.h        # 权限检查
│   ├── src/                          # 服务实现
│   │   ├── nfc_tag_service.cpp       # 服务主逻辑
│   │   ├── nfc_tag_stub.cpp         # IPC 处理
│   │   ├── nfc_tag_callback_proxy.cpp
│   │   ├── nfc_tag_utils.cpp
│   │   ├── nfc_tag_sys_perm.cpp      # 权限实现
│   │   └── hdi/                      # HDI 适配层
│   │       ├── nfc_tag_hdi_adapter.h
│   │       ├── nfc_tag_hdi_impl.h
│   │       ├── nfc_tag_hdi_adapter.cpp
│   │       └── nfc_tag_hdi_impl.cpp
│   └── etc/                          # 配置
│       └── init/
│           ├── nfc_tag_service.cfg    # 服务配置
│           ├── nfc_tag_service.rc
│           └── BUILD.gn
├── sa_profile/                        # SA 配置
│   ├── 1148.json                     # SA 描述文件
│   └── BUILD.gn
├── utils/                             # 工具
│   └── sa_listener/                   # SA 监听器
├── test/                              # 测试代码（不作为业务证据）
├── BUILD.gn                           # 构建入口
├── connected_nfc_tag.gni              # GN 配置
├── bundle.json                        # 组件描述
├── README.md                          # 项目说明
└── figures/                           # 架构图
```

---

## 3. 关键概念

### 3.1 NDEF（NFC Data Exchange Format）

NDEF 是 NFC 论坛定义的标准数据格式，用于在 NFC 设备间交换数据。

```
┌─────────────────────────────────────────┐
│              NDEF Message                │
├─────────────────────────────────────────┤
│  ┌───────────────────────────────────┐  │
│  │     NDEF Record 1 (TNF=Well-Known)│  │
│  │     Type: "urn:nfc:wkt:T"         │  │
│  │     Payload: Text/URI/ MIME       │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │     NDEF Record 2 (Optional)      │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 3.2 RF 状态

| 状态 | 值 | 描述 |
|------|------|------|
| `NFC_RF_LEAVE` | 0 | NFC 射频场已离开 |
| `NFC_RF_ENTER` | 1 | NFC 射频场已进入 |

### 3.3 系统能力 ID

```cpp
#define NFC_CONNECTED_TAG_ABILITY_ID 1148
```

---

## 4. 使用限制

| 限制项 | 值 | 说明 |
|--------|------|------|
| **NDEF 数据长度** | 1-512 字节 | 超出范围返回错误 |
| **最大回调数** | 30 | 每个进程最多注册 30 个回调 |
| **权限要求** | 系统权限 | 需 `ohos.permission.NFC_TAG` |

---

## 5. 相关文档

| 文档 | 链接 |
|------|------|
| JS API 参考 | `docs/zh-cn/application-dev/reference/apis/js-apis-connectedTag.md` |
| 架构设计 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [02_NAPI.md](./02_NAPI.md) |
| 构建说明 | [04_Build.md](./04_Build.md) |
