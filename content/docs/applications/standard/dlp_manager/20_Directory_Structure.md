# 目录结构与模块职责

## 目的

本文档描述DLP Manager项目的目录结构，解释各目录的职责和关键文件，帮助开发者快速定位代码。

## 适用范围

- 新加入项目的开发人员
- 需要查找特定功能的开发者
- 进行代码评审的人员

---

## 顶层目录

```
applications/standard/dlp_manager/
├── AppScope/                   # 应用全局配置和资源
├── entry/                      # 主模块（Entry Module）
│   └── src/
│       ├── main/               # 主代码（非测试）
│       │   ├── ets/            # ArkTS/ETS源码
│       │   └── resources/      # 资源文件
│       └── ohosTest/           # 测试代码（本文档不涉及）
├── figures/                    # README图片
├── signature/                  # 签名证书
├── wiki/                       # 工程Wiki（本文档）
├── BUILD.gn                    # GN构建脚本
├── bundle.json                 # 项目元数据
└── README_zh.md                # 项目README
```

---

## 主代码目录 (entry/src/main/ets)

### 总体结构

```
entry/src/main/ets/
├── Ability/                    # Ability实现（系统入口点）
│   ├── data/                   # RPC数据定义
│   └── *.ets                   # 各Ability实现
├── OpenDlpFile/                # DLP文件打开处理
│   ├── manager/                # 业务管理器
│   ├── handler/                # 处理器
│   ├── data/                   # 数据模型
│   ├── ViewProcessor/          # 视图处理
│   └── common/                 # 公共工具
├── pages/                      # UI页面
├── rpc/                        # RPC通信层
├── manager/                    # 业务管理器
├── common/                     # 公共工具层
│   ├── FileUtils/              # 文件工具
│   ├── huks/                   # 加密工具
│   ├── AlertMessage/           # 提示消息
│   └── enum/                   # 枚举定义
├── bean/                       # 数据Bean
│   ├── request/                # 请求数据
│   ├── response/               # 响应数据
│   ├── data/                   # 实体数据
│   └── base/                   # 基础Bean
└── convertor/                  # 数据转换器
```

---

## 各目录详细说明

### 1. Ability/ - Ability实现

**职责**: 定义应用的Ability组件，作为系统入口点

**关键文件**:

| 文件 | 类型 | 职责 |
|------|------|------|
| `AbilityStage.ets` | AbilityStage | 应用生命周期管理 |
| `MainAbilityEx.ets` | UIExtensionAbility | 主入口，处理DLP文件权限设置和修改（`entry/src/main/ets/Ability/MainAbilityEx.ets:54`） |
| `ViewAbility.ets` | ServiceExtensionAbility | 处理DLP文件打开请求（`entry/src/main/ets/Ability/ViewAbility.ets:31`） |
| `DataAbility.ets` | ServiceExtensionAbility | 数据服务，维护历史记录 |
| `DlpFileProcessAbility.ets` | ServiceExtensionAbility | DLP文件处理服务 |
| `EncryptedSharingAbility.ets` | ServiceExtensionAbility | 加密分享功能 |
| `DialogUIExtAbility.ets` | UIExtensionAbility | 通用对话框UI |
| `OpeningDialogUIExtAbility.ets` | UIExtensionAbility | "正在打开"对话框 |
| `PhoneDialogUIExtAbility.ets` | UIExtensionAbility | 手机专用对话框 |
| `ClearDlpCacheAbility.ets` | ServiceExtensionAbility | 清理DLP缓存 |

**RPC数据定义** (`Ability/data/`):
- `IIdlDlpRpcServiceTs/` - DLP RPC服务IDL定义
  - `i_id_dlpRpc_service.ets` - 接口定义
  - `id_dlpRpc_service_proxy.ets` - 代理实现
  - `id_dlpRpc_service_stub.ets` - Stub实现

### 2. OpenDlpFile/ - DLP文件打开处理

**职责**: 处理DLP文件打开的核心逻辑

**子目录**:

#### manager/ - 业务管理器

