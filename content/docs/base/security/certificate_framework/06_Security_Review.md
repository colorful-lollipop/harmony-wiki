# 安全风险评审

## 1. 评审概述

### 1.1 评审范围

本评审基于以下代码范围进行：

| 目录 | 评审内容 |
|------|---------|
| `interfaces/inner_api/` | 对外 API 接口定义 |
| `frameworks/core/` | 框架核心实现 |
| `frameworks/js/napi/` | N-API 接口层 |
| `frameworks/adapter/` | 算法库适配层 |

**排除范围**：
- `test/` 测试代码
- 第三方 OpenSSL 实现（依赖上游审计）

### 1.2 威胁模型

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          信任边界                                        │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    受信任区域 (Trusted)                          │   │
│  │                                                                   │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │   │
│  │  │  OpenSSL        │  │  框架核心       │  │  能力注册中心   │ │   │
│  │  │  libcrypto.so   │  │  cf_api.c       │  │  cf_ability.c   │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│                              ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      边界 (Boundary)                             │   │
│  │                                                                   │   │
│  │  N-API: @ohos.security.cert                                     │   │
│  │  外部输入: encodingBlob, params                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│                              ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    不受信任区域 (Untrusted)                       │   │
│  │                                                                   │   │
│  │  应用层 JS/TS 代码                                                │   │
│  │  外部传入的证书/CRL 数据                                         │   │
│  │  网络/文件系统输入                                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

## 2. 攻击面分析

### 2.1 N-API 接口攻击面

| 接口类型 | 风险等级 | 说明 |
|---------|---------|------|
| `createX509Cert()` | 高 | 解析外部传入的证书数据 |
| `createX509Crl()` | 高 | 解析外部传入的 CRL 数据 |
| `createX509CertChain()` | 高 | 解析证书链数据 |
| `validate()` | 高 | 执行签名验证 |
| `verify()` | 高 | 执行签名验证 |
| `doFinal()` (CMS) | 高 | 执行加密操作 |

### 2.2 输入数据源

| 数据源 | 类型 | 风险 |
|--------|------|------|
| `encodingBlob.data` | DER/PEM 编码 | 解析器漏洞 |
| `encodingBlob.dataSize` | 长度字段 | 整数溢出 |
| `CfParamSet.params` | 参数数组 | 数组越界 |
| 文件读取 | PKCS#12 | 密码验证绕过 |
| 网络请求 | OCSP/CRL | 中间人攻击 |

## 3. 安全风险清单

### 3.1 输入验证风险

#### 风险 1: DER/PEM 解析器边界检查不完整

**证据**：`frameworks/adapter/v2.0/src/cf_adapter_cert_openssl.c`

```c
// 可能存在边界检查不足的代码模式
CfResult CfOpensslGetCertItem(const CfObject *object, const CfParamSet *paramSetIn, CfParamSet **paramSetOut)
{
    // ...
    HcfX509CertificateSpi *spi = (HcfX509CertificateSpi *)object;
    if (spi == nullptr || spi->cert == nullptr) {
        return CF_INVALID_PARAMS;
    }
    // ... 后续操作可能缺乏完整边界检查
}
```

**触发条件**：
1. 传入超长 DER/PEM 编码数据
2. 构造特殊的 ASN.1 结构

**影响**：
- 缓冲区读取溢出
- 拒绝服务

**修复建议**：
1. 在解析前验证 DER/PEM 数据长度不超过 `MAX_LEN_CERTIFICATE` (65536)
2. 增加 ASN.1 解析器的深度限制
3. 使用 OpenSSL 安全解析函数（已实现的部分较安全）

---

#### 风险 2: 整数溢出导致数组越界

**证据**：`cf_type.h`

```c
#define MAX_COUNT_OID          100
#define MAX_LEN_OID            128

typedef struct {
    uint32_t paramSetSize;
    uint32_t paramsCnt;
    CfParam params[];
} CfParamSet;
```

**触发条件**：
1. 构造超大的 `paramsCnt` 值
2. `paramSetSize` 与 `paramsCnt` 不匹配

