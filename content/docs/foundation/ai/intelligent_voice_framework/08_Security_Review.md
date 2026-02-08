# 安全评审

> **目的**: 对 Intelligent Voice Framework 进行全面的安全风险分析  
> **适用范围**: 安全评审、合规检查、渗透测试  
> **最后更新**: 2026-02-06

---

## 1. 评审范围

### 1.1 评审边界

| 层级 | 范围 |
|------|------|
| **框架层** | N-API 绑定、Native API、内部接口 |
| **服务层** | SA 服务、引擎管理、触发器管理 |
| **工具层** | 公共工具库 |

### 1.2 不在评审范围内

| 层级 | 说明 |
|------|------|
| **HDI 驱动** | 第三方驱动实现（由供应商负责） |
| **DSP 硬件** | 硬件安全（由芯片厂商负责） |
| **测试代码** | `tests/`, `llt/` |

---

## 2. 信任边界

### 2.1 边界定义

```
┌─────────────────────────────────────────────────────────────────┐
│                         不可信区域                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              第三方应用 (Untrusted Apps)                 │  │
│  └─────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                           │ IPC (Binder)
                           │ 边界检查点
┌───────────────────────────┴────────────────────────────────────┐
│                         可信区域                                 │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │
│  │ N-API   │  │ Native  │  │  SA     │  │ 引擎/触发器    │  │
│  │ 绑定    │  │ API     │  │ 服务    │  │                 │  │
│  └─────────┘  └─────────┘  └─────────┘  └─────────────────┘  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              系统服务 (Trusted Services)                 │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────┬────────────────────────────────────┘
                           │ HDI
                           │ 边界检查点
┌───────────────────────────┴────────────────────────────────────┐
│                        半可信区域                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │           HDI 驱动 / DSP 驱动 (Vendor)                 │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 边界检查点

| 位置 | 检查项 | 实现 |
|------|--------|------|
| **N-API 入口** | 系统应用验证 | `IntellVoiceCommonNapi::CheckIsSystemApp()` |
| **IPC 入口** | 接口令牌验证 | `ReadInterfaceToken()` vs `GetDescriptor()` |
| **IPC 入口** | 权限验证 | `VerifyClientPermission()` |
| **服务入口** | 系统应用验证 | `CheckIsSystemApp()` |
| **隐私操作** | 麦克风权限记录 | `PrivacyKit::StartUsingPermission()` |

**证据来源**:
- `frameworks/js/napi/intell_voice_common_napi.cpp:105-115`
- `services/intell_voice_service/server/sa/intell_voice_service_stub.cpp:206-212`
- `utils/intell_voice_util.cpp:123-152`

---

## 3. 权限模型

### 3.1 权限声明

**服务配置权限** (`services/etc/intell_voice_service.cfg`):
```json
"permission": [
    "ohos.permission.MANAGE_INTELLIGENT_VOICE",    // 管理智能语音
    "ohos.permission.MICROPHONE",                    // 麦克风
    "ohos.permission.GET_TELEPHONY_STATE",           // 电话状态
    "ohos.permission.READ_CALL_LOG",                 // 通话记录
    "ohos.permission.START_ABILITIES_FROM_BACKGROUND", // 后台启动
    "ohos.permission.WAKEUP_VOICE",                 // 语音唤醒
    "ohos.permission.WAKEUP_VISION",               // 视觉唤醒
    "ohos.permission.PERMISSION_USED_STATS"         // 权限使用统计
]
```

**API 权限要求**:
| API | 必需权限 |
|-----|---------|
| 所有引擎 API | `ohos.permission.MANAGE_INTELLIGENT_VOICE` |
| 录音相关 API | `ohos.permission.MICROPHONE` (API 22+) |

### 3.2 权限验证实现

```cpp
// utils/intell_voice_util.cpp
bool IntellVoiceUtil::VerifyClientPermission(const std::string &permissionName)
{
    Security::AccessToken::AccessTokenID clientTokenId = IPCSkeleton::GetCallingTokenID();
    int res = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        clientTokenId, permissionName);
    if (res != Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        INTELL_VOICE_LOG_ERROR("Permission denied");
        return false;
    }
    return true;
}

bool IntellVoiceUtil::CheckIsSystemApp()
{
    uint64_t fullTokenId = IPCSkeleton::GetCallingFullTokenID();
    // ...
    if (!Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(fullTokenId)) {
        INTELL_VOICE_LOG_INFO("Not system app, permission reject");
        return false;
    }
    return true;
}
```

**证据来源**: `utils/intell_voice_util.cpp:123-171`

---

## 4. 攻击面分析

### 4.1 N-API 输入点

| 输入类型 | 处理位置 | 风险等级 |
|---------|---------|---------|
| WakeupPhrase | `createEnroll/WakeupIntelligentVoiceEngine` | 中 |
| BundleName | `setWakeupHapInfo()` | 低 |
| 参数 Key/Value | `setParameter()`/`getParameter()` | 低 |
| 文件路径 | `getWakeupSourceFiles()` | 中 |

### 4.2 IPC 数据点

| 数据类型 | 处理位置 | 风险等级 |
|---------|---------|---------|
| 接口令牌 | `OnRemoteRequest()` | 高 |
| MessageParcel | 各 IPC 处理函数 | 中 |
| Ashmem | `GetFileDataFromAshmem()` | 高 |

### 4.3 隐私数据点

| 数据类型 | 处理 | 风险等级 |
|---------|------|---------|
| 声纹特征 | 内存存储 | 高 |
| 唤醒音频 | 临时缓存 | 中 |
| 用户配置 | 持久化存储 | 低 |

---

## 5. 风险清单

### 5.1 高风险项

#### 风险 1: 声纹特征内存暴露

| 属性 | 值 |
|------|-----|
| **编号** | SEC-001 |
| **风险等级** | 高 |
| **攻击面** | 引擎内部内存 |
| **触发条件** | 设备被 root 或调试 |

**证据**: 声纹特征在内存中以 `std::vector<uint8_t>` 形式存储

**影响**: 攻击者可能读取用户声纹数据

**修复建议**:
- 使用内存加密（如 MTE）
- 敏感数据处理后立即清零
- 限制调试模式下的内存访问

---

#### 风险 2: Ashmem 共享内存安全

| 属性 | 值 |
|------|-----|
| **编号** | SEC-002 |
| **风险等级** | 高 |
| **攻击面** | `GetFileDataFromAshmem()` |
| **触发条件** | 恶意应用构造特殊 Ashmem |

**证据**: `intell_voice_manager.cpp` 使用 Ashmem 传递文件内容

```cpp
int32_t IntellVoiceManager::GetFileDataFromAshmem(
    sptr<Ashmem> ashmem, std::vector<uint8_t> &fileData)
