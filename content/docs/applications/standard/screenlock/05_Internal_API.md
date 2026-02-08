# 05. 内部API

> 目的: 详细说明ScreenLock内部模块间的接口和调用关系  
> 适用范围: 模块开发者、维护人员

---

## 1. Common模块接口

### 1.1 EventBus接口

**文件**: `common/src/main/ets/default/event/EventBus.ts`

```typescript
export interface EventBus<T> {
    /**
     * 订阅事件
     * @param event 事件名或事件名数组
     * @param cb 回调函数
     * @returns 取消订阅函数
     */
    on(event: T | T[], cb: Callback): () => void;
    
    /**
     * 订阅事件（一次性）
     * @param event 事件名
     * @param cb 回调函数
     * @returns 取消订阅函数
     */
    once(event: T, cb: Callback): () => void;
    
    /**
     * 取消订阅
     * @param event 事件名或事件名数组
     * @param cb 回调函数
     */
    off(event: T | T[] | undefined, cb: Callback): void;
    
    /**
     * 触发事件
     * @param event 事件名
     * @param args 事件参数
     */
    emit(event: T, args: any): void;
}

// 创建事件总线
export function createEventBus<T extends string>(): EventBus<T>
```

**使用示例**:
```typescript
const eventBus = createEventBus<'event1' | 'event2'>();

// 订阅
const unsubscribe = eventBus.on('event1', (data) => {
    console.log(data);
});

// 触发
eventBus.emit('event1', { key: 'value' });

// 取消订阅
unsubscribe();
```

**稳定性**: ✅ 稳定接口

---

### 1.2 EventManager接口

**文件**: `common/src/main/ets/default/event/EventManager.ts`

```typescript
class EventManager {
    /**
     * 发布本地事件
     * @param event 事件对象
     */
    publish(event: LocalEvent): void;
    
    /**
     * 订阅本地事件
     * @param eventName 事件名
     * @param callback 回调函数
     */
    subscribe(eventName: string, callback: EventCallback): void;
    
    /**
     * 取消订阅
     * @param eventName 事件名
     * @param callback 回调函数
     */
    unsubscribe(eventName: string, callback: EventCallback): void;
    
    /**
     * 启动Ability并传递数据
     * @param want Want对象
     * @param event 事件数据
     */
    startAbility(want: Want, event: LocalEvent): void;
    
    /**
     * 发送远程事件
     * @param event 远程事件
     */
    sendRemoteEvent(event: RemoteEvent): void;
    
    /**
     * 订阅公共事件
     * @param action 事件动作
     * @param callback 回调函数
     */
    subscribeCommonEvent(action: string, callback: CommonEventCallback): void;
}

// 全局实例
export const sEventManager: EventManager;
```

**使用示例**:
```typescript
import { sEventManager, obtainLocalEvent } from '@ohos/common';

// 发布事件
sEventManager.publish(obtainLocalEvent('SCREEN_CHANGE_EVENT', true));

// 订阅事件
sEventManager.subscribe('SCREEN_CHANGE_EVENT', (data) => {
    console.log('Screen changed:', data);
});
```

**稳定性**: ✅ 稳定接口

---

### 1.3 WindowManager接口

**文件**: `common/src/main/ets/default/WindowManager.ts`

