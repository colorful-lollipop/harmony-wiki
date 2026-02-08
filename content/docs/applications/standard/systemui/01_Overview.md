# SystemUI 项目概览

## 1. 项目定位

### 1.1 基本信息

| 属性 | 值 |
|------|------|
| **项目名称** | SystemUI |
| **项目类型** | OpenHarmony 系统应用 |
| **语言** | ArkTS / ArkUI |
| **预置位置** | /system/apps/com.ohos.systemui |
| **用户可见** | 是 (状态栏、通知、控制中心等) |

### 1.2 核心能力

SystemUI 提供以下核心功能：

| 功能模块 | 描述 |
|----------|------|
| **状态栏** | 显示时间、电池、信号、WiFi、蓝牙等状态图标 |
| **导航栏** | 提供系统导航手势和虚拟按键 |
| **通知系统** | 展示、管理系统和应用通知 |
| **控制中心** | 快速设置面板 (WiFi、蓝牙、亮度等) |
| **锁屏界面** | 锁屏状态和时间显示 |
| **系统设置** | 音量、亮度、飞行模式等系统级设置 |

### 1.3 技术特征

- **纯 ArkTS/ArkUI**: 无 N-API 层，直接使用 `@ohos.*` 系统 API
- **模块化设计**: 21 个独立 HAR 组件 + 9 个产品模块
- **ServiceExtension**: 使用后台服务能力实现系统 UI
- **EventBus**: 组件间事件通信机制
- **多设备适配**: 支持 phone 和 pc 设备

## 2. 运行环境

### 2.1 系统要求

| 要求 | 版本 |
|------|------|
| **OpenHarmony SDK** | 12 |
| **hvigor** | 最新版本 |
| **Node.js** | 16+ |
| **设备类型** | phone, pc, tablet |

### 2.2 依赖系统服务

| 服务 | 用途 | 权限 |
|------|------|------|
| NotificationManager | 通知管理 | `NOTIFICATION_CONTROLLER` |
| WindowManager | 窗口管理 | 系统内置 |
| BundleManager | 包管理 | `GET_BUNDLE_INFO_PRIVILEGED` |
| WiFiManager | WiFi 控制 | `MANAGE_WIFI_CONNECTION` |
| Bluetooth | 蓝牙控制 | `MANAGE_BLUETOOTH` |
| AudioManager | 音量控制 | `MODIFY_AUDIO_SETTINGS` |

## 3. 模块总览

### 3.1 模块分类

```
SystemUI
├── common (通用工具)
│   └── 18 个工具类
├── entry (入口模块)
│   ├── phone (手机版)
│   └── pc (PC 版)
├── features (功能组件 - 21 个 HAR)
│   ├── system/ (系统状态)
│   │   ├── batterycomponent
│   │   ├── brightnesscomponent
│   │   ├── clockcomponent
│   │   ├── signalcomponent
│   │   └── volumecomponent
│   ├── network/ (网络连接)
│   │   ├── airplanecomponent
│   │   ├── bluetoothcomponent
│   │   ├── locationcomponent
│   │   ├── nfccomponent
│   │   └── wificomponent
│   └── notification/ (通知相关)
│       ├── capsulecomponent
│       ├── controlcentercomponent
│       ├── managementcomponent
│       ├── navigationservice
│       ├── noticeitem
│       └── notificationservice
└── product (产品模块 - 9 个)
    ├── default/ (默认产品)
    ├── phone/ (手机产品)
    └── pc/ (PC 产品)
```

### 3.2 模块类型说明

| 类型 | 后缀 | 说明 | 编译产物 |
|------|------|------|----------|
| entry | 无 | 主入口，可独立安装 | .hap |
| feature | component | 功能模块，需被引用 | .hap |
| har | 无 | 共享模块，被引用 | .har |

**证据**: `build-profile.json5:14-196` 定义了所有模块的 `srcPath` 和 `type`

## 4. 关键概念

### 4.1 ServiceExtensionAbility

SystemUI 的核心组件都是 `ServiceExtensionAbility`，用于：

- 后台运行，无需用户交互界面
- 响应系统事件 (如电量变化、通知到达)
- 显示系统级 UI (状态栏、通知面板)

**证据**: `entry/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts:23`

