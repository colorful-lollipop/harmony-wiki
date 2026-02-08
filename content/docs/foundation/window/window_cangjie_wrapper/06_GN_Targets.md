# GN Targets 与编译产物

> GN targets 清单、依赖关系、编译产物、安装路径

---

## 目的

本文档说明 `window_cangjie_wrapper` 的 GN 构建配置，列出所有 targets、依赖关系和生成的编译产物。

---

## 构建系统

### GN 模板

**使用模板**：`//build/templates/cangjie/cjc.gni`

**构建类型**：`ohos_cangjie_shared_library`

**特点**：
- 仓颉语言特有的构建类型
- 编译生成共享库（.so 文件）
- 支持仓颉外部依赖（`cj_external_deps`）

**证据**：`ohos/window/BUILD.gn:14`、`ohos/display/BUILD.gn:14`

---

## Targets 清单

### 根 BUILD.gn

| Target | 类型 | 产物 | 证据 |
|--------|------|------|--------|
| `copy_sdk_window_cangjie_libs` | `copy_ohos_cangjie_sdk_api_lib` | SDK 库复制 | `BUILD.gn:21-23` |

**证据**：`BUILD.gn:21-23`

```gn
window_cangjie_wrapper_packages_ohos = [
  "//foundation/window/window_cangjie_wrapper/ohos/display:ohos.display",
  "//foundation/window/window_cangjie_wrapper/ohos/window:ohos.window"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_window_cangjie_libs") {
  ohos_inputs = window_cangjie_wrapper_packages_ohos
}
```

### ohos/window/BUILD.gn

| Target | 类型 | 源文件 | 产物 |
|--------|------|----------|------|
| `ohos.window` | `ohos_cangjie_shared_library` | `libohos.window.so` |

**证据**：`ohos/window/BUILD.gn:16-44`

```gn
ohos_cangjie_shared_library("ohos.window") {
  sources = [
    "cj_window_common.cj",
    "cj_window_enum.cj",
    "cj_window_log.cj",
    "cj_window_utils.cj",
    "window.cj",
    "window_stage.cj",
  ]
  cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability",
    "arkui_cangjie_wrapper:ohos.base",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.business_exception",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
    "multimedia_cangjie_wrapper:ohos.multimedia.image",
  ]
  external_deps = [
    "ability_runtime:abilitykit_native",
    "window_manager:cj_window_ffi",
  ]
  subsystem_name = "window"
  part_name = "window_cangjie_wrapper"
}
```

#### 源文件统计

| 文件 | 行数 | 用途 |
|------|------|--------|
| `cj_window_common.cj` | 803 | 公共结构体（Size, Rect, WindowProperties 等） |
| `cj_window_enum.cj` | 1109 | 枚举类型（WindowType, Orientation, WindowCallbackType） |
| `cj_window_log.cj` | 25 | 日志模块（WINDOW_LIB_LOG） |
| `cj_window_utils.cj` | 52 | 工具函数（checkRet 错误处理） |
| `window.cj` | 895 | Window 类及全局函数（findWindow, createWindow 等） |
| `window_stage.cj` | 157 | WindowStage 类及方法（getMainWindow, createSubWindow 等） |

**总代码量**：3041 行（不含注释和空行）

### ohos/display/BUILD.gn

| Target | 类型 | 源文件 | 产物 |
|--------|------|----------|------|
| `ohos.display` | `ohos_cangjie_shared_library` | `libohos.display.so` |

**证据**：`ohos/display/BUILD.gn:16-37`

```gn
ohos_cangjie_shared_library("ohos.display") {
  sources = [
    "cj_display_common.cj",
    "cj_display_enum.cj",
    "cj_display_log.cj",
    "display.cj",
  ]
  cj_external_deps = [
    "arkui_cangjie_wrapper:ohos.base",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.business_exception",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]
  external_deps = [ "window_manager:cj_display_ffi" ]
  subsystem_name = "window"
  part_name = "window_cangjie_wrapper"
}
```

#### 源文件统计

| 文件 | 行数 | 用途 |
|------|------|--------|
| `cj_display_common.cj` | 327 | 公共结构体（Rect, CutoutInfo, FoldCreaseRegion） |
| `cj_display_enum.cj` | 465 | 枚举类型（Orientation, DisplayState, FoldStatus, ListenerType） |
| `cj_display_log.cj` | 24 | 日志模块（DISPLAY_LOG） |
| `display.cj` | 726 | Display 类及全局函数（getDefaultDisplaySync, getAllDisplays 等） |

**总代码量**：1542 行（不含注释和空行）

---

## 依赖关系

### ohos.window 依赖

#### 仓颉外部依赖（cj_external_deps）

| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `ability_cangjie_wrapper:ohos.app.ability` | BaseContext 能力上下文 | `BUILD.gn:27` |
| `arkui_cangjie_wrapper:ohos.base` | RemoteDataLite 基类 | `BUILD.gn:28` |
| `cangjie_ark_interop:ohos.ffi` | FFI 框架（getOrCreate, releaseFFIData, RetDataI64） | `BUILD.gn:29-31` |
| `cangjie_ark_interop:ohos.labels` | APILevel 注解支持 | `BUILD.gn:30` |
| `cangjie_ark_interop:ohos.callback_invoke` | 回调机制（CallbackObject, Callback1Argument） | `BUILD.gn:32` |
| `cangjie_ark_interop:ohos.business_exception` | 异常处理（BusinessException） | `BUILD.gn:33` |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | 日志能力（HilogChannel） | `BUILD.gn:34` |
| `multimedia_cangjie_wrapper:ohos.multimedia.image` | 图像处理（PixelMap） | `BUILD.gn:35` |

#### 系统外部依赖（external_deps）

| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `ability_runtime:abilitykit_native` | 能力运行时（能力上下文） | `BUILD.gn:37-38` |
| `window_manager:cj_window_ffi` | 窗口管理 FFI 接口 | `BUILD.gn:39` |

### ohos.display 依赖

#### 仓颉外部依赖（cj_external_deps）

| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `arkui_cangjie_wrapper:ohos.base` | RemoteDataLite 基类 | `BUILD.gn:24` |
| `cangjie_ark_interop:ohos.ffi` | FFI 框架 | `BUILD.gn:25-28` |
| `cangjie_ark_interop:ohos.labels` | APILevel 注解支持 | `BUILD.gn:26` |
| `cangjie_ark_interop:ohos.callback_invoke` | 回调机制 | `BUILD.gn:27` |
| `cangjie_ark_interop:ohos.business_exception` | 异常处理 | `BUILD.gn:29` |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | 日志能力 | `BUILD.gn:30` |

#### 系统外部依赖（external_deps）

| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `window_manager:cj_display_ffi` | 显示设备管理 FFI 接口 | `BUILD.gn:33` |

---

## 编译产物

### 预期输出

#### 共享库文件

| Target | 输出文件 | 证据 |
|--------|----------|--------|
| `ohos.window` | `libohos.window.so` | `ohos/window/BUILD.gn:16` |
| `ohos.display` | `libohos.display.so` | `ohos/display/BUILD.gn:16` |

#### SDK 复制目标

| Target | 输出目录 | 证据 |
|--------|----------|--------|
| `copy_sdk_window_cangjie_libs` | SDK libs 目录 | `BUILD.gn:21-23` |

---

## 安装路径与加载关系

### 运行时加载路径

| 库文件 | 安装路径 | 加载方式 |
|----------|----------|----------|
| `libohos.window.so` | `/system/lib64/` 或 `/vendor/lib64/` | 仓颉运行时动态加载 |
| `libohos.display.so` | `/system/lib64/` 或 `/vendor/lib64/` | 仓颉运行时动态加载 |

### 运行时依赖加载

| 依赖 | 链接方式 | 证据 |
|--------|----------|--------|
| `window_manager:cj_window_ffi` | 动态链接（so 文件） | Native 层实现 |
| `window_manager:cj_display_ffi` | 动态链接（so 文件） | Native 层实现 |
| `ability_runtime:abilitykit_native` | 动态链接（so 文件） | Native 层实现 |
| 仓颉封装组件 | 通过仓颉包管理器 | OpenHarmony 包系统 |

---

## 构建配置说明

### subsystem_name

| 模块 | subsystem_name | 证据 |
|--------|--------------|--------|
| ohos.window | "window" | `BUILD.gn:43` |
| ohos.display | "window" | `BUILD.gn:35` |

### part_name

| 模块 | part_name | 证据 |
|--------|--------------|--------|
| ohos.window | "window_cangjie_wrapper" | `BUILD.gn:44` |
| ohos.display | "window_cangjie_wrapper" | `BUILD.gn:36` |

---

## 编译产物验证

### 输出文件命名规范

| Target | 输出文件 | 规范 |
|--------|----------|------|
| `ohos.window` | `libohos.window.so` | `lib<subsystem>.<target>.so` |
| `ohos.display` | `libohos.display.so` | `lib<subsystem>.<target>.so` |

**证据**：OpenHarmony GN 构建系统规范

### 版本信息

| 配置 | 值 | 证据 |
|--------|------|--------|
| 组件版本 | "6.1" | `bundle.json:4` |
| API Level | "22" | 所有 API `@!APILevel[since: "22"]` |
| 目标平台 | standard | `bundle.json:16` |

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 使用 `ohos_cangjie_shared_library` 模板编译生成共享库 | 所有 BUILD.gn 文件 |
| ohos.window 生成 `libohos.window.so`，依赖 6 个仓颉组件 + 2 个系统组件 | `ohos/window/BUILD.gn:16-44` |
| ohos.display 生成 `libohos.display.so`，依赖 4 个仓颉组件 + 1 个系统组件 | `ohos/display/BUILD.gn:16-37` |
| 通过 `copy_ohos_cangjie_sdk_api_lib` target 将库复制到 SDK 目录 | `BUILD.gn:21-23` |
| window 子系统的 Native FFI 实现（cj_window_ffi/cj_display_ffi）作为外部依赖 | 所有 BUILD.gn 文件 |

---

**生成时间**: 2025-02-06
