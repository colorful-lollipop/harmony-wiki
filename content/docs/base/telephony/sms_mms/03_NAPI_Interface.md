# N-API 接口文档

## 目的

本文档详细说明短彩信模块的对外 N-API（JavaScript API）接口，包括 API 清单、参数/返回值、C++ 实现映射、权限要求和错误码。

## 适用范围

本文档覆盖：
- 完整的 JS API 清单表
- 每个 API 的参数/返回值
- 同步/异步模式说明
- C++ 实现函数映射
- 参数校验和错误码
- 权限要求

不包含：
- 内部 API 定义（见内部 API 文档）
- Taihe/Cangjie API 绑定（仅提及存在）

## 模块注册

### 模块信息

- **模块名**: `telephony.sms`
- **注册文件**: `frameworks/js/napi/src/napi_sms.cpp`
- **导出函数**: `InitNapiSmsRegistry`
- **注册宏**: `napi_module_register`

### 注册代码

**位置**: `frameworks/js/napi/src/napi_sms.cpp:2024-2037`

```cpp
static napi_module g_smsModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = InitNapiSmsRegistry,
    .nm_modname = "telephony.sms",
    .nm_priv = ((void *)0),
    .reserved = {(void *)0},
};

extern "C" __attribute__((constructor)) void RegisterTelephonySmsModule(void)
{
    napi_module_register(&g_smsModule);
}
```

## SMS API 清单

### 短信发送相关

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `sendMessage(options)` | 异步（Callback） | `SendMessage` | napi_sms.cpp:262 | `ohos.permission.SEND_MESSAGES` |
| `sendShortMessage(text)` | 同步 | `SendShortMessage` | napi_sms.cpp:292 | `ohos.permission.SEND_MESSAGES` |
| `setSmscAddr(slotId, smscAddr)` | 异步（Promise/Callback） | `SetSmscAddr` | napi_sms.cpp:692 | `ohos.permission.SET_TELEPHONY_STATE` |
| `getSmscAddr(slotId)` | 异步（Promise/Callback） | `GetSmscAddr` | napi_sms.cpp:771 | `ohos.permission.GET_TELEPHONY_STATE` |

### 短信创建相关

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `createMessage(pdu, specification)` | 异步（Promise/Callback） | `CreateMessage` | napi_sms.cpp:398 | 无 |
| `splitMessage(text, specification)` | 同步 | `SplitMessage` | napi_sms.cpp:1173 | 无 |
| `getSmsSegmentsInfo(text, specification)` | 同步 | `GetSmsSegmentsInfo` | napi_sms.cpp:1079 | 无 |
| `hasSmsCapability()` | 同步 | `HasSmsCapability` | napi_sms.cpp:1201 | 无 |

### SIM 卡管理相关

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `addSimMessage(options)` | 异步（Callback） | `AddSimMessage` | napi_sms.cpp:865 | `ohos.permission.RECEIVE_MESSAGES` |
| `delSimMessage(index)` | 异步（Callback） | `DelSimMessage` | napi_sms.cpp:953 | `ohos.permission.RECEIVE_MESSAGES` |
| `updateSimMessage(options)` | 异步（Callback） | `UpdateSimMessage` | napi_sms.cpp:1054 | `ohos.permission.RECEIVE_MESSAGES` |
| `getAllSimMessages(slotId)` | 异步（Callback） | `GetAllSimMessages` | napi_sms.cpp:1173 | `ohos.permission.RECEIVE_MESSAGES` |

### 配置相关

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `setDefaultSmsSlotId(slotId)` | 异步（Promise/Callback） | `SetDefaultSmsSlotId` | napi_sms.cpp:486 | 无 |
| `getDefaultSmsSlotId()` | 异步（Promise/Callback） | `GetDefaultSmsSlotId` | napi_sms.cpp:559 | 无 |
| `getDefaultSmsSimId()` | 异步（Promise/Callback） | `GetDefaultSmsSimId` | napi_sms.cpp:616 | 无 |
| `getSmsShortCodeType(address)` | 同步 | `GetSmsShortCodeType` | napi_sms.cpp:1250 | 无 |

### 小区广播相关

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `setCBConfig(options)` | 同步 | `SetCBConfig` | napi_sms.cpp:1233 | `ohos.permission.RECEIVE_MESSAGES` |
| `setCBConfigList(slotId, configs)` | 同步 | `SetCBConfigList` | napi_sms.cpp:1336 | `ohos.permission.RECEIVE_MESSAGES` |

