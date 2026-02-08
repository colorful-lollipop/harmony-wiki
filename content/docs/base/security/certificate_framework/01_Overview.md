# 项目概览

**证书算法库框架 (Certificate Framework)** 是 OpenHarmony 安全子系统的核心组件之一，为应用提供统一的证书处理能力。本文档帮助开发者快速理解项目定位、能力边界和使用方式。

## 1. 项目定位

### 一句话定义

证书算法库框架是一个**屏蔽第三方证书算法库实现差异的统一接口框架**，为上层应用提供证书、证书扩展域段、证书吊销列表（CRL）的解析及校验能力，同时支持证书链的整体校验。

**证据来源**: `README.md:4`

> 证书算法库框架是一个屏蔽了第三方证书算法库实现差异的算法框架，提供证书、证书扩展域段、证书吊销列表的解析及校验能力，此外还提供了证书链的校验能力。开发者可以通过调用证书算法库框架接口，忽略底层不同三方算法库的差异，实现迅捷开发。

### 核心价值

| 价值维度 | 说明 |
|---------|------|
| **接口统一** | 提供统一的 JS/N-API 接口，屏蔽 OpenSSL 与 Mbed TLS 的差异 |
| **安全隔离** | 作为应用与底层加密库的中间层，提供输入验证和内存安全保护 |
| **能力扩展** | 通过 SPI 架构支持扩展不同的加密算法后端 |

### 系统能力声明

**证据来源**: `bundle.json:17`

```json
"syscap": [ "SystemCapability.Security.Cert" ]
```

该系统能力声明了框架在设备证书管理中的安全级别定位。

---

## 2. 能力边界

### 支持的能力

| 能力类别 | 具体功能 |
|---------|---------|
| **证书解析** | 版本号、序列号、颁发者、主题、签名算法、公钥、有效期等字段提取 |
| **证书扩展域段** | OID 列表获取、扩展数据解析、关键扩展识别 |
| **CRL 操作** | 被吊销证书查询、吊销原因和时间获取 |
| **证书链校验** | 签发关系验证、有效期校验、信任链构建 |

**证据来源**: `README.md:13-17`

### 不支持的能力

| 能力 | 说明 |
|------|------|
| **证书签发** | 不支持生成新证书 |
| **私钥操作** | 不提供私钥生成和管理功能 |
| **PKI 基础设施** | 不支持 CA 运营相关功能 |
| **实时撤销检查** | OCSP 查询能力有限，主要依赖 CRL |

### 证书格式支持

| 格式 | 支持状态 | 证据来源 |
|------|---------|----------|
| X.509 v3 | ✅ 支持 | SPI 接口定义 |
| DER 编码 | ✅ 支持 | `cf_type.h:161` |
| PEM 编码 | ✅ 支持 | `cf_type.h:160` |
| PKCS#12 | ✅ 部分支持 | CMS 生成器接口 |
| PKCS#7 | ✅ 部分支持 | CMS 生成器接口 |

---

## 3. 运行环境

### 系统依赖

**证据来源**: `bundle.json:22-30`

| 依赖组件 | 用途 | 依赖类型 |
|---------|------|----------|
| `c_utils` | C 语言基础工具库 | 必需 |
| `crypto_framework` | 加密框架集成 | 必需 |
| `hilog` | 日志系统 | 必需 |
| `napi` | Node-API 接口层 | 必需 |
| `openssl` | 底层加密算法库 | 必需 |
| `runtime_core` | 运行时核心 | 必需 |

### 运行平台

- **目标系统**: OpenHarmony Standard (标准版)
- **系统类型**: `standard` 仅支持，不支持 Lite 版本

**证据来源**: `bundle.json:19`

```json
"adapted_system_type": [ "standard" ]
```

### 资源消耗

