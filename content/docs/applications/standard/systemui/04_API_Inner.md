# SystemUI 内部 API

> ⚠️ **重要说明**: 本项目为**纯 ArkTS 项目**，不包含 N-API 层。
>
> 以下文档为 **ArkTS 内部 API**，非 C/C++ 原生接口。

## 1. API 概览

### 1.1 API 分类

| 分类 | 说明 | 稳定性 |
|------|------|--------|
| **系统 API** | @ohos.* OpenHarmony API | 稳定 |
| **内部工具** | common 模块工具类 | 稳定 |
| **组件接口** | features 模块导出 | 稳定 |
| **产品接口** | product 模块导出 | 稳定 |

### 1.2 系统 API 使用

SystemUI 使用的主要系统 API：

| API 包 | 用途 | 证据 |
|--------|------|------|
| `@ohos.notificationManager` | 通知管理 | `NotificationManager.ts:18` |
| `@ohos.notificationSubscribe` | 通知订阅 | `NotificationManager.ts:17` |
| `@ohos.pluginComponent` | 插件组件 | `NotificationManager.ts:20` |
| `@ohos.systemparameter` | 系统参数 | `NotificationManager.ts:21` |
| `@ohos.hilog` | 日志输出 | `Log.ts:15` |
| `@ohos.app.ability.common` | 能力上下文 | `abilityManager.ts:18` |
| `@ohos.application.Want` | 意图对象 | `ServiceExtAbility.ts:16` |

## 2. common 模块 API

### 2.1 Log (日志工具)

**文件**: `common/src/main/ets/default/Log.ts`

**功能**: HiLog 日志封装，支持敏感信息过滤

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `showDebug(tag, format, ...args)` | tag: string, format: string, args: any[] | void | 输出调试日志 |
| `showInfo(tag, format, ...args)` | 同上 | void | 输出信息日志 |
| `showWarn(tag, format, ...args)` | 同上 | void | 输出警告日志 |
| `showError(tag, format, ...args)` | 同上 | void | 输出错误日志 |
| `showFatal(tag, format, ...args)` | 同上 | void | 输出严重日志 |

**使用示例**:

```typescript
import Log from '../../../../../../common/src/main/ets/default/Log';

const TAG = 'MyComponent';

Log.showInfo(TAG, `Component initialized`);
Log.showError(TAG, `Error: ${error.message}`);
```

**敏感信息过滤**:

```typescript
// Log.ts 自动过滤包含 "hide" 的参数
Log.showInfo(TAG, `Password: hide123`); // 输出: Password: **
```

**证据**: `Log.ts:24-35` 过滤装饰器实现

### 2.2 AbilityManager (能力管理)

**文件**: `common/src/main/ets/default/abilitymanager/abilityManager.ts`

**功能**: Ability 上下文管理和启动

#### 静态属性

| 属性 | 值 | 说明 |
|------|------|------|
| `ABILITY_NAME_ENTRY` | `'SystemUi_Entry'` | 主入口 |
| `ABILITY_NAME_STATUS_BAR` | `'SystemUi_StatusBar'` | 状态栏 |
| `ABILITY_NAME_NAVIGATION_BAR` | `'SystemUi_NavigationBar'` | 导航栏 |
| `ABILITY_NAME_VOLUME_PANEL` | `'SystemUi_VolumePanel'` | 音量面板 |

#### 静态方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `setContext(abilityName, context)` | abilityName: string, context: any | void | 设置上下文 |
| `getContext(abilityName?)` | abilityName?: string | any | 获取上下文 |
| `setAbilityContext(abilityName, context)` | 同上 | void | 设置 Ability 上下文 |
| `getAbilityContext(abilityName?)` | 同上 | UIAbilityContext | 获取 Ability 上下文 |
| `startAbility(context, want, callback?)` | context?: any, want: Want, callback?: Function | void | 启动 Ability |
| `startServiceExtensionAbility(context, want, callback?)` | 同上 | void | 启动服务 |

**使用示例**:

```typescript
import AbilityManager from '../../../../../../common/src/main/ets/default/abilitymanager/abilityManager';

let want = {
  bundleName: 'com.ohos.systemui',
  abilityName: 'SystemUi_StatusBar'
};

AbilityManager.startAbility(want);
```

### 2.3 EventManager (事件管理)

**文件**: `common/src/main/ets/default/event/EventManager.ts`

**功能**: 事件发布和订阅管理

#### 事件类型

| 类型 | 常量 | 说明 |
|------|------|------|
| 本地事件 | `'local'` | 组件内通信 |
| Ability 事件 | `'ability'` | 启动其他 Ability |
| 公共事件 | `'commonEvent'` | TODO |
| 远程事件 | `'remote'` | TODO |

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `publish(event, pluginType?)` | event: Event, pluginType?: PluginType | boolean | 发布事件 |
| `subscribe(eventType, callback)` | eventType: Events, callback: Callback | unsubscribe | 订阅事件 |
| `subscribeOnce(eventType, callback)` | 同上 | unsubscribe | 单次订阅 |

**事件对象结构**:

```typescript
interface Event {
  target: 'local' | 'ability' | 'commonEvent' | 'remote';
  data: {
    eventName: string;
    args?: any;
    bundleName?: string;
    abilityName?: string;
  };
}
```

**使用示例**:

```typescript
import EventManager from '../../../../../../common/src/main/ets/default/event/EventManager';

// 发布事件
EventManager.publish({
  target: 'local',
  data: { eventName: 'BATTERY_CHANGED', args: { level: 50 } }
});

// 订阅事件
let unsubscribe = EventManager.subscribe('BATTERY_CHANGED', (args) => {
  console.log(`Battery: ${args.level}%`);
});
```