| 文件 | 职责 |
|------|------|
| `OpenDlpFileManager.ets` | DLP文件打开状态管理（单例，线程安全）（`entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets:38`） |
| `OpeningDialogManager.ets` | "正在打开"对话框管理 |
| `StartFileManager.ets` | 文件管理器启动管理 |
| `ErrorManager.ets` | 错误处理管理 |
| `ApplyEfficiencyManager.ets` | 应用效率管理 |

#### handler/ - 处理器

| 文件 | 职责 |
|------|------|
| `FileParseHandler.ets` | DLP文件解析处理 |
| `AccountHandler.ets` | 账号登录处理 |
| `DecryptHandler.ets` | 文件解密处理 |
| `StartSandboxHandler.ets` | 沙箱启动处理 |
| `FileIdHandler.ets` | 文件ID处理 |
| `ErrorHandler.ets` | 错误处理 |

#### data/ - 数据模型

| 文件 | 职责 |
|------|------|
| `OpenDlpFileData.ets` | DLP文件打开请求数据 |
| `DecryptContent.ets` | 解密内容数据 |

#### ViewProcessor/ - 视图处理

| 文件 | 职责 |
|------|------|
| `ViewProcessor.ets` | DLP文件打开处理主流程（`entry/src/main/ets/OpenDlpFile/ViewProcessor/ViewProcessor.ets:45`） |

#### common/ - 公共工具

| 文件 | 职责 |
|------|------|
| `DlpFileOpenReport.ets` | DLP文件打开上报 |
| `DlpFileOpenReportUtils.ets` | 上报工具 |
| `DataUtils/` | 数据处理工具 |
| `OpenDlpFileError/` | 错误定义 |

### 3. pages/ - UI页面

**职责**: ArkTS UI页面定义

| 文件 | 职责 |
|------|------|
| `encryptionProtection.ets` | 加密保护设置页（主页面）（`entry/src/main/ets/pages/encryptionProtection.ets:87`） |
| `changeEncryption.ets` | 修改加密设置页 |
| `permissionStatus.ets` | 权限状态查看页 |
| `alert.ets` | 错误提示页 |
| `encryptionSuccess.ets` | 加密成功页 |
| `encryptedSharing.ets` | 加密分享页 |
| `OpeningDialog.ets` | "正在打开"对话框页 |
| `PhoneDialog.ets` | 手机对话框页 |

### 4. rpc/ - RPC通信层

**职责**: 跨进程通信（RPC）实现

| 文件/目录 | 职责 |
|-----------|------|
| `CredConnectService.ets` | 凭证服务连接（`entry/src/main/ets/rpc/CredConnectService.ets:25`） |
| `CredCallbackStub.ets` | 凭证回调Stub |
| `DlpPermissionAbilityServiceStub.ets` | DLP权限服务Stub（`entry/src/main/ets/rpc/DlpPermissionAbilityServiceStub.ets:25`） |
| `CredConnectServiceDomain.ets` | 域账号服务连接 |
| `CredConnectServiceFileId.ets` | 文件ID服务连接 |
| `ViewAbility/` | ViewAbility RPC实现 |
| `OpeningDialog/` | OpeningDialog RPC实现 |
| `ClearDlpCache/` | 清理缓存 RPC实现 |

### 5. manager/ - 业务管理器

**职责**: 业务逻辑管理

| 文件 | 职责 |
|------|------|
| `AccountManager.ets` | 账号管理（域账号相关） |
| `AccountAssociationManager.ets` | 账号关联管理 |

### 6. common/ - 公共工具层

**职责**: 通用工具类和常量

