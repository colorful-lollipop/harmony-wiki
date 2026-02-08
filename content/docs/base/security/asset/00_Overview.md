# 项目概述

## 目的

本文档介绍 ASSET（Asset Store Service）项目的整体定位、边界、核心能力、运行环境和关键概念。

## 适用范围

- 面向对象：ASSET 服务开发者、集成者、安全审计人员
- 涵盖内容：项目定位、核心能力、运行环境、关键概念
- 不含：测试代码、外部依赖的内部实现（如 HUKS、UserIAM）

---

## 项目定位

### 核心定位

ASSET（Asset Store Service）是 OpenHarmony 系统中的**关键资产存储服务**，提供用户短敏感数据（<1024 字节）的安全存储和管理能力。

**证据**：`bundle.json:17-19`
```json
{
  "name": "@ohos/asset",
  "description": "The asset store service (ASSET) provides secure storage and management of sensitive data.",
  "subsystem": "security",
  "component": {
    "name": "asset",
    "syscap": [
      "SystemCapability.Security.Asset"
    ]
  }
}
```

### 边界

**ASSET 服务的边界**：
- **存储数据类型**：短敏感数据（密码、令牌、银行卡号等）
- **数据大小限制**：< 1024 字节
- **加密提供者**：HUKS（通用密钥库系统）
- **认证提供者**：UserIAM（统一用户认证）
- **系统类型**：standard（标准系统）

**证据**：`README_zh.md:3-6, README.md:4-10`

---

## 核心能力

### 1. 数据安全存储

ASSET 为应用提供敏感数据的安全存储能力：
- **加密存储**：使用 AES-256-GCM 算法加密数据
- **密钥隔离**：每个应用拥有独立的加密密钥
- **TEE 保护**：加密/解密操作在 TEE（可信执行环境）中完成

**证据**：`README_zh.md:13-16`
```
新增关键资产，ASSET 首先为应用生成独属于它的密钥，
然后使用该密钥对关键资产进行加密，最后将关键资产密文存储到数据库。
```

### 2. 数据管理操作

支持完整的 CRUD 操作：
- **新增 (Add)**：添加新资产
- **查询 (Query)**：根据条件查询资产（支持分页、排序）
- **更新 (Update)**：更新现有资产
- **删除 (Remove)**：删除符合条件的资产

**证据**：`README_zh.md:11-16, README.md:14-21`

### 3. 访问控制

支持两种访问控制模式：

**基础访问控制**：
- 通过锁屏状态限制访问（`DEVICE_POWERED_ON`、`DEVICE_FIRST_UNLOCKED`、`DEVICE_UNLOCKED`）

**增强访问控制**（可选）：
- **条件编译**：`asset_access_control_enabled = true`
- **用户认证**：读取资产前需通过用户身份认证（PIN、指纹、人脸）
- **UserIAM 集成**：调用统一用户认证服务拉起认证界面

**证据**：`README_zh.md:20-21`
```
ASSET 支持应用存储需要用户身份认证通过才允许访问的关键资产。
应用在读取此类关键资产时，需要先拉起统一用户认证服务...
```

### 4. 跨用户/跨设备同步

支持资产的多用户和跨设备同步：
- **多用户支持**：每个用户独立的数据空间
- **同步类型**：NEVER、THIS_DEVICE、TRUSTED_DEVICE、TRUSTED_ACCOUNT
- **组访问**：支持 HAP 间共享资产（通过 group_id）

**证据**：`interfaces/kits/c/inc/asset_type.h:78-144`（ASSET_TAG_SYNC_TYPE 定义）

### 5. 权限验证

严格的权限检查机制：
- **STORE_PERSISTENT_DATA**：存储持久化数据需要此权限
- **INTERACT_ACROSS_LOCAL_ACCOUNTS**：跨用户操作需要此权限
- **系统应用验证**：部分操作仅允许系统应用调用

**证据**：`services/db_operator/src/common/permission_check.rs:23-41`

### 6. 插件扩展

