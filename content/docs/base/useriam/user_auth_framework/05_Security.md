# 安全风险评审

## 1. 概述

本文档对 user_auth_framework 进行安全风险评审，基于代码扫描结果识别潜在攻击面和风险点。

**评审范围**:
- N-API 接口输入校验
- IPC 通信安全
- 凭证存储与传输
- 权限验证机制
- 认证结果处理

**证据来源**: `services/core/src/ipc_common.cpp`, `hisysevent.yaml`, IDL 接口定义

---

## 2. 攻击面清单

| 攻击面 | 类型 | 入口 | 风险等级 |
|--------|------|------|----------|
| **N-API auth()** | JS 接口 | `user_auth_entry.cpp` | 高 |
| **N-API verifyAuthToken()** | JS 接口 | `user_access_ctrl_entry.cpp` | 高 |
| **IPC UserAuthService** | SA 接口 | `IUserAuth.idl` | 高 |
| **IPC UserIdmService** | SA 接口 | `IUserIdm.idl` | 高 |
| **IPC CoAuthService** | SA 接口 | `ICoAuth.idl` | 中 |
| **HDI 接口** | 驱动接口 | `IAuthDriverHDI` | 高 |

---

## 3. 信任边界

```
┌──────────────────────────────────────────────────────────────┐
│                        TEE/安全世界                          │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  IAuthDriverHDI 实现 (厂商提供)                        │  │
│  │  - 凭证存储                                            │  │
│  │  - 生物特征模板                                        │  │
│  │  - 加密/解密操作                                       │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
                              ▲
                              │ HDI IPC
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                      用户态 (Normal World)                     │
│                                                              │
│  ┌──────────────────────┐      ┌──────────────────────────┐ │
│  │  Native Client       │      │  N-API (JS/ArkTS)        │ │
│  │  - UserAuthClient    │ ───▶ │  - auth()               │ │
│  │  - UserIdmClient     │      │  - verifyAuthToken()     │ │
│  └──────────────────────┘      └──────────────────────────┘ │
│              │                          │                     │
│              ▼                          ▼                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  IPC Layer                                              │  │
│  │  - UserAuthService (SA 901)                            │  │
│  │  - UserIdmService (SA 921)                              │  │
│  │  - CoAuthService (SA 931)                               │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. 风险点与修复建议

### 4.1 高风险

#### 风险 1: N-API 参数校验不完整

**证据**:
```
文件: frameworks/js/napi/user_auth/src/user_auth_entry.cpp
问题: auth() 函数接收的 challenge 为 BigInt，authType 为 number
      缺少对 authType 范围的严格校验