```typescript
export enum WindowType {
  STATUS_BAR = "SystemUi_StatusBar",           // 2108
  NAVIGATION_BAR = "SystemUi_NavigationBar",   // 2112
  DROPDOWN_PANEL = "SystemUi_DropdownPanel",   // 2109
  NOTIFICATION_PANEL = "SystemUi_NotificationPanel", // 2111
  CONTROL_PANEL = "SystemUi_ControlPanel",     // 2111
  VOLUME_PANEL = "SystemUi_VolumePanel",       // 2111
  BANNER_NOTICE = 'SystemUi_BannerNotice'      // 2111
}

export interface WindowInfo {
  visibility: boolean;
  rect: Rect;
}

export interface Rect {
  left: number;
  top: number;
  width: number;
  height: number | string;
}

class WindowManager {
    /**
     * 创建窗口
     */
    createWindow(
        context: any, 
        name: WindowType, 
        rect: Rect, 
        loadContent: string
    ): Promise<WindowHandle>;
    
    /**
     * 调整窗口大小
     */
    resetSizeWindow(name: WindowType, rect: Rect): Promise<void>;
    
    /**
     * 显示窗口
     */
    showWindow(name: WindowType): Promise<void>;
    
    /**
     * 隐藏窗口
     */
    hideWindow(name: WindowType): Promise<void>;
    
    /**
     * 获取窗口信息
     */
    getWindowInfo(name: WindowType): WindowInfo | undefined;
    
    /**
     * 设置窗口配置信息
     */
    setWindowInfo(configInfo: ConfigInfo): void;
}

// 全局实例
export const sWindowManager: WindowManager;
```

**使用示例**:
```typescript
import { sWindowManager, WindowType } from '@ohos/common';

// 创建窗口
await sWindowManager.createWindow(
    context,
    WindowType.STATUS_BAR,
    { left: 0, top: 0, width: 1080, height: 80 },
    'pages/statusbar'
);

// 显示窗口
await sWindowManager.showWindow(WindowType.STATUS_BAR);
```

**稳定性**: ✅ 稳定接口

---

### 1.4 AbilityManager接口

**文件**: `common/src/main/ets/default/abilitymanager/abilityManager.ts`

```typescript
class AbilityManager {
    // Ability名称常量
    static ABILITY_NAME_SCREEN_LOCK = "screenLockAbility";
    static ABILITY_NAME_STATUS_BAR = "statusBarAbility";
    static ABILITY_NAME_NAVIGATION_BAR = "navigationBarAbility";
    static ABILITY_NAME_NOTIFICATION_PANEL = "notificationPanelAbility";
    static ABILITY_NAME_CONTROL_PANEL = "controlPanelAbility";
    static ABILITY_NAME_VOLUME_PANEL = "volumePanelAbility";
    static ABILITY_NAME_BANNER_NOTICE = "bannerNoticeAbility";
    
    /**
     * 设置Ability上下文
     */
    setContext(abilityName: string, context: any): void;
    
    /**
     * 获取Ability上下文
     */
    getContext(abilityName: string): any;
    
    /**
     * 设置Ability数据
     */
    setAbilityData(abilityName: string, key: string, value: any): void;
    
    /**
     * 获取Ability数据
     */
    getAbilityData(abilityName: string, key: string): any;
    
    /**
     * 启动Ability
     */
    startAbility(want: Want): Promise<void>;
    
    /**
     * 启动Ability并等待结果
     */
    startAbilityForResult(want: Want): Promise<AbilityResult>;
    
    /**
     * 终止Ability
     */
    terminateAbility(): Promise<void>;
}

// 全局实例
export const AbilityManager: AbilityManager;
```

**使用示例**:
```typescript
import { AbilityManager } from '@ohos/common';

// 设置上下文
AbilityManager.setContext(
    AbilityManager.ABILITY_NAME_SCREEN_LOCK, 
    this.context
);

// 设置数据
AbilityManager.setAbilityData(
    AbilityManager.ABILITY_NAME_STATUS_BAR,
    "rect",
    { left: 0, top: 0, width: 1080, height: 80 }
);

// 获取数据
const rect = AbilityManager.getAbilityData(
    AbilityManager.ABILITY_NAME_STATUS_BAR,
    "rect"
);
```

**稳定性**: ✅ 稳定接口

---

### 1.5 Log接口

**文件**: `common/src/main/ets/default/Log.ts`

