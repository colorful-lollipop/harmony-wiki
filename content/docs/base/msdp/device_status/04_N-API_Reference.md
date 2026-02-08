# 对外 JavaScript API (N-API) 参考

## 目的

本文档完整列出 `device_status` 模块的所有 JavaScript N-API 接口，包括方法、参数、返回值、权限要求和对应的 C++ 实现函数。

---

## N-API 模块清单

本模块包含 **11 个 N-API 模块**，按功能分类：

| 序号 | 模块名称 | JS 模块标识 | 主要功能 |
|------|-----------|-----------------|---------|
| 1 | Stationary | `stationary` | 设备静止状态订阅（legacy） |
| 2 | Device Status V1 | `multimodalAwareness.deviceStatus` | 设备状态 v1，姿态数据获取 |
| 3 | Motion | `multimodalAwareness.motion` | 运动感知 |
| 4 | Distance Measurement | `multimodalAwareness.distanceMeasurement` | 距离测量 |
| 5 | On-Screen | `multimodalAwareness.onScreen` | 屏幕感知 |
| 6 | Screen Event | `screenevent` | 屏幕事件订阅 |
| 7 | User Status (Underage) | `multimodalAwareness.userStatus` | 用户年龄模型 |
| 8 | Metadata Binding (Boomerang) | `multimodalAwareness.metadataBinding` | 元数据绑定 |
| 9 | Drag Interaction | `deviceStatus.dragInteraction` | 拖拽交互 |
| 10 | Input Device Cooperation | `multimodalInput.inputDeviceCooperate` | 输入设备协同 |
| 11 | Coordination (Legacy) | `cooperate` | 设备协同（legacy） |

---

## N-API 导入和使用方式

### ES6/ArkTS 导入

```typescript
// 示例：导入设备状态 v1 模块
import { deviceStatus } from '@ohos.multimodalAwareness.deviceStatus';
```

### C++ N-API 导入

```javascript
// 示例：导入静止状态模块（legacy）
import station from '@ohos.stationary';
```

---

## 模块详细说明

### 1. Stationary 模块（静止状态）

**模块标识**: `stationary`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `on(type, eventMode, latency, callback)` | 订阅设备状态变化 | void |
| `off(type, eventMode, callback)` | 取消订阅 | void |
| `once(type, callback)` | 获取设备状态（一次性） | Promise<Data> |

**参数说明**：

| 参数 | 类型 | 说明 | 可选值 |
|------|--------|------|--------|
| `type` | int32 | 设备状态类型 | `Type::TYPE_STILL`, `Type::TYPE_RELATIVE_STILL` |
| `eventMode` | int32 | 事件模式 | `ActivityEvent::ENTER`, `ActivityEvent::EXIT`, `ActivityEvent::ENTER_EXIT` |
| `latency` | int64 | 报告延迟（纳秒） | 默认 0 |
| `callback` | function | 回调函数 | - |

**导出常量**：

```typescript
// ActivityEvent 事件类型
export enum ActivityEvent {
  ENTER,     // 进入状态
  EXIT,      // 退出状态
  ENTER_EXIT // 进入然后退出
}

// ActivityState 状态类型
export enum ActivityState {
  ENTER,  // 进入
  EXIT    // 退出
}
```

**C++ 实现文件**: `frameworks/js/napi/src/devicestatus_napi.cpp`

**C++ 入口函数**: `SubscribeCallback()`, `UnsubscribeCallback()`, `GetDeviceStatusOnce()`

---

### 2. Device Status V1 模块（设备状态 v1）

**模块标识**: `multimodalAwareness.deviceStatus`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `on(type, callback)` | 订阅设备状态 | void |
| `off(type, callback)` | 取消订阅 | void |
| `getDeviceRotationRadian()` | 获取设备姿态旋转角度（弧度） | Promise<number> |

**参数说明**：

| 参数 | 类型 | 说明 |
|------|--------|------|
| `type` | int32 | 设备状态类型 | - |
| `callback` | function | 回调函数 | - |

**导出常量**：

