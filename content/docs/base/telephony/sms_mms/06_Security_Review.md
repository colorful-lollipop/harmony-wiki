# 安全风险评审

## 目的

本文档提供短彩信模块的安全风险分析，包括攻击面、信任边界、可被利用点和修复建议，帮助安全审计人员识别和修复安全漏洞。

## 适用范围

本文档覆盖：
- 攻击面清单（N-API/IPC/文件/网络/系统能力）
- 信任边界和数据流
- 至少 5 条可被利用点（如存在）
- 输入验证、权限检查、内存安全机制

不包含：
- 模糊测试结果分析（见 test/fuzztest/）
- 具体利用代码（仅分析风险）
- 管理安全配置（如 SELinux 策略）

## 攻击面清单

### 1. N-API 攻击面

| 攻击向量 | 风险等级 | 缓解措施 |
|---------|---------|---------|
| **参数注入** | 高 | 类型检查、正则验证、长度限制 |
| **权限绕过** | 中 | AccessToken 检查、系统应用验证 |
| **资源耗尽** | 中 | 参数数量限制、异步队列限制 |
| **回调劫持** | 低 | 强类型回调引用 |

### 2. IPC 攻击面

| 攻击向量 | 风险等级 | 缓解措施 |
|---------|---------|---------|
| **描述符伪造** | 低 | 接口令牌验证 |
| **RPC 溢出** | 中 | Parcel 大小限制、类型验证 |
| **权限提升** | 中 | Caller 身份验证、权限检查 |

### 3. 文件攻击面

| 攻击向量 | 风险等级 | 缓解措施 |
|---------|---------|---------|
| **路径遍历** | 中 | 路径验证、沙箱隔离 |
| **文件信息泄露** | 低 | 权限检查、文件路径验证 |
| **资源耗尽** | 中 | 文件大小限制、句柄数限制 |

### 4. 网络攻击面（MMS）

| 攻击向量 | 风险等级 | 缓解措施 |
|---------|---------|---------|
| **URL 注入** | 高 | URL 白名单、正则验证 |
| **MITM** | 中 | HTTPS 传输（如支持）|
| **数据泄露** | 中 | 数据加密、访问控制 |
| **SSRF** | 中 | URL 格式验证、域名限制 |

### 5. 系统能力攻击面

| 攻击向量 | 风险等级 | 缓解措施 |
|---------|---------|---------|
| **SA 伪造** | 低 | SAMGR 签名验证 |
| **CommonEvent 劫持** | 低 | 权限检查、事件订阅验证 |
| **数据库注入** | 中 | 参数化查询、权限检查 |

## 信任边界和数据流

### 信任层次

```
┌─────────────────────────────────────────────────────────┐
│          应用层（不可信）                       │
│  - 第三方 App                                       │
│  - 非系统 App（MMS API）                   │
└─────────────────────────────────────────────────────────┘
                    │ 需权限验证
                    ▼
┌─────────────────────────────────────────────────────────┐
│          N-API 绑定层                        │
│  - 参数类型检查                                      │
│  - 权限验证                                        │
│  - 系统应用检查（MMS API）                   │
└─────────────────────────────────────────────────────────┘
                    │ IPC 通信（Binder）
                    ▼
┌─────────────────────────────────────────────────────────┐
│        短彩信服务（可信）                        │
│  - SmsService (SA 4008)                          │
│  - 内部组件无权限检查                              │
└─────────────────────────────────────────────────────────┘
                    │
      ┌─────────────┴─────────────┐
      │                             │
      ▼                             ▼
┌───────────────┐          ┌───────────────┐
│  RIL Adapter  │          │  数据库       │
│  (可信）     │          │  (可信）     │
└───────────────┘          └───────────────┘
      │                             │
      ▼                             ▼
┌───────────────┐          ┌───────────────┐
│   Modem      │          │  Modem       │
│  (可信）     │          │  (可信）     │
└───────────────┘          └───────────────┘
```

### 数据流向

