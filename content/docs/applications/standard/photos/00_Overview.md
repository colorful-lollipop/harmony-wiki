# 项目概览

## 1. 项目定位

### 1.1 核心定位

OpenHarmony **系统级图库应用**（Gallery），提供图片和视频的**管理、浏览、显示、编辑**等核心功能。

| 功能类别 | 具体能力 |
|----------|----------|
| **媒体管理** | 图片/视频浏览、相册管理（默认相册、用户相册） |
| **外部调用** | 支持第三方应用图片选择（单选/多选）、相机回调 |
| **FA 卡片** | 提供图库快捷入口和编辑功能 |
| **图片编辑** | 裁剪、滤镜等基础编辑能力 |

### 1.2 目标设备

| 设备类型 | 支持情况 | 证据 |
|----------|----------|------|
| **standard** (标准设备) | ✅ 支持 | `build-profile.json5:19` |
| **tablet** (平板) | ✅ 支持 | `build-profile.json5:20` |
| **default** | ✅ 默认 | `module.json5:9-11` |

### 1.3 技术栈

| 层级 | 技术/框架 |
|------|-----------|
| **开发语言** | TypeScript / ArkTS |
| **UI 框架** | ArkUI |
| **应用框架** | OpenHarmony Ability 框架 |
| **构建系统** | hvigor |
| **架构模式** | MVVM + MVP |

> **证据**: `README_zh.md:4` - "图库项目采用 TS 语言开发"
> **证据**: `README_zh.md:10` - "整体以 OpenHarmony 既有的 MVVM 的 App 架构设计为基础，向下扩展出一套 MVP 分层架构"

---

## 2. 核心能力

### 2.1 图片浏览

| 能力 | 说明 | 实现模块 |
|------|------|----------|
| 日视图浏览 | 按日期分组展示图片 | `timeline` |
| 相册浏览 | 按相册分组展示 | `album` (在 browser 中) |
| 大图浏览 | 单张图片全屏查看、缩放 | `browser` |
| 视频播放 | 视频文件预览 | `browser` |

### 2.2 图片选择（第三方）

| 能力 | URI Scheme | 页面路由 |
|------|-----------|----------|
| 单选图片 | `singleselect` | `ThirdSelectPhotoGridPage` |
| 多选图片 | `multipleselect` | `ThirdSelectPhotoGridPage` |
| 预设选择 | `preselectedUris` 参数 | 支持 |

> **证据**: `MainAbility.ts:113-130` - `parseWantParameter()` 解析选择参数

### 2.3 图片编辑

| 能力 | 说明 | 证据 |
|------|------|------|
| 裁剪 | 图片裁剪功能 | `feature/editor` |
| FA 编辑 | 卡片图片编辑 | `feature/formAbility` |

### 2.4 FA 卡片

| 能力 | 说明 | 证据 |
|------|------|------|
| 快捷浏览 | 从桌面卡片直接浏览 | `FormAbility` |
| 快速编辑 | 卡片入口图片编辑 | `FormEditorPage` |

---

## 3. 运行环境

### 3.1 系统要求

| 要求 | 最低版本 | 证据 |
|------|----------|------|
| **compileSdkVersion** | 23 | `build-profile.json5:29` |
| **compatibleSdkVersion** | 23 | `build-profile.json5:30` |

### 3.2 权限依赖

| 权限 | 用途 | 敏感度 |
|------|------|--------|
| `ohos.permission.READ_IMAGEVIDEO` | 读取媒体文件 | 高 |
| `ohos.permission.WRITE_IMAGEVIDEO` | 写入媒体文件 | 高 |
| `ohos.permission.MEDIA_LOCATION` | 访问位置信息 | 中 |
| `ohos.permission.PROXY_AUTHORIZATION_URI` | 代理授权 | 低 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 后台启动 | 中 |
| `ohos.permission.GET_BUNDLE_INFO` | 获取包信息 | 低 |

