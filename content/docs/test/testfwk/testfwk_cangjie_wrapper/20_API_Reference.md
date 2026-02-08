# API 参考手册

**文档目的**: 提供完整的 Cangjie API 参考，包括方法签名、参数说明、错误码和调用链  
**目标读者**: 使用本框架编写测试的开发者  
**阅读时间**: 约 30 分钟

---

## 1. API 总览

### 1.1 类层次

```
Object
├── Driver          # 核心驱动类（单例模式）
├── On              # 组件选择器（Builder 模式）
├── Component       # UI 组件代理
├── UiWindow        # 窗口代理
├── UiEventObserver # 事件观察者
├── PointerMatrix   # 多指手势定义
├── Point           # 坐标点
├── Rect            # 矩形区域
├── WindowFilter    # 窗口过滤器
└── UiElementInfo   # 元素信息
```

### 1.2 枚举类型

| 枚举 | 用途 | 值 |
|------|------|-----|
| `MatchPattern` | 字符串匹配模式 | Equals, Contains, StartsWith, EndsWith |
| `DisplayRotation` | 显示旋转角度 | Rotation0, Rotation90, Rotation180, Rotation270 |
| `WindowMode` | 窗口模式 | Fullscreen, Primary, Secondary, Floating |
| `ResizeDirection` | 调整大小方向 | Left, Right, Up, Down, LeftUp, LeftDown, RightUp, RightDown |
| `UiDirection` | UI 方向 | Left, Right, Up, Down |
| `MouseButton` | 鼠标按钮 | MouseButtonLeft, MouseButtonRight, MouseButtonMiddle |
| `OnceType` | 事件类型 | ToastShow, DialogShow |

---

## 2. Driver 类

**位置**: `ohos/ui_test/ui_test_api.cj:115`  
**说明**: UI 测试核心入口，提供全局测试环境管理能力

### 2.1 创建与基础

#### create()
创建 Driver 实例。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Test.UiTest",
    throwexception: true
]
public static func create(): Driver
```

**返回**: Driver 实例

**异常**: 
- `BusinessException` 17000001 - 初始化失败

**调用链**:
```
Driver.create()
  → ApiCallParams(DRIVER_CREATE, "", "[]")
  → CJ_ApiCall(params)
  → arkxtest Driver 创建
  → 返回引用如 "Driver#123"
```

**证据**: `ui_test_api.cj:139-143`

---

#### delayMs()
延迟指定时间。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Test.UiTest",
    throwexception: true,
    workerthread: true
]
public func delayMs(duration: Int32): Unit
```

**参数**:
| 参数 | 类型 | 说明 | 约束 |
|------|------|------|------|
| duration | Int32 | 延迟时间（毫秒） | >= 0 |

**调用链**: `Driver.delayMs` → `CJ_ApiCall(DRIVER_DELAYMS)`

**证据**: `ui_test_api.cj:155-159`

---

### 2.2 组件查找

#### findComponent()
查找第一个匹配的组件。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Test.UiTest",
    throwexception: true,
    workerthread: true
]
public func findComponent(on: On): ?Component
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| on | On | 组件选择条件 |

**返回**: Component 实例，未找到返回 `None`

**调用链**:
```
Driver.findComponent(on)
  → ApiCallParams(DRIVER_FINDCOMPONENT, ref, "[\"On#xxx\"]")
  → CJ_ApiCall()
  → 返回引用或 "null"
```

**证据**: `ui_test_api.cj:172-179`

---

#### findComponents()
查找所有匹配的组件。

```cangjie
public func findComponents(on: On): ?Array<Component>
```

**返回**: Component 数组，未找到返回 `None`

**调用链**: 返回 JSON 数组，解析为 `Array<Component>`

**证据**: `ui_test_api.cj:266-281`

---

#### waitForComponent()
等待组件出现。

```cangjie
public func waitForComponent(on: On, time: Int32): ?Component
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| on | On | 组件选择条件 |
| time | Int32 | 等待超时（毫秒） |

**证据**: `ui_test_api.cj:246-253`

---

#### assertComponentExist()
断言组件存在。

```cangjie
public func assertComponentExist(on: On): Unit
```

**异常**: `BusinessException` 17000003 - 断言失败

**证据**: `ui_test_api.cj:294-298`

---

### 2.3 窗口查找

#### findWindow()
查找窗口。

```cangjie
public func findWindow(filter: WindowFilter): ?UiWindow
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| filter | WindowFilter | 窗口过滤条件（bundleName, title, focused, active, displayId） |