### 2.4 EventBus (事件总线)

**文件**: `common/src/main/ets/default/event/EventBus.ts`

**功能**: 底层事件总线实现

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `emit(eventType, ...args)` | eventType: string, ...args: any[] | boolean | 触发事件 |
| `on(eventType, callback)` | eventType: string, callback: Callback | unsubscribe | 订阅 |
| `once(eventType, callback)` | 同上 | unsubscribe | 单次订阅 |
| `off(eventType, callback?)` | eventType: string, callback?: Callback | void | 取消订阅 |

### 2.5 WindowManager (窗口管理)

**文件**: `common/src/main/ets/default/WindowManager.ts`

**功能**: 系统窗口创建和管理

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `createWindow(name)` | name: string | Promise\<Window\> | 创建窗口 |
| `getWindow(name)` | name: string | Window | 获取窗口 |
| `destroyWindow(name)` | name: string | void | 销毁窗口 |

### 2.6 NotificationManager (通知管理)

**文件**: `common/src/main/ets/default/abilitymanager/notificationManager.ts`

**功能**: 通知发布和管理

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `publish(notification)` | notification: Notification | void | 发布通知 |
| `cancel(hashCode)` | hashCode: string | void | 取消通知 |
| `getAllActive()` | void | Notification[] | 获取所有活动通知 |

## 3. 系统 API 封装

### 3.1 通知系统 API

**API 包**: `@ohos.notificationManager`

| 方法 | 来源 | 用途 |
|------|------|------|
| `publish()` | NtfMgr | 发布通知 |
| `setNotificationEnable()` | NtfMgr | 启用/禁用通知 |
| `getAllActiveNotifications()` | NtfMgr | 获取活动通知 |

**证据**: `features/noticeitem/src/main/ets/com/ohos/noticeItem/model/NotificationManager.ts:17-21`

### 3.2 插件组件 API

**API 包**: `@ohos.pluginComponent`

| 方法 | 用途 |
|------|------|
| `request()` | 请求插件组件 |
| `push()` | 推送插件组件 |
| `on('request')` | 监听请求事件 |
| `on('push')` | 监听推送事件 |

### 3.3 系统参数 API

**API 包**: `@ohos.systemparameter`

| 方法 | 用途 |
|------|------|
| `getSync(key, defaultValue)` | 同步获取参数 |
| `get(key, callback)` | 异步获取参数 |

## 4. features 模块接口

### 4.1 组件导出模式

features 模块通过以下方式导出接口：

1. **导出类**: `export default class XXXManager`
2. **导出函数**: `export function createXXX()`
3. **导出常量**: `export const CONSTANT_NAME = value`

### 4.2 典型接口

| 组件 | 导出类型 | 说明 |
|------|----------|------|
| `batterycomponent` | Manager 类 | 电池状态管理 |
| `brightnesscomponent` | Manager 类 | 亮度状态管理 |
| `volumecomponent` | Manager 类 | 音量状态管理 |
| `wificomponent` | Manager 类 | WiFi 状态管理 |
| `noticeitem` | ViewModel | 通知项视图模型 |

### 4.3 使用方式

```typescript
// 在 product 模块中引用
import BatteryVM from 'batterycomponent';

// 使用
BatteryVM.getInstance().onBatteryChange((level) => {
  // 处理电量变化
});
```

## 5. 产品模块接口

### 5.1 页面路由

| 模块 | 页面 | 路径 |
|------|------|------|
| `phone_statusbar` | index | `product/phone/statusbar/src/main/ets/pages/index.ets` |
| `pc_controlpanel` | index | `product/pc/controlpanel/src/main/ets/pages/index.ets` |

### 5.2 Ability 入口

每个 product 模块都有 `ServiceExtAbility`:

```
[product]/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts
```

## 6. 常量定义

### 6.1 模块名称常量

**文件**: `common/src/main/ets/default/Constants.ts`

| 常量 | 值 | 说明 |
|------|------|------|
| `MODULE_NAME` | 模块名 | - |
| `ABILITY_NAME_*` | Ability 名称 | 见 AbilityManager |

### 6.2 事件名称常量

| 常量 | 值 | 用途 |
|------|------|------|
| `BATTERY_CHANGED` | `'batteryChanged'` | 电量变化 |
| `WIFI_STATE_CHANGED` | `'wifiStateChanged'` | WiFi 状态变化 |
| `VOLUME_CHANGED` | `'volumeChanged'` | 音量变化 |

## 7. 稳定性标注

### 7.1 稳定接口 (推荐使用)

| 接口 | 位置 | 说明 |
|------|------|------|
| `Log.*` | `common` | 日志工具 |
| `AbilityManager` | `common` | 能力管理 |
| `EventManager` | `common` | 事件管理 |
| `WindowManager` | `common` | 窗口管理 |

### 7.2 内部接口 (谨慎使用)

| 接口 | 位置 | 说明 |
|------|------|------|
| `EventBus` | `common` | 底层事件总线 |
| `ScreenLockManager` | `common` | 锁屏管理 |
| `TimeManager` | `common` | 时间管理 |

## 8. 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [目录结构](02_Directory_Structure.md) - 详细目录说明
- [架构设计](03_Architecture.md) - 系统架构图
- [构建指南](05_Build.md) - 编译配置
- [安全评审](06_Security.md) - 安全考虑
