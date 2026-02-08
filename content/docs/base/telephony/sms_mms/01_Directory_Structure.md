# 目录结构与模块职责

## 目的

本文档提供短彩信模块的完整目录结构和各模块职责说明，帮助开发者快速定位代码和理解模块边界。

## 适用范围

本文档覆盖：
- 顶层目录结构（不含测试）
- 各目录的职责划分
- 关键文件位置和用途

不包含：
- 测试目录结构（test/）
- 构建产物目录（out/）
- 临时文件（.git/）

## 目录树结构

```
/base/telephony/sms_mms
├── BUILD.gn                      # 根构建文件，定义主共享库
├── bundle.json                   # 组件元数据和依赖配置
├── smsmms.gni                   # GN 特性开关和全局宏定义
├── LICENSE                       # Apache License 2.0
├── README.md / README_zh.md        # 项目说明文档
├── OAT.xml                      # 开放原子测试配置
├── figures/                      # README 文档资源文件
├── frameworks/                   # 短彩信内部框架接口层
├── interfaces/                   # 接口定义目录
├── sa_profile/                   # 系统能力（SA）配置文件
├── services/                     # 短彩信服务实现代码
├── test/                         # 测试目录（本文档不覆盖）
└── utils/                        # 通用工具库
```

## 目录职责详解

### 1. frameworks/ - 框架接口层

职责：提供不同语言和框架的 API 绑定。

```
frameworks/
├── js/
│   └── napi/                    # JavaScript N-API 绑定
│       ├── BUILD.gn               # N-API 模块构建
│       ├── src/
│       │   ├── napi_sms.cpp             # SMS N-API 实现（主模块）
│       │   ├── napi_mms.cpp             # MMS 编解码 N-API
│       │   ├── napi_mms_pdu.cpp         # MMS PDU 数据库操作
│       │   ├── napi_send_recv_mms.cpp   # MMS 发送/接收 N-API
│       │   ├── napi_sms_util.cpp        # N-API 工具函数
│       │   ├── send_callback.cpp          # 发送回调实现
│       │   └── delivery_callback.cpp       # 送达回调实现
│       └── include/
│           ├── napi_sms.h             # SMS 上下文结构
│           ├── napi_mms.h             # MMS 上下文和 NapiMms 类
│           ├── napi_send_recv_mms.h   # MMS 发送/接收上下文
│           └── napi_sms_util.h        # SMS 工具类声明
│
├── native/
│   ├── sms/                       # SMS 原生源代码
│   │   ├── BUILD.gn               # SMS 源代码集构建
│   │   ├── src/
│   │   │   └── short_message.cpp   # ShortMessage 类实现
│   │   └── include/
│   │       └── short_message.h   # ShortMessage 类定义
│   │
│   ├── mms/                       # MMS 原生源代码
│   │   ├── BUILD.gn               # MMS 源代码集构建
│   │   ├── src/                   # 17 个 MMS 编解码源文件
│   │   │   ├── mms_msg.cpp       # MMS 消息编解码
│   │   │   ├── mms_header.cpp   # MMS 头部处理
│   │   │   ├── mms_body.cpp     # MMS 主体处理
│   │   │   └── ...（其他 MMS 编解码文件）
│   │   └── include/
│   │       └── mms_msg.h       # MMS 消息类定义
│   │
│   └── tel_sms_mms_api.versionscript  # API 版本脚本（符号可见性控制）
│
├── cj/
│   ├── BUILD.gn                   # Cangjie FFI 绑定构建
│   ├── src/
│   │   └── sms_utils.cpp       # Cangjie 工具函数
│   └── include/
│       └── sms_utils.h         # Cangjie 工具类声明
│
└── ets/
    └── taihe/                     # ArkTS Taihe 框架绑定
        ├── BUILD.gn               # Taihe 绑定构建
        ├── src/
        │   └── ohos.telephony.sms.impl.cpp  # Taihe 实现
        └── abc/
            └── ohos.telephony.sms.abc      # Taihe ABC 文件
```

### 2. interfaces/ - 接口定义目录

职责：定义对应用和模块间的接口。

