# 架构说明

## 1. 整体架构

证书算法库框架采用**三层分层架构**，从上层到底层依次为：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        API 接口层 (API Layer)                            │
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │
│  │    N-API        │  │     ANI         │  │    FFI/CJ       │        │
│  │  (JavaScript)   │  │  (ArkTS/N-API)  │  │  (C/FFI)        │        │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘        │
│                                                                          │
│  导出模块: security.cert, security.certificates                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     框架实现层 (Framework Layer)                         │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    对象生命周期管理 (life/)                       │   │
│  │  CfCreate() → 对象创建 → get/check 方法 → destroy() 销毁        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    能力注册中心 (ability/)                       │   │
│  │  RegisterAbility() / GetAbility()                               │   │
│  │  支持: CF_ABILITY_TYPE_ADAPTER, CF_ABILITY_TYPE_OBJECT          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                       证书业务模块 (v1.0/)                       │   │
│  │                                                                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │   │
│  │  │ certificate/ │  │    crl/     │  │   cert_chain/          │ │   │
│  │  │             │  │             │  │                         │ │   │
│  │  │ X509Certificate│ │  X509Crl    │  │  CertChainValidator    │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      SPI 接口层 (SPI/)                           │   │
│  │  定义适配器需要实现的接口规范                                    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     算法库适配层 (Adapter Layer)                          │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      v2.0 (新架构 Ability 模式)                 │   │
│  │                                                                   │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │   │
│  │  │ cf_adapter_     │  │ cf_adapter_     │  │ cf_adapter_     │ │   │
│  │  │ cert_openssl   │  │ extension_      │  │ attestation_    │ │   │
│  │  │                 │  │ openssl          │  │                 │ │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      v1.0 (旧架构 SPI 模式)                      │   │
│  │  直接通过 SpiCreate 函数创建适配器对象                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        第三方算法库                              │   │
│  │                    OpenSSL (当前支持)                           │   │
│  │              Mbed TLS (预留，尚未实现)                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

## 2. 数据流

### 证书解析数据流

```
JS API (createX509Cert)
       │
       ▼
N-API 层 (napi_certificate_init.cpp)
       │
       ▼
框架核心 (cf_api.c: CfCreate)
       │
       ▼
能力注册中心 (cf_ability.c: GetAbility)
       │
       ▼
适配器层 (cf_adapter_cert_openssl.c)
       │
       ▼
OpenSSL X509 接口
```

### 证书链校验数据流

```
JS API (certChain.validate)
       │
       ▼
N-API 层 (napi_x509_cert_chain.cpp: NapiValidate)
       │
       ▼
证书链验证器 (cert_chain_validator.cpp)
       │
       ├──► 证书有效性检查 ──► OpenSSL 签名验证
       │
       ├──► 吊销状态检查 ──► CRL/OCSP
       │
       └──► 信任锚验证 ──► 根证书匹配
```

## 3. 线程模型

### 线程划分

| 线程 | 职责 | 说明 |
|------|------|------|
| **主线程 (ArkTS)** | JS 调用入口 | 调用 N-API 入口点 |
| **N-API 线程** | 异步回调处理 | 执行 Promise/Callback 完成回调 |
| **Worker 线程** | 耗时操作 | 证书解析、链校验等 CPU 密集型操作 |

### 异步操作流程

```javascript
// Promise 模式
async function example() {
    let cert = await certificate.createX509Cert(pemData);
    // 内部:
    // 1. JS 调用 N-API
    // 2. 创建 napi_async_work
    // 3. napi_queue_async_work 放入线程池
    // 4. 执行完成回调返回主线程
    // 5. resolve Promise
}
```

## 4. 关键时序

### 对象创建时序