**证据**: `ui_test_api.cj:192-232`

---

### 2.4 按键操作

#### pressBack()
按下返回键。

```cangjie
public func pressBack(): Unit
```

**证据**: `ui_test_api.cj:307-311`

---

#### pressHome()
按下 Home 键。

```cangjie
public func pressHome(): Unit
```

**证据**: `ui_test_api.cj:573-577`

---

#### triggerKey()
触发指定按键。

```cangjie
public func triggerKey(keyCode: Int32): Unit
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| keyCode | Int32 | 按键码 |

**证据**: `ui_test_api.cj:324-328`

---

#### triggerCombineKeys()
触发组合键。

```cangjie
public func triggerCombineKeys(key0: Int32, key1: Int32, key2!: Int32 = 0): Unit
```

**证据**: `ui_test_api.cj:343-348`

---

### 2.5 触屏手势

#### click()
点击指定坐标。

```cangjie
public func click(x: Int32, y: Int32): Unit
```

**参数**:
| 参数 | 类型 | 说明 | 约束 |
|------|------|------|------|
| x | Int32 | X 坐标 | >= 0 |
| y | Int32 | Y 坐标 | >= 0 |

**证据**: `ui_test_api.cj:361-365`

---

#### doubleClick()
双击指定坐标。

```cangjie
public func doubleClick(x: Int32, y: Int32): Unit
```

**证据**: `ui_test_api.cj:378-382`

---

#### longClick()
长按指定坐标。

```cangjie
public func longClick(x: Int32, y: Int32): Unit
```

**证据**: `ui_test_api.cj:395-399`

---

#### swipe()
滑动操作。

```cangjie
public func swipe(
    startx: Int32,
    starty: Int32,
    endx: Int32,
    endy: Int32,
    speed!: Int32 = 600
): Unit
```

**参数**:
| 参数 | 类型 | 说明 | 约束 |
|------|------|------|------|
| startx, starty | Int32 | 起点坐标 | >= 0 |
| endx, endy | Int32 | 终点坐标 | >= 0 |
| speed | Int32 | 速度（像素/秒） | 200-40000，默认 600 |

**证据**: `ui_test_api.cj:415-426`

---

#### drag()
拖拽操作。

```cangjie
public func drag(
    startx: Int32,
    starty: Int32,
    endx: Int32,
    endy: Int32,
    speed!: Int32 = 600
): Unit
```

**证据**: `ui_test_api.cj:442-453`

---

#### fling()
快速滑动（抛掷）。

```cangjie
public func fling(from: Point, to: Point, stepLen: Int32, speed: Int32): Unit
public func fling(direction: UiDirection, speed: Int32): Unit
```

**证据**: `ui_test_api.cj:610-651`

---

### 2.6 鼠标操作

#### mouseClick()
鼠标点击。

```cangjie
public func mouseClick(p: Point, btnId: MouseButton, key1!: Int32 = 0, key2!: Int32 = 0): Unit
```

**证据**: `ui_test_api.cj:667-671`

---

#### mouseDoubleClick()
鼠标双击。

```cangjie
public func mouseDoubleClick(p: Point, btnId: MouseButton, key1!: Int32 = 0, key2!: Int32 = 0): Unit
```

---

#### mouseLongClick()
鼠标长按。

```cangjie
public func mouseLongClick(p: Point, btnId: MouseButton, key1!: Int32 = 0, key2!: Int32 = 0): Unit
```

---

#### mouseMoveTo()
鼠标移动到指定位置。

```cangjie
public func mouseMoveTo(p: Point): Unit
```

**证据**: `ui_test_api.cj:683-687`

---

#### mouseMoveWithTrack()
带轨迹的鼠标移动。

```cangjie
public func mouseMoveWithTrack(from: Point, to: Point, speed!: Int32 = 600): Unit
```

---

#### mouseDrag()
鼠标拖拽。

```cangjie
public func mouseDrag(from: Point, to: Point, speed!: Int32 = 600): Unit
```

---

#### mouseScroll()
鼠标滚轮滚动。

```cangjie
public func mouseScroll(
    p: Point, 
    down: Bool, 
    d: Int32, 
    key1!: Int32 = 0, 
    key2!: Int32 = 0, 
    speed!: Int32 = 20
): Unit
```

**证据**: `ui_test_api.cj:700-716`

---

### 2.7 显示操作

#### screenCap()
屏幕截图。

```cangjie
public func screenCap(savePath: String): Bool
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| savePath | String | 保存路径（应用沙箱内） |