```typescript
class Log {
    /**
     * 调试日志
     */
    static showDebug(tag: string, message: string): void;
    
    /**
     * 信息日志
     */
    static showInfo(tag: string, message: string): void;
    
    /**
     * 警告日志
     */
    static showWarn(tag: string, message: string): void;
    
    /**
     * 错误日志
     */
    static showError(tag: string, message: string): void;
    
    /**
     * 致命错误日志
     */
    static showFatal(tag: string, message: string): void;
    
    /**
     * 打印日志（简化版）
     */
    static printLog(level: LogLevel, tag: string, message: string): void;
}

// 日志级别
enum LogLevel {
    DEBUG = 3,
    INFO = 4,
    WARN = 5,
    ERROR = 6,
    FATAL = 7
}

export { Log };
```

**使用示例**:
```typescript
import { Log } from '@ohos/common';

const TAG = "MyComponent";

Log.showInfo(TAG, "Application started");
Log.showDebug(TAG, `Debug data: ${JSON.stringify(data)}`);
Log.showError(TAG, `Error occurred: ${error.message}`);
```

**稳定性**: ✅ 稳定接口

---

### 1.6 TimeManager接口

**文件**: `common/src/main/ets/default/TimeManager.ts`

```typescript
class TimeManager {
    /**
     * 初始化时间管理器
     */
    init(context: any): void;
    
    /**
     * 释放资源
     */
    release(): void;
    
    /**
     * 获取当前时间字符串
     */
    getCurrentTime(): string;
    
    /**
     * 获取当前日期字符串
     */
    getCurrentDate(): string;
    
    /**
     * 是否24小时制
     */
    is24HourFormat(): boolean;
    
    /**
     * 获取星期字符串
     */
    getWeekString(): string;
    
    /**
     * 获取农历日期
     */
    getLunarDate(): string;
}

// 全局实例
export const sTimeManager: TimeManager;
```

**使用示例**:
```typescript
import { sTimeManager } from '@ohos/common';

// 初始化
sTimeManager.init(context);

// 获取时间
const time = sTimeManager.getCurrentTime();
const date = sTimeManager.getCurrentDate();
const is24Hour = sTimeManager.is24HourFormat();
```

**稳定性**: ✅ 稳定接口

---

### 1.7 SwitchUserManager接口

**文件**: `common/src/main/ets/default/SwitchUserManager.ts`

```typescript
class SwitchUserManager {
    /**
     * 初始化
     */
    init(): void;
    
    /**
     * 获取当前用户ID
     */
    getCurrentUserId(): number;
    
    /**
     * 获取当前用户名称
     */
    getCurrentUserName(): string;
    
    /**
     * 切换用户
     */
    switchUser(userId: number): Promise<void>;
    
    /**
     * 查询所有用户
     */
    queryAllUsers(): Promise<UserInfo[]>;
    
    /**
     * 注册用户变化监听
     */
    onUserChange(callback: UserChangeCallback): void;
}

// 全局实例
export const sSwitchUserManager: SwitchUserManager;
```

**稳定性**: ✅ 稳定接口

---

### 1.8 ScreenLockManager接口

**文件**: `common/src/main/ets/default/ScreenLockManager.ts`

```typescript
export const SCREEN_CHANGE_EVENT = "screenChangeEvent";

class ScreenLockManager {
    /**
     * 初始化，订阅屏幕事件
     */
    init(): Promise<void>;
    
    /**
     * 通知屏幕事件
     */
    notifyScreenEvent(isScreenOn: boolean): void;
}

// 全局单例
export default sScreenLockManager: ScreenLockManager;
```

**稳定性**: ✅ 稳定接口

---

### 1.9 Trace接口

**文件**: `common/src/main/ets/default/Trace.ts`

```typescript
class Trace {
    // 核心方法跟踪标签
    static CORE_METHOD_SLEEP_TO_LOCK_SCREEN = "sleepToLockScreen";
    static CORE_METHOD_SHOW_LOCK_SCREEN = "showLockScreen";
    
    /**
     * 开始跟踪
     */
    static begin(name: string): void;
    
    /**
     * 结束跟踪
     */
    static end(name: string): void;
}

export { Trace };
```

