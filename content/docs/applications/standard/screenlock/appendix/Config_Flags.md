# 附录B: 配置标志

> 目的: 汇总项目中的编译期配置和功能开关  
> 适用范围: 开发者

---

## 1. 编译期配置

### 1.1 SDK版本配置

**文件**: `build-profile.json5`

```json5
{
  "app": {
    "products": [{
      "compileSdkVersion": 23,      // 编译SDK版本
      "compatibleSdkVersion": 23    // 兼容SDK版本
    }]
  }
}
```

### 1.2 设备类型配置

**文件**: `entry/src/main/module.json5`

```json5
{
  "module": {
    "deviceTypes": [
      "phone",    // 支持手机
      "tablet"    // 支持平板
    ]
  }
}
```

**文件**: `product/phone/src/main/module.json5`

```json5
{
  "module": {
    "deviceTypes": ["phone"]    // 仅手机
  }
}
```

---

## 2. 功能开关

### 2.1 调试开关

**文件**: `common/src/main/ets/default/Log.ts`

```typescript
// 日志级别控制
const LOG_LEVEL_DEBUG = 3;
const LOG_LEVEL_INFO = 4;
const LOG_LEVEL_WARN = 5;
const LOG_LEVEL_ERROR = 6;
const LOG_LEVEL_FATAL = 7;

// 当前日志级别（可通过修改此值控制日志输出）
const CURRENT_LOG_LEVEL = LOG_LEVEL_DEBUG;
```

### 2.2 性能跟踪开关

**文件**: `common/src/main/ets/default/Trace.ts`

```typescript
export class Trace {
    // 核心方法跟踪标签
    static CORE_METHOD_SLEEP_TO_LOCK_SCREEN = "sleepToLockScreen";
    static CORE_METHOD_SHOW_LOCK_SCREEN = "showLockScreen";
    
    // 可通过条件编译控制是否启用跟踪
    // #ifdef ENABLE_TRACE
    static begin(name: string): void {
        hiTraceMeter.startTrace(name, 0);
    }
    // #endif
}
```

### 2.3 功能特性开关

**文件**: `features/screenlock/src/main/ets/com/ohos/common/constants.ts`

```typescript
export default class Constants {
    // 窗口名称
    static WIN_NAME = "ScreenLockWindow"
    
    // 数字键盘配置
    static DIGITALPSD_IC_DIAMETER = 12        // 密码圆点直径
    static DIGITALPSD_BUTTON_DIAMETER = 60    // 键盘按钮直径
    
    // 刷新间隔（毫秒）
    static INTERVAL = 1000    // 1秒刷新
    
    // 最大密码长度
    static PASSWORD_MAX_LEN = 32
}
```

---

## 3. UI配置

### 3.1 尺寸配置

**文件**: `features/screenlock/src/main/ets/com/ohos/common/constants.ts`

```typescript
export default class Constants {
    // 快捷操作尺寸
    static SHORTCUT_CIRCLE_WIDTH = '80px'
    static SHORTCUT_CIRCLE_HEIGHT = '80px'
    static SHORTCUT_TEXT_SIZE = '24px'
    static SHORTCUT_HEIGHT = '150px'
    
    // 布局尺寸
    static FULL_CONTAINER_WIDTH = '100%'
    static FULL_CONTAINER_HEIGHT = '100%'
    static HALF_CONTAINER_WIDTH = '50%'
    
    // 密码输入框尺寸
    static PASSWORD_TEXT_WIDTH ='290px'
    static PASSWORD_TEXT_HEIGHT ='40px'
    static PASSWORD_TEXT_BORDER = 20
    
    // 间距
    static ACCOUNT_SPACE = '24px'
    static ACCOUNT_SPACE_PORTRAIT = '40px'
}
```

### 3.2 颜色配置

**文件**: `features/screenlock/src/main/ets/com/ohos/common/constants.ts`

```typescript
export class StatusBarGroupComponentData {
    backgroundColor: string = "#00000000";    // 透明背景
    contentColor: string = "#FFFFFFFF";        // 白色内容
}
```

---

## 4. 事件常量

### 4.1 核心事件

**文件**: `common/src/main/ets/default/ScreenLockManager.ts`

```typescript
export const SCREEN_CHANGE_EVENT = "screenChangeEvent";
```

**文件**: `common/src/main/ets/default/WindowManager.ts`

```typescript
export const WINDOW_SHOW_HIDE_EVENT = "WindowShowHideEvent";
export const WINDOW_RESIZE_EVENT = "WindowResizeEvent";
```

### 4.2 系统事件

**文件**: `common/src/main/ets/default/ScreenLockManager.ts`

