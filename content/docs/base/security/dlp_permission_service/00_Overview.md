# 项目概览

## 目的

本文档介绍 DLP 权限管理服务 (`dlp_permission_service`) 的项目定位、核心能力、运行环境和关键概念，帮助新人快速理解项目背景和职责。

## 适用范围

- 目标读者：DLP 权限管理服务的开发者、维护者、三方应用开发者
- 覆盖内容：项目定位、核心能力、运行环境、关键概念

---

## 项目定位

### 组件信息

| 属性 | 值 |
|------|-----|
| **组件名** | `dlp_permission_service` |
| **子系统** | `security` |
| **系统能力** | `SystemCapability.Security.DataLossPrevention` |
| **仓库路径** | `/base/security/dlp_permission_service` |
| **ROM 占用** | 2048KB |
| **RAM 占用** | 5102KB |

### DLP (Data Loss Prevention) 简介

数据防泄漏（DLP）通过一系列安全保护技术，实现文档的权限管理功能，保证权限文档的安全性。

**核心场景**：
1. **生成 DLP 文件**：对原始文件添加权限保护，生成加密的 DLP 文件
2. **访问控制**：基于账号的访问控制，只有授权用户才能打开
3. **沙箱隔离**：解密时在隔离沙箱中运行，防止数据泄露
4. **权限修改**：动态修改 DLP 文件的访问权限

---

## 核心能力

### 1. DLP 文件生成

**功能**：将普通文件加密为 DLP 文件，设置访问权限

**流程**：三方应用 → startAbility → DLP 权限应用 → SDK → DLP 权限管理服务 → 加密读写模块 → DLP 文件

**代码证据**：
- N-API 接口：`generateDlpFile` (napi_dlp_permission.cpp)
- IPC 接口：`GenerateDlpCertificate` (IDlpPermissionService.idl:31)
- 实现：`DlpPermissionService::GenerateDlpCertificate()` (dlp_permission_service.cpp)

### 2. DLP 文件访问

**功能**：解密并打开 DLP 文件，在沙箱中运行

**流程**：三方应用 → startAbility → DLP 权限应用 → SDK → DLP 权限管理服务 → 证书管理 + 加密读写 + 沙箱管理 → 沙箱应用

**代码证据**：
- N-API 接口：`openDLPFile` (napi_dlp_permission.cpp)
- IPC 接口：`ParseDlpCertificate` (IDlpPermissionService.idl:34)
- 实现：`DlpPermissionService::ParseDlpCertificate()` (dlp_permission_service.cpp)

### 3. 权限管理

**功能**：修改、删除 DLP 文件权限，设置保留策略

**代码证据**：
- N-API 接口：`setRetentionState`, `cancelRetentionState` (napi_dlp_permission.cpp)
- IPC 接口：`SetRetentionState`, `CancelRetentionState` (IDlpPermissionService.idl:70-71)
- 实现：`DlpPermissionService::SetRetentionState()` (dlp_permission_service.cpp)

### 4. 访问记录

**功能**：获取应用对 DLP 文件的访问记录

**代码证据**：
- N-API 接口：`getDLPFileAccessRecords` (napi_dlp_permission.cpp)
- IPC 接口：`GetDLPFileVisitRecord` (IDlpPermissionService.idl:74)
- 实现：`DlpPermissionService::GetDLPFileVisitRecord()` (dlp_permission_service.cpp)

---

## 运行环境

### 目标平台

| 特性 | 值 |
|------|-----|
| **系统类型** | standard |
| **编译目标** | rk3568 (参考) |
| **支持 JS API** | 是（`support_jsapi` 条件编译） |

### 系统依赖

**关键依赖组件** (bundle.json:32-66)：

| 依赖组件 | 用途 |
|----------|------|
| `safwk` / `samgr` | System Ability 注册和发现 |
| `ipc` | 进程间通信 |
| `access_token` | 权限和 token 管理 |
| `huks` | 硬件支持的密钥管理（加密） |
| `napi` / `ace_engine` | N-API 绑定和 JS 引擎 |
| `libfuse` | 用户态文件系统（DLP link 文件） |
| `ability_runtime` / `bundle_framework` | 沙箱应用生命周期管理 |
| `os_account` | 账户管理 |
| `common_event_service` | 事件订阅 |
| `kv_store` | 本地数据存储 |
| `openssl` | 加密库 |

---

## 关键概念

### DLP 文件

DLP 文件是经过加密的文件，包含：
- 原始文件内容（加密）
- 访问策略（权限列表）
- 数字证书

### 沙箱 (Sandbox)

DLP 沙箱是一个隔离的运行环境，用于：
- 解密 DLP 文件
- 限制应用能力（截图、导出等）
- 防止数据泄露到其他应用

**证据**：`InstallDlpSandbox` IPC 方法 (IDlpPermissionService.idl:49)

### 访问权限 (DLPFileAccess)

DLP 文件支持四级访问权限 (permission_policy.h):

| 权限级别 | 值 | 说明 |
|----------|-----|------|
| `NO_PERMISSION` | 0 | 无权限 |
| `READ_ONLY` | 1 | 只读 |
| `CONTENT_EDIT` | 2 | 可编辑内容但不能另存 |
| `FULL_CONTROL` | 3 | 完全控制 |

### 操作权限 (ActionFlags)

细粒度的操作权限控制，支持：

| 操作 | 说明 |
|------|------|
| `ACTION_VIEW` | 查看文档 |
| `ACTION_SAVE` | 保存修改 |
| `ACTION_SAVE_AS` | 另存为新文件 |
| `ACTION_EDIT` | 编辑内容 |
| `ACTION_SCREEN_CAPTURE` | 截屏 |
| `ACTION_SCREEN_SHARE` | 屏幕共享 |
| `ACTION_SCREEN_RECORD` | 屏幕录制 |
| `ACTION_COPY` | 复制内容 |
| `ACTION_PRINT` | 打印文档 |
| `ACTION_EXPORT` | 导出内容 |
| `ACTION_PERMISSION_CHANGE` | 修改权限 |

### System Ability (SA)

DLP 权限管理服务是一个 System Ability，SA ID 为 **3521** (3521.json:5)

**证据**：
- `REGISTER_SYSTEM_ABILITY_BY_ID(DlpPermissionService, SA_ID_DLP_PERMISSION_SERVICE, true)` (dlp_permission_service.cpp)
- `constexpr const int32_t SA_ID_DLP_PERMISSION_SERVICE = 3521;` (dlp_permission_service.cpp:122)

### 回调机制

支持两类回调：

| 回调类型 | 注册方法 | 用途 |
|----------|----------|------|
| 沙箱变化回调 | `RegisterDlpSandboxChangeCallback` | 监听 DLP 沙箱安装/卸载 |
| 打开文件回调 | `RegisterOpenDlpFileCallback` | 监听 DLP 文件打开事件 |

**证据**：IDlpPermissionService.idl:65-68

---

## 相关跳转链接

- [目录结构与模块职责](01_Directory_Structure.md) - 了解代码组织
- [架构说明](02_Architecture.md) - 深入理解架构设计
- [对外 N-API](03_NAPI.md) - 查看完整的 JS API 列表
- [安全风险评审](07_Security_Review.md) - 了解安全特性

---

最后更新时间：2026-02-06
