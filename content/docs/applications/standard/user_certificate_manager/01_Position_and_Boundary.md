# 项目定位与边界

> 本文档定义用户证书管理部件的项目边界、核心能力、运行环境和关键概念

---

## 目的

本文档明确定义用户证书管理部件的项目范围、边界和定位，避免与其他组件职责混淆。

## 适用范围

- OpenHarmony 系统应用架构师
- 产品经理
- 需要理解项目范围的开发者

## 关键结论

1. **核心职责**: 提供图形化界面管理 CA 证书和凭据，不负责证书的底层存储和验证
2. **边界明确**: 证书管理服务的客户端，不实现证书解析、存储、验证等核心逻辑
3. **依赖清晰**: 依赖 `@ohos.security.certManager` 系统服务完成所有证书操作
4. **无服务端**: 纯客户端应用，不提供对外服务接口
5. **数据隔离**: 仅作为 UI 和操作入口，数据存储在证书管理服务中

## 相关跳转

- [00_Overview.md](wiki/00_Overview.md) - 项目概览
- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计
- [04_External_API.md](wiki/04_External_API.md) - 外部接口

---

## 1. 项目定位

### 1.1 职责范围

**用户证书管理部件** 是 OpenHarmony 的预置系统应用，提供：

| 功能 | 职责 | 非职责 |
|------|--------|---------|
| 证书展示 | ✓ 展示 CA 证书和凭据列表 | ✗ 证书底层存储 |
| 证书安装 | ✓ 通过 UI 触发安装 | ✗ 证书格式解析 |
| 证书删除 | ✓ 通过 UI 触发删除 | ✗ 证书验证逻辑 |
| 证书授权 | ✓ 控制应用访问凭据 | ✗ 访问控制实现 |
| 用户认证 | ✓ 触发生物识别/密码认证 | ✗ 认证算法实现 |

**证据位置**: `README.md:5-13`

### 1.2 与证书管理服务的关系

```
┌─────────────────────────────────────────────────────────────┐
│                  用户证书管理应用                         │
│  (UI 层、业务逻辑层、用户交互层)                      │
│                                                      │
│  ┌──────────────────────────────────────────────────┐      │
│  │   @ohos.security.certManager (系统服务)    │      │
│  │   - 证书解析                                │      │
│  │   - 证书存储                                │      │
│  │   - 证书验证                                │      │
│  │   - 访问控制                                │      │
│  └──────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

**关键点**:
- 用户证书管理应用是证书管理服务的 **客户端**
- 所有证书操作通过 `@ohos.security.certManager` API 完成
- 不直接操作证书文件或证书库

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:21`

---

## 2. 项目边界

### 2.1 边内功能 (在项目范围内)

| 功能 | 说明 | 实现位置 |
|------|------|---------|
| UI 展示 | 证书/凭据列表、详情页面 | `pages/` |
| 用户交互 | 安装、删除、授权操作 | `presenter/` + `pages/` |
| 文件选择 | 通过文件选择器选择证书 | `FileIoModel.ets` |
| 用户认证 | 生物识别/密码认证 | `CheckUserAuthModel.ets` |
| 防截屏 | 密码输入时防截屏 | `PreventScreenshotsModel.ets` |
| 应用信息获取 | 获取授权应用名称和图标 | `BundleModel.ets` |

### 2.2 边外功能 (不在项目范围内)

| 功能 | 说明 | 实际位置 |
|------|------|---------|
| 证书解析 | 解析 PEM/DER/P7B 等格式 | `@ohos.security.certManager` |
| 证书存储 | 存储证书到密钥库 | `@ohos.security.certManager` |
| 证书验证 | 验证证书签名、有效期 | `@ohos.security.certManager` |
| 访问控制 | 控制应用访问凭据 | `@ohos.security.certManager` |
| 用户认证实现 | 生物识别算法 | `@ohos.userIAM.userAuth` |

---

## 3. 核心能力

### 3.1 证书管理能力

| 能力 | 支持操作 | 调用接口 |
|------|---------|---------|
| 系统 CA 证书 | 查看列表、查看详情 | `getSystemTrustedCertificate*` |
| 用户 CA 证书 | 安装、查看、删除、启用/禁用 | `installUserTrustedCertificate`, `getUserTrustedCertificate*`, `uninstallUserTrustedCertificate`, `setCertificateStatus` |
| 用户凭据 | 安装、查看、删除、授权管理 | `installPublicCertificate`, `getPublicCertificate*`, `uninstallPublicCertificate`, `grantPublicCertificate` |
| 系统凭据 | 安装、查看、删除 | `installSystemAppCertificate`, `getSystemAppCertificate*`, `uninstallSystemAppCertificate` |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:319-834`

### 3.2 扩展能力

| 能力 | 说明 | 实现位置 |
|------|------|---------|
| 私有凭据查看 | 查看应用的私钥凭据 | `getAllAppPrivateCertificatesByUid` |
| UKey 证书 | 查看 UKey 证书列表 | `getUkeyCertificateList` |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:788-834`

---

## 4. 运行环境

