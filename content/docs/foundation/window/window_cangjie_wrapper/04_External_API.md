# 对外 API（仓颉接口）

> 仓颉 API 清单、参数、返回值、错误码、权限说明

---

## 目的

本文档列出 `window_cangjie_wrapper` 提供的所有对外仓颉 API，包括方法签名、参数、返回值、对应的 FFI 函数、错误码和权限要求。

---

## API 清单总览

| 模块 | 类/函数数 | 枚举类型数 | 错误码数 |
|--------|------------|----------|---------|
| **ohos.window** | 37 | 8 | 11 |
| **ohos.display** | 11 | 5 | 4 |

**证据**：`ohos/window/` 和 `ohos/display/` 目录统计

---

## ohos.window 模块 API

### 窗口创建与查找

#### findWindow(name: String): Window

**功能**：根据名称查找窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:161-171` |
| **对应 FFI** | `FFiOHOSWindowFindWindow(name: CString): RetDataI64` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:161-171`

---

#### createWindow(config: Configuration): Window

**功能**：创建窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:190-209` |
| **对应 FFI** | `FfiOHOSCreateWindow(name, windowType, ctx, displayID, parentID): RetDataI64` |
| **参数校验** | - name 非空<br>- ctx 必须是 UIAbilityContext（通过 FFiGetContext 检查） |
| **错误码** | - 201 - 权限验证失败（TYPE_FLOAT 需要 SYSTEM_FLOAT_WINDOW）<br>- 401 - 参数错误<br>- 1300003 - 窗口管理服务异常<br>- 1300006 - 窗口上下文异常 |
| **权限** | `ohos.permission.SYSTEM_FLOAT_WINDOW`（仅 windowType == TypeFloat 时） |
| **同步/异步** | 同步 |

**Configuration 参数结构**：
```cangjie
public class Configuration {
    public var name: String          // 窗口名称
    public var windowType: WindowType // 窗口类型
    public var ctx: BaseContext       // 窗口上下文
    public var displayId: Int64 = -1  // 显示 ID（可选）
    public var parentId: Int64 = -1  // 父窗口 ID（可选）
}
```

**证据**：`cj_window_common.cj:195-263`、`window.cj:195-196`

---

#### getLastWindow(ctx: BaseContext): Window

**功能**：获取应用的顶层窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:224-236` |
| **对应 FFI** | `FfiOHOSGetLastWindow(ctx: StageContext): RetDataI64` |
| **参数校验** | ctx 必须是 UIAbilityContext（通过 FFiGetContext 检查） |
| **错误码** | - 1300006 - 窗口上下文异常（仅 UIAbilityContext 支持）<br>- 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:224-236`

---

#### shiftAppWindowFocus(sourceWindowID: Int32, targetWindowID: Int32): Unit

**功能**：在同一应用内转移窗口焦点

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:255-260` |
| **对应 FFI** | `FfiOHOSShiftAppWindowFocus(sourceWindowID, targetWindowID): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | - 401 - 参数错误<br>- 801 - 设备能力不支持<br>- 1300002 - 窗口状态异常<br>- 1300003 - 窗口管理服务异常<br>- 1300004 - 未授权操作 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**Syscap**：`SystemCapability.Window.SessionManager`

**证据**：`window.cj:255-260`

---

### Window 类方法

#### showWindow(): Unit

**功能**：显示窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:310-315` |
| **对应 FFI** | `FfiOHOSWindowShowWindow(id: Int64): Int32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:310-315`

---

#### destroyWindow(): Unit

**功能**：销毁窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:382-387` |
| **对应 FFI** | `FfiOHOSWindowDestroyWindow(id: Int64): Int32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:382-387`

---

#### moveWindowTo(x: Int32, y: Int32): Unit

**功能**：设置窗口位置

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:328-333` |
| **对应 FFI** | `FfiOHOSWindowMoveWindowTo(id, x, y): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:328-333`

---

#### resize(width: UInt32, height: UInt32): Unit

