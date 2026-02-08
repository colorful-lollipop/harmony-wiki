# SystemUI 架构设计

## 1. 架构总览

### 1.1 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony Framework                     │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Ability 生命周期管理                        ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                              ↑
                              │ startAbility()
                              │ onCreate/onDestroy
┌─────────────────────────────────────────────────────────────┐
│                      SystemUI                                 │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  ServiceExtensionAbility (后台服务)                       ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │  entry/phone  ───────────────────────────────────┐  │││
│  │  │  entry/pc      │  initSystemUi()                 │  │││
│  │  └─────────────────────────────────────────────────────┘││
│  │                        ↓                                 ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │  common (EventBus, WindowManager, etc.)             │││
│  │  └─────────────────────────────────────────────────────┘││
│  │                        ↓                                 ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │  features (21 个 HAR 组件)                          │││
│  │  │  ┌─────────┬─────────┬─────────┬─────────┐        │││
│  │  │  │battery  │  wifi   │ clock   │ volume  │        │││
│  │  │  │feature  │ feature │ feature │ feature │        │││
│  │  │  └─────────┴─────────┴─────────┴─────────┘        │││
│  │  └─────────────────────────────────────────────────────┘││
│  │                        ↓                                 ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │  product (9 个产品模块)                              │││
│  │  │  ┌─────────┬─────────┬─────────┐                  │││
│  │  │  │statusbar│ control │ dropdown│                  │││
│  │  │  │ panel   │ center  │ panel   │                  │││
│  │  │  └─────────┴─────────┴─────────┘                  │││
│  │  └─────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                              ↑
                              │ 显示 UI
┌─────────────────────────────────────────────────────────────┐
│                      ArkUI 组件                              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│  │ Text    │ │ Image   │ │ Column  │ │ Row     │         │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 架构分层

| 层级 | 组件 | 职责 |
|------|------|------|
| **L1: 入口层** | entry/phone, entry/pc | HAP 入口，Ability 生命周期 |
| **L2: 服务层** | ServiceExtAbility | 后台服务初始化和事件分发 |
| **L3: 基础层** | common | EventBus、Log、WindowManager 等 |
| **L4: 组件层** | features (21 个) | 独立功能组件 (HAR) |
| **L5: 产品层** | product (9 个) | UI 组合和布局 |
| **L6: 渲染层** | ArkUI | 声明式 UI 渲染 |

## 2. 模块依赖关系

### 2.1 依赖方向

```
entry (HAP)
    ↓
product (feature)
    ↓
features (har) + common (har)
```

### 2.2 具体依赖

| 源模块 | 目标模块 | 依赖类型 | 证据 |
|--------|----------|----------|------|
| `entry/phone` | `product/*` | feature | `module.json5:mainElement` |
| `product/*` | `features/*` | har | `module.json5:dependencies` |
| `features/*` | `common` | har | imports |

### 2.3 模块类型转换

| 编译时 | 运行时 |
|--------|--------|
| HAR (features) | 打包到 product/entry |
| feature (product) | 独立 HAP |
| entry | 独立 HAP |

**证据**: `build-profile.json5:12-196` 定义了模块配置

## 3. 组件交互模式

### 3.1 EventBus 通信

```
┌──────────────┐      publish()       ┌──────────────┐
│   Source     │ ───────────────────→ │  EventBus    │
│  Component   │                      │              │
└──────────────┘                      └──────┬───────┘
                                             │
                                             │ emit()
                                             ↓
┌──────────────┐      subscribe()      ┌──────────────┐
│   Target     │ ←─────────────────── │  Component   │
│  Component   │                      │              │
└──────────────┘                      └──────────────┘
```

**证据**: `common/src/main/ets/default/event/EventManager.ts:30-60`

```typescript
class EventManager {
  publish(event: Event, pluginType?: PluginType): boolean {
    return this.eventParser[event.target].call(this, event.data, pluginType);
  }

  subscribe(eventType: Events, callback: Callback): unsubscribe {
    return this.mEventBus.on(eventType, callback);
  }
}
```

### 3.2 事件类型

| 类型 | 目标 | 说明 |
|------|------|------|
| `local` | 本地 EventBus | 组件内通信 |
| `ability` | 其他 Ability | 启动其他 UI |
| `commonEvent` | 公共事件 | TODO |
| `remote` | 远程设备 | TODO |

## 4. ServiceExtension 生命周期

### 4.1 生命周期图