```typescript
class ServiceExtAbility extends ServiceExtension {
  onCreate(want: Want): void {
    Log.showInfo(TAG, `onCreate, want: ${JSON.stringify(want)}`);
    initSystemUi(this.context);
  }
}
```

### 4.2 EventBus

组件间通信采用发布-订阅模式：

```typescript
// 发布事件
EventManager.publish({
  target: 'local',
  data: { eventName: 'BATTERY_CHANGED', args: { level: 50 } }
});

// 订阅事件
EventManager.subscribe('BATTERY_CHANGED', (args) => {
  console.log(`Battery: ${args.level}%`);
});
```

**证据**: `common/src/main/ets/default/event/EventManager.ts:30-60`

### 4.3 Ability 名称

| 常量 | 值 | 用途 |
|------|------|------|
| `ABILITY_NAME_ENTRY` | `SystemUi_Entry` | 主入口 |
| `ABILITY_NAME_STATUS_BAR` | `SystemUi_StatusBar` | 状态栏 |
| `ABILITY_NAME_NAVIGATION_BAR` | `SystemUi_NavigationBar` | 导航栏 |
| `ABILITY_NAME_VOLUME_PANEL` | `SystemUi_VolumePanel` | 音量面板 |

**证据**: `common/src/main/ets/default/abilitymanager/abilityManager.ts:24-34`

## 5. 数据流概述

### 5.1 典型数据流

```
系统事件 (如电量变化)
    ↓
Framework 通知 SystemUI
    ↓
ServiceExtensionAbility.onCreate/onRequest()
    ↓
EventManager.publish() 事件分发
    ↓
Feature 组件订阅并更新 UI
    ↓
用户看到状态变化
```

### 5.2 事件类型

| 类型 | 说明 | 状态 |
|------|------|------|
| `local` | 本地组件间事件 | ✅ 已实现 |
| `ability` | 启动其他 Ability | ✅ 已实现 |
| `commonEvent` | 公共事件 | 🔶 TODO |
| `remote` | 跨设备事件 | 🔶 TODO |

## 6. 与其他系统的交互

### 6.1 向上交互

| 目标系统 | 交互方式 | 用途 |
|----------|----------|------|
| OpenHarmony Framework | Ability 生命周期 | 接收系统事件 |
| NotificationManager | @ohos.notificationManager | 发布/管理通知 |
| WindowManager | @ohos.window | 创建系统窗口 |

### 6.2 向下交互

| 目标组件 | 交互方式 | 用途 |
|----------|----------|------|
| Features 组件 | EventBus | 状态同步 |
| Product 模块 | HAR 引用 | UI 组合 |

## 7. 快速入门

### 7.1 代码结构

```
systemui/
├── entry/              # 入口模块
│   ├── phone/          # 手机版 HAP
│   └── pc/             # PC 版 HAP
├── features/           # 功能组件 (HAR)
│   └── [21 个组件]/
├── product/            # 产品模块 (HAR)
│   ├── default/
│   ├── phone/
│   └── pc/
└── common/             # 通用工具 (HAR)
    └── src/main/ets/default/
```

### 7.2 添加新功能

1. 在 `features/` 创建新组件目录
2. 实现 `module.json5` 配置
3. 实现业务逻辑 (ArkTS)
4. 在 `product/` 中引用
5. 配置权限 (如需要)

## 8. 限制与注意事项

### 8.1 无 N-API 层

⚠️ 本项目**不包含** N-API 或 C/C++ 代码，所有系统调用通过 ArkTS API 完成。

### 8.2 权限限制

部分敏感操作需要声明权限：

| 权限 | 用途 | 风险 |
|------|------|------|
| `MANAGE_SECURE_SETTINGS` | 修改安全设置 | 高 |
| `CAPTURE_SCREEN` | 截屏 | 高 |
| `GET_TELEPHONY_STATE` | 获取电话状态 | 中 |

### 8.3 设备适配

- `features/*`: 通用组件，适用于所有设备
- `product/phone/`: 手机专用 UI
- `product/pc/`: PC 专用 UI

## 9. 相关文档

- [目录结构](02_Directory_Structure.md) - 详细目录说明
- [架构设计](03_Architecture.md) - 系统架构图
- [内部 API](04_API_Inner.md) - 模块接口
- [构建指南](05_Build.md) - 编译配置
- [安全评审](06_Security.md) - 安全考虑