| 资源类型 | 限制值 | 说明 |
|---------|--------|------|
| 运行时内存 | ≤ 5MB | `MAX_MEMORY_SIZE` 限制 |
| 证书数据 | ≤ 64KB | `MAX_LEN_CERTIFICATE` 限制 |
| 扩展数据 | ≤ 64KB | `MAX_LEN_EXTENSIONS` 限制 |
| OID 数量 | ≤ 100 | `MAX_COUNT_OID` 限制 |
| OID 长度 | ≤ 128 | `MAX_LEN_OID` 限制 |

**证据来源**: `cf_type.h:238-243`, `cf_memory.h:29`

---

## 4. 快速开始

### 前置条件

1. 已配置 OpenHarmony 开发环境
2. 获取源码并完成基础编译
3. 了解 ArkTS/JS 应用开发基础

### 最小使用示例

#### 创建 X.509 证书对象

```typescript
import { x509Cert } from '@ohos/security.cert';

// 方法1: 从 PEM 字符串创建
let pemCert = '-----BEGIN CERTIFICATE-----\n...';
let cert = x509Cert.createX509Cert(pemCert);

// 方法2: 从 DER 二进制创建
let derCert: Uint8Array = ...;
let cert2 = x509Cert.createX509Cert(derCert);

// 获取证书字段
let version = cert.getVersion();        // 版本号
let serialNumber = cert.getSerialNumber(); // 序列号
let issuerName = cert.getIssuer();       // 颁发者
let subjectName = cert.getSubject();     // 主题
let notBefore = cert.getNotBeforeTime(); // 生效时间
let notAfter = cert.getNotAfterTime();   // 过期时间
```

#### 证书链验证

```typescript
import { certChainValidator } from '@ohos.security.cert';

// 验证证书链
let chain = x509CertChain.createX509CertChain([rootCert, intermediateCert, leafCert]);
let result = await certChainValidator.validate(chain, {
    date: '2024-01-01T00:00:00Z',
    revocationCheck: 'prefer_crl'
});

if (result.isValid) {
    console.log('证书链验证通过');
} else {
    console.log('验证失败:', result.getFailReason());
}
```

### 错误处理

**证据来源**: `cf_result.h`

| 错误码 | 值 | 处理建议 |
|--------|-----|----------|
| `CF_SUCCESS` | 0 | 成功，无需处理 |
| `CF_INVALID_PARAMS` | -10001 | 检查输入参数有效性 |
| `CF_ERR_CERT_NOT_YET_VALID` | -30003 | 证书尚未生效，等待或检查日期 |
| `CF_ERR_CERT_HAS_EXPIRED` | -30004 | 证书已过期，需要 |
| `CF更新证书_ERR_CERT_SIGNATURE_FAILURE` | -30002 | 签名验证失败，证书可能被篡改 |

---

## 5. 架构概览

### 三层架构设计

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      API 接口层 (API Layer)                              │
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │
│  │    N-API        │  │     ANI         │  │    FFI/CJ       │        │
│  │  (JavaScript)   │  │  (ArkTS/N-API)  │  │  (C/FFI)        │        │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘        │
│                                                                          │
│  导出模块: security.cert                                                 │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     框架实现层 (Framework Layer)                          │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    对象生命周期管理 (life/)                       │   │
│  │  CfCreate() → 对象创建 → get/check 方法 → destroy() 销毁        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    能力注册中心 (ability/)                       │   │
│  │  RegisterAbility() / GetAbility()                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                       证书业务模块 (v1.0/)                        │   │
│  │  certificate/ │ crl/ │ cert_chain/                              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     算法库适配层 (Adapter Layer)                          │
│                                                                          │
│  ┌──────────────────────────────┐  ┌──────────────────────────────┐   │
│  │         v2.0 (新架构)         │  │        v1.0 (旧架构)          │   │
│  │  CfAdapterCertOpenssl        │  │  X509CertificateOpenssl      │   │
│  │  CfAdapterExtensionOpenssl   │  │  X509CertChainOpenssl        │   │
│  └──────────────────────────────┘  └──────────────────────────────┘   │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                         第三方算法库                               │   │
│  │                     OpenSSL (当前支持)                            │   │
│  │               Mbed TLS (预留，尚未实现)                           │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

