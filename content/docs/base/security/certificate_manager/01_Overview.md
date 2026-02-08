# 项目概览 (Project Overview)

## 文档目的
本文档帮助读者快速理解 certificate_manager 模块的定位、能力和使用方式。

---

## 一句话定义

OpenHarmony **certificate_manager**（证书管理）是系统级证书管理服务，实现证书**全生命周期**（安装、存储、使用、销毁）的管理和安全使用，满足生态应用和上层业务的诉求。

---

## 能力边界

### ✅ 能做什么

| 功能类别 | 具体能力 |
|---------|---------|
| **证书安装** | 支持应用证书（P12/PFX）、用户信任证书（PEM/DER）、系统 CA 证书安装 |
| **证书查询** | 查询系统证书、应用证书、用户证书的列表和详细信息 |
| **证书管理** | 启用/禁用证书、批量卸载、按用户/应用筛选 |
| **授权管理** | 授权证书给其他应用、查询授权列表、撤销授权 |
| **密码学操作** | 基于证书进行签名、验签、加密、解密操作 |
| **UKey 支持** | 读取 USB Key 证书列表和信息 |
| **对话框** | 提供证书安装、卸载、详情、授权等 UI 对话框 |

### ❌ 不能做什么

| 限制 | 说明 |
|------|------|
| 不提供证书生成功能 | 只能安装已存在的证书，不生成新证书 |
| 不管理私钥的完整生命周期 | 私钥通过 HUKS 管理，证书管理模块仅引用 |
| 不支持证书撤销验证 | 不实现 CRL（证书撤销列表）查询 |

---

## 运行环境

### 系统要求

| 属性 | 要求 |
|------|------|
| **操作系统** | OpenHarmony (mini, small, standard) |
| **OpenHarmony 版本** | 依赖 HUKS 能力（OpenHarmony 3.0+） |
| **ROM 占用** | 5000KB |
| **RAM 占用** | 500KB |

### 依赖的系统服务

| 依赖组件 | 用途 |
|---------|------|
| **HUKS** | 硬件通用密钥库，用于私钥的安全存储和密码学操作 |
| **Access Token** | 访问令牌服务，用于权限检查 |
| **Bundle Framework** | Bundle 管理，用于应用信息获取 |
| **IPC / Samgr** | 系统能力管理器，用于服务注册和发现 |
| **HiSysEvent** | 系统事件服务，用于日志和安全事件上报 |
| **Relational Store** | 关系型数据库，用于证书元数据存储 |
| **Security Guard** | 安全防护服务，用于安全事件上报 |

---

## 快速开始

### 最小使用示例

#### 场景 1：安装应用证书

```typescript
import { certificateManager } from '@kit.DeviceCertificateKit';

// 准备证书数据
let keystore: Uint8Array = new Uint8Array([
    0x30, 0x82, 0x04, 0x6a, 0x02, 0x01  // 示例 P12 数据
]);
let keystorePwd: string = '123456';
let alias: string = 'MyAppCert';

try {
    // 安装证书
    const res: certificateManager.CMResult = await certificateManager.installPrivateCertificate(
        keystore,
        keystorePwd,
        alias
    );

    if (res.uri !== undefined) {
        console.log('证书安装成功，URI:', res.uri);
        // res.uri 是证书的唯一标识，后续操作使用
    }
} catch (err) {
    console.error('证书安装失败:', err.code, err.message);
}
```

#### 场景 2：获取应用证书列表

```typescript
import { certificateManager } from '@kit.DeviceCertificateKit';

try {
    // 获取调用应用的所有证书
    const certList = await certificateManager.getPrivateCertificate();

    console.log('证书数量:', certList.credentialCount);

    for (let i = 0; i < certList.credentialCount; i++) {
        console.log(`证书 ${i + 1}:`, {
            uri: certList.credentials[i].keyUri,
            alias: certList.credentials[i].alias,
            type: certList.credentials[i].type
        });
    }
} catch (err) {
    console.error('获取证书列表失败:', err.code, err.message);
}
```

#### 场景 3：使用证书签名

