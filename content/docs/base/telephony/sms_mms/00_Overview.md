# 项目概览

## 目的

本文档提供 OpenHarmony 短彩信模块 (telephony/sms_mms) 的项目定位、边界、核心能力和运行环境概述，帮助读者快速建立整体概念。

## 适用范围

本文档覆盖：
- 项目定位和核心职责
- 功能边界和依赖关系
- 运行环境和硬件要求
- 关键概念和术语定义

不包含：
- 具体实现细节（见架构文档）
- API 使用方法（见 N-API 接口文档）
- 构建配置（见 GN 目标文档）

## 项目定位

### 系统定位

短彩信模块是 OpenHarmony **电话服务子系统 (telephony)** 的核心组件之一，负责移动数据用户的短信收发和彩信编解码功能。

- **子系统**: telephony
- **组件名**: sms_mms
- **SA ID**: 4008 (TELEPHONY_SMS_MMS_SYS_ABILITY_ID)
- **进程名**: telephony
- **系统能力**: SystemCapability.Telephony.SmsMms

### 职责边界

```
┌─────────────────────────────────────────────────────────────────┐
│           telephony/sms_mms 职责范围                  │
├─────────────────────────────────────────────────────────────────┤
│                                                              │
│  ✅ 短信收发处理                                           │
│     - GSM/CDMA 网络短信发送                                │
│     - GSM/CDMA 网络短信接收                                │
│     - IMS 短信发送（可选特性）                             │
│                                                              │
│  ✅ 短信协议处理                                            │
│     - PDU（协议数据单元）编解码                                │
│     - TPDU（传输协议数据单元）处理                             │
│     - 用户数据编码/解码（7-bit, 8-bit, UCS2）              │
│                                                              │
│  ✅ 特殊短信类型                                            │
│     - WAP Push 消息接收和处理                                │
│     - 小区广播（Cell Broadcast）接收                            │
│     - 彩信通知和内容下载                                    │
│                                                              │
│  ✅ SIM 卡短信管理                                          │
│     - SIM 卡短信记录的增删改查                              │
│     - SMSC（短信服务中心）地址配置                            │
│     - 默认卡槽配置                                          │
│                                                              │
│  ✅ 彩信支持（可选特性）                                    │
│     - 彩信 PDU 编解码                                       │
│     - 彩信发送（HTTP）                                       │
│     - 彩信接收和下载                                        │
│                                                              │
└─────────────────────────────────────────────────────────────────┘
```

### 不在职责范围

❌ **语音呼叫处理** - 由 telephony_call_manager 负责
❌ **数据连接管理** - 由 telephony_data 负责数据连接
❌ **消息存储和展示** - 由 MMS 应用负责
❌ **电话号码解析和验证** - 使用 libphonenumber 库
❌ **核心网络管理** - 由 telephony_core_service 负责

## 核心功能

### 1. 短信发送

支持多种短信发送场景：

- **文本短信**：普通文本短信发送
- **数据短信**：二进制数据短信（WAP Push 等）
- **长短信分段**：自动将超长短信分段发送（最多 15 段）
- **分段重组**：接收端自动重组分段短信
- **送达报告**：支持请求和接收送达状态报告

### 2. 短信接收

完整的短信接收流程：

- **新短信通知**：监听 RIL 层新短信事件
- **PDU 解析**：解析 GSM/CDMA PDU 协议
- **重复检测**：检测和过滤重复短信
- **分段重组**：自动重组分段短信
- **数据库存储**：将接收的短信存储到数据库
- **应用通知**：通过 CommonEvent 广播到 MMS 应用

### 3. 网络制式支持

支持三种网络制式的短信处理：

- **GSM**：3GPP 标准（全球主流）
- **CDMA**：3GPP2 标准（部分运营商）
- **IMS**：IP 多媒体子系统（通过 IMS 服务转发）

**证据**：`services/sms/gsm/gsm_sms_sender.cpp:1-100` 和 `services/sms/cdma/cdma_sms_sender.cpp:1-100`

### 4. 特殊短信类型

#### WAP Push
- 接收和处理 WAP Push 消息
- 支持类型：Service Indication、SI（Service Indication）等
- 代码位置：`services/sms/sms_wap_push_handler.cpp`

