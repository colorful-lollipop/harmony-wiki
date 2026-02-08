# 项目概览

## 1. 项目定位与边界

### 1.1 定位

`code_signature` 是 OpenHarmony 安全子系统的代码签名组件，负责：
- 应用安装时的代码签名验证
- 运行时的代码完整性保护
- 本地代码签名服务
- 代码属性（Owner ID、XPM）管理

### 1.2 边界

**输入**：
- 待签名的 HAP 包或文件
- 签名文件（.sig）
- 证书/Profile 数据

**输出**：
- 代码签名启用状态
- 本地签名证书
- 签名后的代码文件
- 完整性验证结果

**依赖**：
- HUKS：密钥存储
- fsverity-utils：文件系统完整性工具
- OpenSSL：加密操作
- IPC 框架：服务间通信
- SELinux：安全策略

## 2. 目录结构

```
/base/security/code_signature
├── interfaces/inner_api/          # 接口层
│   ├── code_sign_utils/          # 代码签名启用 API
│   ├── code_sign_attr_utils/     # 代码签名属性设置 API
│   ├── local_code_sign/          # 本地代码签名 API
│   ├── jit_code_sign/            # JIT 代码签名 API
│   └── common/                   # 公共基础类型
├── services/                     # 服务层
│   ├── key_enable/               # 证书初始化服务 (Rust)
│   └── local_code_sign/          # 本地代码签名服务 (IPC SA)
├── utils/                        # 公共基础能力
│   ├── include/                  # 头文件
│   └── src/                      # 实现
├── figures/                      # 架构图
└── test/                         # 测试用例 (不计入本文档范围)
```

## 3. 核心能力详解

### 3.1 可信证书管理

证书管理由 `services/key_enable` 服务提供：

- **设备证书初始化**：加载并验证设备根证书
- **开发者证书信任**：`EnableKeyInProfile` / `RemoveKeyInProfile`
- **企业重签名证书**：`EnableKeyForEnterpriseResign` / `RemoveKeyForEnterpriseResign`

### 3.2 代码签名启用

`interfaces/inner_api/code_sign_utils` 提供以下能力：

- `EnforceCodeSignForApp`：对 HAP 包进行代码签名
- `EnforceCodeSignForFile`：对单个文件进行代码签名
- `EnforceCodeSignForAppWithOwnerId`：带 Owner ID 的签名
- `EnforceCodeSignForAppWithPluginId`：带 Plugin ID 的签名

### 3.3 本地代码签名

`interfaces/inner_api/local_code_sign` 提供本地签名能力：

- `InitLocalCertificate`：初始化本地签名证书
- `SignLocalCode`：对本地代码进行签名

服务实现：`services/local_code_sign` (SA ID: 3507)

### 3.4 代码属性设置

`interfaces/inner_api/code_sign_attr_utils` 提供 XPM 相关设置：

- `InitXpm`：初始化 XPM 区域、JIT Fortify、OwnerId 等
- `SetXpmOwnerId`：设置 Owner ID

### 3.5 JIT 代码签名

`interfaces/inner_api/jit_code_sign` 提供 JIT 代码签名能力：

- 基于 ARMv8.3-A PAC 机制的 JIT 代码签名
- `JitCodeSigner`：JIT 代码签名器
- `JitFortHelper`：JIT Fortify 辅助工具

## 4. 运行环境

### 4.1 系统能力要求

- **子系统**：security
- **系统类型**：standard
- **ROM 要求**：1024KB
- **RAM 要求**：2048KB

### 4.2 Feature Flags

| 开关 | 描述 |
|------|------|
| `code_signature_support_oh_code_sign` | OH SDK 代码签名支持 |
| `code_signature_enable_xpm_mode` | XPM 模式 (0-5) |
| `code_signature_support_oh_release_app` | Release 应用支持 |
| `code_signature_support_app_allow_list` | 应用白名单支持 |
| `code_signature_support_binary_enable` | Binary enable 支持 |

## 5. 关键概念

### 5.1 Owner ID

Owner ID 用于标识代码的所有者，支持多租户代码归属：

- `SYSTEM_LIB_ID`：系统库
- `DEBUG_LIB_ID`：调试库
- `APP`：应用
- `SHARED_LIB_ID`：共享库
- 等等...

### 5.2 fs-verity

Linux kernel 提供的文件完整性验证机制：

- Merkle Tree 哈希结构
- 内核 fs-verity keyring 集成
- 只读文件系统支持

### 5.3 Code Sign Block

ELF 文件中的签名块结构：

- 魔数验证
- 版本信息
- 分段信息
- 签名数据

## 6. 错误码体系

详见 [Inner API 参考](02_API_Reference.md#错误码)
