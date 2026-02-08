# 调用链附录

## 目的

本文档提供短彩信模块关键功能的完整调用链，帮助开发者理解代码执行流程和调试问题。

## 适用范围

本文档覆盖：
- 短信发送完整调用链
- 短信接收完整调用链
- MMS 发送/接收调用链
- PDU 编解码调用链
- IPC 通信调用链

## 短信发送调用链

### 文本短信发送 (GSM 网络)

```
JS API: sms.sendMessage(options)
    │
    ▼
frameworks/js/napi/src/napi_sms.cpp:SendMessage()
    │
    ├── 1. 参数解析和验证
    │   ├── napi_sms_util.cpp:MatchSendMessageParameters() — 检测消息类型
    │   └── napi_sms_util.cpp:IsValidSlotId() — 验证卡槽ID
    │
    ├── 2. 权限检查
    │   └── TelephonyPermission::CheckPermission(SEND_MESSAGES)
    │
    ├── 3. 创建异步工作
    │   └── NapiUtil::HandleAsyncWork()
    │       └── NativeSendMessage() (线程池执行)
    │
    ▼
frameworks/native/sms/src/sms_service_manager_client.cpp:GetSmsServiceProxy()
    │
    ├── DelayedSingleton::GetInstance() — 获取单例
    └── GetSmsServiceProxy(slotId) — 获取 IPC Proxy
    │
    ▼
interfaces/innerkits/sms_service_proxy.h:SmsServiceProxy::SendMessage()
    │
    ├── MessageParcel 构造
    ├── dataParcel.WriteInterfaceToken() — 写入描述符
    ├── dataParcel.WriteInt32(slotId) — 写入卡槽ID
    ├── dataParcel.WriteString16(desAddr) — 写入目标地址
    ├── dataParcel.WriteString16(scAddr) — 写入短信中心
    ├── dataParcel.WriteString16(text) — 写入内容
    └── dataParcel.WriteRemoteObject(callback) — 写入回调
    │
    ▼
services/sms/sms_interface_stub.cpp:OnRemoteRequest(TEXT_BASED_SMS_DELIVERY)
    │
    ├── 验证接口描述符
    │   └── ReadInterfaceToken() == GetDescriptor()
    │
    ├── 查找处理函数
    │   └── memberFuncMap_[TEXT_BASED_SMS_DELIVERY]
    │       └── OnSendSmsTextRequest()
    │
    ▼
services/sms/sms_interface_stub.cpp:OnSendSmsTextRequest()
    │
    ├── 从 MessageParcel 读取参数
    ├── slotId = data.ReadInt32()
    ├── desAddr = data.ReadString16()
    ├── scAddr = data.ReadString16()
    └── text = data.ReadString16()
    │
    ▼
services/sms/sms_service.cpp:SendMessage()
    │
    ├── 1. 权限检查
    │   └── CheckSmsPermission() — 检查 SEND_MESSAGES
    │
    ├── 2. 系统应用检查
    │   └── CheckCallerIsSystemApp() (仅 MMS 应用)
    │
    ├── 3. 获取 InterfaceManager
    │   └── GetSmsInterfaceManager(slotId)
    │
    ▼
services/sms/sms_interface_manager.cpp:TextBasedSmsDelivery()
    │
    ├── 检查 smsSendManager_ 是否初始化
    └── smsSendManager_->TextBasedSmsDelivery(...)
    │
    ▼
services/sms/sms_send_manager.cpp:TextBasedSmsDelivery()
    │
    ├── 1. 参数验证
    │   ├── !desAddr.empty()
    │   └── !text.empty()
    │
    ├── 2. 短码匹配检查
    │   └── smsShortCodeMatcher_->CheckShortCode()
    │
    ├── 3. 检查是否正在发送同一消息
    │   └── FindSameSendingMsg()
    │
    ├── 4. 网络检查
    │   └── CheckNetworkState()
    │
    ├── 5. 分段发送 (长短信)
    │   └── TextBasedSmsSplitDelivery()
    │       ├── SplitMessage() — 分割消息
    │       └── 循环调用 SendMessageInCallback()
    │
    └── 6. 直接发送 (短短信)
        └── SendMessageInCallback()
            │
            ▼
            GetNetWorkType() — 获取网络类型
            │
            ├── 如果是 IMS 网络 ──► SendImsSms()
            │   │
            │   └── services/sms/gsm/gsm_sms_sender.cpp:SendImsSms()
            │       ├── 构造 ImsMessageInfo
            │       └── ImsSmsClient::ImsSendMessage()
            │           │
            │           └── 转发到 IMS 服务
            │
            └── 如果是 CS 网络 ──► SendCsSms()
                │
                └── services/sms/gsm/gsm_sms_sender.cpp:SendCsSms()
                    │
                    ├── 1. 编码 PDU
                    │   └── GsmSmsMessage::CreateDefaultSubmitSmsTpdu()
                    │       │
                    │       ├── 构造 SmsTpdu
                    │       ├── EncodeAddress() — 编码目标地址
                    │       ├── EncodeTime() — 编码时间戳
                    │       └── EncodeUserData() — 编码用户数据
                    │           │
                    │           ├── gsm_user_data_encode.cpp:EncodeGsmUserData()
                    │           │   ├── 7-bit 编码
                    │           │   ├── 8-bit 编码
                    │           │   └── UCS2 编码
                    │           │
                    │           └── 添加 UDH (用户数据头，如果是分段短信)
                    │
                    ├── 2. 发送给 RIL
                    │   └── CoreManagerInner::SendGsmSms()
                    │       │
                    │       └── 发送到 RIL Adapter 服务
                    │
                    └── 3. 注册发送回调
                        └── SendResultCallBack()
```

