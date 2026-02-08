# GN Targets 参考

## 目的

本文档详细说明 ace_engine_lite 的 GN 构建目标（targets）、依赖关系、编译配置和产物输出。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 所有 BUILD.gn 和 .gni 配置文件

---

## 构建系统概览

### 目标层次结构

```
┌─────────────────────────────────────────────────────────┐
│              入口 Target: jsfwk                    │
│  (lite_component - 简单包装)                        │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│        核心 Target: ace_lite                      │
│  (shared_library / static_library)                   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │        主要依赖 Targets               │   │
│  │                                     │   │
│  ┌────────┐  ┌────────┐  ┌────────┐   │
│  │targets│  │  ace_   │  │  gen_   │   │
│  │(group) │  │ common │  │  syscap │   │
│  └────────┘  └────────┘  └────────┘   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 主要 Targets

### 1. jsfwk (入口目标)

**位置**：`frameworks/BUILD.gn:24-26`  
**类型**：`lite_component`

**作用**：顶层组件入口，包装 ace_lite 库

**定义**：
```gn
lite_component("jsfwk") {
  features = [ ":ace_lite" ]
}
```

**证据**：`frameworks/BUILD.gn:24-26`

---

### 2. ace_lite (核心库目标)

**位置**：`frameworks/BUILD.gn:50-157`

**类型**：
- `shared_library`（liteos_a/linux 平台）
- `static_library`（liteos_m 平台）

**输出**：
- `libace_lite.so`（动态库）
- `libace_lite.a`（静态库）

**作用**：主框架库，包含所有核心功能

#### 源文件列表

**数量**：102 个源文件（`ace_lite_sources` 变量）

**主要类别**：
- **animation/**：1 个文件（transition_impl.cpp）
- **base/**：15 个文件（工具类和基础功能）
- **components/**：41 个文件（UI 组件）
- **context/**：12 个文件（Ability 和应用上下文）
- **dialog/**：1 个文件（对话框）
- **directive/**：2 个文件（指令系统）
- **modules/**：4 个文件（核心模块）
- **modules/presets/**：18 个文件（预设模块）
- **router/**：3 个文件（路由系统）
- **stylemgr/**：9 个文件（样式管理）
- **wrapper/**：1 个文件（JS 包装）
- **resource/**：2 个文件（资源）
- **targets/**：1 个文件（平台适配）

**证据**：`ace_lite.gni:160-262`

#### 依赖关系

**public_deps**：
```gn
public_deps = [
    "$ACE_LITE_COMMON_PATH:ace_common_lite",
    "$MODULE_MANAGER_PATH:ace_module_manager_lite",
    "$NATIVE_ENGINE_PATH:ace_native_engine_lite",
    "//base/global/i18n_lite/frameworks/i18n:global_i18n",
    "//base/global/resource_management_lite/frameworks/resmgr_lite:global_resmgr",
    "//commonlibrary/utils_lite/timer_task:ace_kit_timer",
    "//foundation/arkui/ui_lite:ui_lite",
]
```

**条件依赖**（根据 feature flags）：
- `surface_lite`（如果 `ace_engine_lite_surface_lite_enable`）
- `media_lite`（如果 `ace_engine_lite_media_lite_enable`）
- `camera_lite`（如果 `ace_engine_lite_camera_lite_enable`）

**证据**：`frameworks/BUILD.gn:69-77, 103-121`

#### 编译定义

```gn
defines += [
    "GRAPHIC_ENABLE_LINECAP_FLAG=1",
    "GRAPHIC_ENABLE_LINEJOIN_FLAG=1",
    "GRAPHIC_ENABLE_ELLIPSE_FLAG=1",
    "GRAPHIC_ENABLE_BEZIER_ARC_FLAG=1",
    "GRAPHIC_ENABLE_ARC_FLAG=1",
    "GRAPHIC_ENABLE_ROUNDEDRECT_FLAG=1",
    "GRAPHIC_ENABLE_DASH_GENERATE_FLAG=1",
    "GRAPHIC_ENABLE_BLUR_EFFECT_FLAG=1",
    "GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG=1",
    "GRAPHIC_ENABLE_GRADIENT_FILL_FLAG=1",
    "GRAPHIC_ENABLE_PATTERN_FILL_FLAG=1",
    "GRAPHIC_ENABLE_DRAW_IMAGE_FLAG=1",
    "GRAPHIC_ENABLE_DRAW_TEXT_FLAG=1",
    "FEATURE_COMPONENT_TEXT_SPANNABLE=0",
]
```

**条件编译定义**：
```gn
if (ohos_build_type == "debug") {
    defines += [ "JS_PROFILER=1" ]
} else {
    defines += [ "JS_PROFILER=0" ]
}

