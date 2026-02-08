# Connected NFC Tag - 安全风险评估

## 1. 威胁模型概述

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           信任边界图                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      信任边界内 (Trusted)                           │   │
│  │   - NfcTagService (SA 1148)                                       │   │
│  │   - NfcTagHdiAdapter / NfcTagHdiImpl                              │   │
│  │   - HDI 驱动 (libconnected_nfc_tag_proxy_1.1.z.so)                 │   │
│  │   - NFC 芯片硬件                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    ↑                                       │
│                            IPC 调用                                         │
│                                    ↑                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      信任边界外 (Untrusted)                        │   │
│  │   - 应用进程 (JS/Native)                                          │   │
│  │   - connectedTag N-API 接口                                       │   │
│  │   - 用户输入 (NDEF 数据、事件回调)                                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 外部输入点

| 输入类型 | 来源 | 处理位置 |
|----------|------|----------|
| JS API 参数 | 应用层 | `nfc_napi_adapter.cpp` |
| 回调注册 | 应用层 | `nfc_napi_event.cpp` |
| IPC 请求 | 客户端进程 | `nfc_tag_stub.cpp` |
| HDI 事件 | NFC 芯片驱动 | `nfc_tag_hdi_impl.cpp` |

---

## 2. 攻击面清单

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| **N-API 参数注入** | JS 传入的 NDEF 数据可能包含恶意内容 | 中 |
| **回调劫持** | 事件回调可能被恶意利用 | 中 |
| **IPC 命令注入** | IPC 参数可能被恶意构造 | 低 |
| **HDI 事件伪造** | 驱动事件可能被伪造 | 低 |
| **权限提升** | 普通应用尝试调用系统 API | 高 |
| **资源耗尽** | 大量回调注册耗尽系统资源 | 中 |

---

## 3. 已发现的可利用点

### 3.1 权限验证绕过高风险

**证据**:
- 文件: `services/src/nfc_tag_service.cpp:34-43`
- 代码:
```cpp
__attribute__((weak)) ErrCode NfcTagServiceVerifyPermissionsHook(void)
{
    AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    ATokenTypeEnum tokenType = AccessTokenKit::GetTokenTypeFlag(callerToken);
    HILOGI("tokenType: %{public}d", static_cast<int>(tokenType));
    if (tokenType == ATokenTypeEnum::TOKEN_INVALID) {
        return NFC_SYS_PERM_FAILED;
    }
    return NFC_SUCCESS;
}
```

**问题分析**:
- `weak` 属性允许子类覆盖实现
- 如果未正确配置，权限检查可能被绕过

**触发条件**:
1. `connected_nfc_tag_only_system_app_access_api` 未启用
2. 系统权限检查被覆盖

**影响**:
- 权限检查失效
- 普通应用可能访问受限 NFC 功能

**修复建议**:
- 确保 `connected_nfc_tag_only_system_app_access_api` 在生产环境启用
- 移除 `weak` 属性，改为强制检查

---

### 3.2 回调注册无身份验证

**证据**:
- 文件: `services/src/nfc_tag_service.cpp:45-65`
- 代码:
```cpp
ErrCode NfcTagCallBackManager::RegisterListener(sptr<INfcTagCallback> listener)
{
    uint64_t tokenId = IPCSkeleton::GetCallingFullTokenID();
    // ...
    if (listenerMap_.count(tokenId) != 0) {
        return NFC_CALLBACK_REGISTERED;
    }
    if (listenerMap_.size() >= MAX_LISTENER_NUM) {
        return NFC_TOO_MANY_CALLBACK;
    }
    listenerMap_[tokenId] = listener;
    return NFC_SUCCESS;
}
```

**问题分析**:
- 使用 `GetCallingFullTokenID()` 区分调用者
- 但验证仅基于 ID 存在性，不验证回调对象有效性

**触发条件**:
1. 恶意应用注册大量回调（达到 30 个限制）

**影响**:
- 资源耗尽 DoS
- 合法应用无法注册回调

**修复建议**:
- 增加回调对象有效性验证
- 考虑使用白名单机制

---

### 3.3 NDEF 数据长度无上限检查

**证据**:
- 文件: `nfc_napi_adapter.cpp:119-122`
- 代码:
```cpp
if (!ParseString(env, inputWrittenNdefData, argv[0])) {
    CheckNfcStatusCodeAndThrow(env, NFC_INVALID_PARAMETER);
    return CreateUndefined(env);
}
HILOGI("WriteNdefTag argc = %{public}zu, len = %{public}zu", argc, inputWrittenNdefData.length());
```

**问题分析**:
- 服务端 `nfc_tag_service.cpp:192-199` 有检查，但 N-API 层无检查
```cpp
ErrCode NfcTagService::WriteNdefTag(const std::string &data)
{
    ErrCode ret = VerifyPermissionsBeforeEntry();
    if (ret != NFC_SUCCESS) {
        return ret;
    }
    return hdiAdapter_.WriteNdefTag(data);
}
```

**触发条件**:
1. 发送超长 NDEF 数据（> 512 字节）

