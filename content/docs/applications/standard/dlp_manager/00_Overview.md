# 项目概览

## 目的

本文档介绍DLP权限管理应用的项目定位、核心能力、运行环境和关键概念，帮助读者建立对项目的整体认知。

## 适用范围

- 新加入项目的开发人员
- 需要了解DLP功能的产品经理
- 进行安全评审的审计人员
- 集成DLP能力的第三方开发者

---

## 项目定位

### 什么是DLP

DLP（Data Leak Prevention，数据防泄漏）是通过一系列安全保护技术实现文档权限管理的功能，保证权限文档的安全性。

### DLP权限管理应用的角色

本应用是OpenHarmony系统中的**系统级DLP管理应用**，负责：

1. **DLP文件创建** - 接收文件并添加权限保护，生成`.dlp`文件
2. **权限管理** - 设置文件授权用户、权限级别（只读/编辑）、有效期
3. **文件打开** - 验证用户权限，启动沙箱应用安全打开DLP文件
4. **权限修改** - 支持已有DLP文件的权限变更和解密

### 在系统中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                     用户层 (User Layer)                      │
│                   ┌──────────────────┐                      │
│                   │   文件管理应用     │                      │
│                   └────────┬─────────┘                      │
│                            │                                │
│                   ┌────────▼─────────┐                      │
│                   │  DLP权限管理应用  │ ← 本应用             │
│                   │  (dlp_manager)   │                      │
│                   └────────┬─────────┘                      │
│                            │                                │
├────────────────────────────┼────────────────────────────────┤
│                     系统框架层 (Framework)                    │
│                   ┌────────▼─────────┐                      │
│                   │  DLP Permission  │                      │
│                   │     Service      │                      │
│                   └──────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

**相关仓位置**:
- 本应用: `applications/standard/dlp_manager`
- 系统服务: `security_dlp_permission_service`

---

## 核心能力

### 1. 生成DLP文件

**能力描述**: 将普通文件转换为受保护的DLP文件

**流程**:
1. 接收原始文件URI和授权配置
2. 验证当前用户身份（域账号认证）
3. 调用DLP Permission Service生成加密文件
4. 保存.dlp文件到指定路径

**代码入口**: `entry/src/main/ets/pages/encryptionProtection.ets:87`

### 2. 打开DLP文件

**能力描述**: 验证权限并安全打开DLP文件

**流程**:
1. 解析DLP文件头获取权限信息
2. 验证当前用户是否有权限访问
3. 创建沙箱环境（Sandbox）
4. 解密文件内容到沙箱
5. 启动对应应用打开文件

**代码入口**: `entry/src/main/ets/OpenDlpFile/ViewProcessor/ViewProcessor.ets:46`

### 3. 修改DLP权限

**能力描述**: 修改已有DLP文件的权限设置

**支持操作**:
- 添加/删除授权用户
- 修改用户权限级别（只读→编辑）
- 修改有效期
- 解除加密（完全解密）

**代码入口**: `entry/src/main/ets/pages/changeEncryption.ets`

### 4. 权限检查

**能力描述**: 查询当前用户对DLP文件的权限

**检查内容**:
- 是否文件所有者
- 是否在授权用户列表
- 权限级别（只读/编辑/完全控制）
- 有效期是否过期

**代码入口**: `entry/src/main/ets/common/FileUtils/utils.ets:173`

---

## 运行环境

### 系统要求

| 属性 | 要求 |
|------|------|
| 系统类型 | OpenHarmony Standard（标准系统） |
| 最低API版本 | 12 |
| 目标API版本 | 12 |
| 设备类型 | phone、tablet、2in1 |

**配置来源**: `AppScope/app.json:9-10`

### 必要权限

应用需要以下系统权限（`entry/src/main/module.json:95-126`）:

