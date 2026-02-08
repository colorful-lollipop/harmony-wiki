# 目录结构与模块职责

## 文档信息

| 项目 | 内容 |
|------|------|
| **目的** | 描述 NFC 组件的源码目录结构和各模块职责 |
| **适用范围** | NFC 组件开发、维护人员 |
| **相关文档** | [架构设计](01_Architecture.md)、[内部接口](04_Inner_API.md) |

---

## 1. 顶层目录结构

```
/foundation/communication/nfc
├── interfaces/              # 接口层（IDL + 内部 API）
│   └── inner_api/          # 系统内部接口
├── frameworks/              # 框架层（多语言绑定）
│   ├── js/napi/            # JavaScript N-API 实现
│   ├── cj/                 # Cangjie FFI 实现
│   └── ets/taihe/          # ArkTS/ETS 实现
├── services/                # 服务层（核心服务实现）
│   ├── src/                # 源码
│   ├── include/            # 头文件
│   └── etc/init/           # 初始化配置
├── sa_profile/              # 系统能力配置
├── test/                    # 测试代码（IGNORE）
├── wiki/                    # 本文档
├── bundle.json              # 组件配置
├── nfc.gni                  # GN 构建配置
├── hisysevent.yaml          # 系统事件配置
└── README.md                # 项目说明
```

---

## 2. interfaces/inner_api/ - 内部接口层

### 2.1 目录结构

```
interfaces/inner_api/
├── common/                  # 通用接口和数据结构
│   ├── inci_ce_interface.h
│   ├── inci_nfcc_interface.h
│   ├── inci_tag_interface.h
│   ├── inci_native_interface.h
│   ├── ndef_message.h
│   ├── taginfo.h
│   ├── nfc_sdk_common.h
│   └── ...
├── controller/              # NFC 控制器接口
│   ├── idl/
│   │   └── INfcController.idl
│   ├── nfc_controller.h
│   ├── nfc_sa_client.h
│   └── ...
├── tags/                    # 标签操作接口
│   ├── idl/
│   │   └── ITagSession.idl
│   ├── basic_tag_session.h
│   ├── ndef_tag.h
│   ├── nfca_tag.h
│   ├── mifare_classic_tag.h
│   └── ...
└── cardEmulation/           # 卡模拟接口
    ├── idl/
    │   └── IHceSession.idl
    ├── hce_service.h
    └── ...
```

### 2.2 模块职责

#### common/
- **NCI 接口定义**：`INciNfccInterface`, `INciTagInterface`, `INciCeInterface`
- **数据结构**：`TagInfo`, `NdefMessage`, `NdefRecord`
- **错误码定义**：`ErrorCode` 枚举
- **工具类**：`NfcSdkCommon`, `NfcBasicProxy`

#### controller/
- **IDL 接口**：INfcController - NFC 开关、状态查询
- **客户端 API**：`NfcController` - 客户端调用封装
- **SA 客户端**：`NfcSaClient` - System Ability 连接管理

#### tags/
- **IDL 接口**：ITagSession - 标签连接、读写
- **基础标签**：`BasicTagSession` - 所有标签类型的基类
- **标签类型**：NdefTag, IsoDepTag, NfcATag, MifareClassicTag 等
- **回调接口**：`IForegroundCallback`, `IReaderModeCallback`

#### cardEmulation/
- **IDL 接口**：IHceSession - HCE 启停、APDU 收发
- **服务 API**：`HceService` - 卡模拟服务客户端
- **回调接口**：`IHceCmdCallback` - APDU 命令回调

---

## 3. frameworks/ - 框架层

### 3.1 JS N-API 实现 (frameworks/js/napi/)

```
frameworks/js/napi/
├── common/                  # N-API 通用工具
│   ├── nfc_napi_common_utils.cpp  # 参数解析、错误处理
│   ├── nfc_napi_common_utils.h
│   ├── nfc_api_control.cpp        # NFC 支持检查
│   └── nfc_ha_event_report.cpp    # 事件上报
├── controller/              # NFC 控制器 N-API
│   ├── nfc_napi_controller.cpp    # 模块注册
│   ├── nfc_napi_controller_adapter.cpp
│   ├── nfc_napi_controller_event.cpp
│   └── BUILD.gn
├── tag/                     # 标签 N-API
│   ├── nfc_napi_tag.cpp           # 模块注册、Tag 类定义
│   ├── nfc_napi_tag_session.cpp   # 基础标签操作
│   ├── nfc_napi_tag_ndef.cpp      # NDEF 操作
│   ├── nfc_napi_tag_mifare_classic.cpp
│   ├── nfc_napi_foreground_dispatch.cpp
│   └── BUILD.gn
└── cardEmulation/           # 卡模拟 N-API
    ├── nfc_napi_cardEmulation.cpp
    ├── nfc_napi_hce_adapter.cpp
    └── BUILD.gn
```

#### 模块职责