**返回**: true 成功，false 失败

**说明**: 路径中的特殊字符会被 `eatEscape()` 转义

**证据**: `ui_test_api.cj:466-470`

---

#### screenCapture()
带区域的屏幕截图。

```cangjie
public func screenCapture(savePath: String, rect!: Rect = Rect(0, 0, 0, 0)): Bool
```

---

#### setDisplayRotation()
设置显示旋转。

```cangjie
public func setDisplayRotation(rotation: DisplayRotation): Unit
```

**证据**: `ui_test_api.cj:483-487`

---

#### getDisplayRotation()
获取显示旋转。

```cangjie
public func getDisplayRotation(): DisplayRotation
```

**证据**: `ui_test_api.cj:498-502`

---

#### setDisplayRotationEnabled()
启用/禁用旋转。

```cangjie
public func setDisplayRotationEnabled(enabled: Bool): Unit
```

---

#### getDisplaySize()
获取显示尺寸。

```cangjie
public func getDisplaySize(): Point
```

**证据**: `ui_test_api.cj:530-534`

---

#### getDisplayDensity()
获取显示密度。

```cangjie
public func getDisplayDensity(): Point
```

---

#### wakeUpDisplay()
唤醒显示。

```cangjie
public func wakeUpDisplay(): Unit
```

**证据**: `ui_test_api.cj:559-563`

---

### 2.8 输入操作

#### inputText()
输入文本。