```
participant JS as "JavaScript"
participant NAPI as "N-API Layer"
participant Core as "Framework Core"
participant Ability as "Ability Center"
participant Adapter as "Adapter Layer"
participant OpenSSL as "OpenSSL"

JS->>NAPI: createX509Cert(pemData)
NAPI->>NAPI: 创建 AsyncContext
NAPI->>Core: CfCreate(CF_OBJ_TYPE_CERT, ...)
Core->>Ability: GetAbility(CF_ABILITY_TYPE_ADAPTER, ...)
Ability-->>Core: 返回适配器函数表
Core->>Adapter: CfOpensslCreateCert(...)
Adapter->>OpenSSL: X509_new()
OpenSSL-->>Adapter: X509*
Adapter-->>Core: HcfX509CertificateImpl*
Core-->>NAPI: CfObject*
NAPI-->>JS: X509Cert 实例
```

## 5. 模块依赖关系

### 依赖方向图

```
interfaces/inner_api  (对外接口定义)
    │
    ▼
frameworks/core       (核心实现)
    │
    ├──► frameworks/ability  (能力注册)
    │
    ├──► frameworks/common   (公共工具)
    │
    └──► frameworks/adapter  (适配器依赖)
            │
            ▼
        OpenSSL
```

### 具体依赖

| 模块 | 依赖 | 被依赖 |
|------|------|--------|
| core/life | ability, common, adapter | js/napi, js/ani |
| core/cert | life, common | core/v1.0 |
| core/extension | life, common | core/v1.0 |
| core/v1.0 | cert, extension, life | - |
| js/napi | core | - |
| js/ani | core | - |
| adapter/v2.0 | openssl | core |

## 6. 版本演进

### v1.0 vs v2.0 架构对比

| 特性 | v1.0 (SPI) | v2.0 (Ability) |
|------|------------|----------------|
| **创建方式** | SpiCreate() 显式调用 | RegisterAbility() 自动注册 |
| **适配器发现** | 编译时静态链接 | 运行时动态加载 |
| **灵活性** | 较低 | 较高 |
| **多算法库** | 需要条件编译 | 运行时可选 |
| **当前状态** | 已废弃 | 正在使用 |

### 迁移状态

| 模块 | 迁移状态 | 说明 |
|------|---------|------|
| 证书 | ✅ v2.0 | cf_adapter_cert_openssl.c |
| 扩展 | ✅ v2.0 | cf_adapter_extension_openssl.c |
| 证书链 | 🔄 迁移中 | 部分迁移 |
| CRL | ⏳ 待迁移 | 仍用 v1.0 |
| CMS | ⏳ 待迁移 | 仍用 v1.0 |

## 7. 设计模式

### 能力注册模式 (Ability Pattern)

```c
// 适配器注册 (cf_adapter_ability.c)
__attribute__((constructor)) static void LoadAdapterAbility(void)
{
    RegisterAbility(CF_ABILITY(CF_ABILITY_TYPE_ADAPTER, CF_OBJ_TYPE_CERT), 
                    &g_certAdapterFunc.base);
}

// 框架使用
CfCertAdapterAbilityFunc *func = (CfCertAdapterAbilityFunc *)GetAbility(
    CF_ABILITY_TYPE_ADAPTER, CF_OBJ_TYPE_CERT);
func->adapterCreate(...);
```

### SPI 模式 (旧架构)

```c
// 框架定义接口 (x509_certificate_spi.h)
struct HcfX509CertificateSpi {
    CfResult (*parseCert)(...);
    CfResult (*verifySignature)(...);
};

// 适配器实现 (x509_certificate_openssl.c)
CfResult OpensslX509CertSpiCreate(const CfEncodingBlob *inStream, 
                                   HcfX509CertificateSpi **spi)
{
    // OpenSSL 实现
}
```

## 8. 关键文件索引

| 功能 | 关键文件 |
|------|---------|
| 模块注册 | `frameworks/js/napi/certificate/src/napi_certificate_init.cpp:443-455` |
| 能力注册中心 | `frameworks/ability/inc/cf_ability.h` |
| 核心 API | `frameworks/core/life/cf_api.c` |
| OpenSSL 适配 | `frameworks/adapter/v2.0/src/cf_adapter_cert_openssl.c` |
| 类型定义 | `interfaces/inner_api/include/cf_type.h` |