```

**影响**: 可能导致越界读取或拒绝服务

**修复建议**:
- 严格校验 Ashmem 大小
- 添加长度限制
- 使用安全的内存复制函数

---

### 5.2 中风险项

#### 风险 3: WakeupPhrase 未加密传输

| 属性 | 值 |
|------|-----|
| **编号** | SEC-003 |
| **风险等级** | 中 |
| **攻击面** | IPC 参数传递 |
| **触发条件** | IPC 监听 |

**证据**: WakeupPhrase 通过 `MessageParcel` 传递

```cpp
// IIntellVoiceService 接口
virtual int32_t CreateIntellVoiceEngine(IntellVoiceEngineType type, 
                                         sptr<IIntellVoiceEngine> &inst) = 0;
```

**影响**: 可能被中间人窃听

**修复建议**:
- 使用加密 IPC 通道
- 敏感参数使用安全传输

---

#### 风险 4: BundleName 注入

| 属性 | 值 |
|------|-----|
| **编号** | SEC-004 |
| **风险等级** | 中 |
| **攻击面** | `setWakeupHapInfo()` |
| **触发条件** | 恶意构造 BundleName |

**证据**: `intell_voice_info.h`
```cpp
struct WakeupHapInfo {
    std::string bundleName;
    std::string abilityName;
};
```

**影响**: 可能导致权限提升或拒绝服务

**修复建议**:
- 验证 BundleName 格式
- 白名单校验
- 使用安全的 Want 设置 API

---

### 5.3 低风险项

#### 风险 5: 日志信息泄露

| 属性 | 值 |
|------|-----|
| **编号** | SEC-005 |
| **风险等级** | 低 |
| **攻击面** | `INTELL_VOICE_LOG_*` |
| **触发条件** | 日志级别设置为 DEBUG |

**证据**: 代码中使用 `HILOG_ENABLE` 和 `ENABLE_DEBUG`

**修复建议**:
- 生产版本禁用 DEBUG 日志
- 敏感信息脱敏后打印

---

#### 风险 6: 配置文件权限

| 属性 | 值 |
|------|-----|
| **编号** | SEC-006 |
| **风险等级** | 低 |
| **攻击面** | `intell_voice_service.cfg` |
| **触发条件** | 文件权限配置不当 |

**证据**: `services/etc/intell_voice_service.cfg`

**修复建议**:
- 确保配置文件权限为 600
- 敏感配置加密存储

---

## 6. 安全机制评估

### 6.1 已实现的安全机制

| 机制 | 实现 | 有效性 |
|------|------|--------|
| 系统应用检查 | `IsSystemAppByFullTokenID()` | ✅ 有效 |
| 权限验证 | `VerifyAccessToken()` | ✅ 有效 |
| IPC 令牌校验 | `ReadInterfaceToken()` | ✅ 有效 |
| 隐私权限记录 | `PrivacyKit` | ✅ 有效 |
| SELinux 隔离 | `u:r:intell_voice_service:s0` | ✅ 有效 |
| CFI 保护 | `sanitize { cfi = true }` | ✅ 有效 |

### 6.2 待加强的安全机制

| 机制 | 建议 | 优先级 |
|------|------|--------|
| 内存加密 | 使用 MTE 或 SE 保护敏感数据 | 高 |
| 输入校验 | 添加参数白名单校验 | 高 |
| 审计日志 | 记录所有敏感操作 | 中 |

---

## 7. 总结

### 7.1 整体评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 权限控制 | ⭐⭐⭐⭐⭐ | 完善的两层权限检查 |
| 输入校验 | ⭐⭐⭐☆☆ | 需加强参数白名单 |
| 内存安全 | ⭐⭐⭐⭐☆ | CFI 启用，敏感数据需加密 |
| 隐私保护 | ⭐⭐⭐⭐☆ | 隐私权限记录完整 |
| 隔离性 | ⭐⭐⭐⭐⭐ | SELinux + SA 隔离 |

### 7.2 建议优先级

| 优先级 | 建议 |
|--------|------|
| **P0** | 敏感内存数据加密 |
| **P1** | 添加参数白名单校验 |
| **P1** | Ashmem 安全加固 |
| **P2** | 完善审计日志 |
| **P3** | 日志脱敏 |

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Overview.md) | 权限要求 |
| [架构设计](./02_Architecture.md) | 数据流 |
| [N-API 参考](./04_NAPI_Reference.md) | API 安全考量 |
