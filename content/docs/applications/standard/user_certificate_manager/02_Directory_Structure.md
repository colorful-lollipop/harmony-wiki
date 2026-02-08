# 目录结构与模块职责

> 本文档描述用户证书管理部件的目录结构、文件组织和模块职责

---

## 目的

本文档帮助开发者快速理解代码组织结构，找到需要的文件和模块。

## 适用范围

- OpenHarmony 应用开发者
- 需要修改代码的技术人员
- 需要理解项目结构的架构师

## 关键结论

1. **模块化设计**: 采用 Model-Presenter-Page (MVP) 分层架构
2. **清晰分层**: 业务逻辑、页面逻辑、UI 界面分离
3. **公共组件**: 公共组件和工具集中在 `common/` 目录
4. **单模块应用**: 主业务逻辑集中在 `certmanager/` 模块
5. **49 个源文件**: ArkTS (.ets/.ts) 文件共 49 个

## 相关跳转

- [00_Overview.md](wiki/00_Overview.md) - 项目概览
- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计
- [05_Internal_API.md](wiki/05_Internal_API.md) - 内部接口

---

## 1. 顶层目录结构

### 1.1 目录树

```
user_certificate_manager/
├── BUILD.gn                    # GN 构建配置
├── bundle.json                 # Bundle 元数据
├── build-profile.json5         # 构建 profile
├── oh-package.json5            # 依赖配置
├── LICENSE                    # Apache License 2.0
├── README.md                  # 项目说明
├── certmanager/               # ⭐ 业务证书管理模块
│   ├── BUILD.gn              # 模块构建配置
│   ├── oh-package.json5       # 模块依赖
│   └── src/main/
│       ├── ets/             # ArkTS 源码
│       │   ├── MainAbility/  # Ability 实现
│       │   ├── model/        # Model 层 (业务逻辑)
│       │   ├── presenter/    # Presenter 层 (页面逻辑)
│       │   ├── pages/        # Pages 层 (UI 界面)
│       │   └── common/       # 公共组件和工具
│       └── resources/       # 资源文件
├── entry/                     # Entry 模块 (HAP 打包)
│   └── src/main/
├── hvigor/                   # 构建工具
├── signature/                 # 签名配置
├── patches/                  # 补丁
├── doc/                      # 文档
└── wiki/                     # Wiki 文档 (本目录)
```

### 1.2 目录职责

| 目录 | 职责 | 说明 |
|------|--------|------|
| `certmanager/` | 主业务模块 | 包含所有证书管理相关代码 |
| `entry/` | HAP 打包入口 | 负责 HAP 最终打包 |
| `AppScope/` | 应用级配置 | 应用名称、版本、图标等 |
| `signature/` | 签名配置 | 签名证书和 profile |
| `hvigor/` | 构建工具 | Hvigor 构建脚本 |
| `patches/` | 补丁 | 第三方依赖补丁 |

**证据位置**: `README.md:30-81`, `BUILD.gn:15-49`

---

## 2. certmanager 模块结构

### 2.1 源码统计

```
certmanager/src/main/ets/
├── MainAbility/          1 个文件  (Ability)
├── model/              6 个文件  (业务逻辑)
├── presenter/          9 个文件  (页面逻辑)
├── pages/             17 个文件  (UI 界面)
├── common/              9 个文件  (公共组件)
└── common/util/         3 个文件  (工具函数)
                        ──────
                         45 个源文件
```

**证据位置**: 统计 `certmanager/src/main/ets/` 目录

### 2.2 MainAbility/ - Ability 实现

| 文件 | 职责 | 说明 |
|------|--------|------|
| `CertPickerUiExtAbility.ets` | UI Extension Ability | 证书选择器入口，支持多种页面类型 |

**关键功能**:
- 支持四种页面类型: 安装页面、授权页面、UKey 认证页面、证书选择器
- 管理全局上下文和会话
- 处理 Want 参数

**证据位置**: `certmanager/src/main/ets/MainAbility/CertPickerUiExtAbility.ets:27-111`

---

## 3. Model 层 - 业务逻辑

### 3.1 模块清单

| 文件 | 职责 | 关键类/函数 |
|------|--------|------------|
| `CertMangerModel.ets` | ⭐ 证书管理核心模型 | 证书/凭据的增删改查 |
| `CheckUserAuthModel.ets` | 用户认证模型 | 生物识别/密码认证 |
| `BundleModel.ets` | 包信息模型 | 获取应用名称和图标 |
| `FileIoModel.ets` | 文件 I/O 模型 | 读取证书文件 |
| `PreventScreenshotsModel.ets` | 防截屏模型 | 设置窗口隐私模式 |
| `CertManagerVo/` | 数据对象 | VO (Value Object) 定义 |

