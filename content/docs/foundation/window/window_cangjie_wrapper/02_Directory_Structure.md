# 目录结构与模块职责

> 源码目录结构、文件职责划分、模块依赖关系

---

## 目的

本文档说明 `window_cangjie_wrapper` 项目的目录结构、文件职责和模块边界，帮助开发者快速定位代码位置。

---

## 目录结构（不含测试）

```
foundation/window/window_cangjie_wrapper/
├── figures/                           # README 使用的架构图
├── ohos/                              # 仓颉接口实现源码目录
│   ├── display/                        # 屏幕属性相关接口实现
│   │   ├── cj_display_common.cj         # 公共工具函数和结构体定义
│   │   ├── cj_display_enum.cj           # 枚举类型定义（465 行）
│   │   ├── cj_display_log.cj             # 日志模块（24 行）
│   │   └── display.cj                  # Display 类及全局函数（726 行）
│   └── window/                         # 窗口相关接口实现
│       ├── cj_window_common.cj           # 公共工具函数和结构体定义（803 行）
│       ├── cj_window_enum.cj            # 枚举类型定义（1109 行）
│       ├── cj_window_log.cj              # 日志模块（25 行）
│       ├── cj_window_utils.cj            # 工具函数（52 行）
│       ├── window.cj                    # Window 类及全局函数（895 行）
│       └── window_stage.cj              # WindowStage 类（157 行）
├── test/                              # 测试用例（不在本文档覆盖范围）
│   ├── display_test/
│   └── window_test/
├── wiki/                              # 本 Wiki 文档
└── _work/                             # 工作笔记和计划
```

**证据**：根目录 `ls -la` 输出、`find ohos -type f` 输出

---

## 模块职责划分

### ohos.display 模块

**职责**：显示设备管理和事件监听

#### 文件职责

| 文件 | 行数 | 职责 | 证据 |
|------|------|--------|--------|
| `cj_display_common.cj` | 327 | 公共结构体（Rect, CutoutInfo, FoldCreaseRegion） | 代码注释 + 内容 |
| `cj_display_enum.cj` | 465 | 枚举类型（Orientation, DisplayState, FoldStatus, ListenerType） | 代码注释 + 内容 |
| `cj_display_log.cj` | 24 | 日志通道（DISPLAY_LOG） | 代码注释 + 内容 |
| `display.cj` | 726 | Display 类及全局 API（getDefaultDisplaySync, getAllDisplays, on/off） | 代码注释 + 内容 |

#### 核心类

##### Display 类（display.cj:485-725）

**职责**：显示设备属性访问和生命周期管理

**属性**：
```cangjie
public class Display <: RemoteDataLite {
    static let INSTANCE_MAP = HashMap<Int64, Display>()
    static let REGISTER_MUTEX = Mutex()

    public prop id: UInt32           // Display ID
    public prop name: String           // 显示名称
    public prop alive: Bool            // 是否存活
    public prop state: DisplayState     // 显示状态
    public prop refreshRate: UInt32    // 刷新率
    public prop rotation: UInt32       // 旋转角度
    public prop orientation: Orientation   // 屏幕方向
    public prop width: Int32           // 宽度（像素）
    public prop height: Int32          // 高度（像素）
    public prop densityDpi: Float32    // 密度
    public prop densityPixels: Float32 // 像素比
    public prop xDpi: Float32          // X 轴 DPI
    public prop yDpi: Float32          // Y 轴 DPI
}
```

**证据**：`display.cj:496-681`

#### 全局函数

| 函数 | 位置 | 职责 |
|------|--------|--------|
| `getDefaultDisplaySync()` | display.cj:130-142 | 获取默认显示设备 |
| `getAllDisplays()` | display.cj:155-169 | 获取所有显示设备 |
| `isFoldable()` | display.cj:180-182 | 检测是否支持折叠屏 |
| `getFoldStatus()` | display.cj:193-198 | 获取折叠状态 |
| `getFoldDisplayMode()` | display.cj:209-214 | 获取折叠显示模式 |
| `getCurrentFoldCreaseRegion()` | display.cj:227-247 | 获取折叠缝区域 |
| `on/off()` | display.cj:258-476 | 注册/注销显示事件回调 |