### 发送结果回调流程

```
Modem ──► RIL Adapter ──► 发送响应事件
    │
    ▼
services/sms/gsm/gsm_sms_sender.cpp:ProcessEvent(RADIO_SEND_SMS)
    │
    ├── 获取发送索引器
    │   └── smsSendIndexer_->GetMsgRefId()
    │
    ├── 处理发送结果
    │   └── HandleMessageResponse()
    │       ├── 成功: SEND_SMS_SUCCESS
    │       └── 失败: SEND_SMS_FAILURE_*
    │
    ├── 调用发送回调
    │   └── ISendShortMessageCallback::OnSmsSendResult()
    │       │
    │       └── IPC 回调到客户端
    │
    └── 如果请求了送达报告
        └── 注册状态报告监听
```

## 短信接收调用链

### GSM 短信接收

```
Modem 收到新短信
    │
    ▼
telephony_core_service (RIL Adapter)
    │
    ├── 解析 PDU
    └── 发送事件: RadioEvent::RADIO_GSM_SMS
    │
    ▼
services/sms/sms_receive_manager.cpp:ProcessEvent()
    │
    ├── 获取当前卡槽ID
    └── gsmSmsReceiveHandler_->ProcessEvent(event)
    │
    ▼
services/sms/gsm/gsm_sms_receive_handler.cpp:ProcessEvent()
    │
    └── HandleSmsEvent(event)
        │
        ▼
        HandleSmsByType()
        │
        ├── 根据 PDU 类型分发:
        │   ├── SMS_TYPE_NORMAL ──► 处理普通短信
        │   ├── SMS_TYPE_STATUS_REPORT ──► 处理状态报告
        │   └── SMS_TYPE_WAP_PUSH ──► 处理 WAP Push
        │
        ▼
        TransformMessageInfo()
        │
        ├── 1. 解析 PDU
        │   └── GsmSmsMessage::PduAnalysis()
        │       │
        │       ├── gsm_sms_tpdu_decode.cpp:DecodeSmsTpdu()
        │       │   ├── 解析 SC 地址
        │       │   ├── 解析 TPDU 类型 (Deliver/Submit/Status Report)
        │       │   └── 解析各字段
        │       │
        │       └── gsm_user_data_decode.cpp:DecodeGsmUserData()
        │           ├── 7-bit 解码
        │           ├── 8-bit 解码
        │           └── UCS2 解码
        │
        ├── 2. 检查是否为重复短信
        │   └── IsRepeatedMessagePart()
        │
        ├── 3. 处理分段短信
        │   └── CombineMessagePart()
        │       │
        │       ├── 检查是否已存在分段
        │       │   └── FindSegmentMessage()
        │       │
        │       ├── 如果是新分段
        │       │   └── 添加到待重组列表
        │       │
        │       └── 如果所有分段已收到
        │           └── BuildConcatenatedSms() — 重组完整短信
        │
        └── 4. 存储和通知
            │
            ├── AddMsgToDB() — 存储到数据库
            │   └── DataShareHelper::Insert()
            │
            └── PublishBroadcastEvents() — 广播通知
                │
                ├── common.event.SMS_RECEIVE_COMPLETED
                └── 应用层收到广播
```

