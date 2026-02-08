# 项目概览

## 目的

本文档提供 PrintSpooler 项目的快速概览，帮助新人快速了解项目的定位、核心能力、技术栈和运行环境。

## 适用范围

本文档适用于所有开发者，特别是：

- 新加入项目的新人
- 需要了解项目整体架构的开发者
- 进行项目评估的架构师

## 关键结论

1. **项目定位**：OpenHarmony 预置系统应用，提供打印管理功能
2. **技术栈**：纯 ArkTS/TypeScript 应用，无 C++ 代码
3. **支持协议**：仅支持 IPP 无驱动打印协议（Mopria）
4. **主要能力**：打印预览、打印机发现与连接、任务管理
5. **权限要求**：需要 WiFi、网络、文件访问等 9 个系统权限

---

## 项目基本信息

| 属性 | 值 |
|------|-----|
| 项目名称 | PrintSpooler |
| 包名 | `com.ohos.spooler` |
| 版本 | 1.0.0.10 |
| Vendor | ohos |
| SDK 版本 | API 18 (编译), API 11 (兼容) |
| 运行时 | OpenHarmony |
| 代码类型 | 纯 ArkTS/TypeScript |

**证据**：
- 包名：`AppScope/app.json5:3`
- 版本：`AppScope/app.json5:5-6`
- SDK 版本：`build-profile.json5:9-10`

---

## 核心能力

PrintSpooler 提供以下核心功能：

### 1. 打印预览
- 支持图片预览和动态刷新
- 支持页面方向、边距、彩色等设置
- 实时显示打印效果

**证据**：
- 预览组件：`entry/src/main/ets/pages/component/PreviewComponent.ets`
- 文件工具：`entry/src/main/ets/Common/Utils/FileUtil.ts`

### 2. 打印机发现与连接
- 支持 Wifi P2P 发现打印机
- 支持 mDNS 服务发现
- 支持连接管理

**证据**：
- P2P 发现：`feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts`
- mDNS 发现：`feature/ippPrint/src/main/ets/common/discovery/MdnsDiscovery.ts`
- 连接管理：`feature/ippPrint/src/main/ets/common/connect/P2pPrinterConnection.ts`

### 3. 打印参数设置
- 页面范围选择
- 纸张尺寸和类型
- 打印质量、双面打印
- 彩色/黑白模式

**证据**：
- 常量定义：`common/src/main/ets/model/Constants.ts`（枚举：PrintRangeType, PageDirection, PrintQuality, Duplex, ColorCode, MediaType）

### 4. 打印任务管理
- 任务创建和下发
- 任务状态监听
- 任务队列管理

**证据**：
- 任务控制器：`entry/src/main/ets/Controller/PrintJobManager.ts`
- 任务管理页面：`entry/src/main/ets/pages/JobManagerPage.ets`

---

## 技术栈

### 编程语言
- **主要语言**：ArkTS (TypeScript 超集)
- **总代码量**：87 个 .ets/.ts 源代码文件

### 框架和库

| 技术 | 用途 | 证据 |
|------|------|------|
| @kit.BasicServicesKit | 打印框架 API | 29 个文件导入 |
| @ohos.wifi | WiFi P2P 发现和连接 | README.md 示例代码 |
| @ohos.file.fs | 文件操作 | FileUtil.ts |
| @ohos.multimedia.image | 图像处理 | PreviewComponent.ets |
| @ohos.bundle.bundleManager | Bundle 信息获取 | MainAbility.ets:49 |
| @ohos.app.ability | Ability API | 所有 Ability 文件 |
| @ohos.data.preferences | 应用存储 | PreferencesAdapter.ts |

### 构建系统
- **构建工具**：Hvigor
- **配置文件**：build-profile.json5, module.json5

---

## 约束与限制

### 功能约束
- **仅支持 IPP 协议**：不支持需要单独驱动的打印机
- **不支持扫描功能**：当前版本仅支持打印

**证据**：
- README.md:13: "当前只支持对接支持ipp无驱动打印协议的打印机"

### 运行环境
- **系统要求**：OpenHarmony API 11+
- **硬件要求**：支持 WiFi 的设备
- **权限要求**：需要 9 个系统权限（详见 [安全风险评审](08_Security_Review.md)）

---

## 模块概览

项目包含 4 个模块：

| 模块 | 类型 | 职责 | 文件数 |
|------|------|------|--------|
| entry | HAP (entry) | 主入口、UI 界面、业务逻辑 | 32 |
| common | HAR | 公共工具、常量定义 | 24 |
| ippPrint | HAR | IPP 打印功能、P2P 发现 | 20 |
| driverEntry | Feature | 驱动扩展、CUPS 配置 | - |

**证据**：
- entry 模块：`entry/src/main/module.json5:3`
- common 模块：`common/src/main/module.json5:3`
- ippPrint 模块：`feature/ippPrint/src/main/module.json5:3`
- driverEntry 模块：`driverEntry/src/main/module.json5:3`

---

## 数据统计

| 指标 | 数值 |
|------|------|
| 源代码文件数 | 87 |
| Ability 数 | 2 (MainAbility, JobManagerAbility) |
| ExtensionAbility 数 | 2 (PrintExtension, PrintServiceExtAbility) |
| 权限声明数 | 9 |
| 页面数 | 10+ |

---

## 相关跳转

- [项目定位与边界](01_Project_Scope.md) - 详细的功能范围和边界说明
- [目录结构](02_Directory_Structure.md) - 代码组织细节
- [架构设计](03_Architecture.md) - 组件关系和数据流
- [对外 API](04_External_API.md) - OpenHarmony 系统 API 使用