if (LOSCFG_TEST_JS_BUILD) {
    defines += [ "JSFWK_TEST=1" ]
}
```

**证据**：`frameworks/BUILD.gn:79, 123-156`

---

### 3. ace_common (公共库目标)

**位置**：`frameworks/common/BUILD.gn:25-61`

**类型**：
- `shared_library`（liteos_a/linux 平台）
- `static_library`（liteos_m 平台）

**输出**：
- `libace_common.so`（动态库）
- `libace_common.a`（静态库）

**作用**：日志、内存管理和公共工具

#### 源文件列表

```
log/ace_log.cpp
memory/cache/cache_manager.cpp
memory/mem_proc.cpp
memory/memory_heap.cpp
```

**证据**：`frameworks/common/BUILD.gn:48-54`

#### 依赖关系

```gn
deps = [
    "//third_party/bounds_checking_function:libsec_static",  # liteos_m
    "//third_party/bounds_checking_function:libsec_shared",  # liteos_a/linux
    "$ace_target_root",
]
```

**证据**：`frameworks/common/BUILD.gn:56-61`

---

### 4. ace_native_engine (JS 引擎适配目标)

**位置**：`frameworks/native_engine/BUILD.gn:27-72`

**类型**：
- `shared_library`（liteos_a/linux 平台）
- `static_library`（liteos_m 平台）

**输出**：
- `libace_native_engine.so`（动态库）
- `libace_native_engine.a`（静态库）

**作用**：JerryScript 引擎的 C++ 封装层

#### 源文件列表

```
async/js_async_work.cpp
async/message_queue_utils.cpp
jsi/jsi.cpp
```

**证据**：`frameworks/native_engine/BUILD.gn:57-61`

#### 依赖关系

```gn
public_deps = [
    "$ace_common_root:ace_common_lite",
]

