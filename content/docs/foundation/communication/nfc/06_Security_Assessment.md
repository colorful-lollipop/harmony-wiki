# NFC 安全风险评估

## 文档信息

| 项目 | 内容 |
|------|------|
| **目的** | 识别 NFC 组件的安全风险并提供修复建议 |
| **适用范围** | 安全审计、代码审查 |
| **相关文档** | [N-API 接口](03_NAPI_Interfaces.md)、[内部接口](04_Inner_API.md) |

---

## 1. 评估范围

### 1.1 检查范围

| 范围 | 说明 |
|------|------|
| **代码范围** | `/foundation/communication/nfc` 目录下生产代码（排除 test/）|
| **检查类型** | 权限校验、输入验证、内存安全、IPC 安全、数据解析 |
| **评估深度** | 源码级静态分析 + 关键路径代码审查 |

### 1.2 关键检查文件

| 文件路径 | 检查重点 |
|----------|----------|
| `services/src/external_deps/nfc_permission_checker.cpp` | 权限校验实现 |
| `services/src/ipc/card_emulation/hce_session.cpp` | HCE IPC 安全 |
| `services/src/card_emulation/host_card_emulation_manager.cpp` | HCE 管理安全 |
| `services/src/tag/ndef_har_data_parser.cpp` | NDEF 解析安全 |
| `interfaces/inner_api/common/ndef_message.cpp` | NDEF 消息解析 |
| `services/src/nci_adapter/nci_native_selector.cpp` | 动态库加载 |

---

## 2. 攻击面分析

### 2.1 攻击面清单

```
                    ┌─────────────────────────────────────┐
                    │           攻击面总览                 │
                    └─────────────────────────────────────┘
                                      │
        ┌─────────────┬───────────────┼───────────────┬─────────────┐
        ▼             ▼               ▼               ▼             ▼
   ┌─────────┐  ┌─────────┐    ┌─────────┐     ┌─────────┐   ┌─────────┐
   │ N-API   │  │   IPC   │    │  NDEF   │     │  HCE    │   │  File   │
   │   层    │  │  接口   │    │  解析   │     │ APDU    │   │   IO    │
   └────┬────┘  └────┬────┘    └────┬────┘     └────┬────┘   └────┬────┘
        │            │              │               │             │
        ▼            ▼              ▼               ▼             ▼
   JS API 调用   跨进程通信      标签数据解析    卡模拟交互    配置文件
```

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| **N-API 接口** | JS API 调用 (controller/tag/cardEmulation) | 中 |
| **IPC 通信** | INfcController/ITagSession/IHceSession | 高 |
| **NDEF 数据解析** | 标签 NDEF 消息解析 | 中 |
| **APDU 处理** | HCE APDU 命令收发 | 高 |
| **动态库加载** | NCI native 库加载 | 中 |
| **文件操作** | 偏好设置文件读写 | 低 |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不信任区域                                │
│                     (第三方应用代码)                             │
├─────────────────────────────────────────────────────────────────┤
│  N-API 层 (frameworks/js/napi/)                                 │
│  - 参数校验、类型检查、错误转换                                  │
├─────────────────────────────────────────────────────────────────┤
│  IPC 接口层 (interfaces/inner_api/)                             │
│  - IDL 定义、Parcelable 序列化                                   │
├─────────────────────────────────────────────────────────────────┤
│  服务层 (services/src/)                                         │
│  - 权限校验、业务逻辑、状态管理                                  │
│  ★ 信任边界 - 在此层进行严格的权限和输入验证                     │
├─────────────────────────────────────────────────────────────────┤
│  NCI 适配层 (services/src/nci_adapter/)                         │
│  - 硬件抽象、动态库调用                                          │
├─────────────────────────────────────────────────────────────────┤
│                        内核区域                                  │
│                   (NFC 控制器驱动)                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 安全检查结果

### 3.1 权限校验 ✓ 良好

**检查位置**: `services/src/external_deps/nfc_permission_checker.cpp:22-34`