**证据来源**: `README.md:11-19`, `01_Architecture.md`

### 关键设计决策

| 决策点 | 选择 | 理由 |
|--------|------|------|
| **入口模式** | 单入口 CfCreate() | 集中控制对象创建，便于安全审计 |
| **适配模式** | SPI + Ability 双模式 | v1.0 兼容 SPI，v2.0 支持热插拔 |
| **加密库** | OpenSSL 优先 | 成熟稳定，社区支持广泛 |
| **内存管理** | CfMalloc/CfFree | 统一内存管理，支持安全擦除 |

---

## 6. 安全特性

### 输入验证体系

框架实现了多层次的输入验证机制：

| 验证层级 | 函数 | 检查内容 |
|---------|------|----------|
| **Blob 验证** | `CfCheckBlob()` | 指针有效性、尺寸范围 |
| **编码验证** | `CfCheckEncodingBlob()` | 编码格式合法性 |
| **字符串验证** | `CfIsStrValid()` | 字符串长度限制 |
| **URL 验证** | `CfIsUrlValid()` | 协议前缀检查 |

**证据来源**: `frameworks/common/v1.0/src/cf_check.c`

```cpp
// Blob 验证示例
int32_t CfCheckBlob(const CfBlob *blob, uint32_t maxLen)
{
    if ((blob == NULL) || (blob->data == NULL) || (blob->size == 0) || (blob->size > maxLen)) {
        CF_LOG_E("invalid input params");
        return CF_INVALID_PARAMS;
    }
    return CF_SUCCESS;
}
```

### 内存安全

| 特性 | 实现 | 证据来源 |
|------|------|----------|
| **安全擦除** | `CfBlobDataClearAndFree()` | `cf_blob.c:53` |
| **尺寸限制** | MAX_MEMORY_SIZE (5MB) | `cf_memory.h:29` |
| **包装器** | CfMalloc/CfFree | `cf_memory.h:25-27` |

### 安全加固编译选项

**证据来源**: `BUILD.gn` sanitizer 配置

```gn
sanitize = {
  cfi = true;                    // 控制流完整性
  cfi_cross_dso = true;          // 跨 DSO CFI
  boundary_sanitize = true;      // 边界检查
  integer_overflow = true;       // 整数溢出检测
  ubsan = true;                  // 未定义行为检测
}
```

---

## 7. 相关资源

### 官方文档

| 资源 | 链接 |
|------|------|
| 接口文档 | [JS APIs - Certificate](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-device-certificate-kit/js-apis-cert.md) |
| 开发指导 | [DeviceCertificateKit 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/DeviceCertificateKit/) |
| 安全子系统 | [OpenHarmony 安全子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/安全子系统.md) |

### 源码仓库

| 仓库 | 说明 |
|------|------|
| [security_certificate_framework](https://gitee.com/openharmony/security_certificate_framework) | 本仓库 |
| [security_crypto_framework](https://gitee.com/openharmony/security_crypto_framework) | 加密框架（相关依赖） |

---

## 8. 贡献指南

### 文档反馈

如发现文档错误或需要补充内容，请：

1. 在 [Gitee Issues](https://gitee.com/openharmony/security_certificate_framework/issues) 提交问题
2. 或提交 PR 修改文档

### 代码贡献

1. Fork 本仓库
2. 创建特性分支: `git checkout -b feature/new-feature`
3. 提交代码: `git commit -m "feat: add new feature"`
4. 推送分支: `git push origin feature/new-feature`
5. 创建 Pull Request

---

*最后更新: 2025-02-07*
*基于代码版本: certificate_framework v4.0*
