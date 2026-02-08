# 关键调用链

> 核心功能的入口→核心逻辑的调用链分析

---

## 目的

本文档详细说明 `window_cangjie_wrapper` 的关键调用链，帮助理解数据流向和执行流程。

---

## 窗口创建调用链

### 完整流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant CW as createWindow()
    participant FFI as FfiOHOSCreateWindow
    participant Native as window_manager
    participant Map as INSTANCE_MAP
    participant Win as Window

    App->>CW: createWindow(config)
    CW->>CW: 验证参数
    CW->>CW: checkRet(errCode, message)
    CW->>FFI: FFIGetContext(ctx.getID())
    Note over CW: 获取 StageContext
    FFI->>Native: 查询上下文
    Native-->>FFI: 返回 CPointer<Unit>
    CW->>FFI: FfiOHOSCreateWindow(name, type, ctx, displayId, parentId)
    Note over CW: 调用 Native 创建窗口
    FFI->>Native: 创建窗口实例
    Native-->>FFI: 返回 RetDataI64 {code, data: windowId}
    FFI-->>CW: 返回创建结果
    CW->>CW: checkRet(ret.code, "[Window] createWindow: ")
    Note over CW: 验证 FFI 返回码
    CW->>Map: getOrCreate(INSTANCE_MAP, windowId, {id => Window(id)})
    Note over CW: 创建或获取 Window 实例
    Map-->>CW: 返回 Window 实例
    CW-->>App: 返回 Window
```

**代码位置**：`window.cj:190-209`

**关键点**：
1. `FFIGetContext()` - 获取 Ability 上下文的 StageContext
2. `FfiOHOSCreateWindow()` - Native 层创建窗口
3. `getOrCreate(INSTANCE_MAP, ...)` - 单例管理窗口实例
4. `Window(id)` - 创建 Window 实例，继承 RemoteDataLite

---

## Display 查询调用链

### getDefaultDisplaySync 流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant GS as getDefaultDisplaySync()
    participant FFI1 as FfiOHOSGetDefaultDisplaySync
    participant Native as window_manager
    participant FFI2 as FfiOHOSDisplayGet*
    participant Map as INSTANCE_MAP
    participant Disp as Display

    App->>GS: getDefaultDisplaySync()
    GS->>FFI1: FfiOHOSGetDefaultDisplaySync()
    Note over GS: 调用 Native 查询默认显示
    FFI1->>Native: 执行查询
    Native-->>FFI1: 返回 RetStruct {code, len, data: displayId*}
    Note over GS: 返回 Display ID 数组
    FFI1-->>GS: 返回查询结果
    GS->>GS: 检查返回码
    GS->>Map: 遍历数组，为每个 ID 创建 Display 实例
    loop 遍历 displayId[]
        GS->>FFI2: FfiOHOSDisplayGetId(displayId[i])
        Note over GS: 获取 Display 属性
        FFI2->>Native: 查询属性
        Native-->>FFI2: 返回属性值
        FFI2-->>GS: 返回属性
        GS->>Map: getOrCreate(INSTANCE_MAP, displayId[i], {id => Display(id)})
    Note over GS: 创建 Display 实例
    GS-->>App: 返回 Array<Display>
```

**代码位置**：`display.cj:130-142`

**关键点**：
1. `FfiOHOSGetDefaultDisplaySync()` - Native 层查询默认显示
2. 遍历 Display ID 数组，为每个 ID 创建实例
3. `FfiOHOSDisplayGet*()` - 系列获取 Display 属性（id, name, width, height 等）
4. `getOrCreate(INSTANCE_MAP, ...)` - 单例管理 Display 实例

---

## 回调注册调用链

### KeyboardHeightChange 回调

```mermaid
sequenceDiagram
    participant App as 应用
    participant Win as Window
    participant Mutex as REGISTER_MUTEX
    participant Map as callbackMaps
    participant FFI as FfiOHOSOnCallback
    participant Native as window_manager
    participant Callback as 用户回调

    App->>Win: on(KeyboardHeightChange, callback)
    Win->>Mutex: synchronized(REGISTER_MUTEX)
    Note over Win: 进入同步块，保护并发访问
    Mutex->>Map: 获取 callbackMaps.entryView("keyboardHeightChange")
    Map-->>Mutex: 返回 entry
    Mutex-->>Win: 返回 entry
    Mutex-->>Mutex: 退出同步块
    Win->>Win: findCallbackObject(list, callback)
    Note over Win: 检查回调是否已注册
    Win->>Win: 创建 wrapper 函数
    Note over Win: wrapper = {data => callback.invoke(None, val)}
    Win->>FFI: FfiOHOSOnCallback(id, callbackId, "keyboardHeightChange")
    Note over Win: 注册回调到 Native 层
    FFI->>Native: 注册回调
    Native-->>FFI: 返回注册结果
    FFI-->>Win: 返回 callbackId
    Win->>Map: 添加 (callback, callbackId) 到 ArrayList
    Note over Win: 保存回调引用
    Win-->>App: 返回 Unit
```

**代码位置**：`window.cj:776-802`

