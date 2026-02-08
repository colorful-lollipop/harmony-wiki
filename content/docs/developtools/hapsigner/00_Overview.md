# 项目概览

## 目的

本文档介绍 OpenHarmony **hapsigner** 签名工具的项目定位、核心能力和运行环境，帮助开发者快速理解项目全貌。

## 适用范围

- 想要了解 hapsigner 项目的开发者
- 需要集成签名功能的工具链开发者
- 负责应用签名/验签的运维人员

---

## 项目定位

### 功能定位

hapsigner 是 OpenHarmony 生态的**官方签名工具**，用于：

1. **密钥和证书管理** - 生成密钥对、证书签名请求(CSR)、各类证书
2. **应用签名** - 对 HAP(Harmony Ability Package)、HSP、HQF 等应用包进行数字签名
3. **Profile 签名** - 对应用分发配置文件进行签名
4. **二进制工具签名** - 对 ELF 可执行文件、bin 文件进行签名
5. **签名验证** - 验证应用、Profile、二进制文件的签名有效性
6. **代码签名** - 提供强制代码签名机制，确保运行时完整性

### 在 OpenHarmony 生态中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用开发流程                    │
├─────────────────────────────────────────────────────────────┤
│  应用开发 → 应用构建 → 【应用签名】→ 应用分发 → 应用安装 → 应用运行  │
│                              ↑                               │
│                         hapsigner                           │
└─────────────────────────────────────────────────────────────┘
```

### 版本信息

| 属性 | 值 |
|------|-----|
| 组件名称 | @ohos/hapsigner |
| 版本 | 3.1 (bundle.json) |
| 子系统 | developtools |
| 适配系统类型 | standard |

**代码证据**: `hapsigntool_cpp/bundle.json:2-5`

```json
{
    "name": "@ohos/hapsigner",
    "version": "3.1",
    "description": "hap包签名工具，支持.hsp、.hqf、.hap和.app等文件签名"
}
```

---

## 核心能力

### 1. 密钥与证书生成

| 功能 | 命令 | 说明 |
|------|------|------|
| 生成密钥对 | `generate-keypair` | 生成 RSA/ECC 密钥对并存储到 Keystore |
| 生成 CSR | `generate-csr` | 生成证书签名请求 |
| 生成 CA 证书 | `generate-ca` | 生成根 CA 或中间 CA 证书 |
| 生成应用证书 | `generate-app-cert` | 生成应用调试/发布证书 |
| 生成 Profile 证书 | `generate-profile-cert` | 生成 Profile 签名证书 |
| 生成通用证书 | `generate-cert` | 生成自定义用途证书 |

### 2. 签名功能

| 功能 | 命令 | 支持格式 |
|------|------|----------|
| 签名 Profile | `sign-profile` | JSON → p7b |
| 签名应用 | `sign-app` | zip(HAP/HSP/HQF)、elf、bin |
| 代码签名 | `--signCode=1` | 启用 fsverity 代码签名 |

### 3. 验证功能

| 功能 | 命令 | 说明 |
|------|------|------|
| 验证 Profile | `verify-profile` | 验证 Profile 签名并提取内容 |
| 验证应用 | `verify-app` | 验证 HAP/ELF/bin 签名 |

### 4. 签名模式

- **本地签名 (localSign)** - 使用本地 Keystore 进行签名
- **远程签名 (remoteSign)** - 调用远程签名服务
- **远程重签名 (remoteResign)** - 对已有签名进行重新签名

---

## 运行环境

### Java 版本 (hapsigntool)

| 要求 | 版本 |
|------|------|
| JDK | 8 或更高 |
| 构建工具 | Maven 3 |
| 依赖库 | BouncyCastle 1.79, Gson 2.11.0, Log4j 2.23.1 |

**代码证据**: `hapsigntool/pom.xml:13-14, 34-47`

```xml
<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
    <log4j-version>2.23.1</log4j-version>
</properties>

<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcpkix-jdk18on</artifactId>
    <version>1.79</version>
</dependency>
```

### C++ 版本 (hapsigntool_cpp / binary_sign_tool)

| 要求 | 版本/说明 |
|------|----------|
| C++ 标准 | C++17 |
| 编译器 | 支持 -std=c++17 |
| 关键依赖 | OpenSSL 3.x, cJSON, zlib, bzip2, elfio |
| 安全库 | bounds_checking_function |

**代码证据**: `hapsigntool_cpp/BUILD.gn:97-105`

```gn
cflags_cc = [
    "-std=c++17",
    "-fno-rtti",
]

deps = [
    "//third_party/bzip2:libbz2",
    "//third_party/openssl:libcrypto_shared",
    "//third_party/openssl:libssl_shared",
]

external_deps = [
    "c_utils:utils",
    "cJSON:cjson_static",
    "zlib:shared_libz",
]
```

### 一键签名脚本

| 要求 | 版本 |
|------|------|
| Python | 3.5 或更高 |
| 适用场景 | 开发调试阶段快速签名 |

---

## 关键概念

### 1. HAP (Harmony Ability Package)

OpenHarmony 应用包格式，本质是一个 ZIP 文件，包含：
- 应用代码和资源
- `module.json` 配置文件
- 签名块 (Signing Block)

### 2. Profile (分发配置文件)

控制应用分发和运行的配置文件，包含：
- 应用包名、版本信息
- 权限声明
- 设备类型限制
- 签名后存储为 p7b 格式

### 3. 代码签名 (Code Signing)

基于 fsverity 的强制代码签名机制：
- 使用 Merkle Tree 保护文件完整性
- 运行时内核验证签名
- 防止恶意代码篡改

### 4. 签名块 (Signing Block)

HAP 文件尾部的签名数据块，包含：
- 证书链
- 签名算法信息
- 代码签名数据（如启用）

---

## 多语言实现对比

| 特性 | Java (hapsigntool) | C++ (hapsigntool_cpp) | C++ (binary_sign_tool) |
|------|-------------------|----------------------|----------------------|
| 主要用途 | 开发工具/SDK | 系统工具链 | 轻量级二进制签名 |
| 构建方式 | Maven | GN/Ninja | GN/Ninja |
| 功能完整性 | 完整 | 完整 | 精简 |
| 代码签名 | 支持 | 支持 | 支持 |
| OpenSSL | 内置/JCA | 动态链接 | 静态链接 |
| 输出产物 | hap-sign-tool.jar | hap-sign-tool | binary-sign-tool |

---

## 相关链接

- [目录结构](01_Directory_Structure.md) - 了解代码组织
- [架构说明](02_Architecture.md) - 理解系统设计
- [API 参考](03_API_Reference.md) - 查看接口详情