```typescript
// SteadyStandingStatus 状态
export enum SteadyStandingStatus {
  STATUS_ENTER,  // 站立状态
  STATUS_EXIT   // 未站立状态
}
```

**C++ 实现文件**: `frameworks/js/napi/device_status/src/device_status_napi.cpp`

**C++ 入口函数**: `Subscribe()`, `Unsubscribe()`, `GetDevicePostureData()`

---

### 3. Motion 模块（运动感知）

**模块标识**: `multimodalAwareness.motion`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `on(type, callback)` | 订阅运动事件 | void |
| `off(type, callback)` | 取消订阅 | void |
| `getRecentOperatingHandStatus()` | 获取最近操作手状态 | Promise<OperatingHandStatus> |
| `getRecentHoldingHandStatus()` | 获取最近握持状态 | Promise<HoldingHandStatus> |

**参数说明**：

| 参数 | 类型 | 说明 | 可选值 |
|------|--------|------|--------|
| `type` | string | 运动事件类型 | "operatingHandChanged", "steadyStandingDetect", "remotePhotoStandingDetect", "holdingHandChanged" |

**导出常量**：

```typescript
// 操作手状态
export enum OperatingHandStatus {
  UNKNOWN_STATUS = -1,
  LEFT_HAND_OPERATED = 0,
  RIGHT_HAND_OPERATED = 1
}

// 握持状态
export enum HoldingHandStatus {
  UNKNOWN_STATUS = -1,
  NOT_HELD = 0,
  LEFT_HAND_HELD = 1,
  RIGHT_HAND_HELD = 2,
  BOTH_HANDS_HELD = 3
}
```

**C++ 实现文件**: `frameworks/js/napi/motion/src/motion_napi.cpp`

**C++ 入口函数**: `Subscribe()`, `Unsubscribe()`, `GetRecentOperatingHandStatus()`, `GetRecentHoldingHandStatus()`

---

### 4. Distance Measurement 模块（距离测量）

**模块标识**: `multimodalAwareness.distanceMeasurement`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `onDistanceMeasure(config, callback)` | 订阅距离测量 | void |
| `offDistanceMeasure(config, callback)` | 取消订阅 | void |
| `onIndoorOrOutdoorIdentify(config, callback)` | 订阅室内外识别 | void |
| `offIndoorOrOutdoorIdentify(config, callback)` | 取消订阅 | void |

**参数说明**：

| 参数 | 类型 | 说明 | 结构 |
|------|--------|------|--------|
| `config` | DistanceMeasureConfig / IndoorOrOutdoorIdentifyConfig | 配置对象 |
| `callback` | function | 回调函数 | - |

**配置对象**：

```typescript
export interface DistanceMeasureConfig {
  deviceList: Array<string>;           // 设备列表
  techType: TechType;                // 技术类型
  reportMode: ReportMode;             // 报告模式
  reportFrequency: number;            // 报告频率
}

export interface IndoorOrOutdoorIdentifyConfig {
  deviceList: Array<string>;
  techType: TechType;
  reportMode: ReportMode;
  reportFrequency: number;
}
```

**技术类型**：

```typescript
export enum TechType {
  BLE_RSSI = 0,        // BLE 信号强度
  WIFI_RSSI = 1,       // WIFI 信号强度
  ULTRASOUND = 2,      // 超宽带
  NEARLINK = 3,       // UWB
  WIFI_BLE_RSSI = 4   // WIFI + BLE 混合
}
```

**报告模式**：

```typescript
export enum ReportMode {
  AUTO = 0,    // 自动报告
  MANUAL = 1     // 手动报告
}
```

**C++ 实现文件**: `frameworks/js/napi/distance_measurement/src/distance_measurement_napi.cpp`

**C++ 入口函数**: `OnDistanceMeasure()`, `OffDistanceMeasure()`, `OnIndoorOrOutdoorIdentify()`, `OffIndoorOrOutdoorIdentify()`

---

### 5. On-Screen 模块（屏幕感知）