**功能**：设置窗口大小

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:346-351` |
| **对应 FFI** | `FfiOHOSWindowResize(id, width, height): Int32` |
| **参数校验** | width 和 height 应大于 0（文档说明，代码未强制校验） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:346-351`

---

#### setWindowBackgroundColor(color: String): Unit

**功能**：设置窗口背景色

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:456-463` |
| **对应 FFI** | `FfiOHOSSetWindowBackgroundColor(id, color: CString): Int32` |
| **参数校验** | 使用 try-resource 自动释放 C 字符串 |
| **错误码** | - 401 - 参数错误<br>- 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:456-463`

---

#### setWindowBrightness(brightness: Float32): Unit

**功能**：设置窗口亮度

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:475-480` |
| **对应 FFI** | `FfiOHOSWindowSetWindowBrightness(id, brightness): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:475-480`

---

#### setWindowFocusable(isFocusable: Bool): Unit

**功能**：设置窗口是否可获取焦点

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:492-497` |
| **对应 FFI** | `FfiOHOSWindowSetWindowFocusable(id, focusable): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:492-497`

---

#### setWindowKeepScreenOn(isKeepScreenOn: Bool): Unit

**功能**：设置是否保持屏幕常亮

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:509-514` |
| **对应 FFI** | `FfiOHOSWindowSetWindowKeepScreenOn(id, keepScreenOn): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:509-514`

---

#### setWindowPrivacyMode(isPrivacyMode: Bool): Unit

**功能**：设置隐私窗口模式

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:527-532` |
| **对应 FFI** | `FfiOHOSWindowSetWindowPrivacyMode(id, isPrivacyMode): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | `ohos.permission.PRIVACY_WINDOW` |
| **同步/异步** | 同步 |

**证据**：`window.cj:527`（包含 `@!APILevel[permission: "ohos.permission.PRIVACY_WINDOW"]`）

---

#### setWindowTouchable(isTouchable: Bool): Unit

**功能**：设置窗口是否可触摸

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:544-549` |
| **对应 FFI** | `FfiOHOSWindowSetWindowTouchable(id, touchable): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:544-549`

---

#### setPreferredOrientation(orientation: Orientation): Unit

**功能**：设置窗口首选方向

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:565-568` |
| **对应 FFI** | `FFiOHOSWindowSetPreferredOrientation(id, orientation): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | - 401 - 参数错误<br>- 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:565-568`

---

#### getWindowAvoidArea(areaType: AvoidAreaType): AvoidArea

**功能**：获取避让区域

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:584-592` |
| **对应 FFI** | `FFiOHOSWindowGetWindowAvoidArea(id, areaType, retPtr): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | - 401 - 参数错误<br>- 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:584-592`

---

#### setAspectRatio(ratio: Float64): Unit

**功能**：设置窗口宽高比

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:605-608` |
| **对应 FFI** | `FFiOHOSWindowSetAspectRatio(id, ratio): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | - 1300002 - 窗口状态异常<br>- 1300004 - 未授权操作 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:605-608`

---

#### resetAspectRatio(): Unit

**功能**：重置窗口宽高比

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:620-623` |
| **对应 FFI** | `FFiOHOSWindowResetAspectRatio(id): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | - 1300002 - 窗口状态异常<br>- 1300004 - 未授权操作 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:620-623`

---

#### minimize(): Unit

**功能**：最小化窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:653-658` |
| **对应 FFI** | `FFiOHOSWindowMinimize(id): Int32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | - 801 - 设备能力不支持<br>- 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**Syscap**：`SystemCapability.Window.SessionManager`

**证据**：`window.cj:653-658`

---

#### setWindowColorSpace(colorSpace: ColorSpace): Unit

**功能**：设置窗口色彩空间

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:635-640` |
| **对应 FFI** | `FfiOHOSWindowSetWindowColorSpace(id, colorSpace): Int32` |
| **参数校验** | 无（直接传递给 FFI） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:635-640`

---

#### getWindowColorSpace(): ColorSpace

