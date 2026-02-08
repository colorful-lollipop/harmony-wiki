# Ace Engine 构建系统

> **文档版本**: v1.0  
> **更新时间**: 2026-02-06  
> **源码版本**: OpenHarmony ace_engine

---

## 📋 目录

1. [构建系统概述](#构建系统概述)
2. [核心构建配置](#核心构建配置)
3. [主要 Targets](#主要-targets)
4. [组件库](#组件库)
5. [编译产物](#编译产物)
6. [构建命令](#构建命令)

---

## 构建系统概述

### 1.1 构建工具

| 工具 | 用途 | 证据 |
|------|------|------|
| **GN** | 生成 Ninja 构建文件 | `BUILD.gn` 文件 |
| **Ninja** | 执行构建 | 标准 GN 流程 |
| **HB** | OpenHarmony 构建工具 | `build.sh` 脚本 |

### 1.2 构建入口

```bash
# OpenHarmony 根目录执行
./build.sh --product-name <product> --build-target ace_engine
```

**常见产品名称**: `rk3568`, `ohos-sdk`

> **证据来源**: `CLAUDE.md` - "Build System"

---

## 核心构建配置

### 2.1 根构建文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 根构建配置（定义 `ace_config`, `ace_test_config`） |
| `ace_config.gni` | 全局构建配置和 feature flags |
| `build/ace_lib.gni` | 构建模板（`libace_static`, `ace_bridge_engine`） |
| `bundle.json` | 组件元数据和依赖声明 |

### 2.2 配置定义

**`BUILD.gn` 主要配置**:

| 配置名 | 类型 | 用途 |
|--------|------|------|
| `ace_config` | config | 公共 ACE 源码配置（include_dirs, defines, cflags） |
| `ace_test_config` | config | 测试专用配置 |
| `ace_coverage_config` | config | 代码覆盖率检测配置 |

**关键编译选项**:

```gn
cflags = [
  "-fvisibility=hidden",      # 隐藏符号
  "-fdata-sections",         # 数据段分离
  "-ffunction-sections",     # 函数段分离
  "-Os",                     # 优化大小
  "-Werror=return-stack-address",
  "-Werror=enum-compare-switch",
  # ... 更多警告选项
]
```

> **证据来源**: `BUILD.gn:43-68`

### 2.3 Feature Flags

在 `bundle.json` 中定义的 feature flags：

| Feature Flag | 描述 |
|--------------|------|
| `ace_engine_feature_enable_accessibility` | 无障碍功能 |
| `ace_engine_feature_enable_web` | Web 支持 |
| `ace_engine_feature_enable_pgo` | PGO 优化 |
| `ace_engine_feature_enable_coverage` | 代码覆盖率 |
| `ace_engine_feature_enable_gpu` | GPU 支持 |
| `ace_engine_feature_enable_form_size_change_animation` | 卡片尺寸变化动画 |

> **证据来源**: `bundle.json:20-44`

---

## 主要 Targets

### 3.1 核心引擎库

| Target | 类型 | 输出 | 用途 |
|--------|------|------|------|
| `libace_compatible` | ohos_shared_library | `libace_compatible.z.so` | **主兼容引擎库**（当前核心） |
| `libace` | ohos_shared_library | `libace.z.so` | NG 专用引擎库（无遗留支持） |
| `libace_compatible_components` | ohos_shared_library | `libace_compatible_components.z.so` | 兼容组件库 |

**libace_compatible 依赖**:
```
interfaces/inner_api/ace_kit:ace_kit
build:libace_static_ohos
adapter/ohos/services/etc:ace_engine_param
hilog:libhilog
```

### 3.2 核心框架 Source Sets

| Template | 生成 Target | 用途 |
|----------|------------|------|
| `ace_core_source_set` | `ace_core_ohos`, `ace_core_ohos_ng` | 遗留 + NG 核心源码 |
| `ace_core_ng_source_set` | `ace_core_ng_ohos`, `ace_core_ng_ohos_ng` | NG 专用核心源码 |

### 3.3 桥接层

| Template | 生成 Target | 用途 |
|----------|------------|------|
| `framework_bridge` | `framework_bridge_ohos` | 完整桥接（遗留 + NG） |
| `framework_bridge_ng` | `framework_bridge_ng_ohos` | NG 专用桥接 |

### 3.4 前端库

| Target | 类型 | 输出 | 用途 |
|--------|------|------|------|
| `arkts_frontend` | ohos_shared_library | `libarkts_frontend.z.so` | ArkTS 前端桥接 |
| `cj_frontend_ohos` | ohos_shared_library | `libcj_frontend_ohos.z.so` | Cangjie 前端桥接 |
| `ArkoalaNative_ark` | ohos_shared_library | `libarkoala_native_ark.z.so` | Arkoala 原生实现 |

### 3.5 适配层

| Template | 生成 Target | 用途 |
|----------|------------|------|
| `ace_ohos_standard_source_set` | `ace_ohos_standard_entrance_ohos` | OpenHarmony 平台适配入口 |

---

## 组件库

### 4.1 动态组件库 (libarkui_*.so)

组件库按需动态加载，位于 `adapter/ohos/build/BUILD.gn`：

| 组件库 | 组件名 |
|--------|--------|
| `arkui_counter` | Counter |
| `arkui_datapanel` | DataPanel |
| `arkui_gauge` | Gauge |
| `arkui_checkbox` | Checkbox |
| `arkui_radio` | Radio |
| `arkui_slider` | Slider |
| `arkui_rating` | Rating |
| `arkui_qrcode` | QRCode |
| `arkui_patternlock` | PatternLock |
| `arkui_timepicker` | TimePicker |
| `arkui_calendarpicker` | CalendarPicker |
| `arkui_indexer` | Indexer |
| `arkui_symbol` | Symbol |
| `arkui_waterflow` | WaterFlow |
| `arkui_sidebar` | SideBar |
| `arkui_marquee` | Marquee |
| `arkui_stepper` | Stepper |
| `arkui_linearsplit` | LinearSplit |
| `arkui_folderstack` | FolderStack |

**组件库依赖**:
```gn
frameworks/core/components_ng/pattern/{module}:ace_core_components_{module}_pattern_ng_ohos
build:libace_compatible
frameworks/core:ace_container_scope
```

### 4.2 高级组件 (advanced_ui_component)

| 组件 | 命名空间 |
|------|----------|
| Chip | `arkui.advanced.Chip` |
| Dialog | `arkui.advanced.Dialog` |
| Popup | `arkui.advanced.Popup` |
| ToolBar | `arkui.advanced.ToolBar` |
| SegmentButton | `arkui.advanced.SegmentButton` |
| Counter | `arkui.advanced.Counter` |
| ArcButton | `arkui.advanced.ArcButton` |
| ArcSlider | `arkui.advanced.ArcSlider` |

---

## 编译产物

### 5.1 输出目录

```
out/{product}/arkui/ace_engine/
```

### 5.2 核心产物清单

| 产物 | 类型 | 用途 |
|------|------|------|
| `libace.z.so` | 共享库 | NG 核心引擎 |
| `libace_compatible.z.so` | 共享库 | 兼容引擎（当前主产物） |
| `libace_compatible_components.z.so` | 共享库 | 兼容组件库 |
| `libarkts_frontend.z.so` | 共享库 | ArkTS 前端 |
| `libcj_frontend_ohos.z.so` | 共享库 | Cangjie 前端 |
| `libarkui_*.z.so` | 共享库 | 各组件库（动态加载） |

### 5.3 字节码文件 (.abc)

```
out/{product}/arkui/ace_engine/ark*.abc
```

| 字节码 | 用途 |
|--------|------|
| `arkbutton.abc` | Button 组件 |
| `arkslider.abc` | Slider 组件 |
| `modifier.abc` | 属性修饰器 |
| `statemanagement.abc` | 状态管理 |
| `uicontext.abc` | UI 上下文 |

### 5.4 NDK 产物

| 产物 | 类型 | 用途 |
|------|------|------|
| `libace_ndk.z.so` | 共享库 | NDK 接口库 |
| `ace_ndk` | NDK 库 | NDK ROM 库 |

---

## 构建命令

### 6.1 完整构建

```bash
# 从 OpenHarmony 根目录
./build.sh --product-name rk3568 --build-target ace_engine
```

### 6.2 快速构建

```bash
./build.sh --product-name rk3568 --build-target ace_engine -b
```

### 6.3 构建测试

```bash
# 构建单元测试
./build.sh --product-name rk3568 --build-target unittest

# 构建基准测试
./build.sh --product-name rk3568 --build-target benchmark_linux
```

### 6.4 构建 SDK

```bash
./build.sh --product-name ohos-sdk --build-target ace_engine
```

---

## 🔗 相关文档

- 项目概览: [00_Overview](00_Overview.md)
- 架构说明: [01_Architecture](01_Architecture.md)
- N-API 接口: [02_NAPI](02_NAPI.md)
- 安全评审: [04_Security](04_Security.md)