**common/**
- **参数解析**：字符串、数字、数组、对象解析
- **类型检查**：`IsNumber()`, `IsString()`, `IsArray()` 等
- **错误处理**：业务错误码转换、异常抛出
- **异步框架**：`HandleAsyncWork()`, `DoAsyncCallbackOrPromise()`

**controller/**
- **模块注册**：`nfc.controller` - 控制器模块
- **API 实现**：`enableNfc()`, `disableNfc()`, `getNfcState()`
- **事件处理**：`on()`, `off()` - NFC 状态事件

**tag/**
- **模块注册**：`nfc.tag` - 标签模块
- **Tag 类注册**：NfcATag, NfcBTag, IsoDepTag, NdefTag 等
- **静态函数**：`getNfcA()`, `getNdef()`, `registerForegroundDispatch()`
- **NDEF 工具**：`ndef.makeUriRecord()`, `ndef.messageToBytes()`

**cardEmulation/**
- **模块注册**：`nfc.cardEmulation` - 卡模拟模块
- **HceService 类**：`start()`, `stop()`, `transmit()`
- **事件处理**：APDU 命令监听

### 3.2 ETS Taihe 实现 (frameworks/ets/taihe/)

```
frameworks/ets/taihe/
├── nfc_common/             # 通用工具
├── nfc_controller/         # 控制器实现
├── nfc_tag/                # 标签实现
├── nfc_cardEmulation/      # 卡模拟实现
└── BUILD.gn
```

### 3.3 Cangjie FFI (frameworks/cj/)

```
frameworks/cj/
├── controller/             # 控制器 FFI
└── cardEmulation/          # 卡模拟 FFI
```

---

## 4. services/ - 服务层

### 4.1 目录结构

```
services/
├── include/                 # 公共头文件
│   ├── nfc_service.h
│   ├── nfc_sa_manager.h
│   ├── nfc_polling_manager.h
│   └── ...
├── src/
│   ├── nfc_service.cpp              # 主服务实现
│   ├── nfc_sa_manager.cpp           # SA 生命周期
│   ├── nfc_event_handler.cpp        # 事件处理
│   ├── nfc_polling_manager.cpp      # 轮询管理
│   ├── nfc_routing_manager.cpp      # 路由管理
│   ├── nfc_polling_params.cpp       # 轮询参数
│   ├── controller/                  # 控制器 IPC 实现
│   │   └── (ipc/controller/ 下实现)
│   ├── tag/                         # 标签处理
│   │   ├── tag_dispatcher.cpp       # 标签分发
│   │   ├── ndef_har_dispatch.cpp    # HAR 分发
│   │   ├── ndef_har_data_parser.cpp # HAR 数据解析
│   │   ├── ndef_wifi_data_parser.cpp
│   │   ├── ndef_bt_data_parser.cpp
│   │   ├── wifi_connection_manager.cpp
│   │   └── bt_connection_manager.cpp
│   ├── card_emulation/              # 卡模拟
│   │   ├── ce_service.cpp           # CE 服务
│   │   ├── host_card_emulation_manager.cpp
│   │   ├── nfc_ability_connection_callback.cpp
│   │   └── setting_data_share_impl.cpp
│   ├── ipc/                         # IPC 实现
│   │   ├── controller/              # 控制器 IPC
│   │   │   ├── nfc_controller_impl.cpp
│   │   │   ├── nfc_controller_callback_proxy.cpp
│   │   │   └── ...
│   │   ├── tags/                    # 标签 IPC
│   │   │   ├── tag_session.cpp
│   │   │   ├── foreground_callback_proxy.cpp
│   │   │   └── ...
│   │   └── card_emulation/          # 卡模拟 IPC
│   │       ├── hce_session.cpp
│   │       └── hce_cmd_callback_proxy.cpp
│   ├── nci_adapter/                 # NCI 适配层
│   │   ├── nci_native_selector.cpp  # 动态库选择
│   │   ├── nci_nfcc_proxy.cpp       # NFCC 代理
│   │   ├── nci_tag_proxy.cpp        # Tag 代理
│   │   ├── nci_ce_proxy.cpp         # CE 代理
│   │   └── nci_native_default/      # 默认 NCI 实现
│   │       ├── include/
│   │       └── src/
│   ├── external_deps/               # 外部依赖封装
│   │   ├── nfc_permission_checker.cpp
│   │   ├── app_data_parser.cpp
│   │   ├── nfc_data_share_impl.cpp
│   │   ├── nfc_event_publisher.cpp
│   │   ├── tag_ability_dispatcher.cpp
│   │   └── ...
│   ├── utils/                       # 工具类
│   │   ├── nfc_timer.cpp
│   │   ├── nfc_watch_dog.cpp
│   │   └── app_state_observer.cpp
│   └── notification/                # 通知模块
│       └── BUILD.gn
├── etc/init/                # 初始化配置
│   └── BUILD.gn
└── BUILD.gn
```

### 4.2 模块职责

#### 核心服务

**nfc_service.cpp**
- NFC 服务主类，继承 `INciTagInterface::ITagListener`, `INciCeInterface::ICeHostListener`
- 初始化所有子模块
- NFC 状态管理（开启/关闭）
- 标签/CE 事件回调处理

**nfc_sa_manager.cpp**
- System Ability 生命周期管理
- `OnStart()` / `OnStop()` 回调
- SA 动态卸载管理

**nfc_event_handler.cpp**
- NFC 事件处理器基类
- 系统事件订阅（关机、屏幕状态）

#### 管理器

**nfc_polling_manager.cpp**
- 前台分发注册管理
- 读卡模式注册管理
- 屏幕状态变化响应
- 轮询参数配置

**nfc_routing_manager.cpp**
- 路由表计算
- 默认支付应用管理
- 路由提交到 NFC 控制器

#### 标签模块

**tag_dispatcher.cpp**
- 标签分发逻辑
- 优先级：Reader Mode → Foreground → NDEF → Notification
- NDEF 消息解析和分发

**ndef_har_dispatch.cpp / ndef_har_data_parser.cpp**
- HAR（HarmonyOS Ability Runtime）分发
- NDEF 记录解析
- URI、Mime、应用记录处理

**wifi_connection_manager.cpp / bt_connection_manager.cpp**
- NDEF WiFi 记录处理（自动连接 WiFi）
- NDEF 蓝牙记录处理（自动配对蓝牙）

#### 卡模拟模块

**ce_service.cpp**
- 卡模拟服务主类
- AID 路由表管理
- 默认支付应用设置

**host_card_emulation_manager.cpp**
- HCE 管理
- APDU 命令分发
- 应用绑定验证

#### IPC 实现

**ipc/controller/**
- `NfcControllerImpl` - INfcController 服务端实现
- 权限校验、状态管理
- 回调注册/注销

**ipc/tags/**
- `TagSession` - ITagSession 服务端实现
- 标签连接、读写操作
- 前台/读卡模式回调

**ipc/card_emulation/**
- `HceSession` - IHceSession 服务端实现
- HCE 启停、APDU 收发

#### NCI 适配层

**nci_adapter/**
- `NciNativeSelector` - 动态库加载
- `NciNfccProxy` - NFCC 接口代理
- `NciTagProxy` - Tag 接口代理
- `NciCeProxy` - CE 接口代理

**nci_native_default/**
- 默认 NCI 实现
- `nfcc_nci_adapter.cpp` - NFCC 控制
- `tag_nci_adapter.cpp` - Tag 操作
- `nci_ce_impl_default.cpp` - 卡模拟

#### 外部依赖封装

**external_deps/**
- `nfc_permission_checker.cpp` - 权限检查封装
- `app_data_parser.cpp` - 应用信息解析
- `nfc_data_share_impl.cpp` - SettingsData 读写
- `nfc_event_publisher.cpp` - 公共事件发布
- `tag_ability_dispatcher.cpp` - Ability 分发

---

## 5. sa_profile/ - 系统能力配置

```
sa_profile/
└── BUILD.gn          # SA 配置文件构建
```

**内容**：NFC 服务 SA ID 1140 的配置

---

## 6. 关键文件索引

### 6.1 接口定义

| 文件 | 路径 | 说明 |
|------|------|------|
| INfcController.idl | interfaces/inner_api/controller/idl/ | 控制器 IPC 接口 |
| ITagSession.idl | interfaces/inner_api/tags/idl/ | 标签 IPC 接口 |
| IHceSession.idl | interfaces/inner_api/cardEmulation/idl/ | HCE IPC 接口 |
| inci_*.h | interfaces/inner_api/common/ | NCI 接口定义 |

### 6.2 服务入口

| 文件 | 路径 | 说明 |
|------|------|------|
| nfc_service.cpp | services/src/ | 主服务实现 |
| nfc_sa_manager.cpp | services/src/ | SA 管理 |
| nfc_controller_impl.cpp | services/src/ipc/controller/ | 控制器 IPC |
| tag_session.cpp | services/src/ipc/tags/ | 标签 IPC |
| hce_session.cpp | services/src/ipc/card_emulation/ | HCE IPC |

### 6.3 N-API 入口

| 文件 | 路径 | 说明 |
|------|------|------|
| nfc_napi_controller.cpp | frameworks/js/napi/controller/ | 控制器 N-API |
| nfc_napi_tag.cpp | frameworks/js/napi/tag/ | 标签 N-API |
| nfc_napi_cardEmulation.cpp | frameworks/js/napi/cardEmulation/ | HCE N-API |

### 6.4 构建配置

| 文件 | 路径 | 说明 |
|------|------|------|
| BUILD.gn | services/ | 服务构建 |
| nfc.gni | 根目录 | 全局配置 |
| bundle.json | 根目录 | 组件配置 |