**功能**：获取窗口色彩空间

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:670-677` |
| **对应 FFI** | `FfiOHOSWindowGetWindowColorSpace(id, errCode): Int32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:670-677`

---

#### snapshot(): PixelMap

**功能**：获取窗口截图

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:689-696` |
| **对应 FFI** | `FFiOHOSWindowSnapshot(id, errCode): Int64` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:689-696`

---

#### setWindowSystemBarEnabled(names: Array<SystemBarType>): Unit

**功能**：设置系统栏显示

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:708-719` |
| **对应 FFI** | `FFiOHOSWindowSetWindowSystemBarEnable(id, arr): Int32` |
| **参数校验** | 使用 toArrayCString 转换数组，调用后 freeArrCString |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:708-719`

---

#### setWindowSystemBarProperties(systemBarProperties: SystemBarProperties): Unit

**功能**：设置系统栏属性

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:731-745` |
| **对应 FFI** | `FFiOHOSWindowSetWindowSystemBarProperties(id, properties): Int32` |
| **参数校验** | 使用 try-resource 自动释放 6 个 C 字符串 |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window.cj:731-745`

---

### 回调机制

#### on(callbackType: WindowCallbackType, callback: Callback1Argument<UInt32>): Unit

**功能**：注册窗口回调

| 项目 | 说明 |
|------|--------|
| **位置** | `window.cj:769-801` |
| **对应 FFI** | `FfiOHOSOnCallback(id, callbackId, callbackType): Int32` |
| **参数校验** | - callbackType 必须是 KeyboardHeightChange（当前实现限制）<br>- callback 不能重复注册 |
| **错误码** | 1300016 - 参数校验错误 |
| **权限** | 无 |
| **同步/异步** | 同步注册，异步触发 |

**回调类型**（`cj_window_enum.cj:924-1051`）：
- WindowStageEvent
- WindowSizeChange
- WindowAvoidAreaChange
- KeyboardHeightChange
- TouchOutside
- WindowVisibilityChange
- Screenshot
- DialogTargetTouch
- WindowEvent
- WindowStatusChange
- WindowTitleButtonRectChange
- WindowRectChange
- SubWindowClose
- NoInteractionDetected

**证据**：`window.cj:769-801`

---

### WindowStage 类方法

#### getMainWindow(): Window

**功能**：获取主窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window_stage.cj:86-92` |
| **对应 FFI** | `FfiOHOSGetMainWindow(id): RetDataI64` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window_stage.cj:86-92`

---

#### createSubWindow(name: String): Window

**功能**：创建子窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window_stage.cj:105-114` |
| **对应 FFI** | `FfiOHOSCreateSubWindow(id, name): RetDataI64` |
| **参数校验** | 使用 try-resource 自动释放 C 字符串 |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window_stage.cj:105-114`

---

#### getSubWindow(): Array<Window>

**功能**：获取所有子窗口

| 项目 | 说明 |
|------|--------|
| **位置** | `window_stage.cj:126-137` |
| **对应 FFI** | `FfiOHOSGetSubWindow(id): RetStruct` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 1300002 - 窗口状态异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window_stage.cj:126-137`

---

#### loadContent(path: String): Unit

**功能**：加载页面内容

| 项目 | 说明 |
|------|--------|
| **位置** | `window_stage.cj:149-155` |
| **对应 FFI** | `FfiOHOSLoadContent(id, path): Int32` |
| **参数校验** | 使用 try-resource 自动释放 C 字符串 |
| **错误码** | 无（未检查返回值） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`window_stage.cj:149-155`

---

## ohos.display 模块 API

### Display 查询

#### getDefaultDisplaySync(): Display

**功能**：获取默认显示设备

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:130-142` |
| **对应 FFI** | `FfiOHOSGetDefaultDisplaySync(): RetStruct` |
| **参数校验** | 无 |
| **错误码** | 1400001 - 无效的显示或屏幕<br>- 1400003 - 显示管理服务异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:130-142`

---

#### getAllDisplays(): Array<Display>

