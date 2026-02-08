# FilePicker 项目概览

## 目的

本文档提供 FilePicker 应用的整体概览，帮助开发者快速理解项目的定位、边界、核心能力和运行环境。

## 适用范围

本文档适用于：
- OpenHarmony 应用开发者，需要集成文件选择功能
- FilePicker 二次开发者，需要扩展功能
- 系统维护人员，需要理解架构设计

## 关键结论

1. **FilePicker 是系统级应用**，预置在 OpenHarmony 设备中，提供文件选择和保存功能
2. **纯 ArkTS/TypeScript 实现**，无 N-API 模块，通过系统原生 API（@ohos.*）实现功能
3. **采用 Ability 框架**，不使用传统 ServiceAbility/SystemAbility，通过 Want 对象进行通信
4. **支持多种启动模式**：choose（选择文件）、save（保存文件）、batchAuth（批量授权）、download（下载授权）
5. **完善的权限授权机制**：URI 权限授权、批量授权、持久化授权

## 项目定位与边界

### 定位

FilePicker 是 OpenHarmony 系统预置应用，为第三方应用提供**文件选择**和**文件保存**功能。

**核心价值**：
- 统一的文件选择 UI，提升用户体验一致性
- 安全的权限授权机制，保护用户数据安全
- 支持跨应用文件访问（通过 URI 权限）

### 边界

FilePicker **不提供**以下功能：
- ❌ 文件内容预览（除音频/视频缩略图）
- ❌ 文件编辑功能
- ❌ 文件上传/下载到网络
- ❌ 云存储集成
- ❌ 文件加密/解密

### 关键特征

| 特征 | 说明 | 证据 |
|------|------|-------|
| 系统应用 | 预置在设备中，签名证书固定 | `AppScope/app.json5:3` - bundleName: "com.ohos.filepicker" |
| 多 Ability 模式 | 支持 UIAbility 和 UIExtensionAbility | `entry/src/main/module.json5:17,37` |
| 纯 ArkTS 实现 | 无 C/C++ 代码 | 全项目搜索：无 N-API 模块 |
| 文件类型过滤 | 支持按后缀、MIME 类型过滤 | `FilePickerUtil.ets:294-346` |
| 批量选择 | 支持单选和多选模式 | `FilePickerUtil.ets:253-282` |

## 核心能力

### 1. 文件选择能力

**功能描述**：允许用户从本地文件系统中选择文件或文件夹。

**关键特性**：
- 支持单选和多选模式（FILE/FOLDER/MIX）
- 支持文件后缀过滤（如 `.jpg`, `.png`）
- 支持 MIME 类型过滤（如 `image/*`）
- 支持设置默认选择路径
- 支持最大选择数量限制

**证据**：
- Want 参数定义：`entry/src/main/ets/base/utils/FilePickerUtil.ets:70-107`
- 选择器 UI：`entry/src/main/ets/pages/browser/storage/MyPhone.ets:45`

### 2. 文件保存能力

**功能描述**：允许用户选择保存位置并创建新文件。

**关键特性**：
- 支持指定文件名和后缀
- 支持多文件保存（自动重命名避免冲突）
- 支持文件后缀选项（用户可选择）
- 自动处理同名文件冲突（添加序号）

**证据**：
- 保存页面：`entry/src/main/ets/pages/PathPicker.ets:39`
- 文件创建：`entry/src/main/ets/pages/PathPicker.ets:95-152`

### 3. 批量授权能力

**功能描述**：为大量文件（>50 个）提供批量 URI 授权机制。

**关键特性**：
- 使用 UDMF 统一数据通道
- 任务池并发处理
- 支持 read/write 权限授权
- 支持持久化权限授权

**证据**：
- 批量授权任务：`entry/src/main/ets/taskpool/task/BatchGrantPermissionTask.ets:21`
- 任务管理器：`entry/src/main/ets/taskpool/manager/TaskManager.ts:28`
- UDMF 集成：`FilePickerUtil.ets:146-179`

### 4. 下载授权能力

**功能描述**：为浏览器下载提供安全保存位置授权。

**关键特性**：
- 在 Download 目录下创建应用子目录
- URI 权限授权
- 显示应用图标和名称（提升用户体验）

**证据**：
- 下载授权页面：`entry/src/main/ets/pages/DownloadAuth.ets:36`
- 目录创建：`DownloadAuth.ets:76-80`
- 权限授权：`DownloadAuth.ets:82`

