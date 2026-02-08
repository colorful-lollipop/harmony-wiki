# 安全风险分析

## 安全配置概述

OpenSSL 在 OpenHarmony 中的安全配置需要特别关注。根据适配分析，OpenHarmony 版本的 OpenSSL 启用了多项兼容旧系统的配置，这些配置虽然提高了与遗留系统的互操作性，但同时降低了默认的安全水位。本章节详细分析这些安全配置的影响，并提供针对性的建议。

当前安全配置的主要风险点包括：SECLEVEL=0 允许使用已知存在问题的弱加密算法；MinProtocol=None 不强制最低 TLS 版本；Legacy Provider 启用使得 DES、3DES、RC4 等不安全算法仍然可用；UnsafeLegacyRenegotiation 选项允许潜在的不安全重协商。这些配置共同构成了一个较低的默认安全基线，需要根据实际部署场景进行调整。

---

## 高风险配置项

### 安全级别（SECLEVEL=0）

**配置位置**：open_harmony_openssl_config/openssl.cnf

```
CipherString = DEFAULT:@SECLEVEL=0
```

**风险分析**：SECLEVEL 控制 OpenSSL 允许使用的加密算法最小强度。级别 0 是最低级别，允许使用：
- 40 位和 56 位对称加密（已知的弱算法）
- 导出级加密套件（设计时即为弱加密）
- 短密钥长度的非对称加密

**影响范围**：所有使用 OpenSSL TLS 功能的模块（curl、grpc、libwebsockets、wpa_supplicant、cups）都可能受此配置影响。

**建议**：对于面向消费者的设备，建议将 SECLEVEL 设置为 2（要求至少 80 位加密强度）或更高。设置方法为修改 openssl.cnf：
```
CipherString = DEFAULT:@SECLEVEL=2
```

### 无最低协议版本（MinProtocol=None）

**配置位置**：open_harmony_openssl_config/openssl.cnf

```
MinProtocol = None
```

**风险分析**：此配置不强制最低 TLS 版本，允许使用：
- SSLv3（已被 POODLE 攻击证明不安全）
- TLSv1.0（存在多种已知漏洞）
- TLSv1.1（也已过时）

**影响范围**：所有 TLS 连接可能协商到不安全的协议版本。

**建议**：将最低协议版本设置为 TLSv1.2 或 TLSv1.3：
```
MinProtocol = TLSv1.2
```

### Legacy Provider 启用

**配置位置**：open_harmony_openssl_config/openssl.cnf

```ini
[provider_sect]
default = default_sect
legacy = legacy_sect

[legacy_sect]
activate = 1
```

**风险分析**：Legacy Provider 包含以下已知不安全的算法：
- DES、3DES（56 位密钥，易受暴力攻击）
- RC4（已被证明不安全，禁用是 TLS 1.2+ 标准要求）
- Blowfish（密钥长度限制）
- IDEA（不再推荐使用）

**影响范围**：使用这些算法的应用可能受到相关攻击。

**建议**：在不需要兼容旧系统的情况下，禁用 Legacy Provider：
```ini
[provider_sect]
default = default_sect
# legacy = legacy_sect  # 注释掉此行
```

### 不安全重协商（UnsafeLegacyRenegotiation）

**配置位置**：open_harmony_openssl_config/openssl.cnf

```
Options = UnsafeLegacyRenegotiation
```

**风险分析**：TLS Renegotiation 功能曾存在 ClientHello 注入漏洞（CVE-2009-3555）。虽然 RFC 5746 修复了这一漏洞，但某些旧系统可能不支持修复后的安全重协商。UnsafeLegacyRenegotiation 选项允许与这些旧系统兼容，但可能受到降级攻击。

**影响范围**：所有 TLS 重协商操作。

**建议**：如果不需要与旧系统互操作，移除此选项：
```
# Options = UnsafeLegacyRenegotiation  # 注释掉此行
```

---

## OpenSSL 3.0.9 已知漏洞

### CVE 列表

OpenSSL 3.0.9（上游版本）包含以下已知安全修复：

| CVE 编号 | 严重程度 | 影响 | 是否已修复 |
|----------|----------|------|------------|
| CVE-2022-3786 | 中 | X.509 证书处理整数溢出 | 是（3.0.9） |
| CVE-2022-3602 | 高 | X.509 证书处理缓冲区溢出 | 是（3.0.9） |
| CVE-2022-1292 | 高 | 命令注入（c_rehash） | 是（3.0.9） |
| CVE-2022-2068 | 高 | 命令注入（CMS） | 是（3.0.9） |

**建议**：定期关注 OpenSSL 安全公告，及时进行版本升级。OpenSSL 安全公告发布地址：https://www.openssl.org/news/security/

