# Data Share 安全风险分析

## 目的

本文档基于代码分析 Data Share 的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- 安全工程师
- 代码审计人员
- 安全架构师

## 分析范围

- **分析范围**: frameworks/native/ 下非测试代码
- **版本**: 3.2.0
- **时间**: 2025-02-06

## 威胁模型

### 资产识别

| 资产 | 说明 | 敏感级别 |
|------|------|----------|
| 应用数据 | 通过 Data Share 共享的数据 | 高 |
| 权限凭证 | AccessToken | 高 |
| Provider 配置 | proxyData 中的权限配置 | 中 |
| 用户隐私 | 联系人、通话记录等 | 高 |

### 攻击者模型

| 攻击者类型 | 能力 | 目标 |
|-----------|------|------|
| 恶意应用 | 普通应用权限 | 访问未授权数据 |
| 本地攻击者 | Shell/Root 权限 | 绕过权限检查 |
| 远程攻击者 | 网络访问能力 | 间接攻击（通过 IPC）|

### 信任边界

```
[不可信] 普通应用
    │
    │ NAPI 调用
    ▼
[半可信] DataShare NAPI 层 (JS/Native 边界)
    │
    │ IPC (Binder)
    ▼
[可信] DataShare Stub (Provider)
    │
    │ UV Queue 调度
    ▼
[半可信] ExtensionAbility (JS/ArkTS 代码)
```

## 攻击面清单

### 1. N-API 接口攻击面

| 入口 | 风险 | 防护措施 |
|------|------|----------|
| `createDataShareHelper` | 非系统应用创建 Helper | 检查 `IsSystemApp()` (line 41-45) |
| `insert/update/delete` | 数据注入 | 参数类型检查 |
| `query` | 信息泄露 | 权限检查 + 列过滤 |
| `openFile` | 路径遍历 | URI 验证 |
| `registerObserver` | 拒绝服务 | 权限检查 + 数量限制 |

### 2. IPC 通信攻击面

| 入口 | 风险 | 防护措施 |
|------|------|----------|
| `CMD_INSERT` | 越权写入 | `CheckCallingPermission()` |
| `CMD_QUERY` | 越权读取 | `CheckCallingPermission()` |
| `CMD_REGISTER_OBSERVER` | 观察者洪泛 | 权限检查 |
| IPC Parcel | 反序列化攻击 | 边界检查 |

### 3. 权限系统攻击面

| 入口 | 风险 | 防护措施 |
|------|------|----------|
| `VerifyPermission` | 权限绕过 | AccessTokenKit |
| `UriIsTrust` | URI 欺骗 | Scheme 白名单 |
| `GetProviderInfo` | BMS 信息泄露 | IPC 身份重置 |

### 4. 文件系统攻击面

| 入口 | 风险 | 防护措施 |
|------|------|----------|
| `OpenFile` | 路径遍历 | URI 规范化 |
| `GetFileTypes` | 信息泄露 | 权限检查 |

## 安全风险点（基于证据）

### 风险 1: VerifyProvider 返回值被忽略

**风险等级**: 🔴 **高**

**位置**: `frameworks/native/provider/src/datashare_stub_impl.cpp:229`

**代码证据**:
```cpp
// Line 229
VerifyProvider(callingInfo, callingInfo.fullTokenId, uri);  // 返回值被忽略！
auto ret = InsertInner(callingInfo, uri, value);
```

**同样问题出现在**:
- Line 274 (Update)
- Line 321 (Delete)
- Line 363 (Query)
- Line 410 (InsertEx)
- Line 453 (UpdateEx)
- Line 500 (DeleteEx)
- Line 548 (BatchInsert)
- Line 633 (BatchUpdate)

**问题描述**:
`VerifyProvider()` 函数检查调用者是否为系统应用或白名单 Provider，但返回值被忽略，导致非白名单 Provider 也能继续执行操作。

**攻击路径**:
```
恶意应用 ──► IPC 调用 CMD_INSERT
    │
    ▼
DataShareStubImpl::Insert()
    │
    ▼
VerifyProvider() ──► 返回 false (非白名单)
    │
    ▼
[应该拒绝，但实际继续执行]
    │
    ▼
InsertInner() ──► 数据被写入
```

**修复建议**:
```cpp
if (!VerifyProvider(callingInfo, callingInfo.fullTokenId, uri)) {
    LOG_ERROR("Provider verification failed");
    return ERR_PERMISSION_DENIED;
}
```

### 风险 2: 空权限直接通过

**风险等级**: 🟡 **中**

