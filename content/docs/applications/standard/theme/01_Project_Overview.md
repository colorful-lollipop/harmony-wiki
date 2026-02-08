# 项目概览

## 项目定位

**applications_theme** 是 OpenHarmony 系统的预置系统应用，属于系统应用层（System Application）的壁纸主题模块。

### 在 OpenHarmony 架构中的位置

```
┌─────────────────────────────────────────────────┐
│              系统应用层 (System Apps)            │
│  ┌───────────────────────────────────────────┐  │
│  │         applications_theme (本项目)        │  │
│  │  - 壁纸设置                                │  │
│  │  - 主题管理                                │  │
│  └───────────────────────────────────────────┘  │
├─────────────────────────────────────────────────┤
│              框架层 (Framework)                  │
│  - AbilityFramework                            │
│  - WindowManager                               │
│  - WallpaperService                            │
├─────────────────────────────────────────────────┤
│              系统服务层 (System Services)        │
│  - BundleManager                               │
│  - WallpaperManager                            │
├─────────────────────────────────────────────────┤
│              内核层 (Kernel)                     │
└─────────────────────────────────────────────────┘
```

## 核心能力

### 主要功能

| 功能 | 说明 | 代码位置 |
|------|------|----------|
| **壁纸显示** | 在系统锁屏/桌面显示壁纸图片 | `WallpaperExtAbility.onCreated()` |
| **壁纸变化监听** | 监听壁纸设置变化并更新显示 | `WallpaperExtAbility.onWallpaperChanged()` |
| **壁纸数据获取** | 从壁纸服务获取像素数据 | `wallPaper.getPixelMap()` |
| **窗口管理** | 创建和管理壁纸窗口 | `windowManager.create()` |

### 支持的设备类型

| 设备类型 | 支持状态 | 产品配置 |
|----------|----------|----------|
| **phone** | ✅ 支持 | `product/phone/` |
| **pad** | ✅ 支持 | `product/pad/` |

## 运行环境

### 运行时依赖

| 依赖项 | 最低版本 | 说明 |
|--------|----------|------|
| **OpenHarmony SDK** | 9 | 编译和运行所需 SDK |
| **hvigor** | 任意稳定版本 | 构建工具 |
| **Node.js** | 14+ | hvigor 运行时 |

### 系统服务依赖

| 服务 | 用途 | 必需 |
|------|------|------|
| **WallpaperManager** | 壁纸数据管理 | ✅ 必需 |
| **WindowManager** | 窗口创建管理 | ✅ 必需 |
| **BundleManager** | 包管理验证 | ✅ 必需 |
| **PermissionManager** | 权限校验 | ✅ 必需 |

## 关键概念

### WallpaperExtAbility

壁纸扩展能力，是本项目的核心组件。

- **类型**: Extension (扩展能力)
- **类型值**: `wallpaper`
- **可见性**: `visible: true`
- **职责**: 壁纸窗口生命周期管理、壁纸数据同步

**代码证据**:
```
路径: product/phone/src/main/module.json5:23-32
符号: WallpaperExtAbility
```

### Ability 生命周期

```
┌─────────────┐
│  onCreate   │ ← 能力创建
└──────┬──────┘
       │
┌──────▼──────┐
│ onStart     │ ← 能力启动（Extension）
└──────┬──────┘
       │
┌──────▼──────┐
│ onCreated   │ ← 窗口创建完成
└──────┬──────┘
       │
┌──────▼──────┐
│ onForeground│ ← 转到前台
└──────┬──────┘
       │
┌──────▼──────┐
│ onBackground│ ← 转到后台
└──────┬──────┘
       │
┌──────▼──────┐
│  onDestroy  │ ← 能力销毁
└─────────────┘
```

### AppStorage

应用级数据存储，用于组件间数据同步。

- **用途**: 存储壁纸像素数据 (`slPixelData`)
- **访问方式**: `@StorageLink` 装饰器
- **同步机制**: 双向绑定

**代码证据**:
```
路径: product/phone/src/main/ets/pages/index.ets:21
符号: @StorageLink('slPixelData')
```

## 包信息

| 属性 | 值 |
|------|-----|
| **Bundle Name** | `com.ohos.wallpaper` |
| **Vendor** | `example` |
| **Version Code** | `1000000` |
| **Version Name** | `1.0.0` |
| **模块类型** | feature (功能模块) |
| **安装方式** | 随系统预置 |

**代码证据**:
```
路径: AppScope/app.json5:3-6
符号: bundleName, vendor, versionCode, versionName
```

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [API 文档](04_API.md)
- [安全评审](08_Security.md)