```
┌─────────┐
│  onLoad │ ← Ability 加载
└────┬────┘
     ↓
┌─────────┐
│ onCreate│ ← 初始化
│  ────── │   initSystemUi()
└────┬────┘   AbilityManager.setContext()
     ↓
┌─────────┐
│ onStart │ ← 开始服务
│  ────── │
│ onStop  │ ← 停止服务
└────┬────┘
     ↓
┌─────────┐
│onDestroy│ ← 销毁
└─────────┘
```

### 4.2 初始化流程

**证据**: `entry/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts:24-28`

```typescript
onCreate(want: Want): void {
  Log.showInfo(TAG, `onCreate, want: ${JSON.stringify(want)}`);
  initSystemUi(this.context);
  AbilityManager.setContext(ABILITY_NAME_ENTRY, this.context);
}
```

**证据**: `common/src/main/ets/default/InitSystemUi.ts:24-31`

```typescript
export default function initSystemUi(context: ServiceExtensionContext): void {
  EventManager.setContext(context);
  ScreenLockManager.init();
  TimeManager.init(context);
}
```

### 4.3 状态管理

| Ability | 状态 | 常量 |
|---------|------|------|
| Entry | 运行中 | `SystemUi_Entry` |
| StatusBar | 运行中 | `SystemUi_StatusBar` |
| NavigationBar | 运行中 | `SystemUi_NavigationBar` |
| VolumePanel | 按需启动 | `SystemUi_VolumePanel` |

**证据**: `common/src/main/ets/default/abilitymanager/abilityManager.ts:24-34`

## 5. 窗口与 UI 渲染

### 5.1 窗口创建流程

```
AbilityStage.onCreate()
    ↓
WindowManager.createWindow()
    ↓
loadContent() 加载 ArkUI 页面
    ↓
UI 渲染完成
```

### 5.2 窗口类型

| 类型 | 用途 | 证据 |
|------|------|------|
| 主窗口 | 状态栏、导航栏 | `WindowManager.ts` |
| 子窗口 | 音量面板、控制中心 | `WindowManager.createSubWindow()` |

### 5.3 UI 组件层级

```
Window
├── StatusBar (系统窗口)
├── NavigationBar (系统窗口)
├── ControlCenter (悬浮窗口)
├── VolumePanel (悬浮窗口)
└── NotificationPanel (悬浮窗口)
```

## 6. 数据流

### 6.1 系统事件到 UI 更新

```
系统事件 (如电量变化)
    ↓
Framework → Notification/Broadcast
    ↓
EventManager.subscribe()
    ↓
Feature 组件接收事件
    ↓
状态更新
    ↓
ArkUI 组件重新渲染
```

### 6.2 用户交互到系统操作

```
用户点击 (如开关 WiFi)
    ↓
ArkUI onClick 事件
    ↓
调用 @ohos.wifiManager API
    ↓
系统服务处理
    ↓
结果返回 → UI 更新
```

## 7. 线程模型

### 7.1 主线程

ArkUI 组件的渲染和用户交互在主线程执行。

### 7.2 约束

- **禁止阻塞主线程**: 长时间操作应使用 async/await
- **事件处理**: EventBus 回调在主线程执行
- **API 调用**: @ohos.* API 多数为异步

### 7.3 最佳实践

```typescript
// ✅ 正确: 异步调用
async fetchBatteryInfo(): Promise<void> {
  let batteryInfo = await batteryInfoManager.getBatteryInfo();
  this.updateUI(batteryInfo);
}

// ❌ 错误: 阻塞调用 (除非确定快速完成)
let info = getBatteryInfoSync(); // 不推荐
```

## 8. 错误处理

### 8.1 错误传播

```
API 错误
    ↓
catch 捕获
    ↓
Log.showError()
    ↓
用户提示 (可选)
```

### 8.2 日志级别

| 级别 | 方法 | 用途 |
|------|------|------|
| DEBUG | `Log.showDebug()` | 调试信息 |
| INFO | `Log.showInfo()` | 普通信息 |
| WARN | `Log.showWarn()` | 警告信息 |
| ERROR | `Log.showError()` | 错误信息 |
| FATAL | `Log.showFatal()` | 严重错误 |

**证据**: `common/src/main/ets/default/Log.ts:40-121`

## 9. 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [目录结构](02_Directory_Structure.md) - 详细目录说明
- [内部 API](04_API_Inner.md) - 模块接口
- [构建指南](05_Build.md) - 编译配置
- [安全评审](06_Security.md) - 安全考虑
