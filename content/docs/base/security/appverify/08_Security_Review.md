# 安全风险评审

## 目的

基于代码证据，分析 appverify 模块的安全风险、攻击面和潜在利用点。

## 适用范围

- 目标读者：安全审计人员、模块开发者
- 涵盖内容：威胁模型、安全机制、风险清单、修复建议

## 关键结论

1. **攻击面**：文件输入、配置文件、IPC 调用（非标准系统）
2. **安全机制**：证书验证、CRL 检查、完整性格式、边界检查
3. **风险等级**：中等（存在潜在利用点，但缓解措施完善）
4. **代码安全性**：启用 CFI、边界检查、整数溢出检查

## 威胁模型

### 数据流

```
外部输入
  ├─ HAP 文件路径
  ├─ HAP 文件内容（签名块、Profile）
  ├─ 配置文件（JSON）
  └─ IPC 调用参数（非标准系统）
      ↓
  输入校验
      ├─ 路径校验
      ├─ 大小检查
      ├─ 格式验证
      └─ 边界检查
      ↓
  处理逻辑
      ├─ PKCS7 解析
      ├─ 证书验证
      ├─ 签名验证
      └─ JSON 解析
      ↓
  安全检查
      ├─ 可信源匹配
      ├─ CRL 检查
      ├─ 设备授权
      └─ 完整性验证
      ↓
  输出
      └─ 验证结果 / 错误码
```

### 信任边界

```
[不可信区域]
  HAP 文件（用户提供）
  配置文件（可能被篡改）
  IPC 参数（调用方提供）
      ↓
[边界：输入校验]
  CheckFilePath()
  大小检查（< 2GB）
  OpenSSL DER 解码
  JSON 解析
      ↓
[可信区域]
  系统根证书
  可信源配置
  CRL 数据
      ↓
[边界：签名验证]
  PKCS7 签名验证
  证书链验证
  完整性验证
      ↓
[可信操作]
  允许/拒绝安装
  返回应用信息
```

---

## 安全机制

### 1. 证书验证

**证据位置**：hap_cert_verify_openssl_utils.cpp

**机制**：
- 证书链验证（X509_verify）
- 证书有效期检查
- CRL 吊销检查
- 根证书匹配

**代码证据**：
```cpp
// hap_cert_verify_openssl_utils.cpp:346
bool HapCertVerifyOpensslUtils::VerifyCertChainPeriodOfValidity(
    const CertChain& certsChain)
{
    // 检查每个证书的有效期
    for (const auto& cert : certsChain) {
        X509* x509Cert = cert;
        ASN1_TIME* notBefore = X509_get0_notBefore(x509Cert);
        ASN1_TIME* notAfter = X509_get0_notAfter(x509Cert);
        // 验证有效期
    }
    return true;
}
```

**缓解措施**：✅ 有效

---

### 2. CRL 检查

**证据位置**：hap_cert_verify_openssl_utils.cpp:420

**机制**：检查证书是否在吊销列表中

**代码证据**：
```cpp
// hap_cert_verify_openssl_utils.cpp:420
bool HapCertVerifyOpensslUtils::VerifyCrl(
    const std::string& certId,
    const Pkcs7Context& pkcs7Context)
{
    // 检查 CRL 列表
    HapCrlManager& hapCrlManager = HapCrlManager::GetInstance();
    if (hapCrlManager.IsCertRevoked(certId)) {
        return false;
    }
    return true;
}
```

**缓解措施**：✅ 有效

**局限性**：CRL 更新频率依赖系统配置

---

### 3. 完整性验证

**证据位置**：hap_signing_block_utils.cpp:428

**机制**：计算文件内容摘要，与签名中的摘要比对