**模块标识**: `multimodalAwareness.onScreen`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `sendControlEvent(event)` | 发送控制事件 | Promise<void> |
| `getPageContent(option)` | 获取页面内容 | Promise<string> |
| `subscribe(cap, callback, option)` | 注册感知回调 | void |
| `unsubscribe(cap, callback)` | 取消注册感知回调 | void |
| `trigger(cap, option)` | 触发感知能力 | Promise<void> |

**参数说明**：

| 参数 | 类型 | 说明 | 示例 |
|------|--------|------|--------|
| `event` | string | 控制事件类型 | "SCROLL_TO_HOOK" |
| `option` | PageContentOption | 页面内容选项 | `{ scenario: 'ARTICLE' }` |
| `cap` | OnScreenCapability | 感知能力枚举 | 见下方 |

**感知能力枚举**：

```typescript
export enum OnScreenCapability {
  contentUiTree = 0,                    // UI 树内容
  contentUiOcr = 1,                      // OCR 内容
  contentScreenshot = 2,                 // 截图内容
  contentLink = 3,                        // 链接内容
  contentUiTreeWithImage = 4,          // UI 树+图片
  interactionTextSelection = 5,          // 文本选择
  interactionClick = 6,                  // 点击事件
  interactionScroll = 7,                   // 滚轮事件
  scenarioReading = 8,                  // 场景读取
  scenarioShortVideo = 9,               // 短视频读取
  scenarioActivity = 10,                 // 活动检测
  scenarioTodo = 11,                    // 待办事项
  screenshotIntent = 12                // 截图意图
}
```

**C++ 实现文件**: `frameworks/js/napi/onscreen/src/on_screen_napi.cpp`

**C++ 入口函数**: `SendControlEvent()`, `GetPageContent()`, `Subscribe()`, `Unsubscribe()`, `Trigger()`

**权限要求**：
- `ohos.permission.GET_SCREEN_CONTENT` - 获取页面内容权限

---

### 6. Screen Event 模块（屏幕事件）

**模块标识**: `screenevent`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `registerScreenEvent(windowId, event, callback)` | 注册屏幕事件监听 | void |
| `unregisterScreenEvent(windowId, event, callback)` | 取消注册 | void |
| `isParallelFeatureEnabled(windowId)` | 检查并行功能是否启用 | Promise<boolean> |
| `getLiveStatus()` | 获取直播状态 | Promise<number> |

**参数说明**：

| 参数 | 类型 | 说明 | 示例 |
|------|--------|------|--------|
| `windowId` | number | 窗口 ID | 0 表示主窗口 |
| `event` | string | 事件类型 | "visibility_change" 等 |
| `callback` | function | 回调函数 | - |

**C++ 实现文件**: `frameworks/js/napi/onscreen/src/screen_event_napi.cpp`

**C++ 入口函数**: `RegisterScreenEvent()`, `UnregisterScreenEvent()`, `IsParallelFeatureEnabled()`, `GetLiveStatus()`

---

### 7. User Status (Underage) 模块（用户年龄模型）

**模块标识**: `multimodalAwareness.userStatus`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `on(type, callback)` | 订阅用户年龄组检测 | void |
| `off(type, callback)` | 取消订阅 | void |

**参数说明**：

| 参数 | 类型 | 说明 | 枚举值 |
|------|--------|------|--------|
| `type` | string | 用户年龄组类型 | "OTHERS", "CHILD" |

**导出常量**：

```typescript
export enum UserAgeGroup {
  OTHERS,  // 其他
  CHILD    // 儿童
}
```

**C++ 实现文件**: `frameworks/js/napi/underage_model/src/underage_model_napi.cpp`

**C++ 入口函数**: `Subscribe()`, `Unsubscribe()`

---

### 8. Boomerang 模块（元数据绑定）

**模块标识**: `multimodalAwareness.metadataBinding`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `on(eventType, bundleName, callback)` | 注册元数据绑定监听 | void |
| `off(eventType, bundleName)` | 取消注册 | void |
| `notifyMetadataBindingEvent(bundleName)` | 通知元数据绑定事件 | Promise<void> |
| `submitMetadata(metadata)` | 提交元数据 | Promise<void> |
| `encodeImage(pixelMap, metadata)` | 编码图片元数据 | Promise<void> |
| `decodeImage(pixelMap)` | 解码图片元数据 | Promise<void> |