支持通过插件扩展服务能力：
- **备份/恢复**：支持数据备份和恢复扩展
- **RSS 扩展**：资源调度系统扩展

**证据**：`sa_profile/8100.json:33`，`interfaces/inner_kits/plugin_interface/src/plugin_interface.rs`

---

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **操作系统** | OpenHarmony 4.1+ |
| **子系统** | security |
| **系统类型** | standard（标准系统） |
| **依赖服务** | HUKS、UserIAM、AccessToken、BundleManager 等 |
| **硬件要求** | 支持 TEE（可信执行环境） |

**证据**：`bundle.json:19-21, 27-29`

### 资源消耗

| 资源 | 大小 | 说明 |
|------|------|------|
| **ROM** | 5120KB | 静态存储占用 |
| **RAM** | 4828KB | 运行时内存占用 |

**证据**：`bundle.json:34-35`

---

## 关键概念

### 1. 资产 (Asset)

用户敏感数据的加密存储单元，包含：
- **Secret（敏感数据）**：密码、令牌、银行卡号等（必填）
- **Alias（别名）**：资产的唯一标识符（必填）
- **Accessibility（可访问性）**：锁屏状态要求
- **Sync Type（同步类型）**：同步范围
- **Data Labels（数据标签）**：自定义字段（Critical/Normal）

**证据**：`interfaces/kits/c/inc/asset_type.h:78-150`

### 2. 认证类型

| 类型 | 说明 | HUKS 常量 |
|------|------|-----------|
| NONE | 无需认证 | HKS_USER_AUTH_TYPE_NONE |
| ANY | 任意认证方式 | HKS_USER_AUTH_TYPE_ANY |
| PIN | PIN 码 | HKS_USER_AUTH_TYPE_PIN |
| FINGERPRINT | 指纹 | HKS_USER_AUTH_TYPE_FINGERPRINT |
| FACE | 人脸 | HKS_USER_AUTH_TYPE_FACE |

**证据**：`interfaces/kits/c/inc/asset_type.h:94-111`

### 3. 可访问性级别

| 级别 | 说明 | HKS 存储级别 |
|------|------|-------------|
| DEVICE_POWERED_ON | 设备开机即可访问 | HKS_AUTH_STORAGE_LEVEL_DE |
| DEVICE_FIRST_UNLOCKED | 首次解锁后可访问 | HKS_AUTH_STORAGE_LEVEL_CE |
| DEVICE_UNLOCKED | 设备解锁时才可访问 | HKS_AUTH_STORAGE_LEVEL_ECE |

**证据**：`interfaces/kits/c/inc/asset_type.h:89-93`

### 4. 系统能力 (System Ability)

ASSET 作为 System Ability 运行：
- **SA ID**：8100
- **SA 名称**：`security_asset_service`
- **进程名**：`asset_service`
- **启动方式**：按需启动（on-demand）

**证据**：`sa_profile/8100.json:4-6`，`frameworks/ipc/src/lib.rs:25-27`

### 5. IPC 通信

使用 OpenHarmony Rust IPC 框架：
- **客户端**：通过 `SystemAbilityManager` 加载 SA
- **服务端**：实现 `RemoteStub` trait 处理请求
- **序列化**：使用 `MsgParcel` 进行参数序列化/反序列化

**证据**：`frameworks/ipc/src/lib.rs:38-87`，`services/core_service/src/stub.rs`

### 6. 加密体系

- **算法**：AES-256-GCM
- **密钥管理**：HUKS（硬件密钥库）
- **数据库加密**：SQLCipher（AES-256-GCM）
- **哈希/随机数**：OpenSSL（SHA-256, RAND_priv_bytes）

**证据**：`services/crypto_manager/src/huks_wrapper.c:1-100`，`frameworks/os_dependency/openssl/src/openssl_wrapper.c`

---

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md) - 了解代码组织
- [架构说明](02_Architecture.md) - 深入了解架构细节
- [对外 N-API](03_NAPI_API.md) - 学习如何调用服务
- [安全风险评审](07_Security_Review.md) - 了解安全威胁