**代码证据**：
```cpp
// hap_signing_block_utils.cpp:428
int32_t HapSigningBlockUtils::VerifyHapIntegrity(
    RandomAccessFile& hapFile,
    const HapSignatureBlock& hapSignatureBlock)
{
    // 计算文件内容摘要
    if (!HapVerifyOpensslUtils::DigestUpdate(...)) {
        return GET_DIGEST_FAIL;
    }

    // 比对摘要
    if (memcmp(calculatedDigest, signedDigest, digestLen) != 0) {
        return VERIFY_INTEGRITY_FAIL;
    }
    return VERIFY_SUCCESS;
}
```

**缓解措施**：✅ 有效

---

### 4. 边界检查

**编译选项**：
```gn
# libhapverify BUILD.gn:23-32
sanitize = {
  boundary_sanitize = true,    # 边界检查
  integer_overflow = true,    # 整数溢出检查
}
```

**使用库**：bounds_checking_function（libsec_shared）

**缓解措施**：✅ 有效

---

### 5. 控制流完整性 (CFI)

**编译选项**：
```gn
# libhapverify BUILD.gn:26-29
sanitize = {
  cfi = true,           # 控制流完整性
  cfi_cross_dso = true, # 跨 DSO CFI
}
```

**缓解措施**：✅ 有效（防止 ROP/JOP 攻击）

---

## 安全风险清单

### 风险 1：路径遍历（低风险）

**证据**：
- 文件位置：hap_verify_v2.cpp:82（CheckFilePath）
- 代码：
```cpp
bool HapVerifyV2::CheckFilePath(const std::string& filePath, ...)
{
    // 检查文件扩展名
    std::regex pattern(HAP_APP_PATTERN);
    if (!std::regex_match(filePath, pattern)) {
        return false;
    }
    // 后续直接使用 filePath 打开文件
}
```

**风险描述**：
- 未显式检查路径遍历字符（`../`）
- 但通过文件扩展名正则过滤
- 实际文件权限由 OS 控制

**利用路径**：
1. 攻击者构造恶意路径：`/path/to/../../../etc/passwd`
2. 但正则匹配失败（不符合 .hap/.hsp/.hqf/.app）
3. 无法利用

**影响**：❌ 无法利用（缓解措施有效）

**修复建议**：
```cpp
// 添加路径遍历检查
if (filePath.find("..") != std::string::npos) {
    HAPVERIFY_LOG_ERROR("Path traversal detected");
    return false;
}
```

---

### 风险 2：JSON 解析注入（中风险）

**证据**：
- 文件位置：json_parser_utils.cpp
- 使用库：cJSON
- 代码：
```cpp
bool JsonParserUtils::ParseJson(const std::string& jsonStr, cJSON*& root)
{
    root = cJSON_Parse(jsonStr.c_str());
    if (root == nullptr) {
        return false;
    }
    return true;
}
```

**风险描述**：
- 配置文件（trusted_*.json）使用 JSON 格式
- cJSON 库解析
- 但配置文件由系统安装，由 root 管理员控制

**利用路径**：
1. 攻击者需要 root 权限篡改 /system/etc/security/
2. 有 root 权限可直接破坏系统，无需此漏洞

**影响**：⚠️ 需要提升权限，实际风险低

**缓解措施**：
- 配置文件路径：/system/etc/security/（仅 root 可写）
- 使用 cJSON 库（防护解析漏洞）

**修复建议**：
- 当前缓解措施已足够
- 可添加配置文件签名验证（防止篡改）

---

### 风险 3：整数溢出（低风险）

**证据**：
- 编译选项：integer_overflow = true（启用 UBSan）
- 代码证据：hap_signing_block_utils.cpp
```cpp
// hap_signing_block_utils.cpp:600
uint32_t chunkSize = endOffset - startOffset;
// 可能溢出（但启用了整数溢出检查）
```

**风险描述**：
- 文件大小和偏移计算可能溢出
- 但编译时启用 UBSan 检测

**利用路径**：
1. 构造畸形 HAP 文件（超大偏移）
2. 触发整数溢出
3. 可能导致越界访问

**影响**：⚠️ 理论可能，但缓解措施有效