**影响**:
- 拒绝服务（驱动层崩溃）
- 内存耗尽

**修复建议**:
- 在 N-API 层添加数据长度检查
- 返回明确的错误码

---

### 3.4 事件类型字符串无长度限制

**证据**:
- 文件: `nfc_napi_event.cpp:31, 129-131`
- 代码:
```cpp
constexpr int NOTIFY_TYPE_LEN = 64;

char type[NOTIFY_TYPE_LEN] = {0};
size_t typeLen = 0;
napi_get_value_string_utf8(env, argv[0], type, sizeof(type), &typeLen);
```

**问题分析**:
- 使用固定大小缓冲区（64 字节）
- 超出部分被截断，但无错误报告

**触发条件**:
1. 传入超过 64 字节的事件类型字符串

**影响**:
- 静默失败
- 调试困难

**修复建议**:
- 返回明确的参数错误
- 记录警告日志

---

### 3.5 PAC-RET 保护不完整

**证据**:
- 文件: `connected_nfc_tag.gni:24`
- 代码:
```gn
branch_protector_ret = "pac_ret"
```

**问题分析**:
- PAC-RET 仅保护返回地址
- 其他控制流指针未受保护

**影响**:
- 控制流劫持风险
- ROP 攻击可能

**修复建议**:
- 启用完整 CFI (Control Flow Integrity)
- 考虑使用 BTI (Branch Target Identification)

---

## 4. 安全机制说明

### 4.1 权限模型

| 权限 | 描述 | 保护级别 |
|------|------|----------|
| `ohos.permission.NFC_TAG` | NFC Tag 操作权限 | 系统权限 |
| 系统应用 | 运行在 system privilege | 最高 |

**权限检查流程**:
```
应用调用 API
    ↓
VerifyPermissionsBeforeEntry()
    ↓
Utils::IsGranted(TAG_PERMISSION)
    ↓
AccessTokenKit::VerifyAccessToken()
    ↓
返回: PERMISSION_GRANTED / PERMISSION_DENIED
```

---

### 4.2 编译时安全

| 安全机制 | 状态 | 配置位置 |
|----------|------|----------|
| FORTIFY_SOURCE | ✓ 启用 | `common_cflags: "-D_FORTIFY_SOURCE=2"` |
| Stack Canaries | ✓ 启用 | GCC 默认 |
| PAC-RET | ✓ 启用 | `BUILD.gn: branch_protector_ret` |
| CFI | ✓ 启用 | `global_sanitize: cfi = true` |
| Bounds Sanitize | ✓ 启用 | `global_sanitize: boundary_sanitize = true` |
| Integer Overflow | ✓ 启用 | `global_sanitize: integer_overflow = true` |
| UBSan | ✓ 启用 | `global_sanitize: ubsan = true` |

---

### 4.3 运行时安全

| 机制 | 说明 | 文件位置 |
|------|------|----------|
| 回调数量限制 | 最多 30 个回调 | `nfc_tag_service.cpp:32` |
| TokenID 隔离 | 每个进程独立回调 | `nfc_tag_service.cpp:47` |
| 异步事件队列 | 高优先级事件处理 | `nfc_napi_event.cpp:71` |

---

## 5. 安全建议

### 5.1 高优先级

| 问题 | 建议 | 预期效果 |
|------|------|----------|
| 权限检查可能被绕过 | 移除 `weak` 属性，强制权限检查 | 防止权限提升 |
| NDEF 数据无长度检查 | 在 N-API 层添加 512 字节限制 | 防止 DoS |

### 5.2 中优先级

| 问题 | 建议 | 预期效果 |
|------|------|----------|
| 回调对象无验证 | 增加回调有效性检查 | 防止资源耗尽 |
| 事件字符串静默截断 | 返回明确的参数错误 | 便于调试 |

### 5.3 低优先级

| 问题 | 建议 | 预期效果 |
|------|------|----------|
| PAC-RET 不完整 | 启用完整 CFI/BTI | 增强控制流保护 |
| 缺少输入日志记录 | 添加安全审计日志 | 便于入侵检测 |

---

## 6. 检查范围声明

### 6.1 本次评估覆盖范围

| 组件 | 文件/目录 | 检查结果 |
|------|-----------|----------|
| N-API 层 | `frameworks/js/napi/` | ✓ 已检查 |
| 服务层 | `services/src/` | ✓ 已检查 |
| 内部 API | `interfaces/inner_api/` | ✓ 已检查 |
| 构建配置 | `*.gni`, `BUILD.gn` | ✓ 已检查 |

### 6.2 未覆盖范围

| 组件 | 原因 |
|------|------|
| HDI 驱动实现 | 独立仓库 (`drivers_interface_connected_nfc_tag`) |
| NFC 芯片固件 | 硬件实现 |
| 系统权限框架 | 依赖 `access_token` 子系统 |

---

## 7. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [02_NAPI.md](./02_NAPI.md) |
| 内部 API | [03_InnerAPI.md](./03_InnerAPI.md) |
| 构建说明 | [04_Build.md](./04_Build.md) |
