# 关键宏与 Feature Flags

> GN 配置键、编译条件、API Level 配置

---

## 目的

本文档列出 `window_cangjie_wrapper` 的关键配置参数、宏定义和 Feature Flags，帮助理解构建系统和 API 配置。

---

## BUILD.gn 配置

### 根 BUILD.gn

| 配置项 | 值 | 用途 | 证据 |
|----------|--------|--------|--------|
| `import("//build/templates/cangjie/cjc.gni")` | 引入仓颉 GN 模板 | `BUILD.gn:14` |
| `window_cangjie_wrapper_packages_ohos` | 定义 ohos 包列表 | `BUILD.gn:16-19` |
| `copy_ohos_cangjie_sdk_api_libs` | SDK 库复制 target | `BUILD.gn:21-23` |

**完整配置**：
```gn
import("//build/templates/cangjie/cjc.gni")

window_cangjie_wrapper_packages_ohos = [
  "//foundation/window/window_cangjie_wrapper/ohos/display:ohos.display",
  "//foundation/window/window_cangjie_wrapper/ohos/window:ohos.window"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_window_cangjie_libs") {
  ohos_inputs = window_cangjie_wrapper_packages_ohos
}
```

---

### ohos.window/BUILD.gn

| 配置项 | 值 | 用途 | 证据 |
|----------|--------|--------|--------|
| `target_type` | `ohos_cangjie_shared_library` | 仓颉共享库 | `BUILD.gn:16` |
| `target_name` | `ohos.window` | 库文件名 | `BUILD.gn:16` |
| `sources` | 6 个 .cj 文件 | 源文件列表 | `BUILD.gn:17-24` |

**cj_external_deps**：
| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `ability_cangjie_wrapper:ohos.app.ability` | BaseContext 能力上下文 | `BUILD.gn:27` |
| `arkui_cangjie_wrapper:ohos.base` | RemoteDataLite 基类 | `BUILD.gn:28` |
| `cangjie_ark_interop:ohos.ffi` | FFI 框架（getOrCreate, releaseFFIData） | `BUILD.gn:29-31` |
| `cangjie_ark_interop:ohos.labels` | APILevel 注解支持 | `BUILD.gn:30` |
| `cangjie_ark_interop:ohos.callback_invoke` | 回调机制（CallbackObject, Callback1Argument） | `BUILD.gn:32` |
| `cangjie_ark_interop:ohos.business_exception` | 异常处理（BusinessException） | `BUILD.gn:33` |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | Hilog 日志能力 | `BUILD.gn:34` |
| `multimedia_cangjie_wrapper:ohos.multimedia.image` | 图像处理（PixelMap） | `BUILD.gn:35` |

**external_deps**：
| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `ability_runtime:abilitykit_native` | 能力运行时（提供 Context 查询） | `BUILD.gn:37-38` |
| `window_manager:cj_window_ffi` | 窗口管理 FFI 实现 | `BUILD.gn:39` |

**完整配置**：
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

---

### ohos/display/BUILD.gn

| 配置项 | 值 | 用途 | 证据 |
|----------|--------|--------|--------|
| `target_type` | `ohos_cangjie_shared_library` | 仓颉共享库 | `BUILD.gn:16` |
| `target_name` | `ohos.display` | 库文件名 | `BUILD.gn:16` |
| `sources` | 4 个 .cj 文件 | 源文件列表 | `BUILD.gn:17` |

**cj_external_deps**：
| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `arkui_cangjie_wrapper:ohos.base` | RemoteDataLite 基类 | `BUILD.gn:24` |
| `cangjie_ark_interop:ohos.ffi` | FFI 框架 | `BUILD.gn:25-28` |
| `cangjie_ark_interop:ohos.labels` | APILevel 注解支持 | `BUILD.gn:26` |
| `cangjie_ark_interop:ohos.callback_invoke` | 回调机制 | `BUILD.gn:27` |
| `cangjie_ark_interop:ohos.business_exception` | 异常处理 | `BUILD.gn:29` |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | Hilog 日志能力 | `BUILD.gn:30` |

**external_deps**：
| 组件 | 用途 | 证据 |
|--------|--------|--------|
| `window_manager:cj_display_ffi` | 显示管理 FFI 实现 | `BUILD.gn:33` |

**完整配置**：
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

---

## API Level 配置

### 统一 API Level

| 配置项 | 值 | 证据 |
|----------|--------|--------|
| **API Level** | 22 | 所有 API `@!APILevel[since: "22"]` |

**证据**：所有公共 API 都使用 `@!APILevel[since: "22"]` 注解

**示例**：
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.WindowManager.WindowManager.Core"
]
public func createWindow(config: Configuration): Window {
    // 实现
}
```

### Syscap 配置

| Syscap | 覆盖范围 | 用途 | 证据 |
|----------|--------|--------|
| `SystemCapability.WindowManager.WindowManager.Core` | 核心窗口管理能力 | 大部分 API |
| `SystemCapability.Window.SessionManager` | 会话管理能力 | 部分 API（shiftAppWindowFocus, minimize, 折叠屏管理） |

**证据**：所有 API 都使用 `@!APILevel[syscap: "..."]` 注解

---

## 日志配置

### 日志域和标签

| 模块 | 域 ID | 标签 | 证据 |
|------|--------|--------|--------|
| **Window** | `0xD004200` | `CJ-Window-Manager` | `cj_window_log.cj:24` |
| **Display** | `0xD004201` | `CJ-Display` | `cj_display_log.cj:23` |

**代码定义**：
```cangjie
// window 模块
const HILOG_DOMAIN_WINDOW: UInt32 = 0xD004200
let WINDOW_LIB_LOG = HilogChannel(0, HILOG_DOMAIN_WINDOW, "CJ-Window-Manager")