```cpp
bool NfcPermissionChecker::IsGranted(std::string permission)
{
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int result = Security::AccessToken::PermissionState::PERMISSION_GRANTED;
    if (Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(callerToken) ==
        Security::AccessToken::ATokenTypeEnum::TOKEN_NATIVE) {
        result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permission);
    } else if (Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(callerToken) ==
        Security::AccessToken::ATokenTypeEnum::TOKEN_HAP) {
        result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permission);
    }
    return result == Security::AccessToken::PermissionState::PERMISSION_GRANTED;
}
```

**评估结果**:
- ✓ 使用 `IPCSkeleton::GetCallingTokenID()` 正确获取调用者身份
- ✓ 区分原生应用 (TOKEN_NATIVE) 和 HAP (TOKEN_HAP)
- ✓ 使用系统的 `AccessTokenKit` 进行权限验证
- ⚠ 建议：考虑增加权限缓存以提升性能

### 3.2 IPC 安全 ✓ 良好

**检查位置**: `services/src/ipc/card_emulation/hce_session.cpp:192-223`

```cpp
ErrCode HceSession::StartHce(const ElementName& element, const std::vector<std::string>& aids)
{
    if (!ExternalDepsProxy::GetInstance().IsGranted(OHOS::NFC::CARD_EMU_PERM)) {
        return KITS::ERR_NO_PERMISSION;
    }
    Security::AccessToken::HapTokenInfo hapTokenInfo;
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int result = Security::AccessToken::AccessTokenKit::GetHapTokenInfo(callerToken, hapTokenInfo);
    if (result) {
        return KITS::ERR_HCE_PARAMETERS;
    }
    if (hapTokenInfo.bundleName.empty()) {
        return KITS::ERR_HCE_PARAMETERS;
    }
    // ★ 关键安全校验：调用者 bundleName 必须与请求的 element 匹配
    if (hapTokenInfo.bundleName != element.GetBundleName()) {
        ErrorLog("StartHce: wrong bundle name");
        return KITS::ERR_HCE_PARAMETERS;
    }
    // ...
}
```

**评估结果**:
- ✓ 严格的 bundle name 验证 - 防止应用 A 冒充应用 B 启动 HCE
- ✓ 使用 DeathRecipient 处理远程进程死亡
- ✓ `IsCorrespondentService()` 防止应用间 APDU 数据混淆

### 3.3 输入验证 ✓ 良好

**边界检查示例** (start_hce_info_parcelable.cpp:59)：
```cpp
if (aids_.size() > MAX_AID_LIST_NUM_PER_APP) {
    ErrorLog("Aids list is too large.");
    return false;
}
```

**NDEF 解析验证** (ndef_message.cpp:240-263)：
```cpp
bool NdefMessage::IsInvalidRecordLayoutHead(RecordLayout& layout, bool isChunkFound,
    uint32_t parsedRecordSize, bool isMbMeIgnored)
{
    // 检查 MB/ME 标志位一致性
    if (!layout.mb && parsedRecordSize == 0 && !isChunkFound && !isMbMeIgnored) {
        ErrorLog("IsInvalidRecordLayoutHead, 1st error for mb and size.");
        return true;  // 第一条记录必须有 MB 标志
    } else if (layout.mb && (parsedRecordSize != 0 || isChunkFound) && !isMbMeIgnored) {
        ErrorLog("IsInvalidRecordLayoutHead, 2nd error for mb and size");
        return true;  // MB 标志只在第一条记录有效
    } else if (layout.cf && layout.me) {
        ErrorLog("IsInvalidRecordLayoutHead, 4th error for cf and me");
        return true;  // 不能同时有 CF 和 ME
    }
    // ...
}
```

**评估结果**:
- ✓ APDU 数据长度限制 (`MAX_APDU_DATA_HEX_STR`)
- ✓ AID 列表数量限制 (`MAX_AID_LIST_NUM_PER_APP`)
- ✓ NDEF 消息长度限制 (`MAX_NDEF_MESSAGE_LEN`)
- ✓ NDEF 记录结构验证 (MB/ME/CF 标志位检查)

### 3.4 内存安全 ✓ 良好

**安全内存操作** (tag_nci_adapter.cpp:104-108)：
```cpp
errno_t err = memset_s(&g_multiTagParams, sizeof(g_multiTagParams), 0, sizeof(g_multiTagParams));
if (err != EOK) {
    ErrorLog("TagNciAdapter::TagNciAdapter:memset_s for g_multiTagParams error: %{public}d", err);
}
```

