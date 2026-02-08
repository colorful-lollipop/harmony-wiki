# mbedtls OpenHarmony 集成文档

## 库概述

**mbedtls** (现名 Mbed TLS) 是一个开源的、可移植的、易于使用的 SSL/TLS 库，在 OpenHarmony 系统中作为**核心加密安全基础设施**。

| 属性 | 值 |
|------|-----|
| **版本** | v3.6.5 |
| **许可证** | Apache License V2.0 |
| **上游地址** | https://www.trustedfirmware.org/projects/mbed-tls/ |
| **OH 组件** | @ohos/mbedtls |
| **适配系统** | mini, small, standard |

## OpenHarmony 适配特点

### 适配方式

mbedtls 在 OH 中的适配**未采用传统的 `.patch` 文件**，而是通过以下方式实现：

1. **完整 Port 适配层** (`port/` 目录)
2. **内核特定配置文件** (LiteOS-M/LiteOS-A)
3. **GN 构建系统深度集成**

### 核心适配组件

```
port/
├── include/
│   ├── tls_client.h      # TLS 客户端接口
│   ├── tls_certificate.h # 证书接口
│   └── mbedtls_log.h     # OH 日志适配
├── src/
│   ├── tls_client.c      # TLS 客户端实现
│   └── tls_certificate.c # 证书实现（含根证书）
└── config/
    ├── config_liteos_m.h  # LiteOS-M 配置
    ├── config_liteos_a.h  # LiteOS-A 配置
    ├── compat_posix/      # POSIX 兼容层
    └── compat_lwip/       # LwIP 兼容层
```

### 构建适配

- 支持静态库 (`mbedtls_static`)
- 支持共享库 (`mbedtls_shared`)
- 支持 NDK 库 (`mbedtls_ndk`)
- 支持多内核类型 (LiteOS-M/LiteOS-A/标准系统)

## 在 OpenHarmony 中的作用

mbedtls 是 OH 系统安全的**基石组件**，主要用途：

1. **通信加密**: HTTPS、TLS/DTLS 通信
2. **证书管理**: X.509 证书验证
3. **加密操作**: AES、RSA、ECC 等算法
4. **密钥管理**: 密钥生成与使用

## 核心依赖模块

| 模块 | 用途 |
|------|------|
| **dsoftbus** | 分布式软总线 TLS 加密 |
| **huks** | 密钥管理服务加密操作 |
| **appverify_lite** | 应用证书验证 |
| **curl** | HTTPS 客户端 |
| **hiviewdfx** | 安全日志记录 |

## 文档导航

请参阅 [SUMMARY.md](./SUMMARY.md) 获取详细阅读指南。

### 核心文档

- **[02_OH_Adaptation.md](./02_OH_Adaptation.md)**: OH 适配层详细分析
- **[03_Build_Integration.md](./03_Build_Integration.md)**: 构建系统适配
- **[04_Usage_in_OH.md](./04_Usage_in_OH.md)**: 依赖与使用场景

### 相关资源

- [bundle.json](../bundle.json): OH 组件配置
- [BUILD.gn](../BUILD.gn): 主构建配置
- [mbedtls.gni](../mbedtls.gni): 构建变量定义
