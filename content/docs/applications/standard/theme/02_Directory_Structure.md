# 目录结构

## 顶层目录概览

```
/applications/standard/theme/
├── AppScope/                    # 应用级配置
│   ├── app.json5               # 应用配置文件
│   └── resources/              # 应用资源
├── product/                    # 产品配置目录
│   ├── phone/                  # 手机产品
│   │   └── src/main/
│   │       ├── ets/            # ETS 源码
│   │       ├── resources/      # 产品资源
│   │       └── module.json5    # 模块配置
│   └── pad/                    # 平板产品
│       └── src/main/
│           ├── ets/            # ETS 源码
│           ├── resources/      # 产品资源
│           └── module.json5    # 模块配置
├── signature/                  # 签名配置
│   └── signatureProfile.json   # 签名配置
├── wiki/                       # 本 Wiki 文档
│   ├── _work/                  # 工作区
│   └── *.md                    # Wiki 文档
├── build-profile.json5         # 构建配置
├── hvigorfile.js              # hvigor 脚本
├── package.json               # Node.js 配置
└── OAT.xml                    # 安全审计配置
```

## 产品目录结构 (phone/pad)

以 `product/phone/` 为例：

```
product/phone/src/main/
├── ets/                        # ArkTS/ETS 源码目录
│   ├── Application/            # 应用级组件
│   │   └── AbilityStage.ts    # Ability 舞台
│   ├── MainAbility/            # 主能力
│   │   └── MainAbility.ts     # 能力实现
│   ├── WallpaperExtAbility/    # 壁纸扩展能力
│   │   └── WallpaperExtAbility.ts  # 壁纸能力实现
│   └── pages/                  # UI 页面
│       └── index.ets           # 壁纸展示页面
├── resources/                  # 资源目录
│   ├── base/                   # 默认资源
│   │   ├── element/            # 字符串等元素资源
│   │   │   └── string.json     # 字符串配置
│   │   ├── media/              # 图片资源
│   │   │   └── icon.png        # 应用图标
│   │   └── profile/            # 配置资源
│   │       └── main_pages.json # 页面路由配置
│   └── default/                # 默认资源
└── module.json5                # 模块配置文件
```

## 模块职责说明

### 1. Application/AbilityStage.ts

**职责**: 应用级别初始化

| 属性 | 值 |
|------|-----|
| 基类 | `AbilityStage` |
| 生命周期 | `onCreate()` |
| 职责 | 应用启动时的初始化工作 |

**代码位置**: `product/phone/src/main/ets/Application/AbilityStage.ts:18-21`

### 2. MainAbility/MainAbility.ts

**职责**: 主能力生命周期管理

| 属性 | 值 |
|------|-----|
| 基类 | `Ability` |
| 生命周期回调 | `onCreate()`, `onDestroy()`, `onWindowStageCreate()`, `onWindowStageDestroy()`, `onForeground()`, `onBackground()` |
| 职责 | 管理能力的生命周期事件 |

**代码位置**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:18-46`

### 3. WallpaperExtAbility/WallpaperExtAbility.ts

**职责**: 壁纸扩展能力的核心实现

| 属性 | 值 |
|------|-----|
| 基类 | `Extension` (WallpaperExtension) |
| 类型 | `wallpaper` |
| 可见性 | `visible` |
| 核心方法 | `onCreated()`, `onWallpaperChanged()`, `initWallpaperImage()`, `sendPixelMapData()` |
| 职责 | 壁纸窗口创建、壁纸变化监听、壁纸数据同步 |

**代码位置**: `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:23-76`

### 4. pages/index.ets

**职责**: 壁纸展示 UI

| 属性 | 值 |
|------|-----|
| 组件类型 | `@Entry @Component struct Index` |
| 状态管理 | `@StorageLink('slPixelData')` |
| UI 结构 | `Flex` + `Image` |
| 职责 | 展示壁纸像素数据 |

**代码位置**: `product/phone/src/main/ets/pages/index.ets:20-38`

## 资源配置说明

### 字符串资源

| 资源 ID | 值 | 用途 |
|---------|-----|------|
| `app_name` | 应用名称 | 桌面显示名称 |
| `myapplication_desc` | 模块描述 | 模块说明 |

**代码位置**: `product/phone/src/main/resources/base/element/string.json`

### 页面路由

**代码位置**: `product/phone/src/main/resources/base/profile/main_pages.json`

```json
{
  "src": ["pages/index"]
}
```

## 签名配置

| 配置项 | 说明 |
|--------|------|
| 签名文件 | `signature/signatureProfile.json` |
| 签名配置 | `build-profile.json5` 中的 `signingConfig: "default"` |

## 资源目录归类

| 目录/文件 | 职责 | 是否含测试 |
|-----------|------|-----------|
| `AppScope/` | 应用级配置 | 否 |
| `product/phone/` | 手机产品源码 | 否 |
| `product/pad/` | 平板产品源码 | 否 |
| `signature/` | 签名配置 | 否 |
| `wiki/` | 文档 | 否 |

## 相关文档

- [项目概览](01_Project_Overview.md)
- [架构说明](03_Architecture.md)
- [构建配置](06_Build.md)