### IMS 短信相关

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `isImsSmsSupported()` | 异步（Promise/Callback） | `IsImsSmsSupported` | napi_sms.cpp:1385 | 系统应用 |
| `getImsShortMessageFormat()` | 异步（Promise/Callback） | `GetImsShortMessageFormat` | napi_sms.cpp:1445 | 系统应用 |

### MMS API 清单

### MMS 编解码（系统应用专用）

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `decodeMms(data)` | 异步（Promise/Callback） | `NapiMms::DecodeMms` | napi_mms.cpp:681 | 系统应用 |
| `encodeMms(config)` | 异步（Promise/Callback） | `NapiMms::EncodeMms` | napi_mms.cpp:1433 | 系统应用 |

### MMS 发送/接收

| API 名称 | 同步/异步 | C++ 实现 | 文件:行号 | 权限要求 |
|---------|-----------|-----------|-----------|---------|
| `sendMms(context, params)` | 异步（Callback） | `NapiSendRecvMms::SendMms` | napi_send_recv_mms.cpp:291 | 系统应用 |
| `downloadMms(context, params)` | 异步（Callback） | `NapiSendRecvMms::DownloadMms` | napi_send_recv_mms.cpp:514 | 系统应用 |

## 详细 API 说明

### sendMessage

**用途**: 发送短信（支持文本短信、数据短信、长短信分段）

**参数定义**:

```javascript
{
    slotId: number,              // 卡槽 ID（必填）
    destinationHost: string,      // 接收方电话号码（必填）
    serviceCenter?: string,       // 短信中心地址（可选）
    content: string | number[],  // 短信内容（必填）
    destinationPort?: number,      // 接收方端口号（数据短信必填）
    sendCallback: SendResultCallback,  // 发送结果回调（必填）
    deliveryCallback?: DeliveryCallback   // 送达报告回调（可选）
}

// SendResultCallback
(err: BusinessError, data: {
    result: SendSmsResult,  // 0:成功, 1:未知失败, 2:Modem关闭, 3:网络不可用
    url: string,              // 短信中心 URL
    isLastPart: boolean         // 是否是最后一段
}) => void

// DeliveryCallback
(err: BusinessError, pdu: number[]) => void

enum SendSmsResult {
    SEND_SMS_SUCCESS = 0,
    SEND_SMS_FAILURE_UNKNOWN = 1,
    SEND_SMS_FAILURE_RADIO_OFF = 2,
    SEND_SMS_FAILURE_SERVICE_UNAVAILABLE = 3
}
```

**参数校验**:
- `slotId`: 范围检查 [0, 2)
- `destinationHost`: 正则表达式验证（允许 `+` 号头，3-20 位数字）
- `content`: 类型检查（string 或 Uint8Array）
- `destinationPort`: 数据短信必填，范围检查
- `sendCallback`: 必须是函数

**实现位置**: `frameworks/js/napi/src/napi_sms.cpp:262`

**调用链**:
```
sendMessage() [N-API]
  ↓
SmsServiceProxy::SendMessage() [IPC]
  ↓
SmsInterfaceStub::OnSendSmsTextRequest()
  ↓
SmsInterfaceManager::TextBasedSmsDelivery()
  ↓
SmsSendManager::TextBasedSmsDelivery()
  ↓
GsmSmsSender::TextBasedSmsDelivery() [或 CDMA/IMS]
  ↓
EncodeTextSms() [services/sms/gsm/gsm_sms_message.cpp]
  ↓
RIL Adapter: SendGsmSms()
```

### createMessage

**用途**: 根据 PDU 和协议规范创建 ShortMessage 对象

**参数定义**:

```javascript
createMessage(
    pdu: number[],              // PDU 字节数组（必填）
    specification: string       // 协议类型："3gpp" 或 "3gpp2"（必填）
): Promise<ShortMessage>

// ShortMessage 对象
{
    emailAddress?: string,
    emailMessageBody?: string,
    hasReplyPath: boolean,
    isEmailMessage: boolean,
    isReplaceMessage: boolean,
    isSmsStatusReportMessage: boolean,
    messageClass: ShortMessageClass,
    pdu: number[],
    protocolId: number,
    scAddress: string,
    scTimestamp: number,
    status: number,
    userRawData: number[],
    visibleMessageBody: string,
    visibleRawAddress: string
}
```