### 送达报告处理

```
Modem 收到 SMS-STATUS-REPORT
    │
    ▼
RIL Adapter ──► RadioEvent::RADIO_SMS_STATUS_REPORT
    │
    ▼
GsmSmsReceiveHandler::ProcessEvent()
    │
    └── HandleStatusReport()
        │
        ├── 解析状态报告 PDU
        │   └── PduAnalysis()
        │
        ├── 获取消息引用ID
        │   └── GetMsgRefId()
        │
        ├── 查找原始发送记录
        │   └── FindDeliveryCallback()
        │
        └── 调用送达回调
            └── IDeliveryShortMessageCallback::OnSmsDeliveryResult()
                │
                └── IPC 回调到客户端
```

## MMS 发送调用链

```
JS API: sms.sendMms(context, params)
    │
    ├── 需要权限: ohos.permission.SEND_MESSAGES
    └── 需要: 系统应用身份
    │
    ▼
frameworks/js/napi/src/napi_send_recv_mms.cpp:SendMms()
    │
    ├── 1. 系统应用检查
    │   └── CheckCallerIsSystemApp()
    │
    ├── 2. 参数解析
    │   ├── slotId
    │   ├── mmsc (彩信中心 URL)
    │   ├── data (PDU 文件路径)
    │   ├── ua (User-Agent)
    │   └── uaprof (UA Profile)
    │
    └── 3. 异步发送
        └── NativeSendMms()
            │
            ▼
            SmsService::SendMms()
                │
                ├── 权限检查
                └── smsInterfaceManager_->SendMms()
                    │
                    ▼
                    MmsSendManager::SendMms()
                        │
                        ├── 1. 检查 MMS 开关
                        │   └── IsMmsEnabled()
                        │
                        ├── 2. 检查网络连接
                        │   └── CheckNetworkConnectivity()
                        │
                        ├── 3. 读取 PDU 文件
                        │   └── GetMmsPduFromFile()
                        │
                        ├── 4. 创建网络请求
                        │   └── MmsSender::SendMms()
                        │       │
                        │       └── MmsNetworkClient::PostUrl()
                        │           │
                        │           ├── 获取 APN 代理配置
                        │           │   └── GetMmsApnPorxy()
                        │           │
                        │           ├── 构造 HTTP 请求
                        │           │   ├── SetURL(mmsc)
                        │           │   ├── SetMethod(POST)
                        │           │   ├── SetHeader()
                        │           │   └── SetBody(pduData)
                        │           │
                        │           ├── 发送 HTTP 请求
                        │           │   └── HttpClient::SendRequest()
                        │           │       └── 使用 libcurl
                        │           │
                        │           └── 处理响应
                        │               ├── 成功: HTTP 200
                        │               └── 失败: 错误码
                        │
                        └── 5. 更新发送状态
                            └── UpdateMmsStatus()
```

