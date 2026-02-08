# 04_GN_Build - GN 编译配置

## 概述

本文档描述 multimodalinput_input 子系统的 GN 编译配置、关键 targets 和编译产物。

## 代码证据

**根构建文件**: `BUILD.gn`
**模块构建文件**: `*/BUILD.gn`
**GNI 配置**: `multimodalinput_mini.gni`

---

## 4.1 根构建入口

**文件**: `BUILD.gn`

```gn
import("//build/ohos.gni")
import("./multimodalinput_mini.gni")

# 基础配置
config("coverage_flags") {
  if (input_feature_coverage) {
    cflags = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}

# 主服务目标组
group("multimodalinput_mmi_service") {
  deps = [
    "common/property_name_mapper:mmi_property_name_mapper",
    "etc:input_system_etc",
    "etc/joystick:input_joystick_etc",
    "service:libcursor_drawing_adapter",
    "service:libmmi-server",
    "tools/inject_event:uinput",
    "util/screen_capture:libmmi-screen_capture",
    "service:libmmi-server-common",
  ]
}
```

---

## 4.2 主要 Targets 清单

### 4.2.1 服务层 Targets

| Target | 类型 | 输出 | 依赖 |
|--------|------|------|------|
| `libmmi-server` | 静态库 | `libmmi-server.a` | event_handler, device_manager |
| `libmmi-server-common` | 静态库 | `libmmi-server-common.a` | timer_manager, setting_datashare |
| `libcursor_drawing_adapter` | 静态库 | - | display_manager |
| `mmi_property_name_mapper` | 静态库 | - | common |
| `uinput` | 可执行文件 | `uinput` | drivers_interface_input |

### 4.2.2 框架层 Targets

| Target | 类型 | 输出 | 依赖 |
|--------|------|------|------|
| `libmmi-client` | 动态库 | `libmmi-client.z.so` | proxy |
| `oh_input_manager` | 动态库 | `liboh_input_manager.z.so` | native/input |

### 4.2.3 N-API Targets

| Target | 类型 | JS 模块名 |
|--------|------|----------|
| `inputeventclient` | 动态库 | `@ohos.multimodalInput.inputEventClient` |
| `inputdevice` | 动态库 | `@ohos.multimodalInput.inputDevice` |
| `inputmonitor` | 动态库 | `@ohos.multimodalInput.inputMonitor` |
| `inputconsumer` | 动态库 | `@ohos.multimodalInput.inputConsumer` |
| `pointer` | 动态库 | `@ohos.multimodalInput.pointer` |
| `shortkey` | 动态库 | `@ohos.multimodalInput.shortKey` |
| `infraredemitter` | 动态库 | `@ohos.multimodalInput.infraredEmitter` |
| `keyevent` | 动态库 | `@ohos.multimodalInput.keyEvent` |
| `mouseevent` | 动态库 | `@ohos.multimodalInput.mouseEvent` |
| `touchevent` | 动态库 | `@ohos.multimodalInput.touchEvent` |
| `gestureevent` | 动态库 | `@ohos.multimodalInput.gestureEvent` |
| `joystickevent` | 动态库 | `@ohos.multimodalInput.joystickEvent` |
| `keycode` | 动态库 | `@ohos.multimodalInput.keyCode` |
| `intentioncode` | 动态库 | `@ohos.multimodalInput.intentionCode` |

### 4.2.4 编译产物组

```gn
# bundle.json 中的 build 配置
group("base_group") {
  deps = [ "//foundation/multimodalinput/input:multimodalinput_mmi_base" ]
}

group("fwk_group") {
  deps = [
    "//foundation/multimodalinput/input:multimodalinput_mmi_frameworks",
    "//foundation/multimodalinput/input:input_jsapi_group",
    "//foundation/multimodalinput/input:input_ets_group",
    "//foundation/multimodalinput/input/frameworks/native/input:oh_input_manager",
  ]
}

group("service_group") {
  deps = [
    "//foundation/multimodalinput/input:multimodalinput_mmi_service",
    "//foundation/multimodalinput/input/sa_profile:multimodalinput_sa_profile",
    "//foundation/multimodalinput/input:multimodalinput.rc",
    "//foundation/multimodalinput/input:uinput_inject",
    "//foundation/multimodalinput/input:mmi_uinput.rc",
    "//foundation/multimodalinput/input:input-third-mmi",
  ]
}
```

---

## 4.3 N-API 模块构建示例

**文件**: `frameworks/napi/input_event_client/BUILD.gn`

```gn
ohos_shared_library("inputeventclient") {
  sources = [
    "src/js_register_module.cpp",
    "src/js_register_util.cpp",
  ]

  include_dirs = [
    "src/",
    "${mmi_path}/util/common/include",
    "${mmi_path}/interfaces/native/innerkits/event/include",
    "${mmi_path}/interfaces/native/innerkits/common/include",
  ]

  deps = [
    ":inputeventclient_js_assets",
    "//foundation/multimodalinput/input/frameworks/napi/common:inputnapi",
    "//foundation/multimodalinput/input/service:libmmi-server",
    "//third_party/napi:napi",
  ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
  ]

  defines = [ "MMI_DISABLE_LOG_TRACE" ]
}

# JS 资源打包
bundle_resources("inputeventclient_js_assets") {
  sources = [ "src/index.js" ]
  resource_path = "inputeventclient"
  deps = [ "//foundation/multimodalinput/input/frameworks/napi:inputnapi_assets" }
}
```