**发送短信**:
```
App（不可信）→ N-API（类型检查）→ SmsService（权限检查）→ GsmSmsSender（编码）→ RIL（可信）→ Modem（可信）→ 网络
```

**接收短信**:
```
Modem（可信）→ RIL（可信）→ GsmSmsReceiveHandler（解码）→ SmsReceiveManager（处理）→ CommonEvent（广播）→ App（有权限过滤）
```

**MMS 发送**:
```
App（系统应用）→ N-API（系统应用检查）→ SmsService → MmsSendManager → MmsNetworkClient → HTTP 客户端 → MMSC
```

## 可被利用点

### 利用点 1: 目的地址验证不完整

**证据位置**: `services/sms/sms_service.cpp:782-790`

**问题描述**:
`ValidDestinationAddress()` 仅验证地址格式和长度，未验证特殊字符注入。

```cpp
bool SmsService::ValidDestinationAddress(std::string desAddr)
{
    std::regex regexMode("^([0-9_+]{1})([0-9]{2,19})$");
    if (desAddr.empty()) {
        return false;
    }
    return std::regex_match(desAddr, regexMode);
}
```

**风险分析**:
- **触发**: 恶意应用构造特殊格式地址（如 `%00` 控制序列）
- **影响**: 可能导致 Modem 解析错误或注入恶意指令
- **严重程度**: 中（需要特定 Modem 漏洞配合）

**修复建议**:
1. 添加黑名单检查，过滤已知危险字符序列
2. 使用严格的电话号码解析库（libphonenumber）验证
3. 添加长度和字符集限制

---

### 利用点 2: IPC 参数验证缺失错误响应

**证据位置**: `services/sms/sms_interface_stub.cpp:259-263`

**问题描述**:
某些 IPC 处理器在验证失败后直接返回，未向 reply Parcel 写入错误码。

```cpp
int16_t dataLen = data.ReadInt16();
if (dataLen < 1) {
    TELEPHONY_LOGE("dataLen is invalid");
    return;  // ❌ 未写入 reply
}
const uint8_t *buffer = reinterpret_cast<const uint8_t *>(data.ReadRawData(dataLen));
if (buffer == nullptr) {
    return;  // ❌ 未写入 reply
}
```

**风险分析**:
- **触发**: 恶意客户端发送畸形 IPC 请求
- **影响**: 可能导致服务端资源泄露（未释放的缓冲区）或信息泄露（错误状态未反馈）
- **严重程度**: 中（信息泄露）

**修复建议**:
1. 所有验证失败路径都写入错误码到 reply
2. 添加审计日志记录验证失败
3. 统一错误处理模式

---

### 利用点 3: 输入缓冲区整数溢出

**证据位置**: `services/sms/gsm/gsm_sms_message.cpp:74-77`

**问题描述**:
`memcpy_s` 调用时，目标缓冲区大小可能基于攻击者控制的输入计算。

```cpp
ret = memcpy_s(tPdu->data.submit.destAddress.address,
    sizeof(tPdu->data.submit.destAddress.address),
    desAddr.c_str(), MAX_ADDRESS_LEN);  // ⚠️ desAddr.size() 可能接近 MAX_ADDRESS_LEN
```

**风险分析**:
- **触发**: 提供超长地址字符串（接近 MAX_ADDRESS_LEN = 21）
- **影响**: 如果 `desAddr.size()` 等于 MAX_ADDRESS_LEN，可能写入 1 字节越界
- **严重程度**: 中（需要精确边界值）

**修复建议**:
1. 使用 `memcpy_s(dst, dstSize, src, std::min(src.size(), dstSize - 1))`
2. 添加明确的空字节终止符处理
3. 使用安全的字符串复制函数

---

### 利用点 4: MMS URL 验证不充分

**证据位置**: `services/mms/mms_network_client.cpp:405-420`

**问题描述**:
MMS 发送时仅验证文件大小，未验证 MMSC URL 的合法性。

