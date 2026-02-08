# Code Signature - OpenHarmony 代码签名组件

## 项目简介

OpenHarmony `code_signature` 是安全子系统的核心组件，提供运行时代码签名验证和完整性保护能力，防止恶意代码在设备上执行，防止应用代码被攻击者篡改。

## 核心功能

| 功能 | 描述 |
|------|------|
| **可信证书管理** | 导入设备证书和本地代码签名证书，验证证书链及其可信来源 |
| **代码签名启用** | 提供用户态 API，在安装时启用应用或代码文件的代码签名 |
| **本地代码签名** | 在设备上运行签名服务，提供本地代码签名接口（如 AOT 生成的原生代码） |
| **代码属性设置** | 提供设置代码 owner ID 和初始化 XPM 区域的 API |

## 技术特性

- **fs-verity 集成**：使用 Linux kernel fs-verity 机制进行文件完整性验证
- **XPM (Executable Page Monitor)**：内存保护机制，用于代码完整性保护
- **SELinux**：安全策略和上下文管理
- **HUKS 集成**：通用密钥存储
- **JIT 代码签名**：基于 ARMv8.3-A Pointer Authentication (PAC) 的保护
- **Rust/C++ 混合编程**：关键路径使用 Rust 实现

## 快速链接

- [项目概览](01_Overview.md) - 详细功能说明
- [Inner API 参考](02_API_Reference.md) - API 接口文档
- [架构说明](03_Architecture.md) - 组件架构
- [GN 构建系统](04_Build_System.md) - 构建配置
- [安全风险评审](05_Security_Review.md) - 安全分析

## 相关仓库

- [developtools_hapsigner](https://gitee.com/openharmony/developtools_hapsigner)
- [kernel_linux_common_modules](https://gitee.com/openharmony/kernel_linux_common_modules)
- [third_party_fsverity-utils](https://gitee.com/openharmony/third_party_fsverity-utils)