// display 模块
const DISPLAY_LOG = HilogChannel(0, 0xD004201, "CJ-Display")
```

---

## 错误码配置

### 窗口错误码（ohos.window）

| 错误码 | 含义 | 证据 |
|----------|--------|--------|
| 1300001 | 重复操作 | `cj_window_utils.cj:25` |
| 1300002 | 窗口状态异常 | `cj_window_utils.cj:26` |
| 1300003 | 窗口管理服务异常 | `cj_window_utils.cj:27` |
| 1300004 | 未授权操作 | `cj_window_utils.cj:28` |
| 1300005 | 窗口阶段异常 | `cj_window_utils.cj:29` |
| 1300006 | 窗口上下文异常 | `cj_window_utils.cj:30` |
| 1300007 | 启动能力失败 | `cj_window_utils.cj:31` |
| 1300008 | 无效的显示操作 | `cj_window_utils.cj:32` |
| 1300009 | 父窗口无效 | `cj_window_utils.cj:33` |
| 1300010 | 全屏模式下不支持的操作 | `cj_window_utils.cj:34` |
| 1300016 | 参数校验错误 | `window.cj:773,819,837` |

**代码定义**：
```cangjie
let ERR_CODE_MAP = HashMap<Int32, String>(
    [
        (1300001, "Repeated operation."),
        (1300002, "This window state is abnormal."),
        // ... 完整 14 个错误码
    ]
)

func checkRet(errCode: Int32, message: String) {
    if (errCode != 0) {
        if (let Some(errMsg) <- ERR_CODE_MAP.get(errCode)) {
            let msg = message + errMsg
            throw BusinessException(errCode, msg)
        }
        let msg = message + "Unrecognized error code: ${errCode}"
        throw BusinessException(errCode, msg)
    }
}
```

### 显示错误码（ohos.display）

| 错误码 | 含义 | 证据 |
|----------|--------|--------|
| 1400001 | 无效的显示或屏幕 | `display.cj:133-136` |
| 1400003 | 显示管理服务异常 | `display.cj:158-161`、`display.cj:273-276` |

### 通用错误码

| 错误码 | 含义 | 证据 |
|----------|--------|--------|
| 201 | 权限验证失败（系统权限检查失败） | `cj_window_utils.cj:35` |
| 202 | 权限验证失败（非系统应用使用系统 API） | `cj_window_utils.cj:36` |
| 401 | 参数错误（必需参数缺失或类型错误） | `cj_window_utils.cj:37` |
| 801 | 设备能力不支持 | `cj_window_utils.cj:38` |
| 100001 | 内部错误（无法创建目标指针类型） | `display.cj:236,715` |

---

## 编译条件宏

### 当前实现

**检查结果**：
- ❌ 无条件编译宏定义
- ❌ 无 Feature Flags
- ❌ 无调试/Release 版本区分
- ✅ 使用统一 API Level 配置

**证据**：所有 BUILD.gn 文件中未发现编译条件宏

### 建议的改进

| 改进项 | 当前状态 | 建议 |
|----------|----------|--------|
| **编译优化** | 无 | 可添加 `-Os` 优化标志 |
| **调试支持** | 无 | 可添加调试符号生成 |
| **条件编译** | 无 | 可根据功能添加条件编译 |

---

## 运行时配置

### 设备类型支持

| 设备类型 | 支持状态 | 证据 |
|----------|----------|--------|
| **standard** | ✅ 支持 | `bundle.json:16` |
| **small** | ❌ 不支持 | - |
| **mini** | ❌ 不支持 | - |
| **wearable** | ❌ 不支持 | - |

**证据**：`bundle.json:16`（`adapted_system_type: [ "standard" ]`）

### 功能开关

| 功能 | 开关 | 证据 |
|----------|--------|--------|
| **窗口管理** | ✅ 始终支持 | 核心功能 |
| **显示管理** | ✅ 始终支持 | 核心功能 |
| **折叠屏管理** | ✅ 始终支持 | 需要折叠屏设备 |
| **画中画** | ❌ 不支持 | 未实现（README 说明） |
| **闪控球** | ❌ 不支持 | 未实现（README 说明） |
| **屏幕截图** | ❌ 不支持 | 未实现（README 说明） |

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 使用统一 API Level 22 配置 | 所有 API `@!APILevel[since: "22"]` |
| 两个主要 GN targets：`ohos.window` 和 `ohos.display` | 根 BUILD.gn |
| 依赖 6 个仓颉组件 + 2 个系统组件 | cj_external_deps + external_deps |
| 支持 Syscap 能力分级 | 核心能力和会话管理能力 |
| 统一错误处理机制 | checkRet 函数 + 14 个错误码映射 |
| 使用 Hilog 日志，两个独立的日志域和标签 | WINDOW_LIB_LOG + DISPLAY_LOG |

---

**生成时间**: 2025-02-06
