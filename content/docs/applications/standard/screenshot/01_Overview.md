# 项目概览

## 项目定位

**ScreenShot** 是 OpenHarmony 系统预置的系统应用，为用户提供截取当前屏幕并保存图片的功能。

- **项目类型**: OpenHarmony 系统应用
- **bundleName**: `com.ohos.screenshot`
- **vendor**: `ohos`
- **版本**: 1.0.0 (versionCode: 1000000)
- **目标设备**: phone, tablet

## 核心能力

### 1. 截屏功能
- 截取当前系统屏幕内容
- 自动保存为 JPEG 格式图片
- 保存路径: `Screenshots/` 相册目录

### 2. 窗口管理
- 创建浮动截屏预览窗口（40% 缩放）
- 自动定位到屏幕指定位置 (Y=300)
- 5秒后自动关闭

### 3. 隐私保护
- 隐私对话框（用于二次确认场景）
- 用户选择结果上报

### 4. 能力开放
- 支持系统 UI 触发 (`com.ohos.systemui.action.TOGGLE`)
- 可被其他应用调用

## 运行环境

| 环境项 | 要求 |
|-------|------|
| API Type | stageMode |
| Compile SDK | 23 |
| Target SDK | 23 |
| Compatible SDK | 23 |
| 运行时 | OpenHarmony |

## 技术栈

- **开发语言**: ArkTS (ETS)
- **UI 框架**: ArkUI
- **构建工具**: hvigor
- **应用模型**: Stage 模型

## 模块组成

本项目由三个模块组成：

| 模块 | 类型 | 职责 |
|-----|------|------|
| `product/phone` | entry (HAP) | 应用入口与 UI |
| `features/screenshot` | har | 截屏核心逻辑 |
| `common` | har | 通用工具 |

**证据**: `product/phone/src/main/module.json5:4-5`, `features/screenshot/src/main/module.json5:4`, `common/src/main/module.json5:4`

## 功能流程

```
用户触发截屏
    ↓
ServiceExtAbility 接收请求
    ↓
创建浮动窗口 (40% 缩放)
    ↓
加载预览页面 (index.ets)
    ↓
执行截屏 (ScreenshotManager.save)
    ↓
保存图片到相册 (Screenshots/)
    ↓
用户点击图片 → 打开相册
    ↓
自动关闭截屏窗口
```

## 相关文档

- 详细架构: [02_Architecture.md](02_Architecture.md)
- 模块说明: [03_Modules.md](03_Modules.md)
- API 参考: [04_API.md](04_API.md)