| 文件/目录 | 职责 |
|-----------|------|
| `constant.ets` | 全局常量定义（错误码、权限、配置）（`entry/src/main/ets/common/constant.ets:16`） |
| `dlpClass.ets` | DLP相关类定义（`entry/src/main/ets/common/dlpClass.ets:23`） |
| `HiLog.ets` | 日志工具 |
| `GlobalContext.ets` | 全局上下文 |
| `Result.ets` | 结果封装 |
| `ResultMsg.ets` | 结果消息 |
| `StorageUtil.ets` | 存储工具 |
| `Singleton.ets` | 单例基类 |
| `systemUtils.ets` | 系统工具 |
| `UIContextUtil.ets` | UI上下文工具 |
| `CommonUtil.ets` | 通用工具 |
| `AppStorageMgr.ets` | AppStorage管理 |
| `AppStorageConstant.ets` | 存储常量 |
| `CounterLock.ets` | 计数锁 |
| `CommonEventManager.ets` | 公共事件管理 |
| `ReportSecurityEventUtil.ets` | 安全事件上报 |
| `ReportToBigDataUtil.ets` | 大数据上报 |
| `SupportTypesConfig.ets` | 支持类型配置 |
| `AssocUtil/` | 账号关联工具 |

#### FileUtils/ - 文件工具

| 文件 | 职责 |
|------|------|
| `utils.ets` | 文件工具函数（核心工具）（`entry/src/main/ets/common/FileUtils/utils.ets:136`） |
| `FileUtils.ets` | 文件工具类 |

#### huks/ - 加密工具

| 文件 | 职责 |
|------|------|
| `HuksCipherUtil.ets` | HUKs加密工具（`entry/src/main/ets/common/huks/HuksCipherUtil.ets:22`） |
| `HuksProperties.ets` | HUKs属性配置 |

### 7. bean/ - 数据Bean

**职责**: 数据模型定义

| 目录 | 职责 |
|------|------|
| `request/` | 请求数据Bean |
| `response/` | 响应数据Bean |
| `data/` | 实体数据Bean |
| `base/` | 基础Bean类 |

### 8. convertor/ - 数据转换器

**职责**: 数据格式转换

| 文件 | 职责 |
|------|------|
| `DomainAccountConvertor.ts` | 域账号数据转换 |

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Ability)                          │
│  MainAbilityEx    ViewAbility    DataAbility    OtherAbilities   │
└──────────┬────────────────────────┬──────────────────────────────┘
           │                        │
           ▼                        ▼
┌──────────────────┐        ┌──────────────────┐
│   OpenDlpFile    │        │     Pages        │
│   (打开处理)      │        │   (UI页面)        │
└──────────┬───────┘        └──────────────────┘
           │
           ▼
┌──────────────────┐        ┌──────────────────┐
│      RPC         │<──────>│     Manager      │
│   (跨进程通信)    │        │   (业务管理)      │
└──────────┬───────┘        └────────┬─────────┘
           │                         │
           ▼                         ▼
┌──────────────────────────────────────────────┐
│              Common (公共工具层)               │
│   constant   dlpClass   FileUtils   huks     │
└──────────────────────────────────────────────┘
```

---

## 关键文件速查

### 按功能查找

| 功能 | 文件路径 |
|------|---------|
| 查看DLP文件打开流程 | `OpenDlpFile/ViewProcessor/ViewProcessor.ets` |
| 查看权限检查逻辑 | `common/FileUtils/utils.ets:173` |
| 查看加密设置页面 | `pages/encryptionProtection.ets` |
| 查看错误码定义 | `common/constant.ets:134` |
| 查看DLP数据类 | `common/dlpClass.ets` |
| 查看RPC服务连接 | `rpc/CredConnectService.ets` |
| 查看文件解析 | `OpenDlpFile/handler/FileParseHandler.ets` |
| 查看沙箱启动 | `OpenDlpFile/handler/StartSandboxHandler.ets` |

### 按入口查找

| 入口 | 文件路径 |
|------|---------|
| 应用入口 | `Ability/AbilityStage.ets` |
| DLP设置入口 | `Ability/MainAbilityEx.ets` |
| DLP打开入口 | `Ability/ViewAbility.ets` |
| 主页面 | `pages/encryptionProtection.ets` |

---

## 相关链接

- [架构设计](10_Architecture.md) - 了解模块间协作
- [内部接口](40_Inner_API.md) - 了解模块接口定义
- [安全风险](60_Security_Analysis.md) - 了解关键代码安全风险