```
interfaces/
├── innerkits/                   # 模块间内部接口
│   ├── i_sms_service_interface.h           # SMS 服务接口（IRemoteBroker）
│   ├── sms_service_proxy.h              # SMS 服务客户端代理
│   ├── sms_service_manager_client.h      # SMS 服务管理器客户端（单例）
│   ├── sms_send_short_message_proxy.h   # 发送回调代理
│   ├── sms_delivery_short_message_proxy.h # 送达回调代理
│   │
│   ├── short_message.h                  # ShortMessage 数据结构
│   ├── sms_constants_utils.h           # SMS 常量和工具
│   ├── sms_mms_errors.h               # 错误码定义
│   │
│   ├── sms_service_ipc_interface_code.h  # SMS 服务 IPC 接口码（28 个）
│   ├── send_short_message_callback_ipc_interface_code.h   # 发送回调 IPC 码
│   ├── delivery_short_message_callback_ipc_interface_code.h # 送达回调 IPC 码
│   │
│   ├── i_send_short_message_callback.h   # 发送回调接口定义
│   ├── i_delivery_short_message_callback.h # 送达回调接口定义
│   │
│   ├── mms_codec_type.h                # MMS 编解码类型定义
│   ├── mms_msg.h                     # MMS 消息类
│   ├── mms_address.h                 # MMS 地址类
│   └── mms_attachment.h              # MMS 附件类
│
├── innerkits/ims/                  # IMS 短信接口
│   ├── ims_sms_interface.h             # IMS SMS 接口
│   ├── ims_sms_types.h                # IMS SMS 类型定义
│   ├── ims_sms_client.h               # IMS SMS 客户端（单例）
│   ├── ims_sms_proxy.h                # IMS SMS 代理
│   ├── ims_sms_callback_interface.h      # IMS 回调接口
│   ├── ims_sms_callback_proxy.h        # IMS 回调代理
│   ├── ims_sms_callback_stub.h         # IMS 回调 Stub
│   ├── ims_sms_ipc_interface_code.h    # IMS IPC 接口码（4 个）
│   └── ims_sms_callback_ipc_interface_code.h  # IMS 回调 IPC 码
│
└── innerkits/satellite/             # 卫星短信接口
    ├── i_satellite_sms_service.h       # 卫星短信服务接口
    ├── i_satellite_sms_callback.h     # 卫星短信回调接口
    ├── satellite_sms_service_ipc_interface_code.h  # 卫星 IPC 接口码
    └── satellite_sms_callback_ipc_interface_code.h   # 卫星回调 IPC 码
```

### 3. sa_profile/ - 系统能力配置

职责：定义 SMS/MMS 系统能力的启动和依赖配置。

```
sa_profile/
├── BUILD.gn                       # SA 配置构建
├── 4008.json                      # SA 4008 配置（静态启动）
└── 4008_dynamic.json              # SA 4008 配置（动态启动）

4008.json 内容示例：
{
  "name": "4008",
  "libpath": "libtel_sms_mms.z.so",
  "run-on-create": true,
  "depend": [4010],
  "depend_time_out": 60000
}
```

### 4. services/ - 服务实现代码

职责：实现短信/MMS 核心服务逻辑。