**参数说明**：

| 参数 | 类型 | 说明 | 示例 |
|------|--------|------|--------|
| `eventType` | number | 事件类型 | 1-10 |
| `bundleName` | string | 应用包名 | "com.example.app" |
| `pixelMap` | string | 像素映射 | "0,1,2,3..." |
| `metadata` | Metadata | 元数据对象 | 见下方 |

**元数据对象**：

```typescript
export interface Metadata {
  pixelMap: string;        // 像素映射
  fileName: string;          // 文件名
  fileId: string;           // 文件 ID
  scene: string;            // 场景
  timestamp: number;        // 时间戳
}
```

**C++ 实现文件**: `frameworks/js/napi/boomerang/src/boomerang_napi.cpp`

**C++ 入口函数**: `On()`, `Off()`, `NotifyMetadataBindingEvent()`, `SubmitMetadata()`, `EncodeImage()`, `DecodeImage()`

---

### 9. Drag Interaction 模块（拖拽交互）

**模块标识**: `deviceStatus.dragInteraction`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `on(type, callback)` | 注册拖拽监听器 | void |
| `off(type, callback)` | 取消注册拖拽监听器 | void |
| `getDataSummary()` | 获取拖拽数据摘要 | Promise<DragData> |
| `setDragSwitchState(enable)` | 设置拖拽开关状态 | Promise<void> |
| `setAppDragSwitchState(enable, pkgName)` | 设置应用拖拽开关 | Promise<void> |

**参数说明**：

| 参数 | 类型 | 说明 | 枚举值 |
|------|--------|------|--------|
| `type` | string | 监听器类型 | "drag" |
| `enable` | boolean | 是否启用 | true/false |
| `pkgName` | string | 应用包名 | "com.example.app" |

**C++ 实现文件**：
- `frameworks/js/napi/interaction/drag/src/native_register_module.cpp` - 模块注册
- `frameworks/js/napi/interaction/drag/src/js_drag_context.cpp` - 上下文实现
- `frameworks/js/napi/interaction/drag/src/js_drag_manager.cpp` - 管理器实现

---

### 10. Input Device Cooperation 模块（输入设备协同）

**模块标识**: `multimodalInput.inputDeviceCooperate`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `enable(enable, callback)` | 启用/禁用协同 | Promise<void> |
| `start(remoteNetworkDescriptor, inputDeviceId, callback)` | 启动协同 | Promise<void> |
| `stop(callback)` | 停止协同 | Promise<void> |
| `getState(deviceDescriptor, callback)` | 获取协同状态 | Promise<CooperateState> |
| `on(type, callback)` | 注册协同监听器 | void |
| `off(type, callback)` | 取消注册协同监听器 | void |

**参数说明**：

| 参数 | 类型 | 说明 |
|------|--------|------|
| `enable` | boolean | 是否启用 | true/false |
| `remoteNetworkDescriptor` | string | 远程网络描述符 | JSON 字符串 |
| `inputDeviceId` | number | 输入设备 ID | 0 表示主设备 |
| `type` | string | 监听器类型 | "cooperation", "cooperateMessage", "cooperateMouse" |

**监听器类型**：

```typescript
export type ListenerType = 'cooperation' | 'cooperateMessage' | 'cooperateMouse';

export type CallbackType = (enable: boolean, disable: boolean, start: void, stop: void, getCooperateState: void) => void;
```

**C++ 实现文件**：
- `frameworks/js/napi/interaction/cooperate/src/native_register_module.cpp` - 模块注册
- `frameworks/js/napi/interaction/cooperate/src/js_cooperate_context.cpp` - 上下文实现
- `frameworks/js/napi/interaction/cooperate/src/js_cooperate_manager.cpp` - 管理器实现

---

### 11. Coordination 模块（设备协同 Legacy）

