# 目录结构与模块职责

## 目的

本文档说明 PrintSpooler 项目的目录组织结构、各模块的职责划分和代码组织方式。

## 适用范围

本文档适用于：

- 代码导航
- 模块定位和职责理解
- 代码组织参考

## 关键结论

1. **4 个主要模块**：entry（主入口）、common（公共库）、ippPrint（IPP 功能）、driverEntry（驱动扩展）
2. **代码语言**：纯 ArkTS/TypeScript（87 个源代码文件）
3. **依赖关系**：entry 依赖 common 和 ippPrint，ippPrint 依赖 common
4. **无测试代码引用**：本文档不包含 test/、unittest/ 等测试目录

---

## 顶层目录结构

```
printspooler/
├── AppScope/                    # 应用作用域
│   ├── app.json5               # 应用配置（包名、版本）
│   └── resources/               # 全局资源
├── common/                     # 公共模块（HAR）
│   ├── build-profile.json5       # 模块构建配置
│   ├── oh-package.json5         # 模块依赖
│   └── src/main/
│       ├── module.json5         # 模块配置
│       └── ets/              # ArkTS 源代码
├── entry/                     # 主入口模块（HAP）
│   ├── build-profile.json5       # 模块构建配置
│   ├── oh-package.json5         # 模块依赖
│   └── src/main/
│       ├── module.json5         # 模块配置（Ability、权限）
│       ├── resources/           # 资源文件
│       └── ets/              # ArkTS 源代码
├── feature/                   # 功能模块
│   └── ippPrint/            # IPP 打印模块（HAR）
│       ├── build-profile.json5
│       ├── oh-package.json5
│       └── src/main/
│           ├── module.json5
│           └── ets/         # ArkTS 源代码
├── driverEntry/              # 驱动入口模块（Feature）
│   ├── build-profile.json5
│   ├── oh-package.json5
│   ├── libs/                # 驱动库文件
│   └── src/main/
│       ├── module.json5
│       └── ets/            # ArkTS 源代码
├── figures/                 # 文档图片
├── signature/               # 签名文件
├── wiki/                    # 工程文档
└── build-profile.json5        # 根构建配置
```

---

## 模块详细结构

### 1. common 模块

**模块类型**：HAR（Harmony Archive）

**职责**：
- 提供公共工具类和常量定义
- 定义打印相关的枚举和数据模型
- 提供全局对象和存储辅助类

**代码组织**：
```
common/src/main/ets/
├── framework/              # 框架层
│   ├── Print.ts            # 打印框架封装
│   ├── ConfigManager.ts    # 配置管理
│   └── Interfaces.ts       # 接口定义
├── utils/                 # 工具类
│   ├── MediaSizeUtil.ts    # 纸张尺寸工具
│   ├── PermissionUtils.ts   # 权限工具
│   ├── SingletonHelper.ts  # 单例辅助
│   ├── DateUtils.ts        # 日期工具
│   ├── CopyUtil.ts        # 复制工具
│   ├── PrintUtil.ts       # 打印工具
│   ├── UuidGenerator.ts   # UUID 生成
│   ├── PrinterUtils.ts    # 打印机工具
│   ├── Log.ts            # 日志工具
│   ├── CheckEmptyUtils.ts # 空值检查
│   ├── GlobalObject.ts    # 全局对象
│   └── StringUtil.ts      # 字符串工具
└── model/                # 数据模型
    ├── PrintBean.ts       # 打印 Bean
    ├── PrintConstants.ts  # 打印常量
    ├── MediaSize.ts       # 纸张尺寸
    ├── MediaType.ts       # 纸张类型
    ├── ErrorMessage.ts    # 错误消息
    ├── GlobalThisHelper.ts   # 全局辅助
    ├── Constants.ts      # 全局常量
    └── GlobalThisStorageKey.ts  # 存储键
```

**文件数**：24 个 .ets/.ts 文件

**关键文件**：
- `Constants.ts:141` - Bundle name 定义：`com.ohos.spooler`
- `PrintConstants.ts:16-26` - IPP 相关常量
- `Log.ts` - 统一日志接口

**依赖**：无

---

### 2. entry 模块