```cpp
if (fileLen <= 0 || fileLen > static_cast<long>(MMS_PDU_MAX_SIZE)) {
    TELEPHONY_LOGE("fileLen exceeds limit");
    return nullptr;
}

// ❌ 未验证 mmsc 参数
std::string requestUrl = mmsc;
```

**风险分析**:
- **触发**: 恶意系统应用提供伪造的 MMSC URL（如 `file:///` 协议）
- **触发**: SSRF（服务端请求伪造），访问内网资源
- **影响**: 可能泄露内部系统信息或执行未授权操作
- **严重程度**: 高（SSRF 攻击）

**修复建议**:
1. 添加 URL 白名单验证，仅允许 `http://` 和 `https://`
2. 添加域名/地址格式正则验证
3. 添加 MMSC URL 配置签名验证
4. 实施网络隔离和防火墙规则

---

### 利用点 5: 权限检查竞态条件

**证据位置**: `frameworks/js/napi/src/napi_mms.cpp:NativeDecodeMms()`

**问题描述**:
权限检查使用 `TelephonyPermission::CheckCallerIsSystemApp()`，但检查发生在异步工作线程，可能与调用者上下文不同步。

```cpp
void NativeDecodeMms(napi_env env, void *data)
{
    if (data == nullptr) {
        TELEPHONY_LOGE("napi_mms data nullptr");
        return;
    }
    auto context = static_cast<DecodeMmsContext *>(data);

    // ⚠️ 在工作线程检查权限，可能存在竞态
    if (!TelephonyPermission::CheckCallerIsSystemApp()) {
        TELEPHONY_LOGE("Non-system applications use system APIs!");
        context->errorCode = TELEPHONY_ERR_ILLEGAL_USE_OF_SYSTEM_API;
        return;
    }

    // ... 继续处理
}
```

**风险分析**:
- **触发**: 恶意应用在权限检查后快速切换调用者身份
- **影响**: 可能绕过系统应用检查，获取敏感 API 访问权限
- **严重程度**: 中（权限提升）

**修复建议**:
1. 在主线程（调用 N-API 时）检查权限
2. 将权限检查结果传递到工作线程上下文
3. 使用访问令牌而不是应用身份
4. 添加审计日志记录权限检查结果

---

### 利用点 6: 文件路径遍历（MMS）

**证据位置**: `services/mms/mms_network_client.cpp:480-500`

**问题描述**:
MMS 发送时使用应用提供的文件路径，未验证路径是否沙箱内。

```cpp
std::string filePath = data;  // 来自应用参数

// ❌ 未验证路径是否在沙箱内
std::ifstream file(filePath, std::ios::binary);
```

**风险分析**:
- **触发**: 恶意应用提供路径如 `../../system/etc/passwd`
- **影响**: 可能读取沙箱外文件，泄露系统信息
- **严重程度**: 高（信息泄露）

**修复建议**:
1. 使用应用沙箱 API 获取文件句柄
2. 验证文件路径在允许的目录内
3. 添加路径规范化检查（解析 `..` 符号）
4. 使用安全的文件打开 API

---

### 利用点 7: PDU 缓冲区未初始化清理

**证据位置**: `services/sms/sms_pdu_buffer.cpp:132-140`

**问题描述**:
PDU 缓冲区在验证失败时可能返回未初始化的数据。

```cpp
bool PduFromHex(const std::string &hex, std::vector<uint8_t> &pdu)
{
    std::string::size_type len = hex.length();
    if (len < PDU_BUFFER_MIN_SIZE || len > PDU_BUFFER_MAX_SIZE + 1) {
        TELEPHONY_LOGE("PduFromHex len error");
        return false;  // ❌ pdu 可能未清理
    }

    // 处理 hex...
    return true;
}
```

**风险分析**:
- **触发**: 提供格式错误的 hex 字符串
- **影响**: 返回的 pdu 向量可能包含随机内存数据，泄露信息
- **严重程度**: 低（信息泄露）