### 5. 音频选择能力

**功能描述**：独立的音频选择器，提供音频文件选择功能。

**关键特性**：
- 纯音频类型文件选择
- 系统 Picker 集成
- 音频播放预览

**证据**：
- AudioPicker 模块：`audiopicker/src/main/ets/audiopickerability/AudioPickerUIExtensionAbility.ets:31`
- 模块配置：`audiopicker/src/main/module.json5:54`

## 运行环境

### 系统要求

| 要求 | 版本 | 证据 |
|------|-------|-------|
| OpenHarmony | API 9+ | `build-profile.json5:7-8` - compileSdkVersion: 23 |
| 运行时 OS | OpenHarmony | `build-profile.json5:9` |
| 支持设备 | default, tablet | `entry/src/main/module.json5:8-11` |
| 签名模式 | 系统签名 | `build-profile.json5:12-24` |

### 权限要求

应用声明了以下权限（详细列表见 [权限机制](06_Permissions.md)）：

**媒体库权限**：
- `ohos.permission.MEDIA_LOCATION`
- `ohos.permission.READ_MEDIA`
- `ohos.permission.WRITE_MEDIA`

**文件管理权限**：
- `ohos.permission.FILE_ACCESS_MANAGER`

**特权权限**：
- `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED`
- `ohos.permission.PROXY_AUTHORIZATION_URI`

**音频选择器权限**：
- `ohos.permission.INTERNET`
- `ohos.permission.READ_AUDIO`
- `ohos.permission.WRITE_AUDIO`

**证据**：`entry/src/main/module.json5:44-82`, `audiopicker/src/main/module.json5:14-46`

### 系统依赖

FilePicker 依赖以下系统模块：

| 模块 | 用途 | 证据 |
|--------|-------|-------|
| @ohos.file.fileAccess | 文件系统访问 | `FileAccessExec.ets:23` |
| @ohos.file.fs | 文件系统 API | `FileAccessExec.ets:30` |
| @ohos.data.preferences | 本地存储 | `FilePickerUIExtAbility.ets:19` |
| @ohos.app.ability.UIAbility | UI 能力 | `MainAbility.ets:16` |
| @ohos.app.ability.UIExtensionAbility | UI 扩展能力 | `FilePickerUIExtAbility.ets:16` |
| @ohos.router | 路由导航 | `MyPhone.ets:21` |
| @kit.MediaLibraryKit | 媒体库 | `MyPhone.ets:38` |
| @ohos.multimedia.image | 图像处理 | `MyPhone.ets:30` |
| @ohos.bundle.bundleResourceManager | 包资源管理 | `FilePickerUIExtAbility.ets:26` |
| @ohos.application.uriPermissionManager | URI 权限管理 | `FilePickerUtil.ets:123` |
| @kit.ArkData | 统一数据通道 | `FilePickerUtil.ets:29` |

## 关键概念

### Ability 模型

OpenHarmony 的 Ability 是组件的基本单元，类似于 Android 的 Activity/Service。FilePicker 使用以下类型：

| Ability 类型 | 用途 | 实现位置 |
|------------|-------|----------|
| UIAbility | 带界面的完整 Ability | `MainAbility.ets:28` |
| UIExtensionAbility | UI 扩展能力（系统 Picker） | `FilePickerUIExtAbility.ets:32` |

### Want 通信

Want 是 OpenHarmony 中用于组件间通信的数据结构，包含：
- `action`：操作类型（如 `ohos.want.action.OPEN_FILE`）
- `parameters`：自定义参数（文件过滤、选择模式等）
- `bundleName`：目标应用包名
- `abilityName`：目标 Ability 名称

**证据**：`FilePickerUtil.ets:70-107`

### URI 权限

URI（Uniform Resource Identifier）是 OpenHarmony 文件访问的安全机制：
- 文件通过 URI 访问，而非直接文件路径
- 权限通过 `grantUriPermission` 授予
- 支持读/写/持久化权限标志

**证据**：`FilePickerUtil.ets:123-144`

### UDMF

UDMF（Unified Data Management Framework）是统一数据管理框架：
- 用于跨应用数据共享
- FilePicker 用于批量文件授权（>50 个文件）
- 通过 udKey 访问共享数据

**证据**：`FilePickerUtil.ets:146-179`

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 了解代码组织
- [架构设计](02_Architecture.md) - 深入理解架构
- [对外 API](03_Public_API.md) - 学习如何调用 FilePicker