#### 小区广播（Cell Broadcast）
- 接收运营商发送的小区广播消息
- 支持配置接收范围（消息 ID 范围）
- 代码位置：`services/sms/gsm/gsm_cb_codec.cpp`

### 5. 彩信支持（可选特性）

彩信功能通过编译特性开关控制：

- **编码**：将多媒体文件编码成彩信 PDU
- **解码**：解析彩信 PDU 内容
- **发送**：通过 HTTP 发送到彩信中心（MMSC）
- **接收**：接收彩信通知并下载内容

**证据**：`BUILD.gn:28-32` 定义 `SMS_SUPPORT_MMS` 宏

### 6. SIM 卡短信管理

- **添加**：向 SIM 卡写入短信
- **删除**：从 SIM 卡删除短信
- **更新**：更新 SIM 卡短信状态
- **查询**：获取 SIM 卡所有短信
- **SMSC 配置**：设置/获取短信服务中心地址

**证据**：`services/sms/sms_misc_manager.cpp:1-200`

### 7. IMS 短信支持（可选特性）

通过 IMS 服务发送短信：

- **检测 IMS 支持**：查询网络是否支持 IMS
- **IMS 格式配置**：获取/设置 IMS 短信格式
- **通过 IMS 发送**：将短信请求转发到 IMS 服务

**证据**：`services/sms/ims_service_interaction/src/ims_sms_client.cpp:1-150`

### 8. 卫星短信支持（可选特性）

通过卫星服务发送短信（用于应急通信场景）：

- **卫星注册**：注册到卫星服务通知
- **卫星发送**：通过卫星网络发送短信

**证据**：`BUILD.gn:109-116` 定义卫星服务交互代码

## 运行环境

### 硬件要求

- **Modem**：支持独立蜂窝通信的调制解调器
- **SIM 卡**：至少一张 SIM 卡（支持双卡槽）
- **存储**：用于短信数据库存储

### 软件依赖

从 `bundle.json:36-65` 识别的关键依赖：

| 依赖组件 | 用途 |
|---------|------|
| **core_service** | 电话核心服务，提供 RIL 接口 |
| **ability_runtime** | Ability 运行时框架 |
| **safwk / samgr** | 系统能力框架和服务管理器 |
| **ipc** | IPC 框架，支持 Binder 通信 |
| **napi** | Node-API 框架，用于 JS 绑定 |
| **hilog / hisysevent** | 日志和系统事件框架 |
| **data_share** | 数据共享框架，用于数据库访问 |
| **netstack / curl** | HTTP 客户端，用于 MMS 发送 |
| **power_manager** | 电源管理（可选特性）|
| **access_token** | 访问令牌，用于权限校验 |
| **libphonenumber** | 电话号码解析和验证 |
| **icu** | 国际化组件，支持编码转换 |

### 第三方库依赖

- **glib**：C 语言通用库（见 README_zh.md:58）
- **curl**：HTTP 客户端（MMS 发送）
- **protobuf**：Protocol Buffers（用于数据序列化）

## 关键概念

### PDU（Protocol Data Unit）

协议数据单元，短信在 Modem 和网络之间传输的二进制格式。

- **GSM TPDU**：传输协议数据单元（3GPP TS 23.040）
- **CDMA Teleservice**：CDMA 网络短信格式
- 编码代码：`services/sms/gsm/gsm_sms_tpdu_codec.cpp`

### SMSC（Short Message Service Center）

短信服务中心，负责短信的存储转发。

- 作用：接收发送方短信、转发到接收方
- 配置：通过 `setSmscAddr`/`getSmscAddr` API
- 验证：`services/sms/sms_service.cpp:782-790` 检查地址合法性

### 分段短信（Concatenated SMS）

超长短信自动分成多段发送。

- **GSM 标准**：每段最多 160 字符（7-bit 编码）或 70 字符（UCS2）
- **段数限制**：最多 15 段
- 重组：接收端根据 `msgRefId`、`seqNum`、`totalSeg` 重组
- 代码位置：`services/sms/sms_receive_handler.cpp:CombineMessagePart()`

