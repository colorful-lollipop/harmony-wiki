# 关键配置项

> 本文档记录 SystemUI 项目中的关键配置项和功能开关。

## 1. 模块配置

### 1.1 build-profile.json5

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `compileSdkVersion` | `12` | 编译 SDK 版本 |
| `compatibleSdkVersion` | `12` | 兼容 SDK 版本 |
| `signingConfig` | `release` | 签名配置 |

**证据**: `build-profile.json5:4-9`

### 1.2 module.json5 通用配置

| 配置项 | 类型 | 说明 |
|--------|------|------|
| `module.name` | string | 模块名称 |
| `module.type` | string | 模块类型 (entry/feature/har) |
| `module.uiSyntax` | string | UI 语法 (ets) |
| `module.mainElement` | string | 主元素 |
| `module.deliveryWithInstall` | boolean | 是否随安装交付 |

**证据**: `entry/phone/src/main/module.json5:1-20`

## 2. 权限配置

### 2.1 权限声明

所有权限在 `module.json5` 的 `requestPermissions` 中声明：

```json5
"requestPermissions": [
  {
    "name": "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED"
  }
]
```

### 2.2 权限使用场景

| 权限 | 使用组件 | 用途 |
|------|----------|------|
| `NOTIFICATION_CONTROLLER` | noticeitem | 管理通知 |
| `GET_WIFI_INFO` | wificomponent | 显示 WiFi 状态 |
| `MANAGE_WIFI_CONNECTION` | wificomponent | 控制 WiFi |
| `GET_TELEPHONY_STATE` | signalcomponent | 显示信号 |

## 3. 功能开关

### 3.1 ArkTS 部分更新

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `metadata.name` | `ArkTSPartialUpdate` | 启用 ArkTS 部分更新 |
| `metadata.value` | `true` | 开启状态 |

**证据**: `product/phone/statusbar/src/main/module.json5:11-16`

```json5
"metadata": [
  {
    "name": "ArkTSPartialUpdate",
    "value": "true"
  }
]
```

### 3.2 设备类型

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `deviceTypes` | `["default"]` | 支持默认设备 |
| `deviceTypes` | `["default", "tablet"]` | 支持多设备 |

## 4. 窗口配置

### 4.1 窗口属性

| 属性 | 值 | 说明 |
|------|-----|------|
| `windowType` | - | 窗口类型 |
| `isLayoutFullScreen` | boolean | 全屏布局 |
| `isKeepScreenOn` | boolean | 保持屏幕常亮 |

**证据**: `common/src/main/ets/default/WindowManager.ts`

### 4.2 悬浮窗口

音量面板、控制中心等使用悬浮窗口：

```typescript
// 创建子窗口
WindowManager.createSubWindow('volumepanel');
```

## 5. 日志配置

### 5.1 日志域

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `DOMAIN` | `0x001A` | SystemUI 日志域 |
| `TAG` | `"SystemUI_Default"` | 默认标签 |

**证据**: `common/src/main/ets/default/Log.ts:17-18`

### 5.2 敏感信息过滤

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `FILTER_KEYS` | `['hide']` | 敏感关键词 |

## 6. 通知配置

### 6.1 通知类型

| 类型 | 常量 | 说明 |
|------|------|------|
| 基础文本 | `TYPE_BASIC` | 基础文本通知 |
| 长文本 | `TYPE_LONG` | 长文本通知 |
| 多行文本 | `TYPE_MULTI` | 多行文本通知 |
| 图片 | `TYPE_PICTURE` | 带图片通知 |

**证据**: `features/noticeitem/src/main/ets/com/ohos/noticeItem/model/NotificationManager.ts:33-36`

### 6.2 调试配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `DEBUG_SETTING_KEY` | `debug.systemui.notificationtemplate` | 调试开关 |
| `DEBUG_BUNDLE_NAME` | `com.ohos.example.notificationtemplate` | 调试包名 |

## 7. 事件配置

### 7.1 事件目标

| 目标 | 说明 |
|------|------|
| `local` | 本地事件 |
| `ability` | 启动 Ability |
| `commonEvent` | 公共事件 |
| `remote` | 远程事件 |

### 7.2 事件名称

| 事件 | 说明 |
|------|------|
| `BATTERY_CHANGED` | 电量变化 |
| `WIFI_STATE_CHANGED` | WiFi 状态变化 |
| `VOLUME_CHANGED` | 音量变化 |
| `NOTIFICATION_ARRIVED` | 通知到达 |

## 8. 相关文档

- [构建指南](05_Build.md) - 构建配置
- [内部 API](04_API_Inner.md) - API 配置
- [架构设计](03_Architecture.md) - 架构配置
