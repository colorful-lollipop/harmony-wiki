# 内部 API

## 说明

本文档描述 Camera 应用内部模块之间的接口定义、依赖关系和使用规范。

## 核心服务接口

### CameraService

**路径**: `common/src/main/ets/default/camera/CameraService.ts`

**类定义**:
```typescript
export class CameraService {
  // 相机实例管理
  private mCameraManager: camera.CameraManager;
  private mCameraInput: camera.CameraInput;
  private mCaptureSession: camera.CaptureSession;
  
  // 输出实例
  private mPreviewOutput: camera.PreviewOutput;
  private mPhotoOutPut: camera.PhotoOutput;
  private mVideoOutput: camera.VideoOutput;
  private mAVRecorder: media.AVRecorder;
  
  // 关键方法
  initCamera(config: CameraConfig): Promise<void>;
  releaseCamera(): Promise<void>;
  capturePhoto(): Promise<void>;
  startRecording(): Promise<void>;
  stopRecording(): Promise<void>;
  switchCamera(cameraId: string): Promise<void>;
  setZoomRatio(ratio: number): void;
  setFlashMode(mode: FlashMode): void;
  setExposureMode(mode: ExposureMode): void;
  setFocusMode(mode: FocusMode): void;
}
```

**接口回调**:
```typescript
export interface FunctionCallBack {
  onCapturePhotoOutput(): void;
  onCaptureSuccess(thumbnail: any, resourceUri: any): void;
  onCaptureFailure(): void;
  onRecordSuccess(thumbnail: any): void;
  onRecordFailure(): void;
  thumbnail(thumbnail: any): void;
}

export interface VideoCallBack {
  videoUri(videoUri: any): void;
  onRecodeError(errorMsg: any): void;
}
```

**稳定性**: 核心服务，高稳定性要求

### FeatureManager

**路径**: `common/src/main/ets/default/featureservice/FeatureManager.ts`

**职责**: 
- 管理拍摄模式生命周期
- 协调 Feature 与相机服务
- 处理模式切换

**核心接口**:
```typescript
export class FeatureManager {
  constructor(mode: string, modeMap: IModeMap);
  init(): void;
  release(): void;
  switchMode(newMode: string): void;
  getCurrentMode(): string;
}
```

### SettingManager

**路径**: `common/src/main/ets/default/setting/SettingManager.ts`

**职责**:
- 相机设置管理
- 设置持久化
- 设置变更通知

**核心接口**:
```typescript
export class SettingManager {
  static getInstance(): SettingManager;
  getSetting(key: string): any;
  setSetting(key: string, value: any): void;
  saveSettings(): Promise<void>;
  loadSettings(): Promise<void>;
  getCameraId(): CameraId;
  setCameraId(id: CameraId): void;
  getResolution(): Resolution;
  setResolution(res: Resolution): void;
}
```

### CameraBasicFunction

**路径**: `common/src/main/ets/default/function/CameraBasicFunction.ts`

**职责**:
- 相机基础功能封装
- 全局相机状态管理
- Ability 生命周期联动

**核心接口**:
```typescript
export class CameraBasicFunction {
  static getInstance(): CameraBasicFunction;
  initCamera(config: CameraConfig, trigger: string): void;
  releaseCamera(): void;
  startPreview(): void;
  stopPreview(): void;
}
```

## 状态管理接口

### Redux Store

**路径**: `common/src/main/ets/default/redux/store.ets`

**导出接口**:
```typescript
export function getStore(): Store<OhCombinedState>;
export type Dispatch = typeof store.dispatch;
export type OhCombinedState = ReturnType<typeof rootReducer>;
```

**Action 定义** (`common/src/main/ets/default/redux/actions/Action.ts`):
```typescript
export class Action {
  type: ActionType;
  payload?: any;
}

export enum UiStateMode {
  UI_STATE_MODE_PHOTO = 'PHOTO',
  UI_STATE_MODE_VIDEO = 'VIDEO',
  UI_STATE_MODE_MULTI = 'MULTI'
}

// 常用 Actions
export const setCameraStatus = (status: CameraStatus) => {...};
export const setCurrentMode = (mode: UiStateMode) => {...};
export const setZoomRatio = (ratio: number) => {...};
```

### EventBus

**路径**: `common/src/main/ets/default/worker/eventbus/EventBus.ts`

**核心接口**:
```typescript
export class EventBus {
  on(event: string, callback: Function): void;
  off(event: string, callback?: Function): void;
  emit(event: string, ...args: any[]): void;
  once(event: string, callback: Function): void;
}

// 全局 EventBus 实例
export class EventBusManager {
  static getInstance(): EventBusManager;
  getEventBus(): EventBus;
}
```