**使用示例**:
```typescript
import { Trace } from '@ohos/common';

Trace.begin(Trace.CORE_METHOD_SHOW_LOCK_SCREEN);
// ... 执行业务逻辑
Trace.end(Trace.CORE_METHOD_SHOW_LOCK_SCREEN);
```

**稳定性**: ✅ 稳定接口

---

## 2. ScreenLock模块接口

### 2.1 ScreenLockService接口

**文件**: `features/screenlock/src/main/ets/com/ohos/model/screenLockService.ts`

```typescript
class ScreenLockService {
    /**
     * 初始化服务
     */
    init(): void;
    
    /**
     * 锁屏
     */
    lockScreen(): void;
    
    /**
     * 解锁（发起解锁流程）
     */
    unlockScreen(): void;
    
    /**
     * 解锁中（验证通过后调用）
     */
    unlocking(): void;
    
    /**
     * 用户认证
     */
    authUser(password?: string): Promise<AuthResult>;
    
    /**
     * 获取锁屏样式模式
     */
    getLockStyleMode(): LockStyleMode;
    
    /**
     * 设置锁屏样式模式
     */
    setLockStyleMode(mode: LockStyleMode): void;
    
    /**
     * 获取认证属性
     */
    checkPinAuthProperty(): AuthProperty;
    
    /**
     * 注册锁屏结果回调
     */
    onLockScreenResult(callback: LockScreenResultCallback): void;
    
    /**
     * 注册解锁结果回调
     */
    onUnlockScreenResult(callback: UnlockScreenResultCallback): void;
}

// 锁屏样式模式
enum LockStyleMode {
    SlideScreenLock,   // 滑动解锁
    JournalScreenLock, // 日志锁屏
    CustomScreenLock   // 自定义锁屏（密码/图案）
}
```

**稳定性**: ✅ 稳定接口

---

### 2.2 ScreenLockModel接口

**文件**: `features/screenlock/src/main/ets/com/ohos/model/screenLockModel.ts`

```typescript
class ScreenLockModel {
    /**
     * 监听系统事件
     */
    eventListener(callback: EventCallback): void;
    
    /**
     * 发送锁屏事件
     */
    sendScreenLockEvent(
        typeName: string, 
        typeNo: number, 
        callback: SendEventCallback
    ): void;
    
    /**
     * 显示锁屏窗口
     */
    showScreenLockWindow(callback: Callback<void>): void;
    
    /**
     * 隐藏锁屏窗口
     */
    hiddenScreenLockWindow(callback: Callback<void>): void;
    
    /**
     * 系统是否就绪
     */
    isSystemReady(): boolean;
}
```

**稳定性**: ✅ 稳定接口

---

### 2.3 AccountsModel接口

**文件**: `features/screenlock/src/main/ets/com/ohos/model/accountsModel.ts`

```typescript
// 认证类型
enum AuthType {
    PIN = 1,
    FACE = 2
}

// 认证子类型
enum AuthSubType {
    PIN_SIX = 10000,
    PIN_MIXED = 10001,
    PIN_NUMBER = 10002,
    FACE_2D = 20000,
    FACE_3D = 20001
}

class AccountsModel {
    /**
     * 初始化
     */
    init(context: any): void;
    
    /**
     * 获取当前用户ID
     */
    getCurrentUserId(): number;
    
    /**
     * 设置当前用户
     */
    setCurrentUser(userId: number): void;
    
    /**
     * 用户认证
     */
    authUser(
        challenge: Uint8Array,
        authType: AuthType,
        authLevel: number,
        callback: AuthCallback
    ): void;
    
    /**
     * 获取认证属性
     */
    getAuthProperty(request: AuthPropertyRequest): Promise<AuthProperty>;
    
    /**
     * 注册密码输入器（用于PIN认证）
     */
    private registerInputer(password: string): boolean;
    
    /**
     * 获取认证类型
     */
    getAuthType(): AuthType;
    
    /**
     * 获取认证子类型
     */
    getAuthSubType(): AuthSubType;
}
```

**稳定性**: ✅ 稳定接口

---