| 权限名 | 用途 |
|--------|------|
| ohos.permission.ACCESS_DLP_FILE | 访问DLP文件 |
| ohos.permission.GET_LOCAL_ACCOUNTS | 获取本地账号信息 |
| ohos.permission.START_ABILITIES_FROM_BACKGROUND | 后台启动Ability |
| ohos.permission.GET_BUNDLE_INFO | 获取应用包信息 |
| ohos.permission.GET_NETWORK_INFO | 获取网络状态 |
| ohos.permission.START_DLP_CRED | 启动DLP凭证服务 |
| ohos.permission.START_SYSTEM_DIALOG | 启动系统对话框 |
| ohos.permission.EXEMPT_PRIVACY_SECURITY_CENTER | 隐私安全中心豁免 |
| ohos.permission.REPORT_SECURITY_EVENT | 上报安全事件 |
| ohos.permission.FILE_ACCESS_MANAGER | 文件访问管理 |

### 依赖服务

| 服务名 | 用途 |
|--------|------|
| DLP Permission Service | 核心DLP功能服务 |
| DLP Credential Manager | 域账号凭证管理 |
| 账号服务 | 域账号认证 |
| 包管理服务 | 沙箱应用安装 |

---

## 关键概念

### DLP文件格式

DLP文件是加密保护的文件，有两种格式：

1. **原始格式** - 单文件加密（magic: `0x087f4922`）
2. **ZIP格式** - 多文件打包加密（magic: `0x04034b50`）

**参考代码**: `entry/src/main/ets/common/constant.ets:197-198`

### 账号类型

支持三种账号类型：

| 类型 | 值 | 说明 |
|------|-----|------|
| DOMAIN_ACCOUNT | 1 | 域账号（企业统一身份） |
| CLOUD_ACCOUNT | 2 | 云账号 |
| ENTERPRISE_ACCOUNT | 4 | 企业账号 |

**参考代码**: `entry/src/main/ets/common/FileUtils/utils.ets:896-906`

### 权限级别

DLP文件访问权限（`@ohos.dlpPermission.DLPFileAccess`）:

| 级别 | 值 | 说明 |
|------|-----|------|
| NO_PERMISSION | 0 | 无权限 |
| READ_ONLY | 1 | 只读 |
| EDIT | 2 | 编辑 |
| FULL_CONTROL | 3 | 完全控制（所有者） |

### 沙箱机制

**概念**: 为每个DLP文件创建隔离的应用运行环境

**特点**:
- 独立的应用实例（appIndex区分）
- 独立的文件系统视图
- 受限的网络访问
- 受限的剪贴板/截屏能力

**参考代码**: `entry/src/main/ets/OpenDlpFile/handler/StartSandboxHandler.ets`

### 信任边界

```
外部输入                    DLP Manager                    沙箱应用
    │                           │                            │
    │  1.文件URI/Want参数        │                            │
    │──────────────────────────>│                            │
    │                           │  2.解析/验证/解密            │
    │                           │───────────────────────────>│
    │                           │  3.沙箱中打开文件            │
    │                           │<───────────────────────────│
    │                           │                            │
```

**关键信任边界**:
1. **入口边界** - Want参数校验（`MainAbilityEx.checkValidWant:197`）
2. **文件边界** - DLP文件解析和验证
3. **权限边界** - 用户权限检查
4. **沙箱边界** - 沙箱应用隔离

---

## 关键结论

1. **系统级应用**: DLP Manager是预置系统应用，需要系统签名和特殊权限
2. **服务依赖**: 核心功能依赖DLP Permission Service系统服务
3. **域账号强依赖**: 当前实现主要支持域账号（DOMAIN_ACCOUNT）认证
4. **沙箱安全**: 通过沙箱机制实现文件内容的隔离访问
5. **无N-API**: 本应用为纯ArkTS应用，对外接口为Ability调用（非N-API）

---

## 相关链接

- [架构设计](10_Architecture.md) - 查看详细架构
- [对外接口](30_Public_API.md) - 查看调用方式
- [安全风险](60_Security_Analysis.md) - 查看安全分析
