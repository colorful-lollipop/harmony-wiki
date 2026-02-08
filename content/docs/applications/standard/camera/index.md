# Camera 项目概览

## 项目定位

Camera 是 OpenHarmony 标准系统中预置的系统相机应用，为用户提供基础的相机拍摄功能。作为系统核心应用，它直接面向最终用户，同时也是展示 OpenHarmony 多媒体能力的标杆应用。

**证据**: `README_zh.md:1-3`
> 相机应用是OpenHarmony标准系统中预置的系统应用，为用户提供基础的相机拍摄功能，包括预览、拍照、摄像、缩略图显示、跳转相册、多机位协同。

## 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| **预览** | 实时相机画面预览 | `common/src/main/ets/default/camera/CameraService.ts:83` (PreviewOutput) |
| **拍照** | 静态图片拍摄与保存 | `common/src/main/ets/default/camera/CameraService.ts:84` (PhotoOutput) |
| **录像** | 视频录制与存储 | `common/src/main/ets/default/camera/CameraService.ts:86-87` (VideoOutput/AVRecorder) |
| **缩略图** | 拍摄后缩略图显示 | `common/src/main/ets/default/camera/ThumbnailGetter.ts` |
| **相册跳转** | 快速跳转到系统相册 | `product/*/src/main/ets/pages/PreviewArea.ets` |
| **多机位协同** | 分布式设备相机调用 | `product/*/src/main/ets/MainAbility/ExtensionPickerAbility.ts` |
| **设置管理** | 分辨率、定时器等设置 | `common/src/main/ets/default/setting/SettingManager.ts` |

## 运行环境

### 系统要求
- **OpenHarmony API 版本**: 23 (compileSdkVersion)
- **系统类型**: standard (标准系统)
- **设备形态**: Phone (手机)、Tablet (平板)
- **开发语言**: ArkTS (纯 ArkUI-TS)

**证据**: `build-profile.json5:22-24`
```json
{
  "compileSdkVersion": 23,
  "compatibleSdkVersion": 23
}
```

### 应用类型
- **Bundle 名称**: `com.ohos.camera`
- **应用类型**: 系统预置应用 (System App)
- **签名方式**: 系统签名
- **模块类型**: 
  - Entry (应用入口): phone, tablet
  - HAR (静态共享包): common, photo, video, multi

## 关键概念

### 1. 三层架构

```
┌─────────────────────────────────────────┐
│           Product 层 (Entry)             │
│  ┌──────────────┐    ┌──────────────┐   │
│  │    Phone     │    │    Tablet    │   │
│  └──────────────┘    └──────────────┘   │
├─────────────────────────────────────────┤
│           Feature 层 (HAR)               │
│  ┌─────────┬─────────┬────────────────┐ │
│  │  Photo  │  Video  │      Multi      │ │
│  └─────────┴─────────┴────────────────┘ │
├─────────────────────────────────────────┤
│           Common 层 (HAR)                │
│   公共服务、UI组件、状态管理、工具类         │
└─────────────────────────────────────────┘
```

**证据**: `README_zh.md:12-14`
> - Product层：区分不同产品，不同屏幕的各形态
> - Feature层：抽象的公共特性组件集合，每个特性解耦独立可打包为har
> - Common层：负责数据服务、UI组件、工具组、数据持久层、动效层、外部交互层

### 2. 状态管理

使用类似 Redux 的架构：
- **Store**: 全局状态存储 (`common/src/main/ets/default/redux/store`)
- **Action**: 描述状态变更 (`common/src/main/ets/default/redux/actions/Action.ts`)
- **EventBus**: 跨组件事件通信 (`common/src/main/ets/default/worker/eventbus/`)

### 3. 相机抽象层

```
┌──────────────────────────────────────┐
│         CameraService (封装层)        │
├──────────────────────────────────────┤
│  @ohos.multimedia.camera (系统 API)   │
│  - CameraManager                     │
│  - CameraInput                       │
│  - CaptureSession                    │
│  - PhotoOutput / VideoOutput         │
├──────────────────────────────────────┤
│       相机 HAL (系统底层)              │
└──────────────────────────────────────┘
```

## 技术栈

| 层次 | 技术/框架 | 说明 |
|------|-----------|------|
| UI | ArkUI | 声明式 UI 开发框架 |
| 语言 | ArkTS | TypeScript 超集 |
| 状态 | Redux-like | 自定义状态管理 |
| 通信 | EventBus | 跨组件事件总线 |
| 存储 | Preferences + RDB | 键值 + 关系型数据库 |
| 线程 | Worker | 多线程处理 |

## 外部依赖

### 系统 SDK
- `@ohos.multimedia.camera` - 相机核心功能
- `@ohos.multimedia.image` - 图片处理
- `@ohos.multimedia.media` - 媒体录制
- `@ohos.app.ability.UIAbility` - Ability 框架
- `@ohos.window` - 窗口管理
- `@ohos.display` - 显示管理
- `@ohos.deviceInfo` - 设备信息

### 开发依赖
- `@ohos/hypium: 1.0.6` - 测试框架

## 快速开始

### 阅读路径

1. **[目录结构](01_Directory_Structure.md)** - 理解代码组织
2. **[架构说明](02_Architecture.md)** - 掌握设计思想
3. **[对外 API](03_External_API.md)** - 了解系统调用
4. **[安全风险](06_Security.md)** - 安全注意事项

### 关键入口点

| 入口 | 路径 | 说明 |
|------|------|------|
| 应用入口 | `product/*/src/main/ets/MainAbility/MainAbility.ts` | Ability 生命周期 |
| 主页面 | `product/phone/src/main/ets/pages/index.ets` | Phone 主界面 |
| 相机服务 | `common/src/main/ets/default/camera/CameraService.ts` | 相机能力封装 |
| 设置管理 | `common/src/main/ets/default/setting/SettingManager.ts` | 相机设置 |
| 全局状态 | `common/src/main/ets/default/redux/store` | Redux Store |

## 文档约定

### 符号标注
- **证据**: 指代代码中的具体位置，格式为 `文件路径:行号`
- **TODO(需确认)**: 需要人工进一步确认的内容
- **注意**: 重要提示信息

### 术语表

| 术语 | 说明 |
|------|------|
| HAP | HarmonyOS Ability Package，应用安装包 |
| HAR | HarmonyOS Archive，静态共享包 |
| Entry | 应用入口模块，可独立安装运行 |
| Ability | 应用组件，包含 UIAbility/ExtensionAbility |
| ArkTS | OpenHarmony 应用开发语言 |
| ArkUI | OpenHarmony UI 开发框架 |