### 4.1 系统要求

| 要求 | 说明 |
|------|------|
| **操作系统** | OpenHarmony 4.0+ |
| **设备类型** | standard 系统类型 |
| **API Level** | API 23 |
| **运行时** | OpenHarmony Runtime |

**证据位置**: `bundle.json:19-21`, `build-profile.json5:21-23`

### 4.2 权限要求

| 权限 | 用途 | 证据 |
|------|------|------|
| `ohos.permission.ACCESS_CERT_MANAGER` | 访问证书管理服务 | `module.json:92` |
| `ohos.permission.ACCESS_CERT_MANAGER_INTERNAL` | 访问证书管理服务内部接口 | `module.json:89` |
| `ohos.permission.ACCESS_SYSTEM_APP_CERT` | 访问系统应用证书 | `module.json:107` |
| `ohos.permission.ACCESS_USER_TRUSTED_CERT` | 访问用户信任证书 | `module.json:110` |
| `ohos.permission.GET_BUNDLE_INFO` | 获取应用信息 | `module.json:86` |
| `ohos.permission.ACCESS_BIOMETRIC` | 生物识别认证 | `module.json:101` |
| `ohos.permission.PRIVACY_WINDOW` | 防截屏窗口 | `module.json:104` |

**证据位置**: `certmanager/src/main/module.json:84-114`

---

## 5. 关键概念

### 5.1 证书类型

| 类型 | 定义 | 用途 |
|------|------|------|
| **CA 证书** | Certificate Authority 证书 | 验证其他证书的签名 |
| **端实体证书** | 终端实体证书 | 标识具体实体（服务器、应用） |
| **凭据** | 包含私钥的证书 | 用于签名、加密 |

### 5.2 存储类型

| 存储 | 特性 | 权限 |
|------|------|------|
| **系统信任存储** | 只读，仅系统可更新 | 所有应用可读取 |
| **用户信任存储** | 可写，应用和用户可操作 | 用户控制 |
| **应用信任存储** | 应用特定，仅应用可写 | 应用专用 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:68-77`

### 5.3 操作类型

| 枚举值 | 操作 | 支持证书类型 |
|--------|------|------------|
| `CM_MODEL_OPT_SYSTEM_CA = 1` | 系统CA证书 | 系统 CA 证书 |
| `CM_MODEL_OPT_USER_CA = 2` | 用户CA证书 | 用户 CA 证书 |
| `CM_MODEL_OPT_APP_CRED = 3` | 应用凭据 | 用户凭据 |
| `CM_MODEL_OPT_SYSTEM_CRED = 5` | 系统凭据 | 系统 凭据 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:40-48`

---

## 6. 非功能性需求

### 6.1 性能要求

| 指标 | 要求 | 证据 |
|--------|------|------|
| ROM 占用 | ≤ 1MB | `bundle.json:23` |
| RAM 占用 | ≤ 1MB | `bundle.json:24` |

### 6.2 安全要求

- 所有证书操作必须通过证书管理服务完成
- 凭据访问需要用户授权
- 密码输入时启用防截屏
- 安装 CA 证书需要用户认证

**证据位置**: `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:34-57`, `certmanager/src/main/ets/model/CheckUserAuthModel.ets:61-117`

---

## 7. 依赖关系

### 7.1 外部依赖

```
用户证书管理应用
├── @ohos.security.certManager (证书管理服务) ⭐
├── @ohos.security.cert (证书解析)
├── @ohos.userIAM.userAuth (用户认证)
├── @ohos.bundle.bundleManager (包管理)
├── @ohos.bundle.bundleResourceManager (包资源)
├── @ohos.file.fs (文件 I/O)
├── @ohos.file.picker (文件选择器)
├── @ohos.window (窗口管理)
├── @ohos.router (路由)
└── @ohos.hilog (日志)
```

**证据位置**: `certmanager/src/main/ets/model/*.ets` (import 语句)

### 7.2 内部依赖

```
CertManager 模块
├── MainAbility (Ability 入口)
│   └── CertPickerUiExtAbility (UI Extension)
├── Model 层 (业务逻辑)
│   ├── CertMangerModel (证书管理核心) ⭐
│   ├── CheckUserAuthModel (用户认证)
│   ├── BundleModel (包信息)
│   ├── FileIoModel (文件 I/O)
│   └── PreventScreenshotsModel (防截屏)
├── Presenter 层 (页面逻辑)
│   ├── CmFaPresenter (主页面)
│   ├── CmInstallPresenter (安装页面)
│   └── CmShow*Presenter (列表页面)
├── Pages 层 (UI 页面)
│   ├── certManagerFa.ets (主页面)
│   ├── certInstallFromStorage.ets (安装页面)
│   └── detail/* (详情页面)
└── Common 层 (公共组件)
    ├── GlobalContext (全局上下文)
    ├── constants/ (常量)
    ├── component/ (组件)
    └── util/ (工具)
```

**证据位置**: `certmanager/src/main/ets/` 目录结构

---

**END OF 01_Position_and_Boundary.md**
