# 攻击面分析 (Attack Surface Analysis)

## 文档目的
本文档全面分析 certificate_manager 模块的攻击面，帮助安全研究员识别所有外部输入入口、敏感操作和潜在安全风险。

---

## 攻击面总览

```
┌─────────────────────────────────────────────────┐
│                  应用层攻击面                          │
│  ┌──────────────────────────────────────────┐   │
│  │ N-API 输入参数（32 个接口）        │   │
│  │ - Uint8Array（证书数据）            │   │
│  │ - String（别名、密码、URI）         │   │
│  │ - Number（存储类型、用途等）         │   │
│  └──────────────────────────────────────────┘   │
│                          ↓ 参数验证               │
└─────────────────────────────────────────────────┘
                    ↓ IPC 通信
┌─────────────────────────────────────────────────┐
│                  IPC 层攻击面                          │
│  ┌──────────────────────────────────────────┐   │
│  │ 27 个 IPC 方法                        │   │
│  │ - MessageParcel 序列化数据        │   │
│  │ - 64KB 缓冲区限制               │   │
│  └──────────────────────────────────────────┘   │
│                          ↓ 权限检查               │
└─────────────────────────────────────────────────┘
                    ↓ 信任边界
┌─────────────────────────────────────────────────┐
│                  系统层攻击面                          │
│  ┌──────────────────────────────────────────┐   │
│  │ 文件系统操作                        │   │
│  │ - 证书存储目录                  │   │
│  │ - 路径拼接和规范化             │   │
│  └──────────────────────────────────────────┘   │
│                          ↓ HUKS 调用              │
└─────────────────────────────────────────────────┘
                    ↓ 密钥操作
┌─────────────────────────────────────────────────┐
│              HUKS 攻击面（外部依赖）              │
│  - 密钥导入/删除                     │
│  - 签名/验签/加密/解密            │
└─────────────────────────────────────────────────┘
```

---

## 外部输入清单

### 1. N-API 输入参数

| 输入类型 | 攻击场景 | 影响 |
|---------|---------|------|
| **证书数据 (Uint8Array)** | 恶意证书注入、格式混淆、缓冲区溢出 | 可能导致任意代码执行 |
| **证书密码 (String)** | 密码爆破、弱密码利用 | 可能导致证书泄露 |
| **证书别名 (String)** | 路径遍历、注入攻击 | 可能导致文件系统攻击 |
| **证书 URI (String)** | URI 注入、权限绕过 | 可能导致未授权访问 |
| **存储类型 (Number)** | 类型混淆、整数溢出 | 可能导致逻辑错误 |
| **用途/目的 (Number)** | 逻辑绕过、功能滥用 | 可能导致权限提升 |
| **用户 ID/UID (Number)** | 越界访问、隐私泄露 | 可能导致隔离绕过 |

### 2. IPC 输入数据

| 输入类型 | 验证位置 | 潜在风险 |
|---------|---------|------|
| **MessageParcel 数据** | 反序列化攻击 | 代码注入 |
| **证书 URI** | 路径遍历 | 文件系统访问 |
| **密钥 URI** | HUKS 注入 | 密钥操作劫持 |
| **用户 ID/UID** | 权限绕过 | 跨用户访问 |
| **缓冲区数据** | 溢出/越界读写 | 拒绝服务/信息泄露 |

### 3. 文件系统输入

| 输入源 | 攻击向量 | 影响范围 |
|---------|---------|--------|
| **证书文件读取** | 文件内容解析 | 证书解析漏洞 |
| **目录遍历** | `../` 路径遍历 | 任意文件读取 |
| **证书文件写入** | 路径拼接、权限检查 | 任意文件写入 |

### 4. HUKS 调用

| 操作类型 | 潜在风险 |
|---------|---------|
| **密钥导入** | 恶意密钥注入 | 密钥劫持 |
| **密钥删除** | 权限绕过 | 未授权删除 |
| **签名操作** | 签名伪造 | 身份伪造 |
| **加密/解密** | 密码学攻击 | 数据泄露 |

---

## 敏感操作清单

### 文件系统操作

| 操作 | 敏感度 | 潜在影响 |
|------|--------|---------|
| **证书文件写入** | 高 | 可能写入到任意位置、覆盖系统证书 |
| **证书文件读取** | 中 | 可能泄露敏感信息 |
| **证书文件删除** | 高 | 可能删除其他用户证书 |
| **目录创建** | 低 | 可能影响文件系统结构 |

### 密钥操作（HUKS）