---

### ohos.window 模块

**职责**：窗口管理和生命周期控制

#### 文件职责

| 文件 | 行数 | 职责 | 证据 |
|------|------|--------|--------|
| `cj_window_common.cj` | 803 | 公共结构体（Size, Rect, WindowProperties, SystemBarProperties, 等） | 代码注释 + 内容 |
| `cj_window_enum.cj` | 1109 | 枚举类型（WindowType, Orientation, WindowCallbackType, SystemBarType） | 代码注释 + 内容 |
| `cj_window_log.cj` | 25 | 日志通道（WINDOW_LIB_LOG） | 代码注释 + 内容 |
| `cj_window_utils.cj` | 52 | 工具函数（checkRet 错误处理） | 代码注释 + 内容 |
| `window.cj` | 895 | Window 类及全局 API（findWindow, createWindow, getLastWindow, 等） | 代码注释 + 内容 |
| `window_stage.cj` | 157 | WindowStage 类（getMainWindow, createSubWindow, loadContent） | 代码注释 + 内容 |

#### 核心类

##### Window 类（window.cj:269-894）

**职责**：窗口实例管理和属性控制

**成员变量**：
```cangjie
public class Window <: RemoteDataLite {
    let callbackMaps = HashMap<String, ArrayList<(CallbackObject, Int64)>>(
        [
            ("windowSizeChange", ...),
            ("avoidAreaChange", ...),
            ("keyboardHeightChange", ...),
            ("touchOutside", ...),
            ("screenshot", ...),
            ("dialogTargetTouch", ...),
            ("windowEvent", ...),
            ("windowVisibilityChange", ...),
            ("windowStatusChange", ...),
            ("windowTitleButtonRectChange", ...),
            ("noInteractionDetected", ...),
            ("windowRectChange", ...),
            ("subWindowClose", ...)
        ]
    )

    static let INSTANCE_MAP = HashMap<Int64, Window>()
}
```

**证据**：`window.cj:270-296`

##### WindowStage 类（window_stage.cj:63-156）

**职责**：窗口阶段管理，管理主窗口和子窗口

**成员变量**：
```cangjie
public class WindowStage <: RemoteDataLite {
    let callbackMaps = HashMap<String, ArrayList<(CallbackObject, Int64)>>(
        [("windowStageEvent", ArrayList<(CallbackObject, Int64)>()]
    )

    protected init(windowStageHandler: Int64) {
        let ret = unsafe { FfiOHOSBindWindowStage(windowStageHandler) }
        checkRet(ret.code, "[WindowStage] bind WindowStage: ")
        dataInit(ret.data)
    }
}
```

**证据**：`window_stage.cj:64-75`

#### 全局函数

| 函数 | 位置 | 职责 |
|------|--------|--------|
| `findWindow(name)` | window.cj:161-171 | 根据名称查找窗口 |
| `createWindow(config)` | window.cj:190-209 | 创建窗口 |
| `getLastWindow(ctx)` | window.cj:224-236 | 获取顶层窗口 |
| `shiftAppWindowFocus(source, target)` | window.cj:255-260 | 转移焦点 |

---

## 数据类型与结构体

### 公共结构体（cj_window_common.cj）

| 结构体 | 用途 | 证据 |
|--------|--------|--------|
| `Size` | 窗口尺寸（width, height） | cj_window_common.cj:30-74 |
| `Rect` | 矩形区域（left, top, width, height） | cj_window_common.cj:272-340 |
| `TitleButtonRect` | 标题按钮矩形区域 | cj_window_common.cj:95-163 |
| `Configuration` | 窗口创建配置 | cj_window_common.cj:191-263 |
| `WindowProperties` | 窗口属性（不可自动更新） | cj_window_common.cj:363-534 |
| `AvoidArea` | 避让区域 | cj_window_common.cj:543-624 |
| `SystemBarProperties` | 系统栏属性 | cj_window_common.cj:644-756 |