## PDU 编解码调用链

### GSM PDU 编码 (发送)

```
GsmSmsSender::EncodeTextSms()
    │
    ▼
GsmSmsMessage::CreateDefaultSubmitSmsTpdu()
    │
    ├── 1. 构造 SmsTpdu 结构
    │   └── smsTpdu_ = std::make_shared<SmsTpdu>()
    │
    ├── 2. 设置 TPDU 类型
    │   └── smsTpdu_-u003e.tpduType = TPDU_TYPE_SUBMIT
    │
    ├── 3. 编码短信中心地址 (可选)
    │   └── EncodeSmscAddress()
    │
    ├── 4. 编码目标地址
    │   └── CalcReplyEncodeAddress()
    │       │
    │       ├── 检查地址长度 < MAX_ADDRESS_LEN (21)
    │       ├── 解析地址类型 (国内/国际)
    │       └── BCD 编码
    │
    ├── 5. 编码协议标识
    │   └── smsTpdu_-u003e.data.submit.pid = 0
    │
    ├── 6. 编码数据编码方案 (DCS)
    │   └── EncodeMsgDcs()
    │       ├── DCS_7BIT (默认)
    │       ├── DCS_8BIT
    │       └── DCS_UCS2 (中文)
    │
    └── 7. 编码用户数据
        └── EncodeUserData()
            │
            ├── 如果是分段短信
            │   └── 添加 UDH (User Data Header)
            │       ├── 信息元素标识 (0x00 = 分段)
            │       ├── 消息引用 ID
            │       ├── 总分段数
            │       └── 当前分段序号
            │
            └── 编码文本数据
                └── gsm_user_data_encode.cpp:EncodeGsmUserData()
                    │
                    ├── 7-bit 编码 (GSM 默认字母表)
                    │   ├── 将 ASCII 映射到 GSM 7-bit
                    │   └── 打包成 8-bit 字节
                    │
                    ├── 8-bit 编码
                    │   └── 直接复制字节
                    │
                    └── UCS2 编码 (中文)
                        └── 使用 big-endian UTF-16
```

### GSM PDU 解码 (接收)

```
GsmSmsMessage::PduAnalysis(pdu)
    │
    ├── 1. 检查 PDU 有效性
    │   └── pdu.length <= MAX_TPDU_DATA_LEN (255*2)
    │
    ├── 2. 解析短信中心地址
    │   └── DecodeSmscAddress()
    │       ├── 读取地址长度
    │       ├── 读取地址类型
    │       └── BCD 解码
    │
    ├── 3. 解析 TPDU
    │   └── gsm_sms_tpdu_decode.cpp:DecodeSmsTpdu()
    │       │
    │       ├── 读取第一个字节
    │       │   └── 判断 TPDU 类型 (Deliver/Submit/Status Report)
    │       │
    │       ├── 解析 Deliver TPDU
    │       │   ├── 消息类型指示器 (MTI)
    │       │   ├── 更多消息发送 (MMS)
    │       │   ├── 回复路径 (RP)
    │       │   ├── 用户数据头指示 (UDHI)
    │       │   ├── 状态报告指示 (SRI)
    │       │   ├── 发件人地址
    │       │   ├── 协议标识符 (PID)
    │       │   ├── 数据编码方案 (DCS)
    │       │   ├── 服务中心时间戳 (SCTS)
    │       │   ├── 用户数据长度 (UDL)
    │       │   └── 用户数据 (UD)
    │       │
    │       └── 解析 Status Report TPDU
    │
    ├── 4. 解析用户数据
    │   └── gsm_user_data_decode.cpp:DecodeGsmUserData()
    │       │
    │       ├── 检查 UDHI (用户数据头指示)
    │       │   └── 如果有 UDH
    │       │       └── 解析信息元素
    │       │           ├── 0x00 = 分段短信
    │       │           ├── 0x01 = 特殊短信指示
    │       │           └── ...
    │       │
    │       └── 根据 DCS 解码内容
    │           ├── DCS_7BIT ──► 7-bit 解码
    │           ├── DCS_8BIT ──► 8-bit 解码
    │           └── DCS_UCS2 ──► UCS2 解码
    │
    └── 5. 提取消息内容
        └── GetMessageContent()
```