**模块类型**：HAP（Harmony Ability Package）

**职责**：
- 应用主入口和 UI 界面
- 打印任务管理和预览
- 打印扩展能力实现
- 业务逻辑和页面路由

**代码组织**：
```
entry/src/main/ets/
├── MainAbility/            # 主 Ability
│   ├── MainAbility.ets    # 主 UIAbility（UIExtensionAbility）
│   └── JobManagerAbility.ts # 任务管理 Ability
├── ServiceExtAbility/     # 打印扩展
│   └── PrintExtension.ts  # PrintExtensionAbility 实现
├── Controller/            # 业务控制器
│   ├── PrintJobManager.ts      # 打印任务管理器
│   ├── PrintExtensionController.ts # 打印扩展控制器
│   └── PrinterDiscController.ts # 打印机发现控制器
├── Model/                # 数据模型
│   ├── SelectionModel.ets   # 选择模型
│   ├── PrinterDiscModel.ts # 打印机发现模型
│   ├── FileModel.ts        # 文件模型
│   ├── PrintExtensionModel.ts # 打印扩展模型
│   └── JobViewModel/      # 任务视图模型
│       ├── PrintPageModel.ets
│       └── PrintJobViewModel.ets
├── Common/               # 公共代码
│   ├── Utils/            # 工具类
│   │   ├── FileUtil.ts
│   │   └── Util.ts
│   └── Adapter/          # 适配器
│       ├── WifiP2pHelper.ts
│       ├── PrintAdapter.ts
│       ├── PreferencesAdapter.ts
│       └── AppStorageHelper.ts
├── workers/              # Worker 线程
│   ├── PrintWorker.ts      # 打印 Worker
│   └── DiscoveryWorker.ts # 发现 Worker
└── pages/               # 页面
    ├── PrintPage.ets      # 打印页面
    ├── JobManagerPage.ets  # 任务管理页面
    ├── AboutPage.ets      # 关于页面
    ├── PrivacyStatementPage.ets # 隐私声明页面
    └── component/        # 页面组件
        ├── PreviewComponent.ets    # 预览组件
        ├── AboutPageComponent.ets
        ├── CusDialogComp.ets
        ├── SelectComponent.ets
        ├── BaseComponent.ets
        ├── PrivacyStatementDialog.ets
        └── PrivacyStatementWebPage.ets
```

**文件数**：32 个 .ets/.ts 文件

**关键文件**：
- `MainAbility.ets:36` - 主 Ability（UIExtensionAbility）
- `JobManagerAbility.ts` - 任务管理 Ability
- `PrintExtension.ts:36` - PrintExtensionAbility 实现
- `PrintPage.ets` - 打印预览主页面
- `JobManagerPage.ets` - 任务管理页面
- `PreviewComponent.ets` - 打印预览组件

**Ability 配置**（entry/src/main/module.json5:24-64）：
- MainAbility - 主 Ability，type: singleton
- JobManagerAbility - 任务管理 Ability
- PrintExtension - 打印扩展
- PrintServiceExtAbility - 打印服务对话框

**权限声明**（entry/src/main/module.json5:66-161）：
- 9 个权限声明（详见 [安全风险评审](08_Security_Review.md)）

**依赖**：
- @ohos/common（common 模块）
- @ohos/ippprint（ippPrint 模块）

---

### 3. ippPrint 模块

**模块类型**：HAR（Harmony Archive）

**职责**：
- IPP 打印协议实现
- Wifi P2P 设备发现和连接
- mDNS 服务发现
- 打印机能力查询和缓存