**位置**: `frameworks/native/permission/src/data_share_permission.cpp:324-326`

**代码证据**:
```cpp
// Line 324-326
if (!isSilentUri && permission.empty()) {
    return true;  // 非 Silent URI 且权限为空时直接通过
}
```

**问题描述**:
对于非 Silent URI，如果配置的权限为空字符串，权限检查直接返回 true。这可能允许未配置权限的 URI 被访问。

**攻击路径**:
```
攻击者 ──► 构造特殊 URI
    │
    ▼
proxyData 未配置 readPermission/writePermission
    │
    ▼
VerifyPermission() ──► permission.empty() ──► 返回 true
    │
    ▼
获得访问权限
```

**修复建议**:
明确区分"无权限要求"和"权限未配置"两种场景，对于未配置权限的 URI 应默认拒绝。

### 风险 3: URI 前缀匹配可能存在绕过

**风险等级**: 🟡 **中**

**位置**: `frameworks/native/permission/src/data_share_permission.cpp:93-109`

**代码证据**:
```cpp
// Line 93-109
bool IsInUriTrusts(Uri &uri) {
    auto config = ConfigFactory::GetInstance().GetDataShareConfig();
    std::string uriStr = uri.ToString();
    for (std::string& item : config->uriTrusts) {
        if (item.length() > uriStr.length() ||
            uriStr.compare(0, item.length(), item) != 0) {
            continue;
        }
        return true;  // 前缀匹配即信任
    }
    return false;
}
```

**问题描述**:
使用简单的前缀匹配检查 URI 是否在信任列表中。如果信任列表包含 `datashare://com.example/`，恶意 URI `datashare://com.example.attacker/` 可能通过前缀匹配。

**攻击路径**:
```
信任列表: ["datashare://com.example/"]
    │
    ▼
恶意 URI: "datashare://com.example.attacker/data"
    │
    ▼
前缀匹配成功 ──► 绕过信任检查
```

**修复建议**:
使用更严格的匹配规则，确保 `/` 分隔符匹配：
```cpp
// 确保匹配到完整的路径段
if (uriStr.compare(0, item.length(), item) == 0 &&
    (uriStr.length() == item.length() || uriStr[item.length()] == '/')) {
    return true;
}
```

### 风险 4: IPC 身份重置可能的风险

**风险等级**: 🟢 **低**

**位置**: `frameworks/native/permission/src/data_share_called_config.cpp:136-139`

**代码证据**:
```cpp
// Line 136-139
// Set the default userId.
// Set ipc identity to shell or root, to avoid check uid fail.
// otherwise BMS may check permission failed.
int32_t uid = getuid();
SetFirstCallerUid(uid >= SYSTEM_UID ? uid : 0);
```

**问题描述**:
为了调用 BMS，代码临时将 IPC 身份设置为 shell/root。虽然目的是获取 Bundle 信息，但这种做法可能带来安全风险。

**风险**:
- 如果 SetFirstCallerUid 被滥用，可能绕过权限检查
- 时间窗口内可能存在竞态条件

**修复建议**:
- 最小化身份重置的范围
- 使用更细粒度的权限机制
- 考虑使用专门的特权服务代理

### 风险 5: Provider 白名单硬编码

**风险等级**: 🟢 **低**

**位置**: `frameworks/native/provider/src/datashare_stub_impl.cpp:40-45`

**代码证据**:
```cpp
// Line 40-45
const std::set<std::string> PROVIDER_LIST = {
    "5765880207853551549",
    "5765880207853570539",
    "5765880207853771197",
    "5765880207854616753"
}; // 对应 datamgr_service providerIdentifiers 列表
```

**问题描述**:
Provider 白名单硬编码在代码中，需要随版本更新同步更新。如果维护不当可能导致：
- 新 Provider 无法通过验证
- 旧 Provider 仍被信任（即使已不安全）

**修复建议**:
- 考虑使用配置文件存储白名单
- 添加版本号或有效期检查
- 定期审计白名单内容

### 风险 6: 缓存清理策略简单粗暴

**风险等级**: 🟢 **低**

**位置**: `frameworks/native/permission/src/data_share_permission.cpp:211-214, 290-293`

**代码证据**:
```cpp
// Line 211-214
if (silentCache_.size() >= CACHE_SIZE) {
    LOG_INFO("silentCache_ full, clear all");
    silentCache_.clear();  // 直接清空所有缓存
}

// Line 290-293
if (extensionCache_.size() >= CACHE_SIZE) {
    LOG_INFO("extensionCache_ full, clear all");
    extensionCache_.clear();  // 直接清空所有缓存
}
```