### 3.2 CertMangerModel - 证书管理核心

**职责**: 封装所有证书和凭据管理操作

**主要方法**:

| 方法类别 | 方法名 | 说明 |
|---------|---------|------|
| 列表查询 | `getCertOrCredList()` | 获取证书/凭据列表 |
| 详情查询 | `getCertOrCred()` | 获取证书/凭据详情 |
| 安装操作 | `installCertOrCred()` | 安装证书/凭据 |
| 删除操作 | `deleteCertOrCred()` | 删除证书/凭据 |
| 状态设置 | `setCertStatus()` | 设置证书启用/禁用状态 |
| 授权管理 | `setAppAuth()` | 授权应用访问凭据 |
| 批量删除 | `delAllCertOrCred()` | 删除所有证书/凭据 |

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:89-834`

### 3.3 CheckUserAuthModel - 用户认证

**职责**: 封装用户认证逻辑（生物识别/密码）

**主要方法**:

| 方法名 | 说明 |
|--------|------|
| `isAuthTypeSupported()` | 检查认证类型是否支持 |
| `auth()` | 执行用户认证 |

**认证流程**:
1. 检查支持的认证类型（指纹/密码）
2. 生成 16 字节随机数作为 challenge
3. 创建认证实例并启动
4. 监听认证结果

**证据位置**: `certmanager/src/main/ets/model/CheckUserAuthModel.ets:31-117`

### 3.4 BundleModel - 包信息获取

**职责**: 获取应用名称和图标

**主要方法**:

| 方法名 | 说明 |
|--------|------|
| `getAppInfoList()` | 根据 appUid 获取应用信息 |

**实现**: 使用 `@ohos.bundle.bundleManager` 和 `@ohos.bundle.bundleResourceManager`

**证据位置**: `certmanager/src/main/ets/model/BundleModel.ets:24-49`

### 3.5 FileIoModel - 文件 I/O

**职责**: 读取证书文件和获取文件后缀

**主要方法**:

| 方法名 | 说明 |
|--------|------|
| `getMediaFileData()` | 读取文件内容为 Uint8Array |
| `getMediaFileSuffix()` | 获取文件后缀名 |

**证据位置**: `certmanager/src/main/ets/model/FileIoModel.ets:20-64`

### 3.6 PreventScreenshotsModel - 防截屏

**职责**: 设置窗口隐私模式，防止密码输入时被截屏

**主要方法**:

| 方法名 | 说明 |
|--------|------|
| `PreventScreenshots()` | 设置/取消防截屏 |

**实现**: 调用 `window.setWindowPrivacyMode()`

**证据位置**: `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:24-57`

---

## 4. Presenter 层 - 页面逻辑

### 4.1 模块清单

| 文件 | 职责 |
|------|--------|
| `CmFaPresenter.ets` | 主页面 Presenter |
| `CmInstallPresenter.ets` | 安装页面 Presenter |
| `CmShowUserCaPresenter.ets` | 用户 CA 列表 Presenter |
| `CmShowSysCaPresenter.ets` | 系统 CA 列表 Presenter |
| `CmShowAppCredPresenter.ets` | 用户凭据列表 Presenter |
| `CmShowSysCredPresenter.ets` | 系统凭据列表 Presenter |
| `CmAppCredAuthPresenter.ets` | 授权应用管理 Presenter |
| `CmCertDataSource.ets` | 证书数据源 |

**代码行数**: 约 1020 行

**证据位置**: 统计 `certmanager/src/main/ets/presenter/` 目录

### 4.2 CmFaPresenter - 主页面

**职责**: 主页面逻辑，包括文件选择和路由

**主要方法**:

| 方法名 | 说明 |
|--------|------|
| `routeToNextInstallCert()` | 路由到证书安装页面 |
| `routeToNextInstallEvidence()` | 路由到凭据安装页面 |
| `startInstallCert()` | 启动证书安装流程 |
| `startInstallEvidence()` | 启动凭据安装流程 |

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:30-100`

---

## 5. Pages 层 - UI 界面

### 5.1 页面清单

| 目录/文件 | 说明 |
|-----------|------|
| `certManagerFa.ets` | 用户证书管理主页面 ⭐ |
| `certInstallFromStorage.ets` | 从存储安装证书页面 |
| `certPwdInput.ets` | 密码输入页面 |
| `trustedCa.ets` | CA 证书页面 |
| `cerEvidenceFa.ets` | 凭据页面 |
| `requestAuth.ets` | 应用请求授权界面 |
| `CertificateInstallPage.ets` | 证书安装页面 (Extension) |
| `RequestAuthSheet.ets` | 授权页面 (Extension) |
| `UKeyAuthSheet.ets` | UKey 认证页面 (Extension) |
| `detail/` | 详情页面目录 |
| `picker/` | 半模态选择页面目录 |