**代码组织**：
```
feature/ippPrint/src/main/ets/common/
├── discovery/              # 发现相关
│   ├── P2pDiscoveryChannel.ts  # P2P 发现通道
│   ├── P2pDiscovery.ts          # P2P 发现实现
│   ├── P2pMonitor.ts          # P2P 监控
│   ├── DiscoveredPrinter.ts     # 已发现打印机
│   ├── MdnsDiscovery.ts        # mDNS 发现
│   └── Discovery.ts           # 发现抽象
├── connect/               # 连接相关
│   ├── ConnectionListener.ts  # 连接监听器
│   └── P2pPrinterConnection.ts # P2P 打印机连接
├── ipp/                   # IPP 协议
│   ├── CapabilitiesCache.ts  # 能力缓存
│   └── Backend.ts          # 后端实现
├── model/                 # 数据模型
│   ├── WifiModel.ts         # WiFi 模型
│   └── WorkerData.ts       # Worker 数据
├── utils/                 # 工具类
│   ├── WorkerUtil.ts       # Worker 工具
│   ├── CommonUtils.ts      # 公共工具
│   └── P2pUtils.ts        # P2P 工具
├── napi/                  # Native API 封装
│   ├── NativeApi.ts        # 系统打印 API 封装
│   └── LocalPrinterCapabilities.ts # 本地打印机能力
├── LocalDiscoverySession.ts # 本地发现会话
├── LocalPrinter.ts         # 本地打印机
└── PrintServiceAdapter.ts  # 打印服务适配器
```

**文件数**：20 个 .ets/.ts 文件

**关键文件**：
- `P2pDiscoveryChannel.ts` - P2P 发现通道
- `MdnsDiscovery.ts` - mDNS 发现
- `P2pPrinterConnection.ts` - P2P 连接
- `NativeApi.ts:23` - 系统 print API 封装
- `PrintServiceAdapter.ts` - 打印服务适配器

**依赖**：
- @ohos/common（common 模块）
- @kit.BasicServicesKit（print API）
- @ohos.wifi（WiFi API）

---

### 4. driverEntry 模块

**模块类型**：Feature（功能模块）

**职责**：
- 提供打印机驱动扩展
- 配置 CUPS（Common Unix Printing System）
- 配置 SANE（Scanner Access Now Easy）

**代码组织**：
```
driverEntry/src/main/
├── module.json5          # 模块配置
├── syscap.json           # 系统能力
├── resources/            # 资源文件
└── ets/
    └── driverentryability/
        └── MyDriverExtensionAbility.ts  # 驱动扩展 Ability

driverEntry/libs/
└── arm64-v8a/          # 驱动库文件
    ├── rastertopwg       # CUPS filter
    ├── HUAWEI_PixLab_xxx.ppd  # PPD 文件
    ├── libsane-pantumxxx.so      # SANE backend
    └── lpd              # CUPS backend
```

**关键配置**（driverEntry/src/main/module.json5:14-53）：
- DriverExtensionAbility - type: "driver"
- CUPS 配置：
  - cupsFilter: `/print_service/cups/serverbin/filter`
  - cupsPpd: `/print_service/cups/datadir/model`
  - cupsBackend: `/print_service/cups/serverbin/backend`
- SANE 配置：
  - saneBackend: `/print_service/sane/backend`

**依赖**：无

---

## 模块依赖关系

```
┌─────────────┐
│   entry    │  (HAP)
│  (主入口)   │
└──────┬──────┘
       │ depends on
       ├──────────────────┐
       │                  │
┌──────▼──────┐   ┌─────▼────────┐
│   common    │   │   ippPrint   │
│   (HAR)     │   │    (HAR)      │
└─────────────┘   └──────┬───────┘
                        │ depends on
                        │
                   ┌────▼─────┐
                   │  common   │
                   └──────────┘

┌───────────────┐
│ driverEntry   │  (Feature)
│ (驱动扩展)     │
└───────────────┘
```

**依赖说明**：

1. **entry** → 依赖 common 和 ippPrint
   - 使用 common 的工具类和常量
   - 使用 ippPrint 的发现和连接功能

2. **ippPrint** → 依赖 common
   - 使用 common 的日志、常量等工具

3. **driverEntry** → 独立模块
   - 不依赖其他模块
   - 仅通过系统服务与 CUPS/SANE 交互

---

## 代码文件统计

| 模块 | 文件数 | 类型 |
|------|--------|------|
| entry | 32 | .ets/.ts |
| common | 24 | .ets/.ts |
| ippPrint | 20 | .ets/.ts |
| driverEntry | 1+ | .ets + 库文件 |
| **总计** | **87** | 源代码文件 |

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目整体介绍
- [架构设计](03_Architecture.md) - 模块间关系和数据流
- [内部 API](05_Internal_API.md) - 模块间接口详情
