# 安全风险评审

## 1. 威胁模型概述

### 1.1 系统边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Trust Boundary                                   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                     Trusted Zone (Code Sign)                         ││
│  │                                                                     ││
│  │  - key_enable (Rust 服务)                                           ││
│  │  - local_code_sign (SA:3507)                                        ││
│  │  - utils/* (签名工具)                                               ││
│  │                                                                     ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                          │
│  ▲                                    │                                 │
│  │ External Input                      │                                 │
│  │                                    ▼                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      Untrusted Zone                                  ││
│  │                                                                     ││
│  │  - HAP/Native 应用                                                 ││
│  │  - 文件系统输入                                                     ││
│  │  - IPC 消息                                                         ││
│  │  - Profile/证书数据                                                 ││
│  │                                                                     ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面清单

| 攻击面 | 类型 | 描述 |
|--------|------|------|
| **文件系统 API** | 输入验证 | `EnforceCodeSignForApp`, `SignLocalCode` 接收文件路径 |
| **IPC 接口** | 权限绕过 | LocalCodeSign SA 暴露的 IPC 方法 |
| **Profile 数据** | 注入/解析 | `EnableKeyInProfile` 接收的 Profile 缓冲区 |
| **证书数据** | 证书伪造 | `EnableKeyForEnterpriseResign` 接收的证书 |
| **Owner ID** | 注入 | `SetXpmOwnerId`, `SignLocalCode` 接收的 Owner ID |
| **Plugin ID** | 注入 | `EnforceCodeSignForAppWithPluginId` 接收的 Plugin ID |

## 2. 信任边界

### 2.1 数据流与信任边界

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│   App        │────►│  Interface Layer │────►│  Service Layer      │
│ (Untrusted)  │     │                  │     │  (Trusted)          │
└──────────────┘     └──────────────────┘     └─────────────────────┘
                            │                            │
                            │ Boundary Check             │ SA Permission
                            ▼                            │
                     ┌──────────────────┐               │
                     │  Input Validation│               │
                     │  & Sanitization  │               │
                     └──────────────────┘               │
                                                          │
                        ┌───────────────────────────────┘
                        │
                        ▼
                ┌──────────────────┐
                │  Kernel/System   │
                │  (fs-verity)     │
                └──────────────────┘
```

### 2.2 关键信任点

| 组件 | 信任级别 | 说明 |
|------|----------|------|
| IPC 调用者 | 不可信 | 需要 `PermissionUtils` 验证 |
| 文件路径 | 不可信 | 需要路径规范化检查 |
| Profile 数据 | 不可信 | 需要格式验证和签名验证 |
| 证书数据 | 不可信 | 需要证书链验证 |
| Owner ID | 不可信 | 需要长度和格式检查 |

## 3. 安全风险分析

### 3.1 输入验证风险

#### 风险 1：路径遍历攻击

**证据**：`services/local_code_sign/src/local_code_sign_service.cpp:130`

```cpp
if (!OHOS::PathToRealPath(filePath, realPath)) {
    LOG_INFO("Get real path failed, path = %{public}s", filePath.c_str());
    return CS_ERR_FILE_PATH;
}
```

**触发条件**：
- 攻击者调用 `SignLocalCode` 传入包含 `../` 的恶意路径
- `PathToRealPath` 未能正确规范化路径

**影响**：
- 可能签名非预期文件
- 可能绕过访问控制

**修复建议**：
- 确认 `PathToRealPath` 是否进行完整的路径规范化
- 添加白名单路径检查
- 记录审计日志

**状态**：待确认 - 需要验证 `PathToRealPath` 的具体实现

---

#### 风险 2：Owner ID 缓冲区溢出

**证据**：`services/local_code_sign/src/local_code_sign_service.cpp:124`

```cpp
if (ownerID.length() > MAX_OWNER_ID_LEN) {
    LOG_ERROR("ownerID len %{public}zu should not exceed %{public}u", ownerID.length(), MAX_OWNER_ID_LEN);
    return CS_ERR_INVALID_OWNER_ID;
}
```

**评估**：✅ **已防护**
- 长度检查在签名生成前执行
- 使用 `memcpy_s` 进行安全内存操作

**建议**：无需额外修复

---

#### 风险 3：证书链解析整数溢出

**证据**：`utils/src/cert_utils.cpp:43-56`

```cpp
struct HksCertChain **certChain = static_cast<struct HksCertChain *>(malloc(sizeof(struct HksCertChain)));
// ...
(*certChain)->certs = static_cast<struct HksBlob *>(malloc(sizeof(struct HksBlob) *
    ((*certChain)->certsCount)));
```

**评估**：⚠️ **低风险**
- 未对 `certsCount` 进行整数溢出检查
- 但 `HksCertChain` 结构由 HUKS 提供

**修复建议**：
- 在计算内存分配前添加 `certsCount` 范围检查
- 使用 `malloc_s` 或检查 `malloc` 返回值

---

### 3.2 IPC 安全风险

#### 风险 4：IPC 消息序列化边界

**证据**：`interfaces/inner_api/local_code_sign/include/local_code_sign_proxy.h`

```cpp
int32_t InitLocalCertificate(const ByteBuffer &challenge, ByteBuffer &cert) override;
int32_t SignLocalCode(const std::string &ownerID, const std::string &filePath, ByteBuffer &signature) override;
```

**评估**：✅ **已防护**
- 使用 `MessageParcel` 进行序列化
- Binder 框架自动进行大小限制

**建议**：无需额外修复

---

#### 风险 5：调用者权限验证

**证据**：`services/local_code_sign/src/permission_utils.cpp:33-51`

```cpp
bool PermissionUtils::IsValidCallerOfCert()
{
    AccessToken::AccessTokenID callerTokenId = IPCSkeleton::GetCallingTokenID();
    if (VerifyCallingProcess(CERTIFICATE_CALLERS, callerTokenId)) {
        return true;
    }
    ReportInvalidCaller("Cert", callerTokenId);
    return false;
}
```

**评估**：✅ **已防护**
- 使用 `AccessTokenKit` 进行调用者验证
- 白名单机制：`CERTIFICATE_CALLERS = {"key_enable"}`, `SIGN_CALLERS = {"compiler_service"}`

**建议**：无需额外修复

---

### 3.3 内存安全风险

#### 风险 6：ByteBuffer 内存管理

**证据**：`interfaces/inner_api/common/include/byte_buffer.h:46-53`

```cpp
~ByteBuffer()
{
    if (data != nullptr) {
        data.reset(nullptr);
        data = nullptr;
    }
    size = 0;
}
```

**评估**：✅ **已防护**
- 使用 `std::unique_ptr` 自动管理内存
- `CopyFrom` 和 `PutData` 进行边界检查

**建议**：无需额外修复

---

#### 风险 7：证书数据缓冲区溢出

**证据**：`utils/src/cert_utils.cpp:91-100`

```cpp
buffer.Resize(totalLen);
if (!buffer.PutData(0, CastToUint8Ptr(&certsCount), sizeof(uint32_t))) {
    return false;
}
// ...后续 PutData 调用
```

**评估**：✅ **已防护**
- `buffer.Resize` 确保缓冲区足够大
- 所有 `PutData` 调用在 Resize 后执行

**建议**：无需额外修复

---

### 3.4 加密操作风险

#### 风险 8：PKCS7 签名生成

**证据**：`utils/src/pkcs7_generator.cpp`

**评估**：✅ **已防护**
- 使用 OpenSSL 进行加密操作
- HUKS 进行密钥管理

**建议**：无需额外修复

---

### 3.5 竞态条件风险

#### 风险 9：SA 延迟卸载竞态

**证据**：`services/local_code_sign/src/local_code_sign_service.cpp:78-97`

```cpp
void LocalCodeSignService::DelayUnloadTask()
{
    std::lock_guard<std::mutex> lock(unloadMutex_);
    // ... 180秒延迟卸载
}
```

**评估**：⚠️ **低风险**
- 使用 `unloadMutex_` 保护
- 但竞态窗口仍然存在（获取锁后释放前）

**修复建议**：
- 添加更细粒度的状态检查
- 使用原子操作进行状态管理

---

### 3.6 SELinux 风险

#### 风险 10：SELinux 上下文配置

**证据**：`services/local_code_sign/local_code_sign.cfg`

```json
"secon": "u:r:local_code_sign:s0"
```

**评估**：✅ **已配置**
- 配置了 SELinux 上下文
- 但需要确认策略是否完整

**建议**：
- 审查 SELinux 策略文件
- 确保最小权限原则

---

## 4. 已确认的安全机制

### 4.1 内存安全

| 机制 | 证据 | 状态 |
|------|------|------|
| 安全的内存复制 | `memcpy_s` | ✅ 已使用 |
| 缓冲区边界检查 | `ByteBuffer::Resize`, `PutData` | ✅ 已使用 |
| 自动内存管理 | `std::unique_ptr` | ✅ 已使用 |
| 整数溢出检查 | (部分) `certsCount` | ⚠️ 待完善 |

### 4.2 输入验证

| 机制 | 证据 | 状态 |
|------|------|------|
| 路径规范化 | `PathToRealPath` | ⚠️ 待验证 |
| 长度检查 | `MAX_OWNER_ID_LEN` | ✅ 已使用 |
| 空指针检查 | `data == nullptr` | ✅ 已使用 |

### 4.3 权限控制

| 机制 | 证据 | 状态 |
|------|------|------|
| IPC 调用者验证 | `AccessTokenKit` | ✅ 已使用 |
| 白名单机制 | `CERTIFICATE_CALLERS`, `SIGN_CALLERS` | ✅ 已使用 |
| SELinux 上下文 | `local_code_sign.cfg` | ✅ 已配置 |

### 4.4 加密安全

| 机制 | 证据 | 状态 |
|------|------|------|
| HUKS 密钥管理 | `HUKS API` | ✅ 已使用 |
| OpenSSL 操作 | `PKCS7` | ✅ 已使用 |
| fs-verity 集成 | `fsverity-utils` | ✅ 已使用 |

## 5. 安全建议

### 5.1 高优先级

1. **验证路径规范化实现**
   - 确认 `PathToRealPath` 正确处理所有路径遍历攻击
   - 添加显式的路径白名单检查

2. **完善整数溢出检查**
   - 在 `ConstructDataToCertChain` 中添加 `certsCount` 范围检查
   - 使用安全的内存分配函数

### 5.2 中优先级

3. **增强 SA 卸载安全性**
   - 使用原子操作管理 SA 状态
   - 添加卸载前的状态验证

4. **审计 SELinux 策略**
   - 确认 `u:r:local_code_sign:s0` 上下文策略完整
   - 最小权限原则检查

### 5.3 低优先级

5. **添加安全审计日志**
   - 记录所有敏感操作
   - 集成 HISYSEVENT 进行安全事件上报

## 6. 总结

### 6.1 已识别风险数量

| 风险等级 | 数量 | 状态 |
|----------|------|------|
| 高风险 | 0 | 未发现 |
| 中风险 | 0 | 未发现 |
| 低风险 | 3 | 需关注 |
| 已防护 | 7 | 无需额外动作 |

### 6.2 总体评估

**评估结论**：`code_signature` 组件整体安全性较好

**主要优势**：
- 完善的内存安全机制（`memcpy_s`, `unique_ptr`）
- 严格的 IPC 调用者权限控制
- 集成 HUKS 进行密钥管理
- 使用 fs-verity 进行文件完整性保护

**待改进**：
- 确认路径规范化的完整性
- 完善整数溢出检查
- 增强 SA 状态管理

### 6.3 检查范围

本文档覆盖的安全检查范围：
- ✅ 文件系统 API（`SignLocalCode`, `EnforceCodeSignForApp`）
- ✅ IPC 接口（`LocalCodeSignInterface`）
- ✅ 证书/Profile 数据解析
- ✅ 内存管理（`ByteBuffer`, `cert_utils`）
- ✅ 权限控制（`PermissionUtils`）
- ✅ SELinux 配置

**未覆盖范围**：
- ❌ `ParseOwnerIdFromSignature` 函数（未使用）
- ❌ Rust 代码详细审计（`key_enable`）
- ❌ fs-verity 内核实现
- ❌ HUKS 内部实现