**问题描述**:
缓存满时直接清空所有缓存，而不是 LRU 淘汰。这可能导致：
- 性能抖动（缓存突然全部失效）
- 缓存击穿风险

**修复建议**:
实现 LRU 淘汰机制：
```cpp
// 使用 LRU 策略淘汰最久未使用的条目
if (cache.size() >= CACHE_SIZE) {
    auto oldest = std::min_element(cache.begin(), cache.end(),
        [](const auto& a, const auto& b) {
            return a.second.lastAccess < b.second.lastAccess;
        });
    cache.erase(oldest);
}
```

### 风险 7: 同 Bundle Silent URI 免权限可能的风险

**风险等级**: 🟢 **低**

**位置**: `frameworks/native/permission/src/data_share_permission.cpp:332-334`

**代码证据**:
```cpp
// Line 332-334
if (permission.empty() && isSilentUri) {
    // 同 Bundle 免权限
    if (result == Security::AccessToken::RET_SUCCESS && 
        tokenInfo.bundleName == providerName) {
        return true;
    }
    return false;
}
```

**问题描述**:
Silent URI 允许同 Bundle 访问免权限。这在设计上是合理的，但需要确保：
- Bundle 名称不能被伪造
- Bundle 隔离机制严格

**风险**:
- 如果 Bundle 名称验证有漏洞，可能绕过权限
- 共享 UID 的应用可能滥用此机制

**验证**:
Bundle 名称通过 `AccessTokenKit::GetHapTokenInfo()` 获取，相对可信。

## 安全防护措施汇总

| 防护措施 | 位置 | 说明 |
|----------|------|------|
| 系统应用检查 | `datashare_stub_impl.cpp:66-112` | `IsCallerSystemApp()` |
| Token 类型检查 | `datashare_stub_impl.cpp:73-77` | NATIVE/SHELL 放行 |
| 权限字符串检查 | `data_share_permission.cpp:298-308` | `VerifyAccessToken()` |
| URI Scheme 白名单 | `data_share_permission.cpp:154-169` | 内置 Scheme 信任 |
| 配置白名单 | `data_share_permission.cpp:93-109` | `uriTrusts` 配置 |
| Provider 白名单 | `datashare_stub_impl.cpp:40-45` | 硬编码 ID 列表 |
| 共享内存隔离 | `ishared_result_set.cpp` | Ashmem 传输 |
| IPC 边界检查 | 所有 IPC 方法 | Parcel 校验 |
| CFI 保护 | BUILD.gn | 控制流完整性 |
| 边界消毒 | BUILD.gn | 缓冲区边界检查 |

## 修复建议优先级

| 优先级 | 风险 | 修复复杂度 | 建议操作 |
|--------|------|-----------|----------|
| 🔴 P0 | VerifyProvider 返回值被忽略 | 低 | **立即修复** - 添加返回值检查 |
| 🟡 P1 | 空权限直接通过 | 中 | 明确区分权限场景 |
| 🟡 P1 | URI 前缀匹配绕过 | 中 | 实现严格匹配 |
| 🟢 P2 | IPC 身份重置 | 高 | 最小化范围 |
| 🟢 P2 | Provider 白名单硬编码 | 中 | 配置化存储 |
| 🟢 P3 | 缓存清理策略 | 低 | 实现 LRU |
| 🟢 P3 | 同 Bundle 免权限 | 低 | 文档说明 |

## 检查局限性

本分析存在以下局限性：

1. **未分析测试代码** - 根据约束条件，test/ 目录代码未纳入分析
2. **未动态验证** - 仅基于静态代码分析，未进行运行时测试
3. **依赖组件未深入** - AccessTokenKit、BMS 等外部组件的安全机制未深入分析
4. **配置依赖** - 部分安全逻辑依赖运行时配置文件（如 uriTrusts）

## 关键结论

1. **高优先级风险**: `VerifyProvider` 返回值被忽略是当前最严重的安全问题，可能导致权限绕过
2. **权限检查机制完整** - 多层权限检查（系统应用、Token 类型、权限字符串、URI 信任）设计合理
3. **IPC 通信安全** - 使用 Binder IPC，配合 Parcel 校验和 CFI 保护
4. **需要修复** - 至少 2 个中优先级和 1 个高优先级安全问题需要修复

## 相关文档

- [架构设计](05_Architecture.md) - 信任边界和组件关系
- [内部 API](06_Inner_API.md) - 权限相关接口
- [构建系统](07_Build_System.md) - 安全编译选项