**缓解措施**：
- integer_overflow = true（编译时检查）
- boundary_sanitize = true（运行时检查）

**修复建议**：
```cpp
// 添加显式检查
if (endOffset < startOffset) {
    return FILE_SIZE_TOO_LARGE;
}
uint32_t chunkSize = endOffset - startOffset;
if (chunkSize > MAX_CHUNK_SIZE) {
    return FILE_SIZE_TOO_LARGE;
}
```

---

### 风险 4：DoS 攻击（大文件）（中风险）

**证据**：
- 文件位置：hap_verify_v2.cpp:82
- 代码：
```cpp
// 文件大小检查
if (hapFile.GetFileSize() > MAX_FILE_SIZE) {  // MAX_FILE_SIZE = 2GB
    return FILE_SIZE_TOO_LARGE;
}
```

**风险描述**：
- 最大文件限制为 2GB
- 攻击者可发送接近 2GB 的文件
- 导致内存耗尽和 CPU 占用

**利用路径**：
1. 攻击者上传 2GB HAP 文件
2. 系统尝试加载和验证
3. 内存和 CPU 被耗尽

**影响**：⚠️ 拒绝服务

**缓解措施**：
- 文件大小限制（2GB）
- 建议：降低到 500MB（应用实际大小）

**修复建议**：
```cpp
// 降低最大文件大小
const uint32_t MAX_FILE_SIZE = 500 * 1024 * 1024;  // 500MB
```

---

### 风险 5：信息泄露（错误消息）（低风险）

**证据**：
- 文件位置：hap_verify_openssl_utils.cpp
- 代码：
```cpp
// hap_verify_openssl_utils.cpp:619
void HapVerifyOpensslUtils::GetOpensslErrorMessage()
{
    unsigned long err = ERR_get_error();
    char err_msg[OPENSSL_ERR_MESSAGE_MAX_LEN];
    ERR_error_string_n(err, err_msg, sizeof(err_msg));
    HAPVERIFY_LOG_ERROR("OpenSSL error: %s", err_msg);
}
```

**风险描述**：
- OpenSSL 错误消息可能泄露内部状态
- 但日志仅输出到 Hilog，需要访问权限

**利用路径**：
1. 攻击者构造畸形签名
2. 触发 OpenSSL 错误
3. 通过 Hilog 获取错误信息

**影响**：❌ 需要日志访问权限

**缓解措施**：
- Hilog 权限控制（普通应用无法访问系统日志）
- 错误消息不返回给调用方

**修复建议**：
- 生产环境可禁用详细错误消息
- 仅记录错误码，不记录详细消息

---

### 风险 6：竞争条件（低风险）

**证据**：
- 文件位置：hap_verify.cpp:34
- 代码：
```cpp
static std::mutex g_mtx;  // 全局互斥锁

bool HapVerifyInit()
{
    g_mtx.lock();
    g_isInit = rootCertsObj.Init() && trustedAppSourceManager.Init();
    g_mtx.unlock();
    return g_isInit;
}
```

**风险描述**：
- 全局初始化使用互斥锁保护
- 但单例模式可能有其他竞争条件

**利用路径**：
1. 多线程同时调用 HapVerify()
2. 可能触发 TOCTOU（Time-of-Check-Time-of-Use）

**影响**：❌ 无明显利用点

**缓解措施**：
- 全局互斥锁保护初始化
- HapVerifyV2 实例无共享状态

**修复建议**：
- 当前缓解措施已足够

---

### 风险 7：IPC 注入（低风险，仅非标准系统）

**证据**：
- 文件位置：device_type_manager.cpp, provision_verify.cpp, ticket_verify.cpp
- 依赖：ipc:ipc_core, os_account:libaccountkits
- 代码：
```cpp
// device_type_manager.cpp (推测，基于 IPC 依赖)
// 通过 IPC 获取设备类型
```

**风险描述**：
- 非标准系统通过 IPC 调用其他服务
- IPC 消息可能被篡改