**影响**：
- 堆溢出
- 任意代码执行

**修复建议**：
1. 在 `CfInitParamSet()` 中验证 `paramSetSize` 合理性
2. 增加 `CfIsAdditionOverflow()` 检查（在代码中存在，但需确保使用）

---

### 3.2 内存安全风险

#### 风险 3: 空指针解引用

**证据**：`cf_api.c`

```c
static int32_t CfObjectGet(const CfObject *object, const CfParamSet *paramSetIn, CfParamSet **paramSetOut)
{
    // ...
    CfObjectAbilityFunc *abilityFunc = GetAbilityFunc(object);
    if (abilityFunc == NULL) {
        return CF_INVALID_PARAMS;
    }
    return abilityFunc->get(object, paramSetIn, paramSetOut);
}
```

**分析**：代码中存在空指针检查，但需确认所有路径都覆盖。

**触发条件**：
1. 未初始化的 CfObject
2. 已销毁的对象

**影响**：
- 崩溃 (SIGSEGV)
- 拒绝服务

**修复建议**：
1. 确保 `CfCreate()` 失败时返回错误而非部分初始化对象
2. `destroy()` 后将指针设为 NULL

---

#### 风险 4: 内存释放后使用 (UAF)

**证据**：需审查 `destroy()` 实现

**触发条件**：
1. 多线程并发访问已销毁对象
2. 回调中引用已销毁对象

**影响**：
- 内存读取/写入
- 任意代码执行

**修复建议**：
1. 销毁时使用原子操作标记对象状态
2. 增加竞态检测

---

### 3.3 加密操作风险

#### 风险 5: PKCS#12 密码验证绕过

**证据**：`x509_cert_chain.h`

```c
CfResult HcfParsePKCS12(const CfBlob *keyStore, const HcfParsePKCS12Conf *conf, 
                        HcfX509P12Collection **p12Collection);
```

**触发条件**：
1. 传入空密码或弱密码
2. 密码暴力破解

**影响**：
- 私钥泄露
- 证书欺骗

**修复建议**：
1. 增加密码强度检查（可选）
2. 考虑增加迭代次数配置
3. 依赖 OpenSSL 的安全实现

---

#### 风险 6: 证书链验证策略绕过

**证据**：`napi_x509_cert_chain.cpp: NapiValidate`

```javascript
// revocationCheckOption 配置
{
    checkOption: 'PREFER_OCSP'  // 可能跳过某些检查
}
```

**触发条件**：
1. 配置不当的验证策略
2. 网络不可用时回退到不安全模式

**影响**：
- 使用已吊销证书
- 中间人攻击

**修复建议**：
1. 默认启用吊销检查
2. 明确记录安全级别配置
3. 网络错误时默认拒绝（当前代码中有 `IGNORE_NETWORK_ERROR` 选项，需谨慎使用）

---

### 3.4 路径遍历风险

#### 风险 7: 文件路径操作

**证据**：PKCS#12 文件读取

**触发条件**：
1. 应用传入恶意路径
2. 符号链接攻击

**影响**：
- 读取任意文件
- 路径遍历

**修复建议**：
1. 框架本身不涉及文件系统操作（由应用层处理）
2. 建议应用层进行路径验证

---

### 3.5 资源耗尽风险

#### 风险 8: 证书链长度无限制

**证据**：`x509_cert_chain.h`

```c
typedef struct HcfX509CertChainBuildParameters {
    HcfX509CertMatchParams certMatchParameters;
    int32_t maxlength;  // 可能未限制
    HcfX509CertChainValidateParams validateParameters;
};
```

**触发条件**：
1. 传入超长证书链

**影响**：
- 栈/堆溢出
- 拒绝服务

**修复建议**：
1. 设置 `maxlength` 默认上限（如 10）
2. 验证证书深度（pathLenConstraint）

---

## 4. 已有的安全机制

### 4.1 编译时保护