**安全分配模式** (host_card_emulation_manager.cpp:58)：
```cpp
abilityConnection_ = new (std::nothrow) NfcAbilityConnectionCallback();
// 后续有 nullptr 检查
```

**评估结果**:
- ✓ 使用 `memset_s`, `memcpy_s` 等边界检查函数
- ✓ 使用 `std::nothrow` + nullptr 检查
- ✓ 智能指针管理内存 (`std::shared_ptr`, `sptr`)

---

## 4. 发现的可利用点

### 4.1 信息泄露风险（低危）

**问题描述**:
多处日志可能泄露敏感信息：

```cpp
// hce_session.cpp - bundle name 被记录
ErrorLog("StartHce: wrong bundle name");
InfoLog("get hap token info, result = %{public}d", result);
```

**影响**:
- 可能帮助攻击者进行侦察
- 泄露有效的 bundle name 格式

**修复建议**:
```cpp
// 建议：减少敏感信息日志，或使用脱敏处理
DebugLog("StartHce: bundle name mismatch");  // 仅在 Debug 级别记录
// 不记录具体的 bundle name
```

### 4.2 动态库加载风险（中危）

**问题描述** (nci_native_selector.cpp:94-99)：
```cpp
handle_ = dlopen(libPath_.c_str(), RTLD_LAZY | RTLD_LOCAL);
if (handle_ == nullptr) {
    ErrorLog("load %{public}s fail, %{public}s", libPath_.c_str(), dlerror());
}
```

**风险**:
- 依赖库搜索路径环境变量
- 理论上可能被劫持加载恶意库

**影响**:
- 需要系统级权限才能修改库路径
- 实际利用难度较高

**修复建议**:
```cpp
// 建议：使用绝对路径加载
#ifdef USE_VENDOR_NCI_NATIVE
    libPath_ = "/system/lib/libnci_native_vendor.z.so";
#else
    libPath_ = "/system/lib/libnci_native_default.z.so";
#endif
```

### 4.3 URI 解析不完全验证（低危）

**问题描述** (ndef_har_data_parser.cpp:245-261)：
```cpp
Uri ndefUri(uriAddress_);
std::string scheme = ndefUri.GetScheme();
if (scheme.empty()) {
    schemeType_ = TYPE_RTP_UNKNOWN;
} else if (scheme == TEL_PREFIX) {
    schemeType_ = TYPE_RTP_SCHEME_TEL;
}
// 仅检查 scheme，未完整验证 URI
```

**风险**:
- 恶意构造的 URI 可能绕过检查
- 依赖 `Uri` 类的内部验证

**影响**:
- 可能导致不期望的跳转或操作

**修复建议**:
```cpp
// 建议：增加 URI 白名单或更严格的验证
if (!IsValidUri(uriAddress_)) {
    WarnLog("Invalid URI format: %{public}s", MaskSensitiveInfo(uriAddress_).c_str());
    return false;
}
```

### 4.4 空指针解引用隐患（低危）

**问题描述** (ndef_har_data_parser.cpp:194-199)：
```cpp
auto nciTagProxyPtr = nciTagProxy_.lock();
if (nciTagProxyPtr == nullptr) {
    ErrorLog("nciTagProxy_ is nullptr");
} else if (!nciTagProxyPtr->VendorParseHarPackage(harPackages, uri)) {
    return false;
}
```

**风险**:
- 空指针检查后仍继续执行
- 可能导致后续空指针解引用

**影响**:
- 可能引发崩溃（实际代码路径需进一步分析）

**修复建议**:
```cpp
auto nciTagProxyPtr = nciTagProxy_.lock();
if (nciTagProxyPtr == nullptr) {
    ErrorLog("nciTagProxy_ is nullptr");
    return false;  // 明确返回失败
}
if (!nciTagProxyPtr->VendorParseHarPackage(harPackages, uri)) {
    return false;
}
```

---

## 5. 缺失的安全检查

| 检查项 | 状态 | 建议 |
|--------|------|------|
| APDU 操作速率限制 | **缺失** | 建议添加速率限制防止 DoS |
| NDEF 文本内容净化 | **部分** | 建议增加敏感内容过滤 |
| 库路径签名验证 | **缺失** | 建议验证加载的库签名 |
| SELinux 上下文检查 | **不可见** | 可能在系统层面实现 |