```
services/
├── sms/                         # SMS 服务核心实现
│   ├── include/
│   │   ├── sms_service.h          # SMS 服务主类（SystemAbility）
│   │   ├── sms_interface_stub.h   # IPC Stub 实现
│   │   ├── sms_interface_manager.h # 接口管理器
│   │   ├── sms_send_manager.h    # 发送管理器
│   │   ├── sms_receive_manager.h  # 接收管理器
│   │   ├── sms_misc_manager.h    # 杂项管理器
│   │   ├── sms_network_policy_manager.h # 网络策略管理器
│   │   ├── sms_sender.h          # 发送器基类
│   │   ├── sms_receive_handler.h  # 接收处理器基类
│   │   ├── sms_base_message.h    # SMS 消息基类
│   │   ├── sms_send_indexer.h    # 发送索引器
│   │   ├── sms_receive_indexer.h # 接收索引器
│   │   ├── sms_pdu_buffer.h      # PDU 缓冲区
│   │   ├── sms_wap_push_handler.h   # WAP Push 处理
│   │   ├── sms_cell_broadcast_handler.h  # 小区广播处理
│   │   ├── gsm/                  # GSM 头文件
│   │   ├── cdma/                 # CDMA 头文件
│   │   └── satellite/            # 卫星头文件
│   │
│   ├── gsm/                        # GSM 网络短信实现
│   │   ├── gsm_sms_message.cpp          # GSM 消息解析
│   │   ├── gsm_sms_sender.cpp          # GSM 发送实现
│   │   ├── gsm_sms_receive_handler.cpp  # GSM 接收处理
│   │   ├── gsm_sms_param_codec.cpp     # GSM 参数编解码
│   │   ├── gsm_sms_tpdu_codec.cpp     # GSM TPDU 编解码
│   │   ├── gsm_user_data_encode.cpp   # GSM 用户数据编码
│   │   ├── gsm_user_data_decode.cpp   # GSM 用户数据解码
│   │   ├── gsm_cb_codec.cpp          # 小区广播编解码
│   │   ├── gsm_cb_gsm_codec.cpp      # GSM CB 编解码
│   │   ├── gsm_cb_umts_codec.cpp     # UMTS CB 编解码
│   │   ├── gsm_cb_pdu_decode_buffer.cpp # CB PDU 解码
│   │   ├── gsm_sms_common_utils.cpp  # GSM 通用工具
│   │   ├── gsm_sms_tpdu_encode.cpp   # GSM TPDU 编码
│   │   ├── gsm_sms_tpdu_decode.cpp   # GSM TPDU 解码
│   │   ├── cb_start_ability.cpp       # CB 启动能力
│   │   └── gsm_sms_cb_handler.cpp     # CB 处理器
│   │
│   ├── cdma/                       # CDMA 网络短信实现
│   │   ├── cdma_sms_message.cpp        # CDMA 消息解析
│   │   ├── cdma_sms_sender.cpp        # CDMA 发送实现
│   │   ├── cdma_sms_receive_handler.cpp    # CDMA 接收处理
│   │   ├── cdma_sms_parameter_record.cpp    # CDMA 参数记录
│   │   ├── cdma_sms_sub_parameter.cpp     # CDMA 子参数
│   │   ├── cdma_sms_teleservice_message.cpp # CDMA teleservice 消息
│   │   └── cdma_sms_transport_message.cpp   # CDMA 传输消息
│   │
│   ├── ims_service_interaction/         # IMS 服务交互层
│   │   └── src/
│   │       ├── ims_sms_client.cpp      # IMS SMS 客户端
│   │       ├── ims_sms_proxy.cpp       # IMS SMS 代理
│   │       └── ims_sms_callback_stub.cpp # IMS 回调 Stub
│   │
│   ├── satellite_service_interaction/  # 卫星服务交互层（可选）
│   │   └── src/
│   │       ├── satellite_sms_client.cpp      # 卫星 SMS 客户端
│   │       └── satellite_sms_callback_stub.cpp # 卫星回调 Stub
│   │
│   ├── proxy/                       # IPC 代理实现
│   │   ├── sms_send_short_message_proxy.cpp     # 发送代理
│   │   └── sms_delivery_short_message_proxy.cpp  # 送达代理
│   │
│   └── (其他实现文件)
│       ├── sms_interface_manager.cpp
│       ├── sms_send_manager.cpp
│       ├── sms_receive_manager.cpp
│       ├── sms_misc_manager.cpp
│       ├── sms_network_policy_manager.cpp
│       ├── sms_service.cpp
│       ├── sms_interface_stub.cpp
│       ├── sms_wap_push_handler.cpp
│       ├── sms_pdu_buffer.cpp
│       └── ...（其他实现）
│
├── mms/                         # MMS 服务实现（可选特性）
│   ├── include/
│   │   ├── mms_persist_helper.h   # MMS 持久化助手
│   │   ├── mms_network_manager.h  # MMS 网络管理器
│   │   ├── mms_send_manager.h     # MMS 发送管理器
│   │   ├── mms_receive_manager.h   # MMS 接收管理器
│   │   ├── mms_sender.h          # MMS 发送器
│   │   └── mms_receive.h         # MMS 接收器
│   │
│   ├── mms_persist_helper.cpp       # MMS 数据库操作
│   ├── mms_apn_info.cpp           # MMS APN 信息
│   ├── mms_conn_callback_stub.cpp  # MMS 连接回调 Stub
│   ├── mms_network_client.cpp      # MMS 网络客户端（HTTP）
│   ├── mms_network_manager.cpp     # MMS 网络管理器
│   ├── mms_receive.cpp           # MMS 接收实现
│   ├── mms_receive_manager.cpp   # MMS 接收管理器
│   ├── mms_send_manager.cpp      # MMS 发送管理器
│   └── mms_sender.cpp           # MMS 发送器
│
└── telephony_ext_wrapper/        # 电话扩展包装器（可选特性）
    ├── include/
    │   └── telephony_ext_wrapper.h   # 扩展包装器头
    └── src/
        └── telephony_ext_wrapper.cpp # 扩展包装器实现
```