if (ohos_kernel_type == "liteos_m") {
    public_deps += [
        "//foundation/arkui/ui_lite:ui",
        "//third_party/jerryscript:jerry_engine",
    ]
} else {
    public_deps += [
        "//third_party/jerryscript/jerry-core:jerry-core_shared",
    ]
}
```

**证据**：`frameworks/native_engine/BUILD.gn:63-72`

#### Include Dirs

```gn
include_dirs = [
    "$ace_common_root/log",
    "$ace_common_root/memory",
    "$ace_interface_root/async",
    "$ace_interface_root/base",
    "$ace_interface_root/jsi",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits",
    "//third_party/jerryscript/jerry-core/include",
]
```

**证据**：`frameworks/native_engine/BUILD.gn:41-53`

---

### 5. ace_module_manager (模块管理目标)

**位置**：`frameworks/module_manager/BUILD.gn:26-96`

**类型**：
- `shared_library`（liteos_a/linux 平台）
- `static_library`（liteos_m 平台）

**输出**：
- `libace_module_manager.so`（动态库）
- `libace_module_manager.a`（静态库）

**作用**：JS 模块加载和管理

#### 源文件列表

```
module_manager/module_manager.cpp
```

**证据**：`frameworks/module_manager/BUILD.gn:48`

#### 依赖关系

```gn
public_deps = [
    "$ace_frameworks_root/common:ace_common_lite",
    "$ace_frameworks_root/native_engine:ace_native_engine_lite",
    "//commonlibrary/utils_lite/js/builtin:ace_utils_kits",
]
```

**证据**：`frameworks/module_manager/BUILD.gn:50-54`

#### 条件依赖

**LiteOS-A 平台**（额外依赖）：
```gn
public_deps += [
    "${appexecfwk_lite_path}/interfaces/kits/bundle_lite/js/builtin:capability_api",
    "//base/hiviewdfx/hilog_lite/frameworks/js/builtin:ace_kit_hilog",
    "//base/security/huks/frameworks/crypto_lite/js/builtin:ace_kit_cipher",
    "//foundation/communication/netstack/frameworks/js/builtin:http_lite_shared",
    "//test/xts/device_attest_lite/interfaces/kit/js:kit_device_attest",
]
```

**证据**：`frameworks/module_manager/BUILD.gn:76-93`

**条件编译定义**：
```gn
if (defined(global_parts_info.security_huks)) {
    defines += [ "ENABLE_HUKS_ACE_LITE_FEATURE" ]
}
```

**证据**：`frameworks/module_manager/BUILD.gn:73, 56`

---

### 6. targets (平台配置目标)

**位置**：`frameworks/targets/BUILD.gn:35-40`

**类型**：`group`

**作用**：平台特定配置的聚合目标

#### 配置定义

```gn
config("ace_lite_target_config") {
    if (ace_engine_lite_feature_product_config) {
        defines = [ "ENABLE_OHOS_ACELITE_PRODUCT_CONFIG=1" ]
    }
    
    include_dirs = [ "$product_path/ace_lite_config" ]
    
    if (ohos_kernel_type == "liteos_m") {
        include_dirs += [ "liteos_m" ]
    } else if (ohos_kernel_type == "liteos_a") {
        include_dirs += [ "liteos_a" ]
    } else if (ohos_kernel_type == "linux") {
        include_dirs += [ "linux" ]
    }
}
```

**证据**：`frameworks/targets/BUILD.gn:21-33`

#### 公共配置

```gn
public_configs = [
    ":ace_lite_target_config",
    "//foundation/graphic/graphic_utils_lite:graphic_utils_public_config"
]
```

**证据**：`frameworks/targets/BUILD.gn:36-38`

---

### 7. gen_syscap_module_native_mini (代码生成目标)

**位置**：`frameworks/BUILD.gn:160-173`

**类型**：`action`

**作用**：生成系统能力模块代码（liteos_m 平台）

**脚本**：
```gn
script = "${ace_tools_root}/syscap/syscap_native_api_src_gen.py"
```

**输入**：
- `${preloader_output_dir}/system/etc/SystemCapability.json`

**输出**：
- `${target_out_dir}/syscap_module_native_mini.cpp`

**证据**：`frameworks/BUILD.gn:160-173`

---

## Feature Flags

### 平台功能开关

| Flag | GN 变量 | 默认值 | 说明 | 影响的模块 |
|------|----------|--------|------|------|
| `ace_engine_lite_surface_lite_enable` | true | Surface 支持 | ace_lite |
| `ace_engine_lite_netstack_enable` | true | 网络支持 | ace_lite |
| `ace_engine_lite_battery_lite_enable` | true | 电池支持 | ace_lite |
| `ace_engine_lite_kv_store_enable` | true | KV 存储支持 | ace_lite |
| `ace_engine_lite_media_lite_enable` | true | 媒体支持 | ace_lite |
| `ace_engine_lite_camera_lite_enable` | true | 相机支持 | ace_lite |

**证据**：`ace_lite.gni:24-72`

### 模块功能开关

**位置**：`frameworks/targets/liteos_a/acelite_config.h`

| 分类 | Flag | 默认值 | 说明 |
|------|-------|--------|------|
| **模块** | `FEATURE_MODULE_AUDIO` | 0 | 音频模块 |
| | `FEATURE_MODULE_STORAGE` | 0 | 存储模块（file/storage） |
| | `FEATURE_MODULE_DEVICE` | 0 | 设备信息模块 |
| | `FEATURE_MODULE_GEO` | 0 | 地理位置模块 |
| | `FEATURE_MODULE_SENSOR` | 0 | 传感器模块（vibrator/sensor） |
| | `FEATURE_MODULE_BRIGHTNESS` | 0 | 亮度模块 |
| | `FEATURE_MODULE_BATTERY` | 0 | 电池模块 |
| | `FEATURE_MODULE_CONFIGURATION` | 0 | 配置模块 |
| **组件** | `FEATURE_COMPONENT_CANVAS` | 0 | Canvas 组件 |
| | `FEATURE_COMPONENT_CAMERA` | 0 | 相机组件 |
| | `FEATURE_COMPONENT_EDITTEXT` | 1 | 编辑文本组件 |
| | `FEATURE_COMPONENT_QRCODE` | 0 | 二维码组件 |
| | `FEATURE_COMPONENT_VIDEO` | 0 | 视频组件 |
| | `FEATURE_COMPONENT_ANALOG_CLOCK` | 0 | 模拟时钟组件 |
| | `FEATURE_COMPONENT_TABS` | 1 | 标签页组件 |
| **框架** | `FEATURE_SYSCAP_MODULE` | 1 | 系统能力模块 |
| | `FEATURE_TIMER_MODULE` | 1 | 定时器模块 |
| | `FEATURE_DATE_FORMAT` | 1 | 日期格式化 |
| | `FEATURE_NUMBER_FORMAT` | 1 | 数字格式化 |
| | `FEATURE_INTL_MODULE` | 1 | 国际化模块 |
| | `FEATURE_LOCALIZATION_MODULE` | 1 | 本地化模块 |
| | `FEATURE_API_VERSION` | 1 | API 版本支持 |
| **DFX** | `FEATURE_ACELITE_DFX_MODULE` | 1 | DFX 模块 |
| | `FEATURE_ACELITE_LITE_DFX_MODULE` | 1 | 轻量 DFX |
| | `FEATURE_ACELITE_JS_PROFILER` | 0 | JS 性能分析（debug 默认启用） |
| | `FEATURE_ACELITE_SYSTEM_CAPABILITY` | 1 | 系统能力查询 |
| **高级** | `FEATURE_PRODUCT_MODULE` | 0 | 产品自定义模块 |
| | `FEATURE_PRIVATE_MODULE` | 0 | 私有模块 |
| | `FEATURE_LAZY_LOADING_MODULE` | 0 | 懒加载 |

**证据**：`frameworks/targets/liteos_a/acelite_config.h`

### 图形功能开关

**位置**：`frameworks/BUILD.gn:123-137`

| Flag | 值 | 说明 |
|------|------|------|
| `GRAPHIC_ENABLE_LINECAP_FLAG` | 1 | 线帽支持 |
| `GRAPHIC_ENABLE_LINEJOIN_FLAG` | 1 | 线连接支持 |
| `GRAPHIC_ENABLE_ELLIPSE_FLAG` | 1 | 椭圆支持 |
| `GRAPHIC_ENABLE_BEZIER_ARC_FLAG` | 1 | 贝塞尔曲线支持 |
| `GRAPHIC_ENABLE_ARC_FLAG` | 1 | 圆弧支持 |
| `GRAPHIC_ENABLE_ROUNDEDRECT_FLAG` | 1 | 圆角矩形支持 |
| `GRAPHIC_ENABLE_DASH_GENERATE_FLAG` | 1 | 虚线生成支持 |
| `GRAPHIC_ENABLE_BLUR_EFFECT_FLAG` | 1 | 模糊效果支持 |
| `GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG` | 1 | 阴影效果支持 |
| `GRAPHIC_ENABLE_GRADIENT_FILL_FLAG` | 1 | 渐变填充支持 |
| `GRAPHIC_ENABLE_PATTERN_FILL_FLAG` | 1 | 图案填充支持 |
| `GRAPHIC_ENABLE_DRAW_IMAGE_FLAG` | 1 | 图像绘制支持 |
| `GRAPHIC_ENABLE_DRAW_TEXT_FLAG` | 1 | 文本绘制支持 |

**证据**：`frameworks/BUILD.gn:123-137`

---

## 编译产物

### 产物类型

#### 动态库（.so 文件）

**平台**：LiteOS-A, Linux

| 库文件 | 输出目标 | 平台 | 说明 |
|--------|----------|------|------|
| `libace_lite.so` | ace_lite | liteos_a, linux | 主框架库 |
| `libace_common.so` | ace_common | liteos_a, linux | 公共工具库 |
| `libace_native_engine.so` | ace_native_engine | liteos_a, linux | JS 引擎适配 |
| `libace_module_manager.so` | ace_module_manager | liteos_a, linux | 模块管理器 |

#### 静态库（.a 文件）

**平台**：LiteOS-M

| 库文件 | 输出目标 | 平台 | 说明 |
|--------|----------|------|------|
| `libace_lite.a` | ace_lite | liteos_m | 主框架库 |
| `libace_common.a` | ace_common | liteos_m | 公共工具库 |
| `libace_native_engine.a` | ace_native_engine | liteos_m | JS 引擎适配 |
| `libace_module_manager.a` | ace_module_manager | liteos_m | 模块管理器 |

#### 模拟器产物

**平台**：Standard 系统（OS 级别为 standard）

| 库文件 | 输出目标 | 说明 |
|--------|----------|------|------|
| `libace_lite.a` | ace_lite (simulator) | 静态库，用于 Qt 模拟器 |

**证据**：`frameworks/targets/simulator/BUILD.gn:26`

---

## 构建配置

### 内核类型适配

| 内核类型 | ohos_kernel_type | 库类型 | 模拟器支持 |
|---------|-----------------|--------|----------|
| LiteOS-A | liteos_a | shared_library | ❌ |
| LiteOS-M | liteos_m | static_library | ❌ |
| Linux | linux | shared_library | ✅ |

**证据**：`frameworks/BUILD.gn:51-55`

### 模拟器特殊配置

**文件**：`frameworks/targets/simulator/BUILD.gn`

**特殊定义**：
```gn
defines = [
    "TARGET_SIMULATOR=1",
    "JS_ENGINE_EXTERNAL_CONTEXT=1",
    "SCREENSIZE_SPECIFIED=1",
    "MOCK_JS_ASYNC_WORK=1",
]