```typescript
import { certificateManager } from '@kit.DeviceCertificateKit';
import { BusinessError } from '@kit.BasicServicesKit';

let keyUri: string = 'cert://100/100/0/MyAppCert';

try {
    // 初始化签名操作
    const initRes = await certificateManager.init({
        uri: keyUri,
        spec: {
            purpose: certificateManager.CmKeyPurpose.CM_KEY_PURPOSE_SIGN,
            digest: certificateManager.CmKeyDigest.CM_DIGEST_SHA256,
            padding: certificateManager.CmKeyPadding.CM_PADDING_PSS
        }
    });

    if (initRes.uri === undefined) {
        // 添加待签名数据
        const updateRes = await certificateManager.update({
            uri: initRes.uri,
            inData: new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f])  // "Hello"
        });

        // 完成签名
        const finishRes = await certificateManager.finish({
            uri: initRes.uri,
            inData: new Uint8Array([])
        });

        console.log('签名成功，签名数据:', finishRes.data);
    }
} catch (err) {
    let e: BusinessError = err as BusinessError;
    console.error(`签名失败。Code: ${e.code}, message: ${e.message}`);
}
```

---

## 架构概览

certificate_manager 采用**三层架构**：

```
┌─────────────────────────────────────────────────┐
│                   SDK 层                             │
│  • N-API 模块 (security.certmanager)            │
│  • C 接口 (cert_manager_api.h)                │
│  • Dialog N-API (security.certManagerDialog)        │
└─────────────────────────────────────────────────┘
                    ↓ IPC
┌─────────────────────────────────────────────────┐
│                 Service 层                           │
│  • CertManagerService (SA ID: 3512)          │
│  • IPC 调度 (27 个方法）                  │
│  • 权限检查                                   │
└─────────────────────────────────────────────────┘
                    ↓ 内部调用
┌─────────────────────────────────────────────────┐
│                 Engine 层                           │
│  • 证书生命周期管理                             │
│  • 文件系统操作                                │
│  • HUKS 密钥管理                               │
│  • RDB 元数据存储                               │
└─────────────────────────────────────────────────┘
```

详见 [02_Architecture.md](02_Architecture.md)

---

## 证书类型

### CA 证书 (只含公钥)
- **用途**：验签或验证对端身份
- **来源**：
  - 系统预安装（`/etc/security/certificates`）
  - 用户安装（`/data/certificates/user_cacerts/`）
- **典型场景**：HTTPS 连接验证、应用签名验证

### 业务证书 (含私钥)
- **用途**：业务场景的签名和验签
- **格式**：PKCS#12 (P12/PFX)、证书链 + 私钥
- **存储**：证书文件在文件系统，私钥在 HUKS
- **权限**：需要 `ohos.permission.ACCESS_CERT_MANAGER`

---

## 权限要求

### 模块级权限

在应用 `module.json5` 中声明：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCESS_CERT_MANAGER"
      }
    ]
  }
}
```

### 不同操作的权限需求

| 操作类型 | 所需权限 | 系统应用要求 |
|---------|-----------|-------------|
| **安装应用证书** | ACCESS_CERT_MANAGER | 必须是系统应用 |
| **安装用户证书** | ACCESS_CERT_MANAGER | 普通应用即可 |
| **系统证书操作** | ACCESS_CERT_MANAGER_INTERNAL | 仅系统应用 |

---

## 存储路径

### 证书文件存储

| 证书类型 | 存储路径 |
|---------|---------|
| **系统预安装 CA** | `/etc/security/certificates` |
| **用户信任证书** | `/data/service/el1/public/cert_manager_service/certificates/user_open/` |
| **应用证书** | `/data/service/el1/public/cert_manager_service/certificates/` |
| **国密 CA（国密算法）** | `/etc/security/certificates_gm/` |

**隔离机制**：
- 按 `userId` 和 `uid` 隔离证书
- 不同用户和应用无法访问彼此证书

---

## 关键常量

### 存储类型

```c
CM_CREDENTIAL_STORE        0   // 应用凭证（含私钥）
CM_SYSTEM_TRUSTED_STORE    1   // 系统预安装 CA 证书
CM_USER_TRUSTED_STORE      2   // 用户安装的信任证书
CM_PRI_CREDENTIAL_STORE    3   // 私有凭证
CM_SYS_CREDENTIAL_STORE    4   // 系统凭证
```

### 证书状态

```c
CERT_STATUS_ENABLED    0   // 证书已启用
CERT_STATUS_DISABLED   1   // 证书已禁用
```

---

## 相关链接

- [架构详情](02_Architecture.md) - 深入理解三层架构
- [代码地图](03_CodeMap.md) - 快速定位代码位置
- [接口文档](04_Interface.md) - 完整 API 参考
- [攻击面分析](05_AttackSurface.md) - 安全研究必备
- [安全风险评估](06_SecurityReview.md) - 漏洞分析

---

**适用范围**：本文档适用于 OpenHarmony 4.0 版本的 certificate_manager 模块

**最后更新**：2026-02-07