### 漏洞响应流程

当上游发布安全更新时，OpenHarmony 集成团队应当：

1. **评估影响**：确认漏洞是否影响 OpenSSL 3.0.9 的已使用功能
2. **制定计划**：确定是否需要紧急热修复或纳入常规升级
3. **兼容性测试**：在所有支持平台上验证更新后的兼容性
4. **依赖模块测试**：验证主要依赖模块（curl、grpc 等）的功能正常
5. **安全验证**：确认漏洞利用场景已被阻断

---

## OH 特定安全考量

### 动态库加载风险

**当前配置**：ENGINESDIR="" 和 MODULESDIR="" 置空

此配置禁用了 OpenSSL 的动态引擎和模块加载功能，从安全角度看是正面的决策。动态加载机制曾被用于提权攻击（如心脏滴血漏洞的利用链），禁用该功能减小了攻击面。

**潜在风险**：禁用动态功能后，无法使用硬件安全模块（HSM）引擎等高级功能。

### 静态编译考虑

静态链接库（libcrypto_static、libssl_static）用于特定场景时，需要注意：

1. **符号可见性**：确保非导出符号不会被意外使用
2. **内存管理**：静态链接时更需注意正确清理敏感数据
3. **更新困难**：静态链接的应用需要重新编译才能获得安全修复

### liblegacy 禁用（LiteOS-A）

LiteOS-A 平台禁用了 liblegacy provider，这一决定虽然出于技术限制（缺少系统头文件），但客观上提高了该平台的安全性——LiteOS-A 设备无法使用不安全的 legacy 算法。

---

## 安全加固建议

### 配置文件加固

以下是一个针对较高安全要求的 openssl.cnf 配置示例：

```ini
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect
ssl_conf = ssl_conf_sect

[provider_sect]
default = default_sect
# legacy = legacy_sect  # 禁用 legacy provider

[default_sect]
activate = 1

[ssl_conf_system_default_sect]
# 移除 UnsafeLegacyRenegotiation 选项
# CipherString 设置较高的安全级别
CipherString = DEFAULT:@SECLEVEL=2
# 设置最低 TLS 版本
MinProtocol = TLSv1.2
```

### 应用层安全实践

使用 OpenSSL 的应用应当遵循以下安全实践：

1. **证书验证**：始终验证服务器证书的有效性，不跳过证书验证
2. **密码套件选择**：优先选择前向保密（Forward Secrecy）套件，如 ECDHE 系列
3. **密钥管理**：使用安全的随机数生成器，正确存储和销毁密钥材料
4. **错误处理**：不向用户暴露敏感的错误信息
5. **更新策略**：建立 OpenSSL 版本更新机制，及时响应安全公告

### 定期审计

建议定期进行以下安全审计：

| 审计项 | 频率 | 内容 |
|--------|------|------|
| 配置检查 | 季度 | 验证 openssl.cnf 配置符合安全策略 |
| 版本监控 | 持续 | 关注上游安全公告 |
| 依赖审查 | 半年 | 审查依赖模块的 OpenSSL 使用方式 |
| 渗透测试 | 每年 | 测试 OpenSSL 相关功能的漏洞 |

---

## 安全配置检查清单

### 生产环境部署前检查

- [ ] SECLEVEL 设置为 2 或更高
- [ ] MinProtocol 设置为 TLSv1.2 或 TLSv1.3
- [ ] 禁用 Legacy Provider（如不需要兼容旧系统）
- [ ] 移除 UnsafeLegacyRenegotiation 选项（如不需要兼容旧系统）
- [ ] 确认无使用已知不安全的加密套件
- [ ] 验证证书链验证功能正常工作
- [ ] 建立 OpenSSL 版本更新流程

### 持续监控

- [ ] 订阅 OpenSSL 安全公告
- [ ] 建立 CVE 响应流程
- [ ] 定期评估安全配置有效性
- [ ] 监控依赖模块的安全公告

---

## 安全相关链接

| 资源 | 链接 |
|------|------|
| OpenSSL 安全公告 | https://www.openssl.org/news/security/ |
| OpenSSL 文档 | https://www.openssl.org/docs/man3.0/ |
| OpenSSL 3.0 迁移指南 | https://www.openssl.org/docs/man3.0/man7/migration_guide.html |
| TLS 1.3 RFC | https://tools.ietf.org/html/rfc8446 |
| Mozilla TLS 指南 | https://wiki.mozilla.org/Security/Server_Side_TLS |

---

*文档版本：1.0*  
*创建日期：2026-02-08*  
*最后审核：2026-02-08*