### 送达报告（Delivery Report）

短信送达状态通知。

- **请求**：发送时设置 `TP-SRR` 位
- **接收**：收到 SMS-STATUS-REPORT 消息
- **回调**：通过 `IDeliveryShortMessageCallback` 返回
- 错误码：`services/sms/gsm/gsm_sms_message.cpp` 定义状态码

### CommonEvent

OpenHarmony 的跨进程事件机制。

- **用途**：SMS 服务通过 CommonEvent 广播新短信到应用
- **事件类型**：`common.event.SMS_RECEIVE_COMPLETED`
- **代码位置**：`services/sms/sms_broadcast_subscriber_receiver.cpp`

### SystemAbility (SA)

OpenHarmony 的系统能力框架。

- **SA ID 4008**：短信/彩信服务
- **进程**：运行在 `telephony` 进程
- **依赖**：等待 SA 4010 (core_service) 启动
- 代码位置：`services/sms/sms_service.cpp:OnStart()`

### RIL Adapter

无线接口层适配器，负责与 Modem 通信。

- **提供**：发送短信、接收短信、网络状态查询
- **来源**：telephony_core_service 提供
- **调用**：通过 `CoreManagerInner::GetInstance()` 获取 RIL 接口

## 特性开关

从 `smsmms.gni:14-29` 识别的编译特性：

| 特性 | 默认值 | 说明 | 宏定义 |
|------|-------|------|--------|
| `sms_mms_dynamic_start` | false | 动态启动服务 | - |
| `sms_mms_tel_power_mode` | false | 省电模式优化 | `BASE_POWER_IMPROVEMENT_FEATURE` |
| `sms_mms_feature_support_mms` | true | 启用 MMS 功能 | `SMS_SUPPORT_MMS` |
| `sms_mms_satellite` | false | 启用卫星短信 | `SMS_MMS_SATELLITE` |

## 限制和约束

### 功能限制

- **双卡限制**：最多支持 2 个卡槽（SIM_SLOT_COUNT = 2）
- **发送重试**：默认最多重试 3 次（MAX_SEND_RETRIES = 3）
- **PDU 大小限制**：最大 0xFF 字节（PDU_BUFFER_MAX_SIZE）
- **地址长度限制**：最大 20 位数字（MAX_ADDRESS_LEN = 21）

### 开发约束

从 `README_zh.md:57-60`：

- **开发语言**：JavaScript（应用层）/ C++（服务层）
- **平台要求**：搭载支持独立蜂窝通信的 Modem 和 SIM 卡的设备

## 安全概述

### 权限要求

| API | 所需权限 | 用途 |
|-----|---------|------|
| `sendMessage` | `ohos.permission.SEND_MESSAGES` | 发送短信 |
| `setSmscAddr` | `ohos.permission.SET_TELEPHONY_STATE` | 设置 SMSC |
| `getSmscAddr` | `ohos.permission.GET_TELEPHONY_STATE` | 获取 SMSC |
| `addSimMessage` | `ohos.permission.RECEIVE_MESSAGES` | 添加 SIM 短信 |
| MMS 相关 API | 系统应用权限 | 彩信编解码 |

**证据**：`README_zh.md:65-77` 和 `services/sms/sms_service.cpp:TelephonyPermission::CheckPermission()`

### 安全机制

- **输入验证**：参数类型、长度、范围检查
- **权限检查**：所有敏感 API 都有权限校验
- **系统应用限制**：MMS API 仅限系统应用使用
- **边界检查**：缓冲区大小、数组长度验证

详细安全分析见：[06_Security_Review.md](06_Security_Review.md)

## 相关跳转链接

- [目录结构](01_Directory_Structure.md) - 了解代码组织方式
- [架构说明](02_Architecture.md) - 深入了解组件交互
- [N-API 接口](03_NAPI_Interface.md) - 学习 JS API 使用
- [安全评审](06_Security_Review.md) - 了解安全机制

## 参考资料

- **中文 README**: `README_zh.md`
- **英文 README**: `README.md`
- **官方 API 文档**: https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-telephony-kit/js-apis-sms.md
- **相关仓**: telephony_core_service