### 5. utils/ - 通用工具库

职责：提供通用工具函数供各模块使用。

```
utils/
├── sms_common_utils.cpp          # SMS 通用工具
├── string_utils.cpp              # 字符串处理工具
├── text_coder.cpp               # 文本编码/解码工具
├── sms_hisysevent.cpp          # HiSysEvent 系统事件封装
└── sms_policy_utils.cpp          # SMS 策略工具
```

## 模块职责划分

### 核心管理层

| 模块 | 职责 | 关键类 | 文件位置 |
|------|------|---------|---------|
| **服务入口** | SMS 服务生命周期管理、IPC 调度 | `SmsService` | `services/sms/sms_service.cpp` |
| **接口管理** | 创建和管理各子管理器、分发请求 | `SmsInterfaceManager` | `services/sms/sms_interface_manager.cpp` |
| **发送管理** | 根据网络制式调度发送器 | `SmsSendManager` | `services/sms/sms_send_manager.cpp` |
| **接收管理** | 监听新短信、分发接收处理 | `SmsReceiveManager` | `services/sms/sms_receive_manager.cpp` |
| **杂项管理** | SIM 操作、CB 配置、SMSC | `SmsMiscManager` | `services/sms/sms_misc_manager.cpp` |
| **网络策略** | 网络状态检测、IMS 注册 | `SmsNetworkPolicyManager` | `services/sms/sms_network_policy_manager.cpp` |

### 协议处理层

| 模块 | 职责 | 关键类 | 文件位置 |
|------|------|---------|---------|
| **GSM 发送** | GSM 网络短信发送 | `GsmSmsSender` | `services/sms/gsm/gsm_sms_sender.cpp` |
| **GSM 接收** | GSM 网络短信接收 | `GsmSmsReceiveHandler` | `services/sms/gsm/gsm_sms_receive_handler.cpp` |
| **CDMA 发送** | CDMA 网络短信发送 | `CdmaSmsSender` | `services/sms/cdma/cdma_sms_sender.cpp` |
| **CDMA 接收** | CDMA 网络短信接收 | `CdmaSmsReceiveHandler` | `services/sms/cdma/cdma_sms_receive_handler.cpp` |
| **WAP Push** | WAP Push 消息处理 | `SmsWapPushHandler` | `services/sms/sms_wap_push_handler.cpp` |
| **小区广播** | CB 消息编解码 | `GsmSmsCbHandler` | `services/sms/gsm/gsm_cb_handler.cpp` |

### MMS 处理层（可选）

| 模块 | 职责 | 关键类 | 文件位置 |
|------|------|---------|---------|
| **MMS 发送** | 彩信发送管理 | `MmsSendManager` | `services/mms/mms_send_manager.cpp` |
| **MMS 接收** | 彩信接收管理 | `MmsReceiveManager` | `services/mms/mms_receive_manager.cpp` |
| **MMS 网络** | HTTP 客户端、网络管理 | `MmsNetworkClient` | `services/mms/mms_network_client.cpp` |
| **MMS 编解码** | MMS PDU 编解码 | `MmsMsg` | `frameworks/native/mms/src/mms_msg.cpp` |

