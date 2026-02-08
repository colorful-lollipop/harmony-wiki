# 关键调用链

> 本文档记录 SystemUI 项目中的关键调用链，用于理解数据流和模块交互。

## 1. 启动流程

### 1.1 SystemUI 启动

```
用户/系统 启动 SystemUI
    ↓
AbilityStage.onCreate()
    ↓
ServiceExtAbility.onCreate(want)
    ↓
initSystemUi(context)
    ↓
├── EventManager.setContext(context)
├── ScreenLockManager.init()
└── TimeManager.init(context)
    ↓
AbilityManager.setContext(ABILITY_NAME_ENTRY, context)
```

**证据**: 
- `entry/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts:24-28`
- `common/src/main/ets/default/InitSystemUi.ts:24-31`

### 1.2 产品模块启动

```
ServiceExtAbility.onCreate()
    ↓
initSystemUi(context)
    ↓
WindowManager.createWindow()
    ↓
loadContent(页面路径)
    ↓
ArkUI 页面渲染
```

## 2. 状态变化处理

### 2.1 电量变化

```
电池状态变化
    ↓
Framework 通知
    ↓
batterycomponent 接收事件
    ↓
EventManager.publish('BATTERY_CHANGED')
    ↓
EventBus.emit()
    ↓
订阅组件更新 UI
    ↓
状态栏/通知显示
```

### 2.2 WiFi 状态变化

```
WiFi 开关/状态变化
    ↓
Framework 通知
    ↓
wificomponent 接收
    ↓
EventManager.publish('WIFI_STATE_CHANGED')
    ↓
EventBus.emit()
    ↓
控制中心/状态栏更新
```

## 3. 用户交互

### 3.1 点击开关 WiFi

```
用户点击 WiFi 开关
    ↓
ArkUI onClick 事件
    ↓
调用 @ohos.wifiManager API
    ↓
系统处理 WiFi 开关
    ↓
结果返回
    ↓
UI 更新 (开关状态)
```

### 3.2 下拉通知面板

```
用户下拉屏幕
    ↓
WindowManager 接收触摸
    ↓
显示通知面板
    ↓
加载 noticeitem 组件
    ↓
从 NotificationManager 获取通知
    ↓
渲染通知列表
```

## 4. 通知流程

### 4.1 接收系统通知

```
应用发送通知
    ↓
NotificationManager 接收
    ↓
EventManager.publish('NOTIFICATION_ARRIVED')
    ↓
EventBus.emit()
    ↓
noticeitem 组件更新
    ↓
通知显示在面板/胶囊
```

### 4.2 删除通知

```
用户滑动删除通知
    ↓
ArkUI 事件处理
    ↓
NotificationManager.remove(hashCode)
    ↓
调用 @ohos.notificationManager API
    ↓
系统删除通知
    ↓
UI 更新
```

## 5. 窗口管理

### 5.1 创建状态栏窗口

```
SystemUI 启动
    ↓
WindowManager.createWindow('statusbar')
    ↓
设置窗口属性 (悬浮、全屏等)
    ↓
loadContent('pages/index.ets')
    ↓
ArkUI 渲染状态栏
```

### 5.2 音量面板弹出

```
用户按音量键
    ↓
SystemUI 接收按键事件
    ↓
WindowManager.createSubWindow('volumepanel')
    ↓
loadContent('pages/index.ets')
    ↓
显示音量滑块
```

## 6. 事件订阅/发布

### 6.1 订阅电量变化

```typescript
// 在组件中
EventManager.subscribe('BATTERY_CHANGED', (args) => {
  this.updateBatteryUI(args.level);
});
```

### 6.2 发布电量变化

```typescript
// 在 batterycomponent 中
EventManager.publish({
  target: 'local',
  data: { eventName: 'BATTERY_CHANGED', args: { level: 50 } }
});
```

## 7. 相关文档

- [架构设计](03_Architecture.md) - 系统架构图
- [内部 API](04_API_Inner.md) - 模块接口
- [项目概览](01_Overview.md) - 项目定位