**参数校验**:
- `pdu`: 必须是数组，长度验证（1-255 字节）
- `specification`: 必须是 "3gpp" 或 "3gpp2"

**实现位置**: `frameworks/js/napi/src/napi_sms.cpp:398`

### setSmscAddr / getSmscAddr

**用途**: 设置/获取短信服务中心（SMSC）地址

**参数定义**:

```javascript
setSmscAddr(
    slotId: number,        // 卡槽 ID（必填）
    smscAddr: string       // SMSC 地址（必填）
): Promise<void>

getSmscAddr(
    slotId: number        // 卡槽 ID（必填）
): Promise<string>
```

**参数校验**:
- `slotId`: 范围检查 [0, 2)
- `smscAddr`: 长度验证（MAX_ADDRESS_LEN = 21）

**实现位置**: `frameworks/js/napi/src/napi_sms.cpp:692` (Set), `:771` (Get)

### addSimMessage / delSimMessage / updateSimMessage

**用途**: SIM 卡短信记录的增删改查

**参数定义**:

```javascript
addSimMessage({
    slotId: number,           // 卡槽 ID（必填）
    smsc: string,           // SMSC 地址（必填）
    status: number,          // 状态码（必填）
    pdu: string             // PDU 字符串（必填）
}, callback: Function): void

delSimMessage(
    slotId: number,           // 卡槽 ID（必填）
    index: number,           // 短信索引（必填）
    callback: Function): void

updateSimMessage({
    slotId: number,           // 卡槽 ID（必填）
    index: number,           // 短信索引（必填）
    newStatus: number,        // 新状态（必填）
    pdu: string,            // 新 PDU（必填）
}, callback: Function): void

getAllSimMessages(
    slotId: number,           // 卡槽 ID（必填）
    callback: Function): void
```

**参数校验**:
- `slotId`: 范围检查 [0, 2)
- `index`: 范围验证
- `status`: 范围验证 [SIM_MESSAGE_STATUS_UNREAD, SIM_MESSAGE_STATUS_SENT]

**实现位置**:
- Add: `frameworks/js/napi/src/napi_sms.cpp:865`
- Del: `frameworks/js/napi/src/napi_sms.cpp:953`
- Update: `frameworks/js/napi/src/napi_sms.cpp:1054`
- GetAll: `frameworks/js/napi/src/napi_sms.cpp:1173`

### encodeMms / decodeMms

**用途**: 彩信 PDU 编解码（系统应用专用 API）

**参数定义**:

```javascript
encodeMms({
    messageType: number,       // 消息类型：m-send-req, m-send-conf 等
    mmsType: MmsType,       // 彩信类型定义
    attachment: Attachment[]   // 附件列表
}): Promise<number[]>        // 返回编码后的字节数组

decodeMms(
    data: ArrayBuffer        // 彩信数据（必填）
): Promise<MmsMsg>        // 返回解析后的 MMS 消息

// Attachment 定义
{
    path: string,              // 文件路径
    fileName: string,          // 文件名
    contentId: string,         // Content-ID
    contentLocation: string,    // Content-Location
    contentDisposition: number,  // disposition
    contentTransferEncoding: string,
    contentType: string,        // Content-Type
    isSmil: boolean,          // 是否是 SMIL 文件
    inBuff: number[],
    charset: number
}
```

**权限检查**:
- **系统应用限制**: `TelephonyPermission::CheckCallerIsSystemApp()`
- 非系统应用调用返回 `TELEPHONY_ERR_ILLEGAL_USE_OF_SYSTEM_API`

**实现位置**:
- Encode: `frameworks/js/napi/src/napi_mms.cpp:1433`
- Decode: `frameworks/js/napi/src/napi_mms.cpp:681`

### sendMms / downloadMms

**用途**: 发送/下载彩信（系统应用专用 API）

**参数定义**:

```javascript
sendMms(
    context: Context,           // 应用上下文（必填）
    {
        slotId: number,           // 卡槽 ID（必填）
        mmsc: string,            // 彩信中心 URL（必填）
        data: string,            // 彩信数据文件路径（必填）
        mmsConfig?: {            // 彩信配置（可选）
            userAgent: string,
            userAgentProfile: string
        }
    },
    callback: Function): void

downloadMms(
    context: Context,           // 应用上下文（必填）
    {
        slotId: number,           // 卡槽 ID（必填）
        mmsc: string,            // 彩信中心 URL（必填）
        data: string,            // 下载文件路径（必填）
        configId: string,         // 配置 ID（可选）
    },
    callback: Function): void
```