## 3. NoticeItem模块接口

### 3.1 NotificationManager接口

**文件**: `features/noticeitem/src/main/ets/com/ohos/noticeItem/model/NotificationManager.ts`

```typescript
class NotificationManager {
    /**
     * 订阅通知
     */
    static subscribe(
        tag: string, 
        subscriber: NotificationSubscriber, 
        callback: AsyncCallback<void>
    ): void;
    
    /**
     * 取消订阅
     */
    static unsubscribe(
        tag: string, 
        subscriber: NotificationSubscriber, 
        callback: AsyncCallback<void>
    ): void;
    
    /**
     * 获取所有活动通知
     */
    static getAllActiveNotifications(
        tag: string, 
        callback: AsyncCallback<NotificationRequest[]>
    ): void;
    
    /**
     * 移除通知
     */
    static remove(
        tag: string,
        hashCode: string,
        callback: AsyncCallback<void>,
        isClickItem?: boolean
    ): void;
    
    /**
     * 移除所有通知
     */
    static removeAll(tag: string, callback: AsyncCallback<void>): void;
}
```

**稳定性**: ✅ 稳定接口

---

## 4. 组件状态接口

### 4.1 ViewModel基类

**文件**: `features/screenlock/src/main/ets/com/ohos/vm/baseViewModel.ts`

```typescript
class BaseViewModel {
    /**
     * 显示密码错误提示
     */
    showErrorTip(message: string): void;
    
    /**
     * 获取认证剩余重试次数
     */
    getRemainingRetries(): number;
    
    /**
     * 开始冻结倒计时
     */
    startFreezeCountdown(seconds: number): void;
    
    /**
     * 获取认证属性
     */
    getAuthProperty(): AuthProperty;
}
```

---

## 5. 模块依赖关系

### 5.1 依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                        Product层                             │
│  ┌───────────────┐        ┌───────────────┐                 │
│  │     phone     │◄──────►│      pc       │                 │
│  └───────┬───────┘        └───────┬───────┘                 │
└──────────┼────────────────────────┼──────────────────────────┘
           │                        │
           │    ┌───────────────────┘
           │    │
           ▼    ▼
┌─────────────────────────────────────────────────────────────┐
│                       Features层                             │
│  ┌───────────┐ ┌───────────┐ ┌───────────────────────────┐  │
│  │screenlock │ │noticeitem │ │  datetime/wallpaper/...   │  │
│  └─────┬─────┘ └─────┬─────┘ └───────────┬───────────────┘  │
└────────┼─────────────┼───────────────────┼──────────────────┘
         │             │                   │
         └─────────────┼───────────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                        Common层                              │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌──────────────┐ │
│  │EventBus   │ │WindowMgr  │ │AbilityMgr │ │ Log/Time/... │ │
│  └───────────┘ └───────────┘ └───────────┘ └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 导入关系

| 模块 | 导入Common | 被谁导入 |
|------|------------|----------|
| common | - | entry, phone, pc, features/* |
| entry | ✅ | - |
| phone | ✅ | - |
| screenlock | ✅ | phone, pc |
| noticeitem | ✅ | phone, pc |
| 其他features | ✅ | phone, pc |

---

## 6. 接口稳定性说明

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| EventBus | ✅ 稳定 | 核心事件机制 |
| EventManager | ✅ 稳定 | 全局事件管理 |
| WindowManager | ✅ 稳定 | 窗口管理核心 |
| AbilityManager | ✅ 稳定 | Ability管理核心 |
| Log | ✅ 稳定 | 日志工具 |
| ScreenLockService | ⚠️ 可能变更 | 业务逻辑可能调整 |
| ScreenLockModel | ⚠️ 可能变更 | 与系统服务绑定 |
| AccountsModel | ⚠️ 可能变更 | 认证逻辑可能调整 |

---

*关键结论: Common模块提供稳定的基础接口（EventBus、WindowManager、AbilityManager等），Features模块提供业务接口。建议优先使用Common模块的接口。*