**功能**：获取所有显示设备

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:155-169` |
| **对应 FFI** | `FfiOHOSGetAllDisplays(): RetStruct` |
| **参数校验** | 无 |
| **错误码** | 1400001 - 无效的显示或屏幕<br>- 1400003 - 显示管理服务异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:155-169`

---

### Display 类属性

#### id: UInt32

**功能**：显示设备 ID

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:496-500` |
| **对应 FFI** | `FfiOHOSDisplayGetId(id): UInt32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:496-500`

---

#### name: String

**功能**：显示设备名称

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:509-516` |
| **对应 FFI** | `FfiOHOSGetDisplayName(id): CString` |
| **参数校验** | 使用 try-resource 自动释放 C 字符串 |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:509-516`

---

#### alive: Bool

**功能**：显示设备是否存活

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:526-530` |
| **对应 FFI** | `FfiOHOSDisplayGetAlive(id): Bool` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:526-530`

---

#### state: DisplayState

**功能**：显示设备状态

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:539-545` |
| **对应 FFI** | `FfiOHOSDisplayGetState(id): UInt32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:539-545`

---

#### refreshRate: UInt32

**功能**：显示设备刷新率

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:554-558` |
| **对应 FFI** | `FfiOHOSDisplayGetRefreshRate(id): UInt32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:554-558`

---

#### width: Int32

**功能**：显示设备宽度（像素）

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:599-603` |
| **对应 FFI** | `FfiOHOSDisplayGetWidth(id): Int32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:599-603`

---

#### height: Int32

**功能**：显示设备高度（像素）

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:612-616` |
| **对应 FFI** | `FfiOHOSDisplayGetHeight(id): Int32` |
| **参数校验** | 无（使用 getID()） |
| **错误码** | 无（属性 getter） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:612-616`

---

### 折叠屏管理

#### isFoldable(): Bool

**功能**：检测设备是否支持折叠屏

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:180-182` |
| **对应 FFI** | `FfiOHOSIsFoldable(): Bool` |
| **参数校验** | 无 |
| **错误码** | 无 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**Syscap**：`SystemCapability.Window.SessionManager`

**证据**：`display.cj:180-182`

---

#### getFoldStatus(): FoldStatus

**功能**：获取折叠屏状态

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:193-198` |
| **对应 FFI** | `FfiOHOSGetFoldStatus(): UInt32` |
| **参数校验** | 无 |
| **错误码** | 无 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**Syscap**：`SystemCapability.Window.SessionManager`

**证据**：`display.cj:193-198`

---

#### getFoldDisplayMode(): FoldDisplayMode

**功能**：获取折叠显示模式

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:209-214` |
| **对应 FFI** | `FfiOHOSGetFoldDisplayMode(): UInt32` |
| **参数校验** | 无 |
| **错误码** | 无 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**Syscap**：`SystemCapability.Window.SessionManager`

**证据**：`display.cj:209-214`

---

#### getCurrentFoldCreaseRegion(): FoldCreaseRegion

**功能**：获取当前折叠缝区域

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:227-247` |
| **对应 FFI** | `FfiOHOSGetCurrentFoldCreaseRegion(): RetStruct` |
| **参数校验** | 无 |
| **错误码** | 1400003 - 显示管理服务异常<br>- 100001 - 内部错误（无法创建目标指针类型） |
| **权限** | 无 |
| **同步/异步** | 同步 |

**Syscap**：`SystemCapability.Window.SessionManager`

**证据**：`display.cj:227-247`

---

### Display 回调

#### on(listenerType: ListenerType, callback: Callback1Argument<FoldStatus>): Unit

