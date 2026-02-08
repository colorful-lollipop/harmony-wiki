# 项目概览

> 本文档提供用户证书管理部件的快速概览

---

## 目的

本文档面向新人提供用户证书管理部件的快速了解，包括项目定位、核心能力、使用场景等。

## 适用范围

- OpenHarmony 系统应用开发者
- 证书管理相关的产品经理、架构师
- 需要了解证书管理功能的技术人员

## 关键结论

1. **项目定位**: 用户证书管理是 OpenHarmony 的预置系统应用，提供 CA 证书和凭据的图形化管理功能
2. **核心功能**: 四大类 - 系统 CA 证书、用户 CA 证书、用户凭据、系统凭据
3. **技术栈**: 纯 ArkTS 应用，通过 `@ohos.security.certManager` 与证书管理服务交互
4. **无 N-API 模块**: 本项目为纯 ArkTS 应用，无 C/C++ N-API 模块
5. **主要入口**: 设置 → 隐私 → 证书与凭据

## 相关跳转

- [01_Position_and_Boundary.md](wiki/01_Position_and_Boundary.md) - 项目边界和关键概念
- [02_Directory_Structure.md](wiki/02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计

---

## 1. 项目定位

### 1.1 基本信息

| 属性 | 值 |
|------|-----|
| **项目名称** | 用户证书管理部件 (user_certificate_manager) |
| **Bundle 名称** | com.ohos.certmanager |
| **版本** | 1.0.0 (versionCode: 1000000) |
| **类型** | OpenHarmony 系统应用 |
| **子系统** | applications |
| **许可** | Apache License 2.0 |
| **源码路径** | applications/standard/user_certificate_manager |

**证据位置**: `bundle.json:2-6`, `BUILD.gn:15-32`, `AppScope/app.json:3-8`

### 1.2 部件定义

用户证书管理是 OpenHarmony 应用子系统的标准部件，定义在 `bundle.json` 中：

```json
{
  "name": "user_certificate_manager",
  "subsystem": "applications",
  "syscap": [],
  "adapted_system_type": ["standard"],
  "rom": "1MB",
  "ram": "1MB"
}
```

**证据位置**: `bundle.json:14-24`

---

## 2. 核心能力

### 2.1 功能矩阵

| 功能类别 | 具体能力 | 支持的操作 |
|---------|---------|-----------|
| **系统 CA 证书** | 查看系统预置的 CA 证书 | 查看、详情、启用/禁用 |
| **用户 CA 证书** | 安装、管理用户信任的 CA 证书 | 查看、详情、安装、删除、启用/禁用 |
| **用户凭据** | 安装、管理应用使用的证书凭据 | 查看、详情、安装、删除、授权管理 |
| **系统凭据** | 安装、管理 WLAN/VPN 系统凭据 | 查看、详情、安装、删除 |

**证据位置**: `README.md:5-13`

### 2.2 证书格式支持

| 类型 | 支持格式 | 用途 |
|------|---------|------|
| CA 证书 | .cer, .pem, .crt, .der, .p7b, .spc | 公钥证书 |
| 凭据 | .p12, .pfx | 包含私钥的证书 |

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:68-69`, `certmanager/src/main/ets/presenter/CmFaPresenter.ets:82`

### 2.3 证书存储类型

| 存储类型 | 枚举值 | 特性 |
|---------|---------|------|
| 凭据证书存储 | `CERT_MANAGER_CREDENTIAL_STORE = 0` | 端实体证书存储 |
| 系统信任存储 | `CERT_MANAGER_SYSTEM_TRUSTED_STORE = 1` | 只读，仅系统可更新 |
| 用户信任存储 | `CERT_MANAGER_USER_TRUSTED_STORE = 2` | 可修改，应用和用户均可操作 |
| 应用信任存储 | `CERT_MANAGER_APPLICATION_TRUSTED_STORE = 3` | 应用特定，仅应用可修改 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:68-77`

---

## 3. 使用场景

### 3.1 主要使用场景

1. **WLAN/VPN 连接配置**
   - WLAN 和 VPN 系统服务在连接时调用证书管理服务接口
   - 安装、读取和使用系统证书凭据
   - 证据: `README.md:121`

2. **HTTPS 服务器证书校验**
   - 应用开发者在校验 HTTPS 服务器证书时
   - 使用系统预置的 CA 证书或用户 CA 证书
   - 证据: `README.md:123`

3. **Email/文档签名**
   - 应用开发者在对 Email 邮件或文档进行签名时
   - 使用用户证书凭据进行签名
   - 证据: `README.md:125`

### 3.2 应用入口

**路径**: 设置 → 隐私 → 证书与凭据

**入口方式**:
- 通过隐私中心入口打开 (`action.access.privacy.center`)
- 通过 want action 打开 (`ohos.want.action.viewData`)

**证据位置**: `certmanager/src/main/module.json:57-66`

---

## 4. 技术栈

### 4.1 开发语言

- **ArkTS** - 主要开发语言
  - 49 个 ArkTS (.ets/.ts) 文件
  - 证据: 统计 `certmanager/src/main/ets/` 目录

### 4.2 外部依赖 API

| 模块 | 导入路径 | 用途 |
|-------|---------|------|
| 证书管理服务 | `@ohos.security.certManager` | 证书和凭据管理 |
| 证书解析 | `@ohos.security.cert` | 证书解析 |
| 用户认证 | `@ohos.userIAM.userAuth` | 生物识别/密码认证 |
| 包管理 | `@ohos.bundle.bundleManager` | 获取应用信息 |
| 包资源管理 | `@ohos.bundle.bundleResourceManager` | 获取应用资源 |
| 文件 I/O | `@ohos.file.fs`, `@ohos.file.fileuri` | 证书文件读取 |
| 文件选择器 | `@ohos.file.picker` | 文件选择 |
| 窗口管理 | `@ohos.window` | 防截屏 |
| 路由 | `@ohos.router` | 页面跳转 |
| 日志 | `@ohos.hilog` | 日志输出 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:21-24`, `certmanager/src/main/ets/model/CheckUserAuthModel.ets:16`, `certmanager/src/main/ets/model/BundleModel.ets:16-17`, `certmanager/src/main/ets/model/FileIoModel.ets:16-17`, `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:16`

---

## 5. 部署信息

### 5.1 构建产物

- **HAP 文件**: `CertificateManager.hap`
- **安装路径**: `app/com.ohos.certificatemanager`

**证据位置**: `BUILD.gn:22-26`

### 5.2 编译命令

```bash
# 单仓编译
./build.sh --product-name rk3568 --ccache --build-target user_certificate_manager

# 安装到设备
hdc install CertificateManager.hap
```

**证据位置**: `README.md:89-109`, `BUILD.gn:15-32`

---

## 6. 架构概览

### 6.1 应用结构

```
用户证书管理应用 (CertificateManager.hap)
├── CertManager 模块 (certmanager/)
│   ├── MainAbility (UI Extension Ability)
│   ├── Model 层 (业务逻辑)
│   ├── Presenter 层 (页面展示逻辑)
│   ├── Pages 层 (UI 页面)
│   └── Common 层 (公共组件和工具)
└── Entry 模块 (entry/)
    └── HAP 打包入口
```

### 6.2 与外部模块交互

```
┌─────────────────────────────────────────────────────────────┐
│           用户证书管理应用 (ArkTS)                      │
├─────────────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │  Model 层   │  │ 用户认证     │  │ 包管理    │  │
│  └─────────────┘  └──────────────┘  └───────────┘  │
│         ↓                  ↓                  ↓          │
│  ┌──────────────────────────────────────────────────┐  │
│  │    @ohos.security.certManager (系统服务)      │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets`, `certmanager/src/main/ets/model/CheckUserAuthModel.ets`, `certmanager/src/main/ets/model/BundleModel.ets`

---

## 7. 关键概念

### 7.1 证书与凭据

- **证书 (Certificate)**: 用于验证身份的公钥证书
- **凭据 (Credential)**: 包含私钥的证书，用于签名或认证

### 7.2 信任存储

- **系统信任存储**: 只读，存储系统预置的 CA 证书
- **用户信任存储**: 可写，存储用户安装的 CA 证书
- **应用信任存储**: 应用特定，仅该应用可访问

### 7.3 授权

- 凭据访问需要应用授权
- 用户可以控制哪些应用可以访问特定的凭据

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:659-695`

---

**END OF 00_Overview.md**