**模块标识**: `cooperate`

**导出接口**：

| 方法 | 签名 | 说明 | 返回类型 |
|------|--------|------|--------|
| `prepare(callback)` / `prepareCooperate(callback)` | 准备协同 | Promise<void> |
| `unprepare(callback)` / `unprepareCooperate(callback)` | 取消准备 | Promise<void> |
| `activate(targetNetworkId, inputDeviceId, callback)` | 激活协同 | Promise<void> |
| `activateCooperateWithOptions(targetNetworkId, inputDeviceId, options)` | 激活协同（带选项） | Promise<void> |
| `deactivate(isUnchained, callback)` | 去激活协同 | Promise<void> |
| `getCrossingSwitchState(networkId, callback)` | 获取跨设备开关状态 | Promise<boolean> |
| `on(type, callback)` | 注册协同监听器 | void |
| `off(type, callback)` | 取消注册监听器 | void |

**参数说明**：

| 参数 | 类型 | 说明 | 结构 |
|------|--------|------|--------|
| `targetNetworkId` | string | 目标网络 ID | - |
| `inputDeviceId` | number | 输入设备 ID | - |
| `options` | object | 协同选项 | `{ isAllowInputCross: boolean }` |

**C++ 实现文件**：
- `frameworks/js/napi/interaction/coordination/src/native_register_module.cpp` - 模块注册
- `frameworks/js/napi/interaction/coordination/src/js_coordination_context.cpp` - 上下文实现
- `frameworks/js/napi/interaction/coordination/src/js_coordination_manager.cpp` - 管理器实现

---

## 权限要求汇总

以下 N-API 模块需要特定权限：

| 模块 | 所需权限 | 权限描述 |
|--------|-----------|----------|
| **On-Screen** | `ohos.permission.GET_SCREEN_CONTENT` | 获取屏幕页面内容 |
| **Cooperate** | `ohos.permission.COOPERATE_MANAGER` | 设备协同管理 |

**注**：其他模块主要依赖应用自身的权限声明。

---

## 错误码规范

### N-API 通用错误码

| 错误码 | 说明 | 触发场景 |
|--------|--------|----------|
| 201 | 权限检查失败 | 权限不足 |
| -1 | 系统服务不可用 | 服务未启动 |
| -200 | 事件不支持 | 当前场景不支持 |

**C++ 错误处理文件**：
- `frameworks/js/napi/include/devicestatus_napi_error.h`
- `frameworks/js/napi/include/motion_napi_error.h`
- `frameworks/js/napi/include/onscreen/on_screen_napi_error.h`
- `frameworks/js/napi/include/distance_measurement/distance_measurement_napi_error.h`
- `frameworks/js/napi/include/boomerang/boomerang_napi_error.h`

---

## 回调机制

### 事件回调

所有订阅类模块都支持事件回调：

```cpp
// 回调函数签名
void CallbackFunction(Data data) {
    // 处理事件数据
}
```

### Promise 支持

部分异步方法支持 Promise：

```cpp
// Promise 模式
napi_create_promise(env, &deferred)
napi_resolve_deferred(env, deferred, result)
napi_reject_deferred(env, deferred, code, message)
```

---

## 使用示例

### ES6/ArkTS 示例

```typescript
// 导入设备状态模块
import { deviceStatus } from '@ohos.multimodalAwareness.deviceStatus';

// 订阅设备状态
deviceStatus.on(deviceStatus.Type.STILL, (data) => {
  console.log(`Device status changed: ${data.value}`);
});
```

### JavaScript 示例

```javascript
// 导入静止状态模块（legacy）
const station = requireNative('stationary');

// 订阅设备状态
station.on(station.Type.TYPE_STILL, station.Event.ENTER, (data) => {
  console.log('Device entered:', data.value);
});
```

---

## 相关跳转

- **[01_Overview](01_Overview.md)** - 项目概览
- **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构
- **[03_Architecture](03_Architecture.md)** - 架构设计
- **[05_Inner_API](05_Inner_API.md)** - 内部 API
- **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标
- **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