```typescript
const SCREEN_COMMON_EVENT_INFO = {
  events: [
    commonEvent.Support.COMMON_EVENT_SCREEN_OFF,
    commonEvent.Support.COMMON_EVENT_SCREEN_ON
  ],
};
```

---

## 5. 窗口类型配置

### 5.1 窗口类型映射

**文件**: `common/src/main/ets/default/WindowManager.ts`

```typescript
export enum WindowType {
  STATUS_BAR = "SystemUi_StatusBar",           
  NAVIGATION_BAR = "SystemUi_NavigationBar",   
  DROPDOWN_PANEL = "SystemUi_DropdownPanel",   
  NOTIFICATION_PANEL = "SystemUi_NotificationPanel", 
  CONTROL_PANEL = "SystemUi_ControlPanel",     
  VOLUME_PANEL = "SystemUi_VolumePanel",       
  BANNER_NOTICE = 'SystemUi_BannerNotice'      
}

// 窗口类型到系统类型的映射
const SYSTEM_WINDOW_TYPE_MAP: { [key in WindowType]: number } = {
  SystemUi_StatusBar: 2108,
  SystemUi_NavigationBar: 2112,
  SystemUi_DropdownPanel: 2109,
  SystemUi_NotificationPanel: 2111,
  SystemUi_ControlPanel: 2111,
  SystemUi_VolumePanel: 2111,
  SystemUi_BannerNotice: 2111,
};
```

---

## 6. 认证类型配置

### 6.1 认证类型枚举

**文件**: `features/screenlock/src/main/ets/com/ohos/model/accountsModel.ts`

```typescript
enum AuthType {
    PIN = 1,     // PIN码认证
    FACE = 2     // 人脸认证
}

enum AuthSubType {
    PIN_SIX = 10000,      // 6位数字密码
    PIN_MIXED = 10001,    // 混合密码
    PIN_NUMBER = 10002,   // 自定义数字密码
    FACE_2D = 20000,      // 2D人脸识别
    FACE_3D = 20001       // 3D人脸识别
}
```

---

## 7. 锁屏样式配置

### 7.1 锁屏模式

**文件**: `features/screenlock/src/main/ets/com/ohos/model/screenlockStyle.ts`

```typescript
export enum LockStyleMode {
    SlideScreenLock,    // 滑动解锁
    JournalScreenLock,  // 日志锁屏
    CustomScreenLock    // 自定义锁屏（密码/图案）
}
```

---

## 8. 页面状态常量

**文件**: `product/phone/src/main/ets/common/constants.ts`

```typescript
export default class Constants {
    // 页面状态
    static STATUS_ABOUT_TO_APPEAR = 0
    static STATUS_ABOUT_TO_DISAPPEAR = 1
    static STATUS_ON_PAGE_SHOW = 2
    static STATUS_ON_PAGE_HIDE = 3
    
    // 数字键盘特殊按键
    static CALL_PHONE = -1      // 紧急呼叫
    static DEL_PWD = -2         // 删除密码
    static GO_BACK = -3         // 返回
}
```

---

## 9. 能力名称常量

**文件**: `common/src/main/ets/default/abilitymanager/abilityManager.ts`

```typescript
class AbilityManager {
    static ABILITY_NAME_SCREEN_LOCK = "screenLockAbility";
    static ABILITY_NAME_STATUS_BAR = "statusBarAbility";
    static ABILITY_NAME_NAVIGATION_BAR = "navigationBarAbility";
    static ABILITY_NAME_NOTIFICATION_PANEL = "notificationPanelAbility";
    static ABILITY_NAME_CONTROL_PANEL = "controlPanelAbility";
    static ABILITY_NAME_VOLUME_PANEL = "volumePanelAbility";
    static ABILITY_NAME_BANNER_NOTICE = "bannerNoticeAbility";
}
```

---

## 10. 配置修改指南

### 10.1 修改窗口尺寸

```typescript
// WindowManager.ts
const DEFAULT_WINDOW_INFO: WindowInfo = {
  visibility: false,
  rect: { left: 0, top: 0, width: 0, height: 0 },
};

// 修改状态栏高度计算
// ServiceExtAbility.ts
if (dis.width > dis.height) { // Pad、PC horizontalScreen Mode
    rect = {
        left: 0,
        top: 0,
        width: '100%',
        height: (48 * dis.width) / 1280,  // 修改此值调整高度
    }
}
```

### 10.2 修改日志级别

```typescript
// Log.ts
// 生产环境建议提高日志级别
const CURRENT_LOG_LEVEL = LOG_LEVEL_WARN;  // 只输出警告及以上
```

### 10.3 修改刷新间隔

```typescript
// constants.ts
static INTERVAL = 5000    // 修改为5秒刷新
```

---

*说明: 修改配置后需要重新编译才能生效。部分配置可能需要重启应用才能生效。*