```

**触发条件**:
```typescript
// 传入无效的 authType 值
userAuth.auth(0n, 999 /* 无效值 */, AuthTrustLevel.ATL1, callback);
```

**影响**:
- 可能导致认证逻辑异常
- 信任等级检查失效

**修复建议**:
```cpp
// 在 C++ 层增加 authType 范围校验
if (authType < AUTH_TYPE_PIN || authType > AUTH_TYPE_PRIVATE_PIN) {
    IAM_LOGE("Invalid authType: %{public}d", authType);
    return napi_invalid_arg;
}
```

**状态**: 需确认 - 当前实现可能已在 Native 层校验

---

#### 风险 2: Challenge 重放攻击

**证据**:
```
文件: frameworks/native/client/
问题: challenge 值由调用方传入，未验证其随机性
```

**触发条件**:
```typescript
// 使用固定值作为 challenge
userAuth.auth(0n, AuthType.FACE, AuthTrustLevel.ATL2, callback);
// 或
userAuth.auth(12345n, AuthType.FACE, AuthTrustLevel.ATL2, callback);
```

**影响**:
- 重放攻击：攻击者可以重放历史认证请求
- 中间人攻击：在不安全的通道中可能被拦截

**修复建议**:
```cpp
// 建议由框架生成随机 challenge，而非由调用方传入
// 或要求 challenge 必须满足一定随机性要求
uint64_t GenerateSecureChallenge() {
    // 使用安全随机数生成器
    return SecRandom::GenerateRandom64();
}
```

**状态**: 设计决策 - 当前架构允许调用方传入 challenge

---

#### 风险 3: Token 验证逻辑

**证据**:
```
文件: frameworks/js/napi/user_access_ctrl/src/user_access_ctrl_entry.cpp
函数: VerifyAuthToken
```

**触发条件**:
```typescript
// 传入伪造或过期的 token
let result = userAccessCtrl.verifyAuthToken(forgedToken);
```

**影响**:
- 未授权访问
- 权限提升

**修复建议**:
```cpp
// 1. 验证 token 签名
// 2. 验证 token 未过期
// 3. 验证 token 颁发者
int32_t VerifyAuthToken(const uint8_t *token, uint32_t tokenLen) {
    // Token 格式: signature | timestamp | userId | authType
    if (tokenLen < MIN_TOKEN_SIZE) {
        return AUTH_TOKEN_INVALID;
    }
    // 验证时间戳未过期
    uint64_t timestamp = ExtractTimestamp(token);
    if (IsExpired(timestamp)) {
        return AUTH_TOKEN_EXPIRED;
    }
    // 验证签名
    if (!VerifySignature(token, tokenLen)) {
        return AUTH_TOKEN_INVALID_SIGNATURE;
    }
    return SUCCESS;
}
```

---

### 4.2 中风险

#### 风险 4: IPC 权限检查覆盖不全

**证据**:
```
文件: services/core/src/ipc_common.cpp
实现: AccessTokenKit 权限检查
```

**说明**:
- 已在 IPC 层实现 `CheckPermission` 调用
- 但部分内部接口可能未覆盖

**修复建议**:
- 审查所有 IPC 接口的权限检查覆盖情况
- 对敏感操作 (如 DelUser) 增加二次确认

---

#### 风险 5: 远程认证设备信任

**证据**:
```
文件: services/remote_connect/
功能: SoftBus 跨设备认证
```

**说明**:
- 远程认证依赖设备间信任关系
- 未验证的设备可能发起认证请求

**修复建议**:
- 验证远程设备的证书链
- 限制可参与远程认证的设备白名单

---

### 4.3 低风险

#### 风险 6: 日志信息泄露

**证据**:
```
文件: common/logs/iam_logger.h
```

**说明**:
- Debug 日志可能泄露敏感信息 (userId, token 等)

**修复建议**:
```cpp
// 使用隐私日志格式
IAM_LOGI("Auth result for user %{public}d", userId);
// 改为
IAM_LOGI("Auth result for user [MASKED]");
```

---

## 5. 安全事件监控

### 5.1 HiSysEvent 安全事件

**证据**: `hisysevent.yaml`

| 事件 | 类型 | 级别 | 说明 |
|------|------|------|------|
| `USERIAM_CREDENTIAL_CHANGE` | SECURITY | CRITICAL | 凭证变更 |
| `USERIAM_TEMPLATE_CHANGE` | SECURITY | CRITICAL | 模板变更 |
| `USERIAM_USER_AUTH_FWK` | SECURITY | CRITICAL | 认证安全事件 |
| `USERIAM_REMOTE_*` | SECURITY | CRITICAL | 远程操作 |

### 5.2 监控建议

建议监控以下异常模式:

- 短时间内大量认证失败
- 非活跃用户的认证请求
- 跨设备的异常认证模式

---

## 6. 最佳实践

### 6.1 开发者注意事项

1. **Challenge 管理**: 使用框架生成的随机 challenge
2. **Token 安全**: 不要硬编码或长期存储 token
3. **权限检查**: 确保调用者有足够权限
4. **错误处理**: 不要在错误信息中泄露敏感数据

### 6.2 集成注意事项

1. **TEE 实现**: 认证方案和结果评估必须在 TEE 中实现
2. **凭证存储**: 生物特征模板必须存储在安全区域
3. **密钥管理**: 加密密钥必须安全存储

---

## 7. 安全等级定义

### 7.1 AuthTrustLevel

| 等级 | 说明 | 使用场景 |
|------|------|----------|
| ATL1 | 最低 | 敏感度低的操作 |
| ATL2 | 中等 | 一般敏感操作 |
| ATL3 | 较高 | 敏感操作 |
| ATL4 | 最高 | 高价值操作 |

### 7.2 ExecutorSecureLevel

| 等级 | 说明 |
|------|------|
| ESL0 | 无安全要求 |
| ESL1 | 安全环境 |
| ESL2 | 安全环境 + 硬件根信任 |
| ESL3 | 安全环境 + 硬件根信任 + 安全存储 |

---

## 8. 相关文档

- [架构说明](01_Architecture.md)
- [N-API 接口](02_NAPI.md)
- [构建配置](04_Build.md)