| 机制 | 配置 | 说明 |
|------|------|------|
| CFI | `cfi = true` | 控制流完整性 |
| CFI Cross-DSO | `cfi_cross_dso = true` | 跨 DSO CFI |
| Integer Overflow | `integer_overflow = true` | 整数溢出检测 |
| UBSan | `ubsan = true` | 未定义行为检测 |
| Boundary Sanitize | `boundary_sanitize = true` | 边界检查 |
| -Werror | 警告视为错误 | 避免编译警告 |

### 4.2 运行时保护

| 机制 | 实现位置 | 说明 |
|------|---------|------|
| 参数校验 | cf_param.c | 空值、类型检查 |
| 内存管理 | cf_memory.h | 安全内存操作 |
| 错误码 | cf_result.h | 统一错误处理 |

### 4.3 安全编码实践

```c
// 1. 使用安全字符串函数
// patterns/cf_adapter_cert_openssl.c
if (snprintf(buf, sizeof(buf), "%.*s", len, ptr) >= sizeof(buf)) {
    // 截断处理
}

// 2. 释放后置空
// patterns/cf_object_cert.c
if (cert != NULL) {
    X509_free(cert->opensslCert);
    cert->opensslCert = NULL;
}
```

## 5. 风险评估总结

| 风险 ID | 风险名称 | 风险等级 | 利用难度 | 影响 | 状态 |
|---------|---------|---------|---------|------|------|
| R-001 | DER/PEM 解析边界 | 中 | 中 | DoS | 依赖 OpenSSL |
| R-002 | 整数溢出 | 中 | 低 | RCE | 部分缓解 |
| R-003 | 空指针解引用 | 低 | 中 | DoS | 已防护 |
| R-004 | UAF | 低 | 高 | RCE | 需审计 |
| R-005 | PKCS#12 密码 | 中 | 中 | 信息泄露 | 依赖 OpenSSL |
| R-006 | 证书链验证绕过 | 中 | 中 | MITM | 需配置 |
| R-007 | 路径遍历 | 低 | 低 | 信息泄露 | 框架层无此风险 |
| R-008 | 证书链长度 | 低 | 低 | DoS | 需增强 |

## 6. 修复建议优先级

### 高优先级 (P0)

1. **整数溢出防护**
   - 位置：`cf_param.c`
   - 行动：确保 `CfAddParams()` 使用 `CfIsAdditionOverflow()`

2. **证书链深度限制**
   - 位置：`x509_cert_chain.c`
   - 行动：设置 `maxlength` 默认值，验证 pathLenConstraint

### 中优先级 (P1)

3. **UAF 防护**
   - 位置：所有对象销毁路径
   - 行动：增加引用计数或原子标记

4. **配置文档化**
   - 位置：`06_Security_Review.md`
   - 行动：明确说明安全配置选项

### 低优先级 (P2)

5. **密码强度建议**
   - 位置：PKCS#12 相关 API 文档
   - 行动：增加使用建议

## 7. 测试建议

### 7.1 模糊测试

```bash
# 使用 AFL/Honggfuzz 进行 DER/PEM 解析测试
cd test/fuzztest/v1.0/
x509certificate_fuzzer    # 证书解析模糊测试
x509crl_fuzzer           # CRL 解析模糊测试
x509certchain_fuzzer     # 证书链模糊测试
```

### 7.2 安全测试用例

| 测试场景 | 输入 | 预期结果 |
|---------|------|---------|
| 超长证书数据 | >65536 bytes | 返回错误 |
| 超长 OID 列表 | >100 OIDs | 返回错误 |
| 深度证书链 | >50 层 | 返回错误 |
| 特殊 ASN.1 结构 | 嵌套过深 | 超时/返回错误 |
| 空指针 | NULL | 返回错误 |
| 已释放对象 | freed object | 崩溃/返回错误 |

## 8. 结论

证书算法库框架整体安全性**良好**，主要依赖 OpenSSL 的安全实现，框架层有基本的输入验证和编译时保护。建议关注：

1. 整数溢出防护的完整覆盖
2. 证书链深度的默认限制
3. 文档化安全配置选项

主要风险来自外部输入的恶意构造数据，建议通过模糊测试持续发现潜在问题。