```cangjie
public func inputText(p: Point, text: String): Unit
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| p | Point | 输入位置 |
| text | String | 输入文本 |

---

### 2.9 其他操作

#### waitForIdle()
等待 UI 空闲。

```cangjie
public func waitForIdle(idleTime: Int32, timeout: Int32): Bool
```

**证据**: `ui_test_api.cj:591-595`

---

#### injectMultiPointerAction()
注入多指操作。

```cangjie
public func injectMultiPointerAction(pointers: PointerMatrix, speed!: Int32 = 600): Bool
```

**证据**: `ui_test_api.cj:630-634`

---

#### createUiEventObserver()
创建 UI 事件观察者。

```cangjie
public func createUiEventObserver(): UiEventObserver
```

---

## 3. On 类（组件选择器）

**位置**: `ohos/ui_test/ui_test_api.cj:1187`  
**说明**: 用于构建组件选择条件，支持链式调用

### 3.1 属性匹配

#### text()
按文本匹配。

```cangjie
public func text(txt: String, pattern!: MatchPattern = MatchPattern.Equals): On
```

**参数**:
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| txt | String | - | 目标文本 |
| pattern | MatchPattern | Equals | 匹配模式 |

**证据**: `ui_test_api.cj:1229-1234`

---

#### id()
按 ID 匹配。

```cangjie
public func id(id: String): On
```

---

#### onType()
按类型匹配。

```cangjie
public func onType(tp: String): On
```

---

#### description()
按描述匹配。

```cangjie
public func description(val: String, pattern!: MatchPattern = MatchPattern.Equals): On
```

---

### 3.2 状态过滤

| 方法 | 说明 | 默认参数 |
|------|------|----------|
| `enabled(b!: Bool = true)` | 是否启用 | true |
| `focused(b!: Bool = true)` | 是否聚焦 | true |
| `selected(b!: Bool = true)` | 是否选中 | true |
| `clickable(b!: Bool = true)` | 是否可点击 | true |
| `longClickable(b!: Bool = true)` | 是否可长按 | true |
| `scrollable(b!: Bool = true)` | 是否可滚动 | true |
| `checked(b!: Bool = true)` | 是否选中 | true |
| `checkable(b!: Bool = true)` | 是否可选中 | true |

---

### 3.3 相对定位

#### isBefore()
在指定组件之前。

```cangjie
public func isBefore(on: On): On
```

---

#### isAfter()
在指定组件之后。

```cangjie
public func isAfter(on: On): On
```

---

#### within()
在指定组件内部。

```cangjie
public func within(on: On): On
```

---

#### inWindow()
在指定窗口内。

```cangjie
public func inWindow(bundleName: String): On
```

---

## 4. Component 类

**位置**: `ohos/ui_test/ui_test_api.cj:1580`  
**说明**: UI 组件代理，提供组件级操作能力

### 4.1 操作

| 方法 | 说明 |
|------|------|
| `click()` | 点击组件 |
| `doubleClick()` | 双击组件 |
| `longClick()` | 长按组件 |
| `inputText(text: String)` | 输入文本 |
| `clearText()` | 清空文本 |
| `scrollToTop(speed!: Int64 = 600)` | 滚动到顶部 |
| `scrollToBottom(speed!: Int64 = 600)` | 滚动到底部 |
| `scrollSearch(on: On): ?Component` | 滚动搜索 |
| `dragTo(target: Component)` | 拖拽到目标组件 |
| `pinchOut(scale: Float32)` | 双指外扩 |
| `pinchIn(scale: Float32)` | 双指内捏 |

### 4.2 属性获取

| 方法 | 返回类型 | 说明 |
|------|----------|------|
| `getText()` | String | 获取文本 |
| `getId()` | String | 获取 ID |
| `getType()` | String | 获取类型 |
| `getDescription()` | String | 获取描述 |
| `getBounds()` | Rect | 获取边界 |
| `getBoundsCenter()` | Point | 获取中心点 |
| `isClickable()` | Bool | 是否可点击 |
| `isLongClickable()` | Bool | 是否可长按 |
| `isScrollable()` | Bool | 是否可滚动 |
| `isEnabled()` | Bool | 是否启用 |
| `isFocused()` | Bool | 是否聚焦 |
| `isSelected()` | Bool | 是否选中 |
| `isChecked()` | Bool | 是否选中 |
| `isCheckable()` | Bool | 是否可选中 |

---

## 5. UiWindow 类

**位置**: `ohos/ui_test/ui_test_api.cj:908`  
**说明**: 窗口代理，提供窗口操作能力

### 5.1 属性获取

| 方法 | 返回类型 | 说明 |
|------|----------|------|
| `getBundleName()` | String | 获取包名 |
| `getBounds()` | Rect | 获取边界 |
| `getTitle()` | String | 获取标题 |
| `getWindowMode()` | WindowMode | 获取窗口模式 |
| `isFocused()` | Bool | 是否聚焦 |
| `isActive()` | Bool | 是否激活 |

### 5.2 窗口操作

| 方法 | 说明 |
|------|------|
| `focus()` | 聚焦窗口 |
| `moveTo(x: Int32, y: Int32)` | 移动窗口 |
| `resize(wide: Int32, height: Int32, direction: ResizeDirection)` | 调整大小 |
| `split()` | 分屏 |
| `maximize()` | 最大化 |
| `minimize()` | 最小化 |
| `resume()` | 恢复 |
| `close()` | 关闭 |

---

## 6. 其他类

### 6.1 PointerMatrix

多指手势定义。

```cangjie
public class PointerMatrix {
    public static func create(fingers: Int32, steps: Int32): PointerMatrix
    public func setPoint(finger: Int32, step: Int32, point: Point): Unit
}
```

### 6.2 UiEventObserver

UI 事件观察者。

```cangjie
public class UiEventObserver {
    public func once(onceType: OnceType, callback: Callback<UiElementInfo>): Unit
}
```

### 6.3 数据类

#### Point

```cangjie
public class Point {
    public var x: Int32
    public var y: Int32
    public var displayId: ?Int32
    
    public init(x: Int32, y: Int32, displayId!: ?Int32 = None)
    public init(jsonStr: String)  // 从 JSON 解析
}
```

**证据**: `ui_test_common.cj:292-338`

#### Rect

```cangjie
public class Rect {
    public var left: Int32
    public var top: Int32
    public var right: Int32
    public var bottom: Int32
    public var displayId: ?Int32
    