### 5.2 详情页面 (detail/)

| 文件 | 说明 |
|------|--------|
| `CaSystemDetailPage.ets` | 系统 CA 详情页面 |
| `CaUserDetailPage.ets` | 用户 CA 详情页面 |
| `CredSystemDetailPage.ets` | 系统凭据详情页面 |
| `CredUserDetailPage.ets` | 用户凭据详情页面 |
| `AuthorizedAppManagementPage.ets` | 授权应用列表详情 |
| `CredPwdInputPage.ets` | 凭据密码输入页面 |

### 5.3 选择页面 (picker/)

| 文件 | 说明 |
|------|--------|
| `CertManagerSheetFa.ets` | 证书管理半模态入口 ⭐ |
| `CaCertPage.ets` | CA 证书半模态页面 |
| `CredListPage.ets` | 凭据半模态页面 |
| `InstallPage.ets` | 安装半模态页面 |

**证据位置**: `certmanager/src/main/resources/base/profile/main_pages.json:2-14`

---

## 6. Common 层 - 公共组件

### 6.1 组件和工具

| 目录/文件 | 职责 |
|-----------|------|
| `GlobalContext.ts` | 全局上下文管理 ⭐ |
| `component/` | 自定义 UI 组件 |
| `component/headComponent.ets` | 头部组件 |
| `component/subEntryComponent.ets` | 子入口组件 |
| `constants/` | 常量定义 |
| `constants/CertManagerConstants.ets` | 证书管理常量 |
| `constants/FileFilterParams.ets` | 文件过滤参数 |
| `util/` | 工具函数 |
| `util/AlignUtils.ets` | 对齐工具 |
| `util/StringUtil.ets` | 字符串工具 |
| `util/SheetParam.ets` | Sheet 参数 |

### 6.2 GlobalContext - 全局上下文

**职责**: 管理全局状态和上下文

**主要属性**:

| 属性 | 类型 | 说明 |
|------|------|------|
| `context` | UIAbilityContext | Ability 上下文 |
| `want` | Want | 启动参数 |
| `pwdStore` | PwdStore | 密码存储 |
| `session` | UIExtensionContentSession | 会话对象 |
| `flag` | Boolean | 标记位 |

**主要方法**:

| 方法名 | 说明 |
|--------|------|
| `getContext()` | 获取全局上下文实例 (单例) |
| `setCmContext()` / `getCmContext()` | 设置/获取 Ability 上下文 |
| `setAbilityWant()` / `getAbilityWant()` | 设置/获取 Want |
| `setPwdStore()` / `getPwdStore()` | 设置/获取密码存储 |
| `setSession()` / `getSession()` | 设置/获取会话 |

**证据位置**: `certmanager/src/main/ets/common/GlobalContext.ts:36-105`

---

## 7. 资源文件

### 7.1 资源目录

```
certmanager/src/main/resources/
├── base/
│   ├── element/         # 基础元素
│   │   ├── string.json # 字符串资源
│   │   ├── color.json  # 颜色资源
│   │   └── float.json  # 浮点数资源
│   ├── media/           # 媒体资源
│   └── profile/         # 配置文件
│       └── main_pages.json # 页面配置
├── en_US/             # 英文资源
│   └── element/
│       └── string.json
├── zh_CN/             # 中文资源
│   └── element/
│       └── string.json
└── rawfile/           # 原始文件
    └── security_privacy.json # 隐私中心配置
```

**证据位置**: `certmanager/src/main/` 目录结构

---

## 8. 模块依赖关系

### 8.1 Model 层依赖

```
CertMangerModel (证书管理核心)
├── @ohos.security.certManager (证书管理服务) ⭐
├── @ohos.security.cert (证书解析)
└── @ohos.hilog (日志)
```

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:21-24`

### 8.2 Presenter 层依赖

```
CmFaPresenter (主页面逻辑)
├── CertMangerModel (业务逻辑) ⭐
├── FileIoModel (文件 I/O)
├── CheckUserAuthModel (用户认证)
├── CmInstallPresenter (安装页面逻辑)
├── @ohos.file.picker (文件选择器)
├── @ohos.router (路由)
└── @ohos.hilog (日志)
```

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:16-25`

### 8.3 Common 层依赖

```
GlobalContext (全局上下文)
├── UIAbilityContext (Ability 上下文) ⭐
├── Want (启动参数)
└── UIExtensionContentSession (会话)
```

**证据位置**: `certmanager/src/main/ets/common/GlobalContext.ts:16-18`

---

**END OF 02_Directory_Structure.md**