### C 互操作结构体

**C 结构体**用于 FFI 调用：

| C 结构体 | 对应仓颉类 | 证据 |
|----------|--------------|--------|
| `CSize` | `Size` | cj_window_common.cj:77-86 |
| `CRect` | `Rect` | cj_window_common.cj:343-354 |
| `CTitleButtonRect` | `TitleButtonRect` | cj_window_common.cj:166-177 |
| `CWindowProperties` | `WindowProperties` | cj_window_common.cj:511-534 |
| `CAvoidArea` | `AvoidArea` | cj_window_common.cj:627-635 |
| `CJBarProperties` | `SystemBarProperties` | cj_window_common.cj:759-779 |

**证据**：`cj_window_common.cj:76-803`

---

## 模块依赖关系

### 外部依赖（bundle.json）

| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `ability_cangjie_wrapper` | 提供 BaseContext（能力上下文） | bundle.json:22 |
| `arkui_cangjie_wrapper` | 提供基础类型（RemoteDataLite） | bundle.json:23 |
| `cangjie_ark_interop` | 提供 FFI 框架（ohos.ffi, ohos.labels, ohos.callback_invoke） | bundle.json:24 |
| `hiviewdfx_cangjie_wrapper` | 提供 Hilog 日志能力 | bundle.json:25 |
| `multimedia_cangjie_wrapper` | 提供图像处理能力（PixelMap） | bundle.json:26 |
| `ability_runtime` | 提供能力运行时（abilitykit_native） | bundle.json:27 |
| `window_manager` | 提供窗口管理服务（cj_window_ffi, cj_display_ffi） | bundle.json:28 |

**证据**：`bundle.json:21-29`

### 内部导入

#### ohos.window 模块导入

```cangjie
import ohos.ffi.{getOrCreate, releaseFFIData, RemoteDataLite, RetDataI64, CArrString, toArrayCString, freeArrCString, Callback1Param}
import ohos.app.ability.BaseContext
import ohos.business_exception.BusinessException
import ohos.labels.APILevel
import ohos.callback_invoke.{CallbackObject, Callback1Argument}
import ohos.multimedia.image.PixelMap
import std.sync.Mutex
import std.collection.{ArrayList, HashMap}
```

**证据**：`window.cj:20-28`

#### ohos.display 模块导入

```cangjie
import ohos.ffi.{Callback1Param, RemoteDataLite, releaseFFIData, getOrCreate}
import ohos.callback_invoke.{CallbackObject, Callback1Argument}
import ohos.business_exception.BusinessException
import ohos.labels.APILevel
import ohos.hilog.HilogChannel
import std.sync.Mutex
import std.collection.{HashMap, ArrayList}
```

**证据**：`display.cj:20-25`

---

## 跨模块交互

### ohos.display → ohos.window

**交互场景**：应用需要获取 Display 信息后创建 Window

**数据流**：
```
Display.getDefaultDisplaySync()
    → 获取 Display 实例
    → Display.id
    → Window.createWindow(config { displayId: Display.id })
```

**证据**：`window.cj:203`（Configuration.displayId 参数）

### ohos.window → ohos.display

**交互场景**：较少直接调用，主要通过 Native 层协调

**例外**：窗口方向设置会间接影响显示状态，由 window_manager 子系统处理

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 项目分为 display 和 window 两个模块，职责清晰 | `ohos/` 目录结构 |
| display 模块：4 个文件，共 1542 行代码 | 文件统计 |
| window 模块：6 个文件，共 3041 行代码 | 文件统计 |
| 使用 FFI 调用 Native 层，避免重复开发 | `foreign` 块定义 |
| 继承 RemoteDataLite 管理资源生命周期 | `window.cj:269`, `display.cj:485` |

---

**生成时间**: 2025-02-06