**修复建议**:
1. 在验证失败前清空 pdu 缓冲区
2. 使用 RAII 模式管理缓冲区生命周期
3. 添加内存初始化检查

---

### 利用点 8: 短信码匹配逻辑不完整

**证据位置**: `services/sms/sms_short_code_matcher.cpp`

**问题描述**:
短信码匹配基于简单模式匹配，可能被特殊字符序列绕过。

```cpp
// 示意（需查看具体实现）
bool MatchShortCode(const std::string &number) {
    // ⚠️ 如果使用简单的字符串包含或前缀匹配
    return (number.find(codePrefix) == 0);
}
```

**风险分析**:
- **触发**: 使用特殊字符或 Unicode 变体绕过匹配
- **影响**: 恶意短信号过过滤，到达用户手机
- **严重程度**: 中（欺诈攻击）

**修复建议**:
1. 使用完整的号码规范化（libphonenumber）|
2. 实施严格的短信号名单白名单
3. 添加用户举报机制
4. 记录所有短信码访问日志

## 现有安全机制

### 1. 权限检查

**实现位置**: `services/sms/sms_service.cpp`

**检查机制**:

| API | 权限 | 检查位置 |
|-----|-------|---------|
| `SendMessage` | `ohos.permission.SEND_MESSAGES` | `SmsService::SendMessage():TelephonyPermission::CheckPermission()` |
| `SetSmscAddr` | `ohos.permission.SET_TELEPHONY_STATE` | `SmsInterfaceStub::OnSetSmscAddrRequest()` |
| `MMS APIs` | 系统应用 | `NapiMms::NativeDecodeMms():TelephonyPermission::CheckCallerIsSystemApp()` |

**访问令牌**:
```cpp
// 使用 AccessToken 进行权限验证
if (!TelephonyPermission::CheckPermission(Permission::SEND_MESSAGES)) {
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

### 2. 输入验证

**实现位置**: `frameworks/js/napi/src/napi_sms_util.cpp`

**验证类型**:

| 验证项 | 位置 | 说明 |
|--------|------|------|
| **参数类型** | `MatchParameters()` | napi_typeof 检查 |
| **对象属性** | `MatchObjectProperty()` | 检查必需属性 |
| **卡槽 ID** | `IsValidSlotId()` | 范围 [0, 2) |
| **地址长度** | `MAX_ADDRESS_LEN = 21` | 最大长度限制 |
| **PDU 大小** | `PDU_BUFFER_MAX_SIZE = 255` | PDU 缓冲区限制 |
| **MMS 大小** | `MMS_PDU_MAX_SIZE` (~300KB) | 彩信大小限制 |

**常量定义**:
```cpp
constexpr int32_t MAX_ADDRESS_LEN = 21;
constexpr int32_t MAX_USER_DATA_LEN = 160;
constexpr int32_t MAX_SEGMENT_NUM = 15;
constexpr int32_t MAX_TPDU_LEN = 255;
constexpr int32_t MMS_PDU_MAX_SIZE = 300 * 1024;
```

### 3. IPC 安全

**实现位置**: `services/sms/sms_interface_stub.cpp`

**安全机制**:

```cpp
int SmsInterfaceStub::OnRemoteRequest(uint32_t code, MessageParcel &data,
    MessageParcel &reply, MessageOption &option)
{
    // 描述符验证
    std::u16string myDescripter = SmsInterfaceStub::GetDescriptor();
    std::u16string remoteDescripter = data.ReadInterfaceToken();
    if (myDescripter != remoteDescripter) {
        TELEPHONY_LOGE("descriptor checked fail");
        return TELEPHONY_ERR_DESCRIPTOR_MISMATCH;
    }

    // 函数映射分发
    auto itFunc = memberFuncMap_.find(static_cast<SmsServiceInterfaceCode>(code));
    if (itFunc != memberFuncMap_.end()) {
        auto memberFunc = itFunc->second;
        memberFunc(data, reply, option);
        return TELEPHONY_ERR_SUCCESS;
    }
    return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
}
```

### 4. 内存安全

**实现位置**: 多个 `memcpy_s` 使用处

**安全函数**:

```cpp
// 使用安全变体
ret = memcpy_s(dst, dstSize, src, srcSize);
ret = strncpy_s(dst, dstSize, src, srcSize);
ret = memset_s(dst, 0, dstSize);