**使用示例**:
```typescript
// 订阅事件
EventBusManager.getInstance().getEventBus().on('captureComplete', (data) => {
  // 处理拍照完成
});

// 发布事件
EventBusManager.getInstance().getEventBus().emit('captureComplete', {uri: 'xxx'});
```

## 工具类接口

### Log

**路径**: `common/src/main/ets/default/utils/Log.ts`

**核心接口**:
```typescript
export class Log {
  static info(msg: string): void;
  static debug(msg: string): void;
  static warn(msg: string): void;
  static error(msg: string): void;
  static start(tag: string): void;
  static end(tag: string): void;
  
  // 预定义标签
  static readonly ABILITY_WHOLE_LIFE = 'AbilityWholeLife';
  static readonly ABILITY_VISIBLE_LIFE = 'AbilityVisibleLife';
  static readonly APPLICATION_WHOLE_LIFE = 'ApplicationWholeLife';
}
```

### GlobalContext

**路径**: `common/src/main/ets/default/utils/GlobalContext.ts`

**核心接口**:
```typescript
export class GlobalContext {
  static get(): GlobalContext;
  
  // Ability 上下文
  setCameraAbilityContext(context: Context): void;
  getCameraAbilityContext(): Context;
  
  // Want 参数
  setCameraAbilityWant(want: Want): void;
  getCameraAbilityWant(): Want;
  
  // 全局对象存储
  setObject(key: string, value: any): void;
  getObject(key: string): any;
  
  // 窗口对象
  setCameraWinClass(win: Window): void;
  getCameraWinClass(): Window;
  
  // 其他全局状态...
}
```

**稳定性**: 全局单例，高稳定性

### Constants

**路径**: `common/src/main/ets/default/utils/Constants.ts`

**导出常量**:
```typescript
export enum CameraStatus {
  CAMERA_STATUS_PREVIEW = 0,
  CAMERA_STATUS_CAPTURE = 1,
  CAMERA_STATUS_RECORDING = 2
}

export enum CameraNeedStatus {
  CAMERA_NEED_RELEASE = 0,
  CAMERA_NEED_RESTART = 1,
  CAMERA_NEED_RESUME = 2
}

export class Constants {
  static readonly DEFAULT_ZOOM_RATIO = 1.0;
  static readonly MAX_ZOOM_RATIO = 10.0;
  static readonly MIN_ZOOM_RATIO = 1.0;
  // ... 其他常量
}
```

## UI 组件接口

### 通用组件导出

**路径**: `common/index.ets`

**导出组件清单**:
```typescript
// 相机相关
export { CameraPlatformCapability } from './src/main/ets/default/camera/CameraPlatformCapability';

// UI 组件
export { ShowFlashBlack } from './src/main/ets/default/featurecommon/animate/ShowFlashBlack';
export { AssistiveGridView } from './src/main/ets/default/featurecommon/assistivegridview/AssistiveGridView';
export { BigText } from './src/main/ets/default/featurecommon/bigtext/BigText';
export { CameraSwitchButton } from './src/main/ets/default/featurecommon/cameraswitcher/CameraSwitchButton';
export { CameraSwitchController } from './src/main/ets/default/featurecommon/cameraswitcher/CameraSwitchController';
export { GeoLocation } from './src/main/ets/default/featurecommon/geolocation/GeoLocation';
export { MoreList } from './src/main/ets/default/featurecommon/moreList/moreList';
export { PlaySound } from './src/main/ets/default/featurecommon/playsound/playSound';
export { PreferencesService, PersistType } from './src/main/ets/default/featurecommon/preferences/PreferencesService';
export { ScreenLockManager } from './src/main/ets/default/featurecommon/screenlock/ScreenLockManager';
export { SettingData } from './src/main/ets/default/featurecommon/settingview/model/SettingData';
export { SettingListModel } from './src/main/ets/default/featurecommon/settingview/model/SettingListModel';
export { SettingItem } from './src/main/ets/default/featurecommon/settingview/phone/SettingItem';
export { TabletSettingItem } from './src/main/ets/default/featurecommon/settingview/tablet/TabletSettingItem';
export { ShutterButton } from './src/main/ets/default/featurecommon/shutterbutton/ShutterButton';
export { ShutterButtonLand } from './src/main/ets/default/featurecommon/shutterbutton/ShutterButtonLand';
export { TabBar } from './src/main/ets/default/featurecommon/tabbar/TabBar';
export { TabBarLand } from './src/main/ets/default/featurecommon/tabbar/TabBarLand';
export { ThumbnailView } from './src/main/ets/default/featurecommon/thumbnail/ThumbnailView';
export { TimeLapseView } from './src/main/ets/default/featurecommon/timelapseview/TimeLapseView';
export { ZoomText } from './src/main/ets/default/featurecommon/zoomview/ZoomText';
export { ZoomView } from './src/main/ets/default/featurecommon/zoomview/ZoomView';
export { ZoomViewLand } from './src/main/ets/default/featurecommon/zoomview/ZoomViewLand';

// 功能类
export { FunctionId } from './src/main/ets/default/featureservice/FunctionId';
export { FeatureManager } from './src/main/ets/default/featureservice/FeatureManager';
export { IModeMap } from './src/main/ets/default/featureservice/IModeMap';
export { CameraBasicFunction } from './src/main/ets/default/function/CameraBasicFunction';
export { CaptureFunction } from './src/main/ets/default/function/CaptureFunction';
export { RecordFunction } from './src/main/ets/default/function/RecordFunction';
export { ZoomFunction } from './src/main/ets/default/function/ZoomFunction';

// Redux
export { Action, UiStateMode } from './src/main/ets/default/redux/actions/Action';
export { getStore, Dispatch, OhCombinedState } from './src/main/ets/default/redux/store';

// 设置
export { SettingManager } from './src/main/ets/default/setting/SettingManager';
export { CameraId } from './src/main/ets/default/setting/settingitem/CameraId';
export { RdbStoreManager } from './src/main/ets/default/setting/storage/RdbStoreManager';

// 工具
export { ComponentPosition } from './src/main/ets/default/utils/ComponentPosition';
export { Constants, CameraStatus, CameraNeedStatus } from './src/main/ets/default/utils/Constants';
export { Log } from './src/main/ets/default/utils/Log';

// Worker
export { CameraWorker } from './src/main/ets/default/worker/CameraWorker';
export { WorkerManager } from './src/main/ets/default/worker/WorkerManager';
export { EventBus } from './src/main/ets/default/worker/eventbus/EventBus';
export { EventBusManager } from './src/main/ets/default/worker/eventbus/EventBusManager';
export { GlobalContext } from './src/main/ets/default/utils/GlobalContext';
```