| 操作 | 敏感度 | 潜在影响 |
|------|--------|---------|
| **密钥导入** | 极高 | 注入恶意密钥、替换合法密钥 |
| **密钥删除** | 高 | 删除其他应用密钥 |
| **签名操作** | 极高 | 伪造签名、绕过身份验证 |
| **加密/解密** | 极高 | 数据泄露、密钥泄露 |

### 权限检查

| 操作 | 敏感度 | 潜在影响 |
|------|--------|---------|
| **系统应用验证** | 高 | 绕过系统应用检查、提升权限 |
| **访问令牌检查** | 中 | 令牌伪造、权限提升 |
| **UID 隔离验证** | 中 | 跨用户访问、数据泄露 |

---

## 信任边界图

```mermaid
graph TB
    A[应用进程<br/>User Space] -->|应用边界|
    A -->|N-API 层|
    |N-API 层| -->|IPC 边界|
    |N-API 层| -->|权限检查|
    |N-API 层| --> B[证书管理服务<br/>System Space<br/>SA 3512]
    B -->|文件系统边界|
    B -->|HUKS 边界|
    B --> C[HUKS 服务<br/>Keystore]
    B -->|安全边界|
    B --> D[存储介质<br/>Flash + TEE]

    style A fill:#ff9999,stroke:#ff6600,stroke-width:3px
    style B fill:#66ccff,stroke:#0066cc,stroke-width:3px
    style C fill:#00cc66,stroke:#004d00,stroke-width:3px
    style D fill:#e6e600,stroke:#cca300,stroke-width:3px

    subgraph UserSpace [用户空间]
        direction TB
        JS_App[JS 应用] --> NAPI[N-API 接口]
    end

    subgraph TrustBoundary [信任边界]
        direction LR
        NAPI --> IPC_Check[权限检查]
        IPC_Check --> SA[Cer tManager Service]
    end

    subgraph SystemSpace [系统空间]
        direction TB
        SA --> IPC_Handler[IPC 处理器]
        SA --> Engine[Engine 层]
        Engine --> Storage[文件系统存储]
        Engine --> HUKS_Call[HUKS 调用]
    end

    subgraph SecurityBoundary [安全边界]
        direction LR
        HUKS_Call --> Storage_Medium[存储介质]
        Storage_Medium --> TEE[可信执行环境]
    end
```

---

## 输入入口点映射

### N-API 接口 → 代码位置

| JS API | C++ 实现文件 | 输入验证代码位置 |
|---------|--------------|----------------|
| installPublicCertificate | cm_napi_install_app_cert.cpp | interfaces/kits/napi/src/cm_napi_install_app_cert.cpp |
| getPublicCertificate | cm_napi_get_app_cert_info.cpp | interfaces/kits/napi/src/cm_napi_get_app_cert_info.cpp |
| grantPublicCertificate | cm_napi_grant.cpp | interfaces/kits/napi/src/cm_napi_grant.cpp |
| init (签名) | cm_napi_sign_verify.cpp | interfaces/kits/napi/src/cm_napi_sign_verify.cpp |
| getAllPublicCertificates | cm_napi_get_app_cert_list.cpp | interfaces/kits/napi/src/cm_napi_get_app_cert_list.cpp |

### IPC 方法 → 代码位置

| IPC 方法 | 服务处理函数 | 权限检查位置 |
|----------|------------|-------------|
| CM_MSG_INSTALL_APP_CERTIFICATE | CmIpcServiceInstallAppCert | services/.../cm_ipc_service.c |
| CM_MSG_GET_APP_CERTIFICATE_LIST | CmIpcServiceGetAppCertList | services/.../cm_ipc_service.c |
| CM_MSG_GRANT_APP_CERT | CmIpcServiceGrantAppCertificate | services/.../cm_ipc_service.c + permission_check.cpp |
| CM_MSG_INIT (签名) | CmIpcServiceInit | services/.../cm_ipc_service.c + permission_check.cpp |

### 文件操作 → 代码位置

| 操作 | 实现文件 | 路径操作位置 |
|------|---------|-------------|
| 证书存储 | cert_manager_storage.c | services/.../engine/main/core/src/cert_manager_storage.c |
| 路径构建 | cert_manager_file_operator.c | services/.../engine/main/core/src/cert_manager_file_operator.c |
| 文件读取 | cert_manager_storage.c | services/.../engine/main/core/src/cert_manager_storage.c |

---

## 潜在攻击路径

### 攻击路径 1：恶意证书注入