> **证据**: `module.json5:21-55`

### 3.3 运行时依赖

| 依赖项 | 用途 | 说明 |
|--------|------|------|
| **UserFileManager** | 媒体文件管理 | OpenHarmony 系统服务 |
| **MediaLibrary** | 媒体库访问 | 系统级媒体服务 |
| **WindowManager** | 窗口管理 | 窗口生命周期 |

---

## 4. 关键概念

### 4.1 MVP 架构分层

```
┌─────────────────────────────────────────────────┐
│                   View (视图层)                  │
│   负责 UI 显示、触摸/点击事件监听               │
│   证据: README_zh.md:14                          │
├─────────────────────────────────────────────────┤
│                Presenter (展现层)                │
│   接收 View 请求，连通 Model 层获取数据         │
│   证据: README_zh.md:15                          │
├─────────────────────────────────────────────────┤
│                 Model (模型层)                   │
│   数据处理逻辑，返回请求结果                     │
│   证据: README_zh.md:16                          │
└─────────────────────────────────────────────────┘
```

### 4.2 层级映射

| 层级 | 示例组件 | 路径 |
|------|----------|------|
| View | `phone.view.Index` | `product/phone/src/main/ets/view/` |
| View | `TimelinePage` | `feature/timeline/src/main/ets/view/` |
| Presenter | `GroupItemDataSource` | `feature/timeline/src/main/ets/view/` |
| Model | `MediaDataItem` | `common/src/main/ets/default/model/` |

> **证据**: `README_zh.md:20-39`

### 4.3 外部调用模式

| 模式 | 触发方式 | 入口 |
|------|----------|------|
| **首页浏览** | 桌面图标启动 | `MainAbility` → `index` |
| **相机回调** | `startAbility` + `uri=photodetail` | `MainAbility` → `PhotoBrowser` |
| **单选** | `uri=singleselect` | `MainAbility` → `ThirdSelectPhotoGridPage` |
| **多选** | `uri=multipleselect` | `MainAbility` → `ThirdSelectPhotoGridPage` |
| **FA 浏览** | `uri=form` | `MainAbility` → `PhotoBrowser` |

---

## 5. 与系统关系

### 5.1 系统能力依赖

```
Photos App
    │
    ├── UserFileManager (媒体文件操作)
    ├── MediaLibrary (媒体库查询)
    ├── WindowManager (窗口管理)
    ├── AbilityManager (生命周期管理)
    └── Preferences (本地存储)
```

### 5.2 被调用关系

| 调用方 | 调用方式 | 说明 |
|--------|----------|------|
| **桌面系统** | 启动 MainAbility | 首页入口 |
| **相机应用** | `startAbility(uri=photodetail)` | 浏览刚拍摄照片 |
| **第三方应用** | `startAbilityForResult(uri=singleselect)` | 选择单张图片 |
| **第三方应用** | `startAbilityForResult(uri=multipleselect)` | 选择多张图片 |
| **系统设置** | FA 卡片 | 快捷浏览 |

> **证据**: `MainAbility.ts:92-166`

---

## 6. 快速开始

### 6.1 编译构建

```bash
# 根目录执行
hvigor assembleHap --product default
```

### 6.2 安装运行

```bash
# 卸载系统自带图库
hdc shell rm -rf /system/app/com.ohos.photos/Photos.hap

# 安装签名 HAP
hdc install ./build/outputs/hap/release/photos.hap
```

### 6.3 日志查看

```bash
# 过滤图库日志
hilog | grep Photos
```

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [01_Architecture.md](01_Architecture.md) | 详细架构说明 |
| [02_Module_Structure.md](02_Module_Structure.md) | 模块职责 |
| [03_External_API.md](03_External_API.md) | 外部调用接口 |
| [05_Build_System.md](05_Build_System.md) | 构建配置 |
| [README_zh.md](../README_zh.md) | 原始开发说明 |