// Buffer 边界检查
if (len > PDU_BUFFER_MAX_SIZE) {
    return;
}
```

**编译保护**:
- **CFI** (Control Flow Integrity): 防止控制流劫持
- **PAC-RET** (Pointer Authentication): 防止返回地址伪造
- **Fortify Source**: 编译时缓冲区溢出检测

### 5. 模糊测试覆盖

**位置**: `test/fuzztest/`

**覆盖范围**:

| 目标 | 模糊测试文件 | 说明 |
|------|----------|------|
| **PDU 解析** | `cdmasmsencode_fuzzer`, `gsmsmsparamcodec_fuzzer` | GSM/CDMA PDU 编解码 |
| **消息创建** | `createsmsmessage_fuzzer` | ShortMessage 对象创建 |
| **发送短信** | `sendmessage_fuzzer`, `sendmessagedata_fuzzer` | 发送 API |
| **分段短信** | `splitmessage_fuzzer` | 长短信分段 |
| **CB 处理** | `setgetcbconfig_fuzzer` | 小区广播配置 |
| **SIM 操作** | `addsimmessage_fuzzer`, `delsimmessage_fuzzer` | SIM 卡操作 |
| **状态报告** | `textbasedsmsdelivery_fuzzer`, `databasedsmsdelivery_fuzzer` | 送达报告 |

**目标文件总数**: 22 个模糊测试

## 修复建议优先级

### 高优先级（建议立即修复）

1. **MMS URL 验证**（利用点 4）
   - 添加 URL 白名单和格式验证
   - 阻止 SSRF 攻击
   - 严重程度: 高

2. **文件路径遍历**（利用点 6）
   - 验证文件路径在沙箱内
   - 使用应用沙箱 API
   - 严重程度: 高

3. **整数溢出**（利用点 3）
   - 修复 `memcpy_s` 边界检查
   - 严重程度: 中

### 中优先级（建议近期修复）

4. **IPC 错误响应**（利用点 2）
   - 统一错误处理模式
   - 添加审计日志
   - 严重程度: 中

5. **权限竞态**（利用点 5）
   - 在主线程检查权限
   - 使用访问令牌
   - 严重程度: 中

### 低优先级（可选优化）

6. **目的地址验证**（利用点 1）
   - 添加黑名单检查
   - 使用 libphonenumber 解析
   - 严重程度: 中

7. **PDU 缓冲区清理**（利用点 7）
   - 添加 RAII 模式
   - 严重程度: 低

## 检查范围和局限性

### 已检查范围

✅ **N-API 层**: 所有参数验证和权限检查代码
✅ **IPC 层**: 所有 Stub 实现和参数处理
✅ **服务层**: 核心业务逻辑的输入验证
✅ **MMS 网络**: URL 验证和文件处理
✅ **编译配置**: 安全编译选项和宏定义

### 未检查范围

❌ **测试代码**: 未深入分析 test/ 目录的实现
❌ **第三方依赖**: 未审计 curl、protobuf、libphonenumber 的实现
❌ **内核/Modem 层**: 未分析 RIL Adapter 和 Modem 的安全特性
❌ **数据库层**: 未深入分析 data_share 的实现

### 局限性

1. **静态分析**: 基于代码审查，未运行时测试
2. **部分覆盖**: 未审计所有代码路径（如卫星服务交互）
3. **假设性分析**: 某些风险基于代码模式推测，可能不存在实际利用

## 相关跳转链接

- [N-API 接口](03_NAPI_Interface.md) - 了解权限检查和参数验证
- [内部 API](04_Internal_API.md) - 了解 IPC 接口定义
- [GN 目标](05_GN_Targets.md) - 了解编译安全特性