**利用路径**：
1. 攻击者拦截或篡改 IPC 消息
2. 伪造设备类型或账号信息
3. 绕过某些检查

**影响**：❌ 需要系统级权限

**缓解措施**：
- OpenHarmony IPC 机制本身有权限控制
- Binder 驱动验证消息来源

**修复建议**：
- 验证 IPC 返回值的有效性
- 添加消息完整性校验

---

## 代码安全特性评估

### 编译时安全

| 特性 | 启用 | 说明 |
|------|--------|------|
| 边界检查 | ✅ | boundary_sanitize |
| 整数溢出检查 | ✅ | integer_overflow |
| 未定义行为检查 | ✅ | ubsan |
| 控制流完整性 | ✅ | cfi, cfi_cross_dso |
| 返回地址保护 | ✅ | branch_protector_ret = "pac_ret" |
| 符号隐藏 | ✅ | -fvisibility=hidden |

### 运行时安全

| 特性 | 启用 | 说明 |
|------|--------|------|
| 全局状态互斥锁 | ✅ | g_mtx 保护 |
| 单例模式 | ✅ | 防止多实例 |
| 文件大小限制 | ✅ | 2GB 上限 |
| 证书有效期检查 | ✅ | VerifyCertChainPeriodOfValidity |
| CRL 检查 | ✅ | VerifyCrl |
| 完整性验证 | ✅ | VerifyHapIntegrity |

---

## 检查范围与局限性

### 已检查范围

✅ **已覆盖**：
- 输入校验（文件路径、大小）
- PKCS7 解析与验证
- 证书链验证（有效期、CRL）
- 签名验证（RSA-PSS、ECDSA、DSA）
- 完整性验证
- JSON 配置解析
- 全局状态线程安全
- 编译时安全特性

### 未覆盖范围

❌ **未深入分析**：
- OpenSSL 内部实现（依赖 OpenSSL 项目安全）
- mbedtls 内部实现（Lite 系统）
- IPC 驱动层（OpenHarmony 系统安全）
- 文件系统权限（OS 层）
- HAP 文件格式设计（签名工具侧）

---

## 安全建议

### 短期改进

1. **降低最大文件大小**
   - 当前：2GB
   - 建议：500MB（应用实际大小）
   - 位置：hap_verify_v2.cpp

2. **添加路径遍历检查**
   - 检查 `../` 字符
   - 位置：hap_verify_v2.cpp:82

3. **配置文件签名验证**
   - 为 trusted_*.json 添加签名
   - 验证签名完整性
   - 防止配置篡改

### 长期改进

1. **引入 OCSP**
   - 替代或补充 CRL
   - 实时证书状态查询

2. **硬件加速签名验证**
   - 使用 TEE 环境验证签名
   - 提高安全级别

3. **审计日志**
   - 记录所有验证操作
   - 支持安全审计

---

## 合规性

### 安全标准

| 标准 | 符合情况 | 说明 |
|------|----------|------|
| NIST SP 800-57 | ✅ | 支持安全哈希（SHA256/384/512） |
| FIPS 140-2 | ⚠️ | 依赖 OpenSSL（需 FIPS 认证） |
| Common Criteria | ⚠️ | 未评估（需第三方评估） |

### 加密算法

| 算法 | 安全性 | 说明 |
|------|--------|------|
| SHA256 | ✅ 强 | 推荐使用 |
| SHA384 | ✅ 强 | 推荐使用 |
| SHA512 | ✅ 强 | 推荐使用 |
| RSA-PSS | ✅ 强 | 推荐使用 |
| ECDSA | ✅ 强 | 推荐使用 |
| RSA-PKCS1v1.5 | ⚠️ 较弱 | 仅兼容性支持 |

---

## 相关跳转

- [架构详解](03_Architecture.md) - 安全机制流程
- [对外 API](04_Public_API.md) - API 安全使用
- [常见问题](09_FAQ.md) - 安全相关排查