    public init(left: Int32, top: Int32, right: Int32, bottom: Int32, displayId!: ?Int32 = None)
    public init(jsonStr: String)
}
```

**证据**: `ui_test_common.cj:661-729`

#### WindowFilter

```cangjie
public class WindowFilter {
    public var bundleName: ?String
    public var title: ?String
    public var focused: ?Bool
    public var active: ?Bool
    public var displayId: ?Int32
}
```

#### UiElementInfo

```cangjie
public class UiElementInfo {
    public let bundleName: String
    public let componentType: String
    public let text: String
}
```

---

## 7. 常量定义

### 7.1 API 操作标识符

位置: `ohos/ui_test/const.cj`

| 常量名 | 值 | 说明 |
|--------|-----|------|
| DRIVER_CREATE | "Driver.create" | 创建 Driver |
| DRIVER_CLICK | "Driver.click" | 点击 |
| DRIVER_SWIPE | "Driver.swipe" | 滑动 |
| COMPONENT_CLICK | "Component.click" | 组件点击 |
| COMPONENT_GETTEXT | "Component.getText" | 获取文本 |
| ON_TEXT | "On.text" | 文本选择器 |
| UIWINDOW_GETBUNDLENAME | "UiWindow.getBundleName" | 获取包名 |
| OBJ_LOST | 17000004i32 | 对象丢失错误码 |
| TESTMODE_ENABLE | "persist.ace.testmode.enabled" | 测试模式参数 |

完整列表参见 `const.cj:20-124`

---

## 8. 错误码参考

### 8.1 UiTest 错误码

| 错误码 | 名称 | 说明 | 触发位置 |
|--------|------|------|----------|
| 17000001 | - | Initialization failed | Driver.create() |
| 17000003 | - | Assertion failed | assertComponentExist() |
| 17000004 | OBJ_LOST | obj create return null reference | checkRef() |

### 8.2 系统参数错误码

| 错误码 | 说明 | 触发条件 |
|--------|------|----------|
| 14700101 | System parameter can not be found | 参数不存在 |
| 14700102 | System parameter value is invalid | 值无效 |
| 14700103 | System permission operation permission denied | 权限不足 |
| 14700104 | System internal error including out of memory, deadlock etc. | 系统错误 |

**证据**: `systemparameter.cj:29-36`

---

## 9. 调用链附录

### 9.1 典型调用链示例

#### 查找并点击按钮

```
测试脚本
  ├─ Driver.create()
  │   ├─ ApiCallParams("Driver.create", "", "[]")
  │   ├─ CJ_ApiCall() [FFI]
  │   └─ 返回 Driver 实例
  │
  ├─ On.text("确定")
  │   ├─ ApiCallParams("On.text", "On#seed", '["确定"]')
  │   ├─ CJ_ApiCall() [FFI]
  │   └─ 返回 On 实例
  │
  ├─ Driver.findComponent(on)
  │   ├─ ApiCallParams("Driver.findComponent", "Driver#xxx", '["On#xxx"]')
  │   ├─ CJ_ApiCall() [FFI]
  │   └─ 返回 Component 实例
  │
  └─ Component.click()
      ├─ ApiCallParams("Component.click", "Component#xxx", "[]")
      ├─ CJ_ApiCall() [FFI]
      └─ 完成
```

---

## 10. 使用示例

### 10.1 基础测试流程

```cangjie
import ohos.ui_test.*

// 1. 创建 Driver
let driver = Driver.create()

// 2. 查找组件
let on = On().text("登录")
let component = driver.findComponent(on)

// 3. 操作组件
if (let Some(c) <- component) {
    c.click()
}

// 4. 断言
let submitOn = On().text("提交")
driver.assertComponentExist(submitOn)
```

### 10.2 复杂选择器

```cangjie
// 多条件组合
let on = On()
    .text("设置")
    .enabled(true)
    .clickable(true)
    .inWindow("com.example.app")

// 相对定位
let btnOn = On()
    .onType("Button")
    .isAfter(On().text("用户名"))
```

### 10.3 窗口操作

```cangjie
// 查找窗口
let filter = WindowFilter(bundleName: Some("com.example.app"))
let window = driver.findWindow(filter)

if (let Some(w) <- window) {
    w.focus()
    w.moveTo(100, 100)
    w.maximize()
}
```

---

## 11. 下一步阅读

- **[构建系统](30_Build_System.md)** - 了解如何集成到项目
- **[安全分析](40_Security.md)** - 理解 FFI 安全边界
- **[附录：调用链](appendix/Callgraphs.md)** - 详细调用链图

---

*本文档基于代码仓库静态分析生成*  
*证据位置: ohos/ui_test/ui_test_api.cj, ohos/ui_test/const.cj, ohos/ui_test/ui_test_common.cj*