## IPC 通信调用链

### 客户端到服务端调用

```
客户端应用 (JS)
    │
    ▼
N-API 层 (napi_sms.cpp)
    │
    ├── 解析 JS 参数
    └── 创建异步工作
    │
    ▼
Native 客户端 (SmsServiceManagerClient)
    │
    ├── 获取服务代理
    │   └── GetSmsServiceProxy(slotId)
    │       │
    │       └── SystemAbilityManager::GetSystemAbility(4008)
    │           │
    │           └── 向 SAMGR 查询 SA 4008
    │               └── 返回 IRemoteObject
    │
    └── 调用 IPC 方法
        └── SmsServiceProxy::SendMessage(...)
            │
            ├── MessageParcel data
            ├── data.WriteInterfaceToken()
            ├── data.WriteInt32(slotId)
            ├── data.WriteString16(desAddr)
            └── ...
            │
            └── remote->SendRequest(code, data, reply, option)
                │
                ▼
                Binder 驱动
                    │
                    ▼
                    服务端进程 (telephony)
                        │
                        ▼
                        SmsInterfaceStub::OnRemoteRequest()
                            │
                            ├── 验证 InterfaceToken
                            ├── 查找处理函数
                            │   └── memberFuncMap_[code]
                            └── 调用处理函数
                                │
                                ▼
                                SmsService::SendMessage()
                                    │
                                    └── 执行业务逻辑
```

### 服务端到客户端回调

```
服务端 (SmsService)
    │
    ├── 发送完成
    └── 调用回调
        │
        ▼
        ISendShortMessageCallback::OnSmsSendResult()
            │
            └── 这是客户端传入的 IPC 对象
                │
                └── IPC 调用到客户端进程
                    │
                    ▼
                    客户端回调实现
                        │
                        ▼
                        SendCallbackStub::OnSmsSendResult()
                            │
                            └── 调用 JS 回调函数
                                │
                                ▼
                                应用层收到结果
```

## 网络选择调用链

```
SmsSendManager::TextBasedSmsDelivery()
    │
    ├── 获取网络策略
    │   └── SmsNetworkPolicyManager::GetNetWorkType()
    │       │
    │       ├── 1. 获取网络状态
    │       │   └── CoreManagerInner::GetNetworkStatus()
    │       │
    │       ├── 2. 检查是否为电信卡
    │       │   └── CoreManagerInner::IsCTSimCard()
    │       │
    │       ├── 3. 检查漫游状态
    │       │   └── NetworkState::IsRoaming()
    │       │
    │       ├── 4. 检查 IMS 注册状态
    │       │   └── CoreManagerInner::GetImsRegStatus()
    │       │       └── ImsRegState::IMS_REGISTERED
    │       │
    │       └── 5. 决策网络类型
    │           ├── 如果: 电信卡 && !漫游 ──► NET_TYPE_CDMA
    │           ├── 如果: IMS 已注册 ──► NET_TYPE_IMS
    │           └── 否则: ──► NET_TYPE_GSM
    │
    └── 根据网络类型分发
        │
        ├── NET_TYPE_GSM ──► GsmSmsSender::TextBasedSmsDelivery()
        ├── NET_TYPE_CDMA ──► CdmaSmsSender::TextBasedSmsDelivery()
        └── NET_TYPE_IMS ──► GsmSmsSender::SendImsSms()
```

## 相关跳转链接

- [架构说明](../02_Architecture.md) - 了解整体架构
- [内部 API](../04_Internal_API.md) - 了解 IPC 接口定义
- [N-API 接口](../03_NAPI_Interface.md) - 了解 JS API 调用