**实现位置**:
- Send: `frameworks/js/napi/src/napi_send_recv_mms.cpp:291`
- Download: `frameworks/js/napi/src/napi_send_recv_mms.cpp:514`

## 参数校验模式

### 基础类型匹配

**位置**: `frameworks/js/napi/src/napi_sms_util.cpp`

```cpp
// MatchParameters() - 验证参数类型
static bool MatchParameters(napi_env env, const napi_value parameters[],
    const std::vector<napi_valuetype> &expectedTypes)
{
    for (size_t i = 0; i < expectedTypes.size(); ++i) {
        napi_valuetype valueType;
        napi_typeof(env, parameters[i], &valueType);
        if (valueType != expectedTypes[i]) {
            return false;
        }
    }
    return true;
}
```

### 对象属性验证

**位置**: `frameworks/js/napi/src/napi_sms_util.cpp`

```cpp
// MatchObjectProperty() - 验证对象属性
static bool MatchObjectProperty(napi_env env, napi_value object,
    const std::vector<PropertyType> &properties)
{
    for (const auto &prop : properties) {
        napi_value value;
        napi_get_named_property(env, object, prop.name.c_str(), &value);

        napi_valuetype actualType;
        napi_typeof(env, value, &actualType);
        if (actualType != prop.type) {
            return false;
        }
    }
    return true;
}

// PropertyType 定义
struct PropertyType {
    std::string name;
    napi_valuetype type;
};

// 使用示例
MatchObjectProperty(env, object, {
    {"slotId", napi_number},
    {"destinationHost", napi_string},
    {"content", napi_string}  // 或 napi_object（数组）
});
```

### 消息类型检测

**位置**: `frameworks/js/napi/src/napi_sms_util.cpp`

```cpp
// MatchSendMessageParameters() - 检测消息类型
int32_t NapiSmsUtil::MatchSendMessageParameters(
    napi_env env, const napi_value parameters[], size_t parameterCount)
{
    // 检测 content 类型
    napi_value contentValue = NapiUtil::GetNamedProperty(env, object, CONTENT_STR);

    bool contentIsStr = NapiUtil::MatchValueType(env, contentValue, napi_string);
    bool contentIsArray = false;
    if (contentIsObj) {
        napi_is_array(env, contentValue, &contentIsArray);
    }

    // 返回匹配结果
    if (contentIsStr) {
        return TEXT_MESSAGE_PARAMETER_MATCH;
    } else if (contentIsArray) {
        return RAW_DATA_MESSAGE_PARAMETER_MATCH;
    }
    return MESSAGE_PARAMETER_NOT_MATCH;
}
```

### 卡槽 ID 验证

**位置**: `frameworks/js/napi/src/napi_sms_util.cpp`

```cpp
// IsValidSlotId() - 验证卡槽 ID 范围
static inline bool IsValidSlotId(int32_t slotId)
{
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT));
}

// 常量定义
constexpr int32_t DEFAULT_SIM_SLOT_ID = 0;
constexpr int32_t SIM_SLOT_COUNT = 2;
```

## 异步工作模式

### 标准 N-API 异步模式

所有异步 API 遵循相同模式：

```cpp
static napi_value ApiFunction(napi_env env, napi_callback_info info)
{
    // 1. 解析参数
    size_t parameterCount = 1;
    napi_value parameters[1] = {0};
    napi_value thisVar = nullptr;
    void *data = nullptr;
    napi_get_cb_info(env, info, &parameterCount, parameters, &thisVar, &data);

    // 2. 验证参数
    if (!MatchParameters(...)) {
        NapiUtil::ThrowParameterError(env);
        return nullptr;
    }

    // 3. 创建上下文
    auto context = std::make_unique<ContextType>().release();

    // 4. 提取参数值
    napi_get_value_int32(env, parameters[0], &context->slotId);

    // 5. 处理可选回调
    if (parameterCount == EXPECTED_COUNT) {
        napi_create_reference(env, parameters[INDEX],
            DEFAULT_REF_COUNT, &context->callbackRef);
    }

    // 6. 创建并排队异步工作
    napi_value result = NapiUtil::HandleAsyncWork(env, context,
        "ApiFunctionName",
        NativeFunction,      // 原生函数（在线程池执行）
        CallbackFunction    // 回调函数（在主线程执行）
    );
    return result;
}
```

