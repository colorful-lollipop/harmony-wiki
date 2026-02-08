# 05_Security_Review - 安全风险评审

## 评审范围

本文档覆盖 **ecological_rule_manager** 模块的 Inner API 接口、IPC 通信、参数校验等安全机制。

**检查范围**:
- `services/manager/src/` - SA 服务端实现
- `interfaces/innerkits/` - Client SDK 实现
- `profile/6105.json` - SA 配置

**未包含**: 测试代码（test/ 目录）

## 威胁模型

### 外部输入

| 输入源 | 数据类型 | 用途 |
|--------|----------|------|
| IPC 调用（系统服务） | Want, CallerInfo, AbilityInfo | 业务规则判断 |
| Parcel 反序列化 | Parcelable 结构 | 数据传递 |
| 配置/Profile | JSON | SA 启动配置 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                    信任边界内 (Trusted)                      │
├─────────────────────────────────────────────────────────────┤
│  • Foundation 进程内的系统服务                                │
│  • Native Token / Shell Token 进程                          │
│  • ROOT_UID (0) 进程                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC 调用
┌─────────────────────────────────────────────────────────────┐
│                   信任边界外 (Untrusted)                     │
├─────────────────────────────────────────────────────────────┤
│  • 非系统应用（普通三方应用）                                 │
│  • 未通过 VerifySystemApp 校验的调用方                       │
└─────────────────────────────────────────────────────────────┘
```

## 攻击面分析

| 攻击面 | 类型 | 说明 |
|--------|------|------|
| **IPC 接口** | RPC 调用 | 4 个远程接口可能被恶意调用 |
| **Parcel 反序列化** | 数据解析 | Want/CallerInfo/AbilityInfo 反序列化 |
| **内存安全** | C++ 实现 | Parcelable 处理、指针操作 |
| **权限绕过** | 访问控制 | EnforceInterceToken / VerifySystemApp |

## 已实现的安全机制

### 1. 接口令牌校验 (EnforceInterceToken)

**文件**: `services/manager/src/ecological_rule_mgr_service_stub.cpp:227-231`

```cpp
bool EcologicalRuleMgrServiceStub::EnforceInterceToken(MessageParcel &data)
{
    std::u16string interfaceToken = data.ReadInterfaceToken();
    return interfaceToken == ERMS_INTERFACE_TOKEN;
}
```

**说明**: 校验 IPC 调用方的接口令牌，防止伪造请求。

### 2. 系统应用校验 (VerifySystemApp)

**文件**: `services/manager/src/ecological_rule_mgr_service_stub.cpp:233-257`

**校验逻辑**:
1. Native Token / Shell Token → 直接放行
2. ROOT_UID (0) → 直接放行
3. Foundation UID (5523) → 放行
4. 其他 → 检查 fullTokenID 是否为系统应用

### 3. Parcel 空指针校验

**文件**: `services/manager/src/ecologic_rule_mgr_service_stub.cpp`

**示例**（第 73-81 行）:
```cpp
sptr<Want> want = data.ReadParcelable<Want>();
if (want == nullptr) {
    LOG_ERROR("read want failed");
    return ERR_FAILED;
}
sptr<CallerInfo> caller = data.ReadParcelable<CallerInfo>();
if (caller == nullptr) {
    LOG_ERROR("read caller failed");
    return ERR_FAILED;
}
```

### 4. 大小限制

**文件**: `services/manager/include/ecological_rule_mgr_service_stub.h`

| 常量 | 值 | 用途 |
|------|-----|------|
| `MAX_WANT_SIZE` | 15 | IsSupportPublishForm 中 Want 列表上限 |
| `MAX_ABILITY_INFO_SIZE` | 1000 | EvaluateResolveInfos 中 AbilityInfo 上限 |

**检查逻辑**（第 198-201 行）:
```cpp
if (wantSize > MAX_WANT_SIZE) {
    LOG_ERROR("wantSize exceed the maximum limit...");
    return ERR_FAILED;
}
```

## 安全风险与修复建议

### 风险 1: 拒绝服务 (DoS) - 无超时保护

| 项目 | 内容 |
|------|------|
| **风险等级** | 中 |
| **证据** | `ecological_rule_mgr_service_proxy.cpp:44` 使用同步 IPC (`TF_SYNC`) |
| **触发条件** | SA 阻塞时，调用方线程被永久阻塞 |
| **影响** | 调用方线程资源耗尽，导致系统服务不可用 |
| **修复建议** | 1. 使用异步 IPC (`TF_ASYNC`)<br>2. 添加超时机制<br>3. 在 Client 端添加超时保护 |

**参考代码**:
```cpp
// 文件: ecological_rule_mgr_service_proxy.cpp:44
MessageOption option = { MessageOption::TF_SYNC };
```

### 风险 2: Parcel 序列化/反序列化安全

| 项目 | 内容 |
|------|------|
| **风险等级** | 低 |
| **证据** | 使用 Parcelable 机制传递 Want/CallerInfo |
| **触发条件** | Parcel 数据异常或恶意构造 |
| **影响** | 潜在的解析错误或内存问题 |
| **修复建议** | 1. 验证 Parcel 数据完整性<br>2. 增加深度校验防止递归 |

**参考代码**:
```cpp
// 文件: services/manager/src/ecologic_rule_mgr_service_stub.cpp:67-90
// Parcelable ReadParcelable 调用
sptr<Want> want = data.ReadParcelable<Want>();
```

### 风险 3: CallerInfo packageName 空字符串绕过

| 项目 | 内容 |
|------|------|
| **风险等级** | 低 |
| **证据** | `ecological_rule_mgr_service_client.cpp:106-110` |
| **触发条件** | packageName 为空字符串时直接允许 |
| **影响** | 恶意应用可能通过构造空包名绕过检查 |
| **修复建议** | 1. 在服务端也增加 packageName 校验<br>2. 区分空字符串和未设置的情况 |

**参考代码**:
```cpp
// 文件: ecological_rule_mgr_service_client.cpp:106-110
if (callerInfo.packageName.find_first_not_of(' ') == std::string::npos) {
    rule.isAllow = true;
    LOG_DEBUG("callerInfo packageName is empty, allow = true");
    return 0;
}
```

### 风险 4: SA 进程死亡处理

| 项目 | 内容 |
|------|------|
| **风险等级** | 低 |
| **证据** | `ecological_rule_mgr_service_client.cpp` 实现了 DeathRecipient |
| **当前状态** | 已实现 - SA 死亡后会自动重连 |
| **说明** | DeathRecipient 机制确保服务可用性 |

### 风险 5: 缺少输入白名单校验

| 项目 | 内容 |
|------|------|
| **风险等级** | 高 (设计层面) |
| **证据** | 当前所有接口返回 SUCCESS，无实际业务校验 |
| **触发条件** | 后续业务实现时可能缺少白名单 |
| **影响** | 设备厂商的管控规则可能无法生效 |
| **修复建议** | 1. 实现规则引擎<br>2. 添加配置解析模块<br>3. 完善白名单/黑名单机制 |

**当前实现状态**: 所有接口返回 SUCCESS（`ecologic_rule_mgr_service.cpp:62-89`）

```cpp
// 文件: ecologic_rule_mgr_service.cpp:57-63
int32_t EcologicalRuleMgrService::QueryFreeInstallExperience(...)
{
    LOG_DEBUG(...);
    return SUCCESS;  // TODO: 实际业务逻辑待实现
}
```

## 风险汇总表

| # | 风险 | 等级 | 状态 | 责任方 |
|---|------|------|------|--------|
| 1 | 拒绝服务 - 无超时保护 | 中 | 未修复 | 开发者 |
| 2 | Parcel 反序列化安全 | 低 | 已知 | 框架 |
| 3 | CallerInfo 空字符串绕过 | 低 | 需服务端补全 | 开发者 |
| 4 | SA 进程死亡处理 | 低 | 已实现 | 开发者 |
| 5 | 缺少输入白名单校验 | 高 | 设计待完善 | 架构师 |

## 安全相关证据索引

| 证据 | 文件路径 | 行号 |
|------|----------|------|
| EnforceInterceToken | `services/manager/src/ecologic_rule_mgr_service_stub.cpp` | 227-231 |
| VerifySystemApp | `services/manager/src/ecologic_rule_mgr_service_stub.cpp` | 233-257 |
| Parcel 空指针检查 | `services/manager/src/ecologic_rule_mgr_service_stub.cpp` | 67-81 |
| MAX_WANT_SIZE | `services/manager/include/ecological_rule_mgr_service_stub.h` | 74 |
| MAX_ABILITY_INFO_SIZE | `services/manager/include/ecological_rule_mgr_service_stub.h` | 75 |
| Client 空包名检查 | `interfaces/innerkits/src/ecological_rule_mgr_service_client.cpp` | 106-110 |
| DeathRecipient | `interfaces/innerkits/src/ecological_rule_mgr_service_client.cpp` | 77-78 |

## 相关文档

- 架构设计: [02_Architecture.md](02_Architecture.md)
- Inner API: [03_Inner_API.md](03_Inner_API.md)