```
应用 → N-API installPublicCertificate()
    ↓ 输入恶意证书（含恶意代码或漏洞）
    ↓ IPC CM_MSG_INSTALL_APP_CERTIFICATE
    ↓ CmIpcServiceInstallAppCert()
    ↓ CertManagerParseCert()
    ↓ 解析恶意证书格式
    ↓ OpenSSL 解析器漏洞触发
    ↓ 任意代码执行或信息泄露
```

**影响范围**：
- 攻击者可以注入恶意证书格式
- 可能触发 OpenSSL 解析漏洞
- 可能绕过权限检查

**缓解措施**：
- 证书格式严格验证
- 输入大小限制
- OpenSSL 版本安全更新

### 攻击路径 2：路径遍历

```
应用 → N-API installUserTrustedCert()
    ↓ 输入别名：`../../../etc/hosts`
    ↓ 路径拼接
    ↓ /data/service/el1/public/cert_manager_service/certificates/user_open/../../../etc/hosts
    ↓ 写入到系统任意文件
```

**影响范围**：
- 可能写入系统任意位置
- 可能覆盖系统文件
- 可能读取敏感配置

**缓解措施**：
- 路径规范化检查（TODO(证据不足) - 需验证）
- 别名白名单
- 权限严格限制

### 攻击路径 3：权限绕过

```
恶意应用 → N-API installPrivateCertificate()
    ↓ 伪造系统应用身份
    ↓ 绕过 CmIsSystemApp() 检查
    ↓ IPC CM_MSG_INSTALL_APP_CERTIFICATE
    ↓ 安装系统级证书
    ↓ 获得系统权限
```

**影响范围**：
- 普通应用获得系统权限
- 访问受限资源

**缓解措施**：
- 强制系统应用签名验证
- 令牌检查加强
- SELinux 策略限制

---

## IPC 安全机制

### 当前实现的安全措施

| 安全机制 | 实现位置 | 覆盖范围 |
|---------|---------|---------|
| **权限检查** | cert_manager_permission_check.cpp | 5 种权限类型 |
| **URI 验证** | cert_manager_check.c | 路径格式检查 |
| **大小限制** | cm_ipc_service.c | MAX_IPC_BUF_SIZE = 64KB |
| **缓冲区限制** | cm_ipc_service.c | MAX_MALLOC_LEN = 1MB |
| **UID 隔离** | CmContext 结构 | userId + uid 隔离 |
| **同步 IPC** | cm_ipc_service.c | MessageOption::TF_SYNC |

### 安全不足

| 问题 | 风险等级 | 缓解 |
|------|---------|------|
| **未发现输入验证代码** | 高 | 需要确认是否存在 TODO |
| **路径遍历防护不明确** | 高 | 需要验证 CheckUri() 实现 |
| **缓冲区溢出防护** | 中 | 使用 MessageParcel 序列化，但未确认边界检查 |
| **TOCTOU 保护** | 中 | 文件系统操作未发现原子性保证 |

---

## 模糊测试覆盖

项目包含 **70+ 个模糊测试目标**，覆盖：

### 测试类别

| 测试类别 | 目标数量 | 覆盖的攻击面 |
|---------|---------|-------------|
| **证书安装/卸载** | 20+ | 证书数据、别名、URI |
| **权限检查** | 5+ | 系统应用、权限验证 |
| **密码学操作** | 10+ | init、update、finish、abort |
| **文件操作** | 10+ | 路径拼接、证书读写 |
| **授权管理** | 5+ | 授权、撤销授权 |
| **系统证书查询** | 5+ | getCertList、getCertInfo |
| **IPC 通信** | 5+ | 各种 IPC 方法 |

### 测试文件示例

```
test/fuzz_test/
├── cmservicinstallappcert_fuzzer/          # 应用证书安装模糊测试
├── cmsetcertstatus_fuzzer/                # 证书状态设置模糊测试
├── cmgrantappcertificate_fuzzer/           # 授权操作模糊测试
├── cmipcservicecheckisauthorizedapp_fuzzer/  # 授权检查模糊测试
├── cmparsecertchainandprivkey_fuzzer/   # 证书解析模糊测试
├── cmkeyopimportkey_fuzzer/             # HUKS 密钥导入模糊测试
└── cmreadcertdata_fuzzer/                # 证书读取模糊测试
```

---

## 相关链接

- [代码地图](03_CodeMap.md) - 定位安全检查和输入验证代码
- [安全风险评估](06_SecurityReview.md) - 详细的漏洞分析
- [接口文档](04_Interface.md) - 查看参数验证逻辑
- [代码证据汇总](_work/NOTES.md) - 查看具体代码位置

---

**适用范围**：本文档适用于 OpenHarmony 4.0 版本的 certificate_manager 模块

**最后更新**：2026-02-07
