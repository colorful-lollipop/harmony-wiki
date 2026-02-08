# 证书算法库框架 - 快速入门

## 项目定位

证书算法库框架是 OpenHarmony 安全子系统的核心组件，**屏蔽第三方证书算法库实现差异**，提供统一的证书操作能力。

### 核心能力

| 能力 | 描述 | 支持的格式 |
|------|------|-----------|
| **证书解析** | 解析 X.509 证书，提取字段信息 | DER, PEM |
| **证书扩展解析** | 解析证书扩展域段（OID） | DER, PEM |
| **CRL 解析** | 解析证书吊销列表 | DER, PEM |
| **证书链校验** | 验证证书签发关系，构建可信链 | DER, PEM |
| **CMS/PKCS7** | 生成和解析 CMS（PKCS7）签名数据 | DER, PEM |
| **PKCS12** | 解析和创建 PKCS#12 密钥库 | P12, PFX |

### 架构特点

```
┌─────────────────────────────────────────────────────────────┐
│                    API 接口层 (JS/N-API)                   │
│          security.cert (security.certificates)              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  框架实现层 (Core)                           │
│  对象管理 | 生命周期 | 能力注册 | 参数处理                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                算法库适配层 (Adapter)                        │
│          OpenSSL 实现 (v1.0 / v2.0)                        │
│          [Mbed TLS 待实现]                                  │
└─────────────────────────────────────────────────────────────┘
```

## 运行环境

| 要求 | 说明 |
|------|------|
| **系统类型** | standard（标准系统） |
| **依赖组件** | c_utils, crypto_framework, hilog, napi, openssl |
| **SysCap** | SystemCapability.Security.Cert |

## 快速开始

### 1. 引入模块

```javascript
import certificate from '@ohos.security.cert';

// 或使用命名空间
import { x509 } from '@ohos.security.cert';
```

### 2. 创建证书对象

```javascript
// 从 PEM 字符串创建 X509 证书
let pemCert = `-----BEGIN CERTIFICATE-----
MIIC...
-----END CERTIFICATE-----`;

let cert = certificate.createX509Cert(pemCert);
```

### 3. 获取证书字段

```javascript
// 同步方法示例
let version = cert.getVersion();
let serialNumber = cert.getSerialNumber();
let issuerName = cert.getIssuerName();

// 检查有效期
cert.checkValidityWithDate("20250101000000Z");
```

### 4. 证书链校验（异步）

```javascript
async function validateCertChain(anchors) {
    let chain = certificate.createX509CertChain();
    // ... 添加证书到链
    
    let result = await chain.validate({
        trustAnchors: anchors,
        revocationCheckOption: {
            checkOption: 'PREFER_OCSP'
        }
    });
    return result.isValid;
}
```

## 目录结构

```
certificate_framework/
├── bundle.json              # 部件配置
├── cf.gni                   # GN 编译配置
├── frameworks/
│   ├── ability/             # 能力注册中心
│   ├── adapter/             # 算法库适配层
│   │   ├── v1.0/           # OpenSSL v1.0 适配
│   │   ├── v2.0/           # OpenSSL v2.0 适配
│   │   └── attestation/    # 证书证明适配
│   ├── common/              # 公共工具
│   ├── core/                # 框架核心
│   │   ├── cert/           # 证书对象
│   │   ├── extension/      # 扩展对象
│   │   ├── life/           # 生命周期
│   │   ├── param/          # 参数处理
│   │   └── v1.0/           # v1.0 业务实现
│   └── js/
│       ├── ani/             # ANI 接口
│       └── napi/            # N-API 接口
├── interfaces/
│   └── inner_api/           # 内部 C API
└── test/                    # 测试用例
```

## 下一步

- **架构设计**：阅读 [01_Architecture.md](01_Architecture.md) 了解分层设计
- **API 使用**：查阅 [02_NAPI_Reference.md](02_NAPI_Reference.md) 获取完整 API 列表
- **构建配置**：查看 [04_Build_System.md](04_Build_System.md) 了解编译配置

## 常见问题

### Q: 支持哪些证书格式？
A: 支持 DER、PEM 编码格式，PKCS#7/CMS、PKCS#12 等容器格式。

### Q: 如何选择底层算法库？
A: 当前仅支持 OpenSSL。Mbed TLS 适配为预留设计，尚未实现。

### Q: 证书链校验支持哪些策略？
A: 支持 X.509 和 SSL 两种验证策略，可配置 OCSP、CRL 等吊销检查选项。