---

## 4.4 条件编译配置

**文件**: `multimodalinput_mini.gni`

### 4.4.1 Feature Flags

| Feature Flag | 默认值 | 用途 |
|--------------|--------|------|
| `input_feature_product` | "default" | 产品配置 |
| `input_feature_enable_pgo` | false | PGO 优化 |
| `input_feature_combination_key` | false | 组合键支持 |
| `input_feature_input_device` | true | 输入设备支持 |
| `input_feature_interceptor` | true | 事件拦截 |
| `input_feature_keyboard` | true | 键盘支持 |
| `input_feature_monitor` | true | 事件监控 |
| `input_feature_mouse` | true | 鼠标支持 |
| `input_feature_pointer_drawing` | false | 指针绘制 |
| `input_feature_switch` | false | 开关事件 |
| `input_feature_touchscreen` | true | 触摸屏支持 |
| `input_feature_short_key` | true | 快捷键 |
| `input_feature_fingerprint` | false | 指纹 |
| `input_feature_crown` | false | 表冠 |
| `input_feature_joystick` | false | 游戏手柄 |
| `input_feature_virtual_keyboard` | false | 虚拟键盘 |
| `input_feature_knuckle` | false | 指关节 |
| `input_feature_touchpad` | true | 触摸板 |
| `input_feature_pen` | false | 手写笔 |
| `input_feature_touch_gesture` | true | 触摸手势 |

### 4.4.2 产品特定配置

```gn
# rk3568 产品配置示例
input_feature_product = "rk3568"
input_feature_mouse = true
input_feature_keyboard = true
input_feature_touchpad = true

# hi3516dv300 产品配置
input_feature_product = "hi3516dv300"
input_feature_mouse = false
input_feature_keyboard = false
input_feature_touchpad = false
```

---

## 4.5 编译产物

### 4.5.1 动态库产物

| 产物路径 | 大小 | 用途 |
|----------|------|------|
| `system/lib/libmmi-client.z.so` | ~512KB | 客户端代理 |
| `system/lib/liboh_input_manager.z.so` | ~256KB | 输入管理器 |
| `@ohos.multimodalInput-*.har` | - | JS API 资源 |

### 4.5.2 静态库产物

| 产物路径 | 大小 | 用途 |
|----------|------|------|
| `objfoundation/multimodalinput/input/service/libmmi-server.a` | ~2MB | 服务实现 |
| `objfoundation/multimodalinput/input/service/libmmi-server-common.a` | ~512KB | 服务公共代码 |

### 4.5.3 可执行文件产物

| 产物路径 | 用途 |
|----------|------|
| `system/bin/uinput` | uinput 注入工具 |

### 4.5.4 配置文件产物

| 产物路径 | 用途 |
|----------|------|
| `system/etc/init/multimodalinput.cfg` | 服务配置 |
| `system/etc/init/mmi_uinput.rc` | rc 脚本 |
| `sa_profile/3101.json` | SA 配置 |

---

## 4.6 Inner Kits 清单

```json
// bundle.json inner_kits 配置
{
  "name": "//foundation/multimodalinput/input/frameworks/proxy:libmmi-client",
  "type": "so",
  "header": {
    "header_files": [
      "proxy/include/input_manager.h",
      "event/include/key_event.h",
      "event/include/pointer_event.h"
    ],
    "header_base": "//foundation/multimodalinput/input/interfaces/native/innerkits"
  }
}
```

---

## 4.7 依赖配置

### 4.7.1 组件依赖

```json
// bundle.json deps.components
"components": [
  "window_manager",
  "hisysevent",
  "start",
  "napi",
  "c_utils",
  "ipc",
  "hitrace",
  "resource_schedule_service",
  "eventhandler",
  "image_framework",
  "graphic_2d",
  "graphic_surface",
  "drivers_interface_input",
  "drivers_interface_display",
  "safwk",
  "ability_runtime",
  "access_token",
  "ability_base",
  "samgr",
  "config_policy",
  "hicollie",
  "init",
  "preferences",
  "security_component_manager",
  "hilog",
  "common_event_service",
  "data_share",
  "relational_store",
  "faultloggerd",
  "ffrt",
  "hdf_core",
  "bounds_checking_function",
  "call_manager",
  "libinput",
  "screenlock_mgr",
  "openssl"
]
```

### 4.7.2 第三方依赖

```json
"third_party": [
  "libuv",
  "libevdev",
  "mtdev",
  "rust"
]
```

---

## 4.8 编译命令

### 4.8.1 完整编译

```bash
./build.sh --product-name {product} --ccache
```

### 4.8.2 增量编译

```bash
hb build -f  # 只编译 input 组件
```

### 4.8.3 独立编译

```bash
# 编译 N-API 模块
cd frameworks/napi/input_event_client
gn gen out/default
ninja -C out/default inputeventclient

# 编译服务
cd service
gn gen out/default
ninja -C out/default libmmi-server
```