## 模块依赖关系

### 依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                         Product 层                           │
│  Phone/Tablet                                               │
│  ├─ imports: @ohos/common (index.ets)                       │
│  ├─ imports: @ohos/photo                                    │
│  ├─ imports: @ohos/video                                    │
│  └─ imports: @ohos/multi                                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Feature 层                            │
│  Photo/Video/Multi                                          │
│  ├─ imports: @ohos/common (CameraService, etc.)             │
│  └─ exports: {Photo/Video/Multi}Mode, {Photo/Video/Multi}ModeParam │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         Common 层                            │
│  Common                                                     │
│  ├─ camera/ (CameraService, ...)                            │
│  ├─ featurecommon/ (UI 组件)                                │
│  ├─ featureservice/ (FeatureManager)                        │
│  ├─ function/ (功能类)                                       │
│  ├─ redux/ (状态管理)                                        │
│  ├─ setting/ (设置管理)                                      │
│  ├─ utils/ (工具类)                                          │
│  └─ worker/ (多线程)                                         │
└─────────────────────────────────────────────────────────────┘
```

### 依赖规则

1. **Product 层**: 可依赖 Common 层和所有 Feature 层
2. **Feature 层**: 只依赖 Common 层，Feature 之间不直接依赖
3. **Common 层**: 不依赖其他模块，只使用系统 SDK

## 接口稳定性分级

| 分级 | 接口 | 说明 |
|------|------|------|
| **稳定** | CameraService, SettingManager, GlobalContext, Log | 核心服务，变更需兼容性考虑 |
| **稳定** | Redux Store, EventBus | 基础设施，接口固定 |
| **半稳定** | UI 组件接口 | 可能随 UI 调整 |
| **不稳定** | Feature 接口 | 随功能迭代可能调整 |

## 接口使用示例

### 初始化相机

```typescript
import { CameraBasicFunction } from '@ohos/common';

const cameraFunc = CameraBasicFunction.getInstance();
cameraFunc.initCamera({
  cameraId: 'BACK',
  mode: 'PHOTO'
}, 'onCreate');
```

### 状态管理

```typescript
import { getStore, Action, UiStateMode } from '@ohos/common';

const store = getStore();
store.dispatch({
  type: 'SET_CAMERA_STATUS',
  payload: CameraStatus.CAMERA_STATUS_PREVIEW
});
```

### 事件通信

```typescript
import { EventBusManager } from '@ohos/common';

const eventBus = EventBusManager.getInstance().getEventBus();

// 订阅
eventBus.on('captureSuccess', (data) => {
  console.info('Capture success:', data.uri);
});

// 发布
eventBus.emit('captureSuccess', { uri: 'file://xxx.jpg' });
```

### 设置管理

```typescript
import { SettingManager, CameraId } from '@ohos/common';

const settingManager = SettingManager.getInstance();
settingManager.setCameraId(CameraId.BACK);
const currentId = settingManager.getCameraId();
```