if (build_lite_full) {
    defines += [ "LITEWEARABLE_SUPPORTED=1" ]
}

if (is_debug == "debug") {
    defines += [ "JS_PROFILER=1" ]
} else {
    defines += [ "JS_PROFILER=0" ]
}
```

**证据**：`frameworks/targets/simulator/BUILD.gn:65-78`

---

## 依赖关系图

### 完整依赖树

```
jsfwk
  └── ace_lite
        ├── targets (group)
        │   └── ace_lite_target_config
        ├── ace_common_lite
        │   └── bounds_checking_function (libsec)
        ├── ace_module_manager_lite
        │   ├── ace_common_lite
        │   ├── ace_native_engine_lite
        │   │   └── jerryscript
        │   └── ace_utils_kits
        ├── ace_native_engine_lite
        │   ├── ace_common_lite
        │   └── jerryscript (or ui_lite for liteos_m)
        └── [条件依赖]
            ├── surface_lite
            ├── media_lite
            ├── camera_lite
            └── battery_lite
        └── [外部依赖]
            ├── ui_lite (图形框架)
            ├── global_i18n (国际化)
            ├── global_resmgr (资源管理)
            └── ace_kit_timer (定时器)
```

**证据**：各 BUILD.gn 文件的 deps 定义

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [05_Inner_API.md](05_Inner_API.md) - 内部 API
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置标志完整列表