### 特殊服务交互层（可选）

| 模块 | 职责 | 关键类 | 文件位置 |
|------|------|---------|---------|
| **IMS 交互** | 与 IMS 服务通信 | `ImsSmsClient` | `services/sms/ims_service_interaction/src/ims_sms_client.cpp` |
| **卫星交互** | 与卫星服务通信 | `SatelliteSmsClient` | `services/sms/satellite_service_interaction/src/satellite_sms_client.cpp` |

### API 绑定层

| 模块 | 职责 | 导出模块名 | 文件位置 |
|------|------|------------|---------|
| **JS 绑定** | 提供 JavaScript API | `@ohos.telephony.sms` | `frameworks/js/napi/src/napi_sms.cpp` |
| **Taihe 绑定** | 提供 ArkTS API | `@ohos.telephony.sms` | `frameworks/ets/taihe/src/ohos.telephony.sms.impl.cpp` |
| **Cangjie 绑定** | 提供 Cangjie FFI | `libcj_sms_ffi.so` | `frameworks/cj/src/sms_utils.cpp` |
| **Native API** | 提供 C++ API | `libtel_sms_mms_api.so` | `frameworks/native/BUILD.gn` |

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│               SmsService (SA 4008)                    │
│           services/sms/sms_service.cpp               │
└─────────────────────────────────────────────────────────────┘
                      │
                      ▼
    ┌───────────────────────────────────────────┐
    │        SmsInterfaceManager          │
    │  services/sms/sms_interface_manager.cpp  │
    └───────────────────────────────────────────┘
             │         │         │         │
    ┌────────┴──┐   ┌────┴────┐  ┌───┴──────────┐
    ▼            ▼    ▼             ▼             ▼
SendMgr    ReceiveMgr  MiscMgr     MmsSendMgr  MmsRecvMgr
    │            │          │          │           │
    │            │          │          │           │
    ▼            ▼          ▼          ▼           ▼
┌──────┐    ┌──────┐ ┌──────┐  ┌──────┐  ┌──────┐
│Gsm   │    │Gsm   │ │Misc  │  │Mms   │  │Mms   │
│Sender │    │Recv   │ │Mgr   │  │Sender │  │Recv   │
├──────┤    ├──────┤ ├──────┤  ├──────┤  ├──────┤
│Cdma   │    │Cdma   │       │  │       │  │       │
│Sender │    │Recv   │       │  │       │  │       │
└──────┘    └──────┘ └──────┘  └──────┘  └──────┘
     │            │                     │           │
     │            │                     │           │
     ▼            ▼                     ▼           ▼
┌──────────┐  ┌──────────┐      ┌──────────┐ ┌──────────┐
│GsmMsg    │  │GsmMsg    │      │ImsClient  │ │SatClient  │
│Codec     │  │Handler    │      │           │ │           │
└──────────┘  └──────────┘      └──────────┘ └──────────┘
     │            │                     │           │
     └────────────┴─────────────────────┴───────────┘
                               │
                               ▼
                    ┌──────────────────┐
                    │  RIL Adapter    │
                    │ (core_service)  │
                    └──────────────────┘
```

## 关键数据流

### 发送短信流程

1. App → N-API → `SmsInterfaceManager` → `SmsSendManager`
2. `SmsSendManager` → `GsmSmsSender`/`CdmaSmsSender`/`ImsSmsClient`
3. Sender → RIL Adapter → Modem

### 接收短信流程

1. Modem → RIL Adapter → `SmsReceiveManager`
2. `SmsReceiveManager` → `GsmSmsReceiveHandler`/`CdmaSmsReceiveHandler`
3. Handler → 解析 PDU → 组装消息 → 存入数据库
4. 广播到 MMS 应用

## 相关跳转链接

- [架构说明](02_Architecture.md) - 深入了解组件交互和数据流
- [内部 API](04_Internal_API.md) - 了解模块间接口定义
- [N-API 接口](03_NAPI_Interface.md) - 学习如何使用这些 API