**功能**：注册显示设备事件回调

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:295-301` |
| **对应 FFI** | `FfiOHOSRegisterDisplayManagerCallback(type, callbackId): Int32` |
| **参数校验** | - listenerType 必须是 FoldStatusChange 或 FoldDisplayModeChange<br>- callback 不能重复注册 |
| **错误码** | 401 - 参数错误<br>- 1400003 - 显示管理服务异常 |
| **权限** | 无 |
| **同步/异步** | 同步注册，异步触发 |

**回调类型**（`cj_display_enum.cj:377-464`）：
- ListenerTypeAdd
- ListenerTypeRemove
- ListenerTypeChange
- ListenerTypeFoldStatusChange
- ListenerTypeFoldAngleChange
- ListenerTypeCaptureStatusChange
- ListenerTypeFoldDisplayModeChange
- ListenerTypeAvailableAreaChange

**证据**：`display.cj:295-301`

---

#### off(listenerType: ListenerType): Unit

**功能**：注销显示设备事件回调

| 项目 | 说明 |
|------|--------|
| **位置** | `display.cj:258-260` |
| **对应 FFI** | `FfiOHOSUnRegisterAllDisplayManagerCallback(type): Int32` |
| **参数校验** | 无 |
| **错误码** | 1400003 - 显示管理服务异常 |
| **权限** | 无 |
| **同步/异步** | 同步 |

**证据**：`display.cj:258-260`

---

## 错误码映射表

### ohos.window 错误码

| 错误码 | 含义 | 使用场景 | 证据 |
|--------|--------|----------|--------|
| 1300001 | 重复操作 | 多次创建同名窗口 | `cj_window_utils.cj:25` |
| 1300002 | 窗口状态异常 | 窗口已销毁或不存在 | `cj_window_utils.cj:26` |
| 1300003 | 窗口管理服务异常 | Native 层服务异常 | `cj_window_utils.cj:27` |
| 1300004 | 未授权操作 | 无权限执行敏感操作 | `cj_window_utils.cj:28` |
| 1300005 | 窗口阶段异常 | WindowStage 状态异常 | `cj_window_utils.cj:29` |
| 1300006 | 窗口上下文异常 | 仅 UIAbilityContext 支持 | `cj_window_utils.cj:30` |
| 1300007 | 启动能力失败 | Ability 启动失败 | `cj_window_utils.cj:31` |
| 1300008 | 无效的显示操作 | 在无效的显示上执行操作 | `cj_window_utils.cj:32` |
| 1300009 | 父窗口无效 | 父窗口不存在或类型不匹配 | `cj_window_utils.cj:33` |
| 1300010 | 全屏模式下不支持的操作 | 某些操作在全屏模式下不支持 | `cj_window_utils.cj:34` |
| 1300016 | 参数校验错误 | 参数类型不匹配或无效值 | `window.cj:773,819,837` |

### ohos.display 错误码

| 错误码 | 含义 | 使用场景 |
|--------|--------|----------|
| 1400001 | 无效的显示或屏幕 | Display ID 无效或不存在 | `display.cj:133-136` |
| 1400003 | 显示管理服务异常 | Native 层服务异常 | `display.cj:158-161,273-276` |

### 通用错误码

| 错误码 | 含义 | 使用场景 | 证据 |
|--------|--------|----------|--------|
| 201 | 权限验证失败 | 系统权限检查失败 | `cj_window_utils.cj:35` |
| 202 | 权限验证失败 | 非系统应用使用系统 API | `cj_window_utils.cj:36` |
| 401 | 参数错误 | 必需参数缺失或类型错误 | `cj_window_utils.cj:37` |
| 801 | 设备能力不支持 | 设备不支持该功能 | `cj_window_utils.cj:38` |
| 100001 | 内部错误 | 无法创建目标指针类型 | `display.cj:236,715` |

**证据**：`cj_window_utils.cj:23-40`、`window.cj`、`display.cj`

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| ohos.window 模块提供 37 个公共 API，覆盖窗口创建、属性设置、回调管理 | 代码行数统计 |
| ohos.display 模块提供 11 个公共 API，覆盖 Display 查询、折叠屏管理、回调监听 | 代码行数统计 |
| 所有 API 通过 FFI 接口调用 Native 层实现 | FFI 函数定义 |
| 错误处理统一使用 checkRet 函数，支持 14 种错误码 | `cj_window_utils.cj:42-51` |
| 权限检查通过 Native 层进行，仓颉层仅传递注解 | `@!APILevel[permission: "..."]` |

---

**生成时间**: 2025-02-06
