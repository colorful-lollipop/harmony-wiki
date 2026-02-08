# 02_目录结构与模块职责

> 本文档基于代码证据编写，证据来源见各章节引用。

## 完整目录树

```
admin_provisioning/
├── AppScope/                                  # 应用全局配置
│   ├── app.json                               # 应用配置文件
│   └── resources/                             # 应用级资源
│       ├── base/                              # 默认资源
│       ├── en_AS.element/                     # 英语资源
│       └── zh_CN.element/                     # 中文资源
├── BUILD.gn                                   # GN 构建入口
├── LICENSE                                    # Apache-2.0 许可证
├── OAT.xml                                   # OpenHarmony 审计表
├── README.md                                 # 英文项目说明
├── README_zh.md                              # 中文项目说明
├── build-profile.json5                       # 构建配置
├── doc/                                      # 文档目录
│   └── Instructions.md                       # 使用说明
├── entry/                                    # 入口模块 ⭐
│   └── src
│       └── main
│           ├── ets/                          # ArkTS 源码 ⭐
│           │   ├── Application/
│           │   │   └── AbilityStage.ts       # Ability 生命周期
│           │   └── MainAbility/              # 主要 Ability
│           │       ├── MainAbility.ts         # DA 入口 (MainAbility)
│           │       ├── AutoManagerAbility.ts   # SDA 入口 (AutoManager)
│           │       └── UIExtensionAbility.ets # UI 扩展能力
│           ├── pages/                        # 页面组件 ⭐
│           │   ├── applicationInfo.ets        # 应用信息页
│           │   ├── autoManager/               # SDA 相关页面
│           │   │   ├── managerStart.ets       # 管理开始页
│           │   │   ├── loadingInfo.ets       # 加载页面
│           │   │   ├── termsShowPage.ets     # 条款展示页
│           │   │   ├── termsListComponent.ets# 条款列表组件
│           │   │   ├── unitManagerShowPage.ets# 设备管理页
│           │   │   ├── setFinishSuccess.ets  # 设置成功页
│           │   │   ├── setFinishFail.ets     # 设置失败页
│           │   │   └── returnInfo.ets         # 返回信息组件
│           │   ├── byod/                      # BYOD 相关页面
│           │   │   ├── byodActivationPage.ets
│           │   │   └── baseByodAdminPage.ets
│           │   ├── custProvisioning/           # 定制化发放页面
│           │   │   └── custProvisioning.ets
│           │   └── component/                 # 公共组件 ⭐
│           │       ├── headComponent.ets       # 头部组件
│           │       ├── entryComponent.ets     # 条目组件
│           │       ├── permissionListComponent.ets # 权限列表组件
│           │       ├── deployComponent.ets    # 部署组件
│           │       ├── PageComponent.ets      # 页面组件
│           │       └── autoManager/           # SDA 专用组件
│           │           ├── doubleButtonComponent.ets
│           │           └── termsListComponent.ets
│           ├── common/                       # 公共模块 ⭐
│           │   ├── logger.ts                  # 日志工具
│           │   ├── utils.ts                   # 工具函数
│           │   ├── baseData.ets               # 基础数据常量
│           │   ├── myApplicationInfo.ets       # 应用信息接口
│           │   ├── accountManager.ets         # 账户管理
│           │   ├── resetFactory.ets           # 恢复出厂设置
│           │   └── appManagement/             # 应用管理
│           │       └── appDetailData.ets      # 应用详情数据
│           ├── module.json5                   # 模块配置
│           └── resources/                    # 模块资源
│               ├── base/                      # 默认资源
│               ├── en_US.element/             # 英语资源
│               ├── rawfile/                  # 原始文件
│               └── zh_CN.element/            # 中文资源
├── figures/                                  # 图片资源
│   └── adminProvisioning_architecture.png    # 架构图
├── hvigorfile.js                            # Hvigor 构建脚本
├── local.properties                          # 本地配置
├── package.json                             # NPM 包配置
└── signature/                               # 签名证书
    └── adminprovisioning.p7b                # 签名文件
```

## 核心模块职责

### 1. entry/src/main/ets (ArkTS 源码)

**职责**: 存放所有 ArkTS 业务代码

**模块划分**:

| 目录 | 职责 | 关键文件 |
|-----|-----|---------|
| `Application/` | 应用生命周期 | `AbilityStage.ts` |
| `MainAbility/` | Ability 入口 | `MainAbility.ts`, `AutoManagerAbility.ts`, `UIExtensionAbility.ets` |
| `pages/` | UI 页面 | 10+ 个页面文件 |
| `pages/component/` | 可复用组件 | 7+ 个组件 |
| `pages/autoManager/` | SDA 专用页面 | 8+ 个文件 |
| `pages/byod/` | BYOD 页面 | 2 个文件 |
| `common/` | 公共模块 | 6 个核心模块 |

### 2. entry/src/main/resources (资源文件)

**职责**: 存放 UI 资源（字符串、图片、颜色等）

| 目录 | 用途 |
|-----|-----|
| `base/` | 默认资源（中文为主） |
| `en_US.element/` | 英语资源 |
| `zh_CN.element/` | 中文资源（与 base 相同） |
| `rawfile/` | 原始文件（可能包括配置文件） |

### 3. AppScope (应用配置)

**职责**: 应用级全局配置

| 文件 | 用途 |
|-----|-----|
| `app.json` | 应用包名、版本、图标、标签 |
| `resources/` | 应用级资源 |

### 4. signature (签名)

**职责**: 应用签名配置

| 文件 | 用途 |
|-----|-----|
| `adminprovisioning.p7b` | 签名证书文件 |

## 模块依赖关系

```
AbilityStage.ts
    ↓ (生命周期)
MainAbility.ts / AutoManagerAbility.ts / UIExtensionAbility.ets
    ↓ (页面路由)
pages/applicationInfo.ets
pages/autoManager/*.ets
pages/custProvisioning/*.ets
pages/byod/*.ets
    ↓ (组件复用)
pages/component/*.ets
    ↓ (数据层)
common/*.ets
    ↓ (系统 API)
@ohos.enterprise.adminManager
@ohos.bundle
@ohos.account
@ohos.update
...
```

## 文件分类统计

| 类型 | 数量 | 说明 |
|-----|-----|-----|
| ArkTS 文件 (.ets) | 40+ | 页面、组件、公共模块 |
| TypeScript 文件 (.ts) | 2 | 工具类 |
| 配置文件 (.json5/.json) | 5+ | 模块配置、应用配置 |
| 构建文件 (BUILD.gn) | 1 | GN 构建入口 |
| 文档 (.md) | 2+ | README、使用说明 |

## 排除的目录

以下目录**不包含在本 Wiki 分析范围**内（遵循规范）：

- `test/` - 测试代码
- `tests/` - 测试代码
- `build/` - 构建输出
- `node_modules/` - 依赖包

---

*文档版本: 1.0*