**关键点**：
1. `synchronized(REGISTER_MUTEX)` - Mutex 保护并发访问
2. `findCallbackObject()` - 避免重复注册
3. `FfiOHOSOnCallback()` - Native 层注册回调
4. 将 `(callback, callbackId)` 保存到 `callbackMaps` - 便于后续注销

---

## WindowStage 创建子窗口调用链

```mermaid
sequenceDiagram
    participant App as 应用
    participant WS as WindowStage
    participant FFI as FfiOHOSCreateSubWindow
    participant Native as window_manager
    participant Map as INSTANCE_MAP
    participant Win as Window

    App->>WS: createSubWindow(name)
    WS->>FFI: FfiOHOSCreateSubWindow(id, name)
    Note over WS: 调用 Native 创建子窗口
    FFI->>Native: 创建子窗口
    Native-->>FFI: 返回 RetDataI64 {code, data: subWindowId}
    FFI-->>WS: 返回创建结果
    WS->>WS: checkRet(ret.code, "[WindowStage] create subwindow: ")
    WS->>Map: getOrCreate(INSTANCE_MAP, subWindowId, {id => Window(id)})
    Note over WS: 创建或获取子窗口 Window 实例
    Map-->>WS: 返回 Window
    WS-->>App: 返回 Window
```

**代码位置**：`window_stage.cj:105-114`

**关键点**：
1. `FfiOHOSCreateSubWindow()` - Native 层创建子窗口
2. `checkRet()` - 验证 Native 层返回码
3. `getOrCreate(INSTANCE_MAP, ...)` - 使用 Window 单例管理子窗口
4. 子窗口返回 `Window` 实例，与主窗口共享类型

---

## 窗口属性设置调用链

### setWindowBackgroundColor 流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant Win as Window
    participant FFI as FfiOHOSSetWindowBackgroundColor
    participant Native as window_manager

    App->>Win: setWindowBackgroundColor(color)
    Win->>Win: getID()
    Win->>Win: LibC.mallocCString(color)
    Note over Win: 分配 C 字符串
    Win->>FFI: FfiOHOSSetWindowBackgroundColor(id, cString)
    Note over Win: 调用 Native 设置背景色
    FFI->>Native: 设置窗口背景色
    Native-->>FFI: 返回结果码
    FFI-->>Win: 返回设置结果
    Win->>Win: checkRet(ret, "[Window] setWindowBackgroundColor: ")
    Note over Win: 验证 FFI 返回码
    Win-->>App: 返回 Unit
    Note over Win: C 字符串通过 try-resource 自动释放
```

**代码位置**：`window.cj:456-463`

**关键点**：
1. `LibC.mallocCString(color)` - 分配 C 字符串
2. `try-resource` 模式 - 自动释放 C 字符串
3. `FfiOHOSSetWindowBackgroundColor()` - Native 层设置背景色
4. `checkRet()` - 统一错误处理

---

## 显示事件回调调用链

### Display 添加事件回调

```mermaid
sequenceDiagram
    participant App as 应用
    participant DM as DisplayManager
    participant Mutex as REGISTER_MUTEX
    participant Map as CALLBACK_MAP
    participant FFI as FfiOHOSRegisterDisplayManagerCallback
    participant Native as window_manager
    participant Callback as 用户回调

    App->>DM: on(ListenerTypeAdd, callback)
    DM->>Mutex: synchronized(REGISTER_MUTEX)
    Mutex->>Map: 获取 CALLBACK_MAP.entryView("add")
    Map-->>Mutex: 返回 entry
    Mutex-->>Mutex: 退出同步块
    DM->>DM: findCallbackObject(list, callback)
    DM->>DM: 创建 wrapper 函数
    DM->>FFI: FfiOHOSRegisterDisplayManagerCallback("add", callbackId)
    FFI->>Native: 注册显示添加事件回调
    Native-->>FFI: 返回注册结果
    FFI-->>DM: 返回 callbackId
    DM->>Map: 添加 (callback, callbackId)
    DM-->>App: 返回 Unit
```

**代码位置**：`display.cj:295-301`

**关键点**：
1. `FfiOHOSRegisterDisplayManagerCallback()` - Native 层注册全局显示事件回调
2. `findCallbackObject()` - 检查是否已注册
3. 将 `(callback, callbackId)` 保存到 `CALLBACK_MAP`
4. 使用 `Mutex` 保证线程安全

---

## 关键结论

| 调用链类型 | 关键方法 | 代码位置 |
|----------|----------|--------|
| 窗口创建 | `createWindow()` | `window.cj:190-209` |
| Display 查询 | `getDefaultDisplaySync()` | `display.cj:130-142` |
| 回调注册 | `on(KeyboardHeightChange, callback)` | `window.cj:776-802` |
| 子窗口创建 | `createSubWindow()` | `window_stage.cj:105-114` |
| 属性设置 | `setWindowBackgroundColor()` | `window.cj:456-463` |
| 显示事件回调 | `on(ListenerTypeAdd, callback)` | `display.cj:295-301` |

**数据流特点**：
- 应用层 → 仓颉封装层 → FFI 接口 → Native 服务层
- 使用 `Mutex` 保护并发访问
- 使用单例 HashMap 管理实例
- 统一错误处理机制（`checkRet`）

---

**生成时间**: 2025-02-06