### 原生函数示例

```cpp
static void NativeFunction(napi_env env, void *data)
{
    auto *context = static_cast<ContextType *>(data);

    // 执行实际的业务逻辑
    int32_t result = SmsServiceProxy::SendMessage(...);
    context->errorCode = result;
}
```

### 回调函数示例

```cpp
static void CallbackFunction(napi_env env, napi_status status, void *data)
{
    auto *context = static_cast<ContextType *>(data);

    if (status != napi_ok) {
        // 处理错误
        napi_throw_error(env, context->errorCode);
        return;
    }

    // 创建返回对象
    napi_value result = CreateJsResult(env, context);

    // 调用 JavaScript 回调
    napi_value callback;
    napi_get_reference_value(env, context->callbackRef, &callback);
    napi_call_function(env, callback, 1, &result, nullptr);

    // 清理
    napi_delete_reference(env, context->callbackRef);
    delete context;
}
```

## 错误码处理

### 错误码映射

**位置**: `interfaces/innerkits/sms_mms_errors.h`

| 错误码 | 值 | 说明 |
|-------|----|------|
| `TELEPHONY_ERR_SUCCESS` | 0 | 成功 |
| `TELEPHONY_ERR_PERMISSION_ERR` | 201 | 权限错误 |
| `TELEPHONY_ERR_SLOTID_INVALID` | 401 | 卡槽 ID 无效 |
| `TELEPHONY_ERR_ILLEGAL_USE_OF_SYSTEM_API` | 202 | 非法使用系统 API |
| `TELEPHONY_ERR_RPC` | 204 | RPC 错误 |
| `TELEPHONY_ERR_READ_ERR` | 203 | 读取错误 |
| `TELEPHONY_ERR_WRITE_ERR` | 204 | 写入错误 |
| `TELEPHONY_ERR_SLOTID_NOT_AVAILABLE` | 403 | 卡槽不可用 |
| `TELEPHONY_ERR_LOCAL_PTR` | 205 | 本地指针错误 |
| `TELEPHONY_ERR_MEMORY_FAILURE` | 402 | 内存失败 |
| `TELEPHONY_ERR_DESCRIPTOR_MISMATCH` | 206 | 描述符不匹配 |
| `TELEPHONY_ERR_NETWORK_NOT_AVAILABLE` | 404 | 网络不可用 |

### 错误抛出

**位置**: `frameworks/js/napi/src/napi_sms_util.cpp`

```cpp
static void ThrowParameterError(napi_env env)
{
    napi_throw_error(env, TELEPHONY_ERR_INVALID_PARAMETER,
        "Invalid parameter.");
}

static void ThrowPermissionError(napi_env env, int32_t errorCode)
{
    napi_throw_error(env, errorCode,
        "Permission denied.");
}
```

## 权限要求总结

| API | 所需权限 | 说明 |
|-----|---------|------|
| `sendMessage` | `ohos.permission.SEND_MESSAGES` | 发送短信 |
| `sendShortMessage` | `ohos.permission.SEND_MESSAGES` | 发送短信 |
| `setSmscAddr` | `ohos.permission.SET_TELEPHONY_STATE` | 设置 SMSC |
| `getSmscAddr` | `ohos.permission.GET_TELEPHONY_STATE` | 获取 SMSC |
| `addSimMessage` | `ohos.permission.RECEIVE_MESSAGES` | 添加 SIM 短信 |
| `delSimMessage` | `ohos.permission.RECEIVE_MESSAGES` | 删除 SIM 短信 |
| `updateSimMessage` | `ohos.permission.RECEIVE_MESSAGES` | 更新 SIM 短信 |
| `getAllSimMessages` | `ohos.permission.RECEIVE_MESSAGES` | 获取 SIM 短信 |
| `setCBConfig` | `ohos.permission.RECEIVE_MESSAGES` | 设置 CB 配置 |
| `setCBConfigList` | `ohos.permission.RECEIVE_MESSAGES` | 设置 CB 配置列表 |
| MMS 相关 API | 系统应用 | MMS 编解码 |

## 相关跳转链接

- [项目概览](00_Overview.md) - 了解项目定位
- [架构说明](02_Architecture.md) - 了解 API 调用链
- [内部 API](04_Internal_API.md) - 了解 IPC 接口定义