---

## 6. 安全编码实践总结

### 6.1 优秀实践 ✓

1. **一致的权限校验模式**
   - 所有敏感操作都通过 `NfcPermissionChecker`
   - 调用者身份验证通过 `IPCSkeleton::GetCallingTokenID()`

2. **严格的 IPC 调用者验证**
   - Bundle name 匹配验证
   - DeathRecipient 清理机制

3. **完善的输入验证**
   - 长度检查：APDU、AID、NDEF 都有大小限制
   - 结构验证：NDEF 记录标志位验证

4. **内存安全**
   - 使用边界检查函数 (`memset_s`, `memcpy_s`)
   - 安全分配模式 (`std::nothrow`)
   - 智能指针管理

5. **错误处理**
   - 错误码分层：内部错误 → 业务错误
   - 异常安全：RAII 模式

### 6.2 改进建议

| 优先级 | 建议 | 实现位置 |
|--------|------|----------|
| 高 | 添加 HCE APDU 速率限制 | `hce_session.cpp` |
| 中 | 使用绝对路径加载动态库 | `nci_native_selector.cpp` |
| 中 | 审查并减少敏感日志 | 全局 |
| 低 | 增加 NDEF URI 白名单 | `ndef_har_data_parser.cpp` |
| 低 | 空指针检查后明确返回 | `ndef_har_data_parser.cpp` |

---

## 7. 风险矩阵

| 风险 | 概率 | 影响 | 等级 | 状态 |
|------|------|------|------|------|
| 权限绕过 | 低 | 高 | 中 | ✓ 已防护 |
| IPC 冒充 | 低 | 高 | 中 | ✓ 已防护 |
| NDEF 解析崩溃 | 低 | 中 | 低 | ✓ 已防护 |
| APDU DoS | 中 | 中 | 中 | ⚠ 需改进 |
| 库劫持 | 低 | 高 | 中 | ⚠ 需改进 |
| 信息泄露 | 中 | 低 | 低 | ⚠ 建议修复 |

**风险等级说明**:
- 🔴 高危：需要立即修复
- 🟡 中危：建议近期修复
- 🟢 低危：可以后续优化
- ✓ 已防护：已有良好防护措施

---

## 8. 修复建议汇总

### 8.1 高优先级

**1. 添加 APDU 速率限制**
```cpp
// 建议：在 hce_session.cpp 中添加
class ApduRateLimiter {
    static constexpr int MAX_APDU_PER_SECOND = 100;
    bool CheckRateLimit();
};
```

### 8.2 中优先级

**2. 使用绝对路径加载动态库**
```cpp
// nci_native_selector.cpp
const std::string VENDOR_LIB_PATH = "/system/lib/libnci_native_vendor.z.so";
const std::string DEFAULT_LIB_PATH = "/system/lib/libnci_native_default.z.so";
```

**3. 日志脱敏处理**
```cpp
// 建议：添加敏感信息掩码函数
std::string MaskBundleName(const std::string& bundleName) {
    if (bundleName.length() <= 4) return "***";
    return bundleName.substr(0, 2) + "***" + bundleName.substr(bundleName.length()-2);
}
```

### 8.3 低优先级

**4. URI 白名单验证**
```cpp
// ndef_har_data_parser.cpp
bool IsAllowedUriScheme(const std::string& scheme) {
    static const std::set<std::string> ALLOWED_SCHEMES = {
        "http", "https", "tel", "mailto", "sms"
    };
    return ALLOWED_SCHEMES.count(scheme) > 0;
}
```

---

## 9. 检查局限性说明

1. **动态分析缺失**: 本报告基于静态代码分析，未进行动态模糊测试
2. **二进制分析缺失**: 未分析编译后的二进制文件
3. **依赖库未检查**: NCI native 库等第三方组件未深入审查
4. **系统级安全**: SELinux、系统调用过滤等系统级防护未涉及

---

## 10. 参考文档

- [OpenHarmony 安全开发指南](https://gitee.com/openharmony/docs)
- [NFC 安全最佳实践](https://nfc-forum.org/)
- [OWASP 移动安全测试指南](https://owasp.org/www-project-mobile-security-testing-guide/)

