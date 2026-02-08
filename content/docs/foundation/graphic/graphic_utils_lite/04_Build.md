# GN 构建配置（Build）

本文档详细说明 `graphic_utils_lite` 的 GN 构建配置，包括 Targets 列表、依赖关系与编译产物。

## 构建系统

| 项目 | 值 |
|------|-----|
| 构建系统 | GN（Generate Ninja） |
| 构建命令 | `hb build graphic_utils_lite` |
| 配置文件 | `BUILD.gn`, `utils.gni`, `bundle.json` |

## 配置文件说明

### 主要配置文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 主构建入口，定义所有 targets |
| `utils.gni` | GN 变量定义，被 `BUILD.gn` 包含 |
| `bundle.json` | 组件配置，描述产物与依赖 |

## Targets 清单

### 1. utils_lite（组件聚合）

**证据来源**：`BUILD.gn` 第 27-30 行

```gn
lite_component("utils_lite") {
    features = [ ":graphic_utils_lite" ]
    public_deps = features
}
```

| 属性 | 值 |
|------|-----|
| 类型 | lite_component |
| 依赖 | `graphic_utils_lite` |

### 2. graphic_utils_lite（核心库）

**证据来源**：`BUILD.gn` 第 38-102 行

```gn
lite_library("graphic_utils_lite") {
    # 根据内核类型确定库类型
    if (ohos_kernel_type == "liteos_m") {
        target_type = "static_library"     # 静态库
    } else {
        target_type = "shared_library"    # 动态库
    }
    # ...
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | lite_library | 根据 `ohos_kernel_type` 动态确定 |
| 静态库输出 | `libgraphic_utils.a` | liteos_m 系统 |
| 动态库输出 | `libgraphic_utils.so` | 其他系统 |
| ROM 占用 | ~450KB | `bundle.json` |
| RAM 占用 | ~50KB | `bundle.json` |

#### Sources 列表

**证据来源**：`BUILD.gn` 第 66-91 行

| 模块 | 文件 |
|------|------|
| 核心工具 | `color.cpp`, `geometry2d.cpp`, `graphic_math.cpp`, `style.cpp` |
| 变换 | `trans_affine.cpp`, `transform.cpp` |
| 内存 | `mem_api.cpp` |
| 格式 | `pixel_format_utils.cpp` |
| 系统 | `version.cpp` |
| 计时 | `graphic_timer.cpp`, `hal_tick.cpp`, `hal_cpu.cpp` |
| 性能 | `graphic_performance.cpp` |
| Diagram common | `diagram/common/paint.cpp` |
| Diagram depiction | `diagram/depiction/depict_curve.cpp` |
| Diagram rasterizer | `diagram/rasterizer/*.cpp` (3 个文件) |
| Diagram vertexgen | `diagram/vertexgenerate/*.cpp` (2 个文件) |
| Diagram vertexprim | `diagram/vertexprimitive/*.cpp` (4 个文件) |

#### Include 目录

**证据来源**：`BUILD.gn` 第 44 行、第 105-109 行

| 目录 | 用途 |
|------|------|
| `frameworks/` | 框架源码目录 |
| `interfaces/innerkits/` | 内部 API 头文件 |
| `interfaces/kits/` | 外部 API 头文件 |
| `$product_path/graphic_config/` | 产品配置目录 |

#### Dependencies

**证据来源**：`BUILD.gn` 第 93-101 行

| 依赖类型 | 依赖项 | 来源 | 用途 |
|----------|--------|------|------|
| `deps` | `bounds_checking_function:libsec_static` | third_party | 静态库版本内存安全 |
| `deps` | `bounds_checking_function:libsec_shared` | third_party | 动态库版本内存安全 |
| `public_deps` | `hilog_lite:mini` | base/hiviewdfx | 日志输出 |
| `public_deps` | `hilog_lite:featured:hilog_shared` | base/hiviewdfx | 日志输出 |

#### Defines（Feature Flags）

**证据来源**：`BUILD.gn` 第 111-125 行 `graphic_utils_public_config`

| Define | 默认值 | 功能 |
|--------|--------|------|
| `GRAPHIC_ENABLE_LINECAP_FLAG` | 启用 | 线帽样式 |
| `GRAPHIC_ENABLE_LINEJOIN_FLAG` | 启用 | 线连接样式 |
| `GRAPHIC_ENABLE_ELLIPSE_FLAG` | 启用 | 椭圆 |
| `GRAPHIC_ENABLE_BEZIER_ARC_FLAG` | 启用 | 贝塞尔弧线 |
| `GRAPHIC_ENABLE_ARC_FLAG` | 启用 | 弧线 |
| `GRAPHIC_ENABLE_ROUNDEDRECT_FLAG` | 启用 | 圆角矩形 |
| `GRAPHIC_ENABLE_DASH_GENERATE_FLAG` | 启用 | 虚线生成 |
| `GRAPHIC_ENABLE_BLUR_EFFECT_FLAG` | 启用 | 模糊效果 |
| `GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG` | 启用 | 阴影效果 |
| `GRAPHIC_ENABLE_GRADIENT_FILL_FLAG` | 启用 | 渐变填充 |
| `GRAPHIC_ENABLE_PATTERN_FILL_FLAG` | 启用 | 图案填充 |
| `GRAPHIC_ENABLE_DRAW_IMAGE_FLAG` | 启用 | 图像绘制 |
| `GRAPHIC_ENABLE_DRAW_TEXT_FLAG` | 启用 | 文本绘制 |
| `ENABLE_OHOS_GRAPHIC_UTILS_PRODUCT_CONFIG` | 可配置 | 产品配置开关 |

### 3. graphic_utils_lite_ndk（NDK 包）

**证据来源**：`BUILD.gn` 第 32-36 行

```gn
ndk_lib("graphic_utils_lite_ndk") {
    lib_extension = ".so"
    deps = [ ":graphic_utils_lite" ]
    head_files = [ "interfaces/kits" ]
}
```

| 属性 | 值 |
|------|-----|
| 类型 | ndk_lib |
| 依赖 | `graphic_utils_lite` |
| 头文件 | `interfaces/kits/` |

### 4. lite_graphic_hals（HAL 组件，仅非 liteos_m）

**证据来源**：`BUILD.gn` 第 129-132 行

```gn
if (ohos_kernel_type != "liteos_m") {
    lite_component("lite_graphic_hals") {
        features = [ ":graphic_hals" ]
        public_deps = features
    }
}
```

### 5. graphic_hals（HAL 库，仅非 liteos_m）

**证据来源**：`BUILD.gn` 第 140-157 行

```gn
shared_library("graphic_hals") {
    sources = [
        "frameworks/hals/gfx_engines.cpp",
        "frameworks/hals/hi_fbdev.cpp",
    ]
    include_dirs = [ "//foundation/window/window_manager_lite/interfaces/innerkits" ]
    deps = [
        ":utils_lite",
        "//drivers/peripheral/display/hal:hdi_display",
    ]
    ldflags = [
        "-ldisplay_gfx",
        "-ldisplay_gralloc",
        "-ldisplay_layer",
    ]
}
```

| 属性 | 值 |
|------|-----|
| 类型 | shared_library |
| 输出 | `libgraphic_hals.so` |
| 依赖 | `utils_lite`, `hdi_display` |
| 链接参数 | `display_gfx`, `display_gralloc`, `display_layer` |

### 6. utils_lite（标准系统版本）

**证据来源**：`BUILD.gn` 第 177-186 行（os_level == "standard" 分支）

```gn
ohos_static_library("utils_lite") {
    sources = graphic_utils_sources  # 引用 utils.gni
    public_configs = [":graphic_utils_config"]
    external_deps = [ "bounds_checking_function:libsec_static" ]
    part_name = "graphic_utils_lite"
    subsystem_name = "graphic"
}
```

## 产物清单

### 编译产物

| 产物 | 类型 | 生成条件 |
|------|------|----------|
| `libgraphic_utils.so` | 动态库 | 非 liteos_m 系统 |
| `libgraphic_utils.a` | 静态库 | liteos_m 系统 |
| `libgraphic_hals.so` | 动态库 | 非 liteos_m 系统 |
| `libgraphic_utils_lite_ndk.so` | NDK 包 | 所有系统 |

### 预计输出目录

```
out/{product}/foundation/graphic/graphic_utils_lite/
├── libgraphic_utils.so           # Utils + Diagram 库
├── libgraphic_utils.a            # liteos_m 静态库
├── libgraphic_hals.so            # HAL 库（非 liteos_m）
└── libgraphic_utils_lite_ndk.so  # NDK 分发包
```

### 安装路径

| 产物 | 目标路径 |
|------|----------|
| `.so` 库 | `/system/lib64/` 或 `/system/lib/` |
| `.a` 库 | 链接时使用，不安装 |
| NDK | `/prebuilt/libs/` |

## 平台差异处理

### liteos_m vs 非 liteos_m

| 特性 | liteos_m | 非 liteos_m |
|------|----------|-------------|
| 库类型 | 静态库 `.a` | 动态库 `.so` |
| HAL | 不编译 graphic_hals | 编译 graphic_hals |
| 日志依赖 | `hilog_lite:mini` | `hilog_lite:featured:hilog_shared` |
| 内存安全 | `libsec_static` | `libsec_shared` |

**证据来源**：`BUILD.gn` 第 39-43 行、第 93-101 行

### 工具链差异

**证据来源**：`BUILD.gn` 第 45-59 行

| 工具链类型 | 处理方式 |
|------------|----------|
| ICCARM | 添加特定 CFLAGS 警告抑制 |
| Clang | 添加 `-Wno-float-equal` |

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                         依赖关系图                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              graphic_utils_lite (utils_lite)           │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ Sources: color.cpp, geometry2d.cpp, ...          │   │   │
│  │  │                                               │   │   │
│  │  │ Defines: GRAPHIC_ENABLE_*_FLAG (12 个)          │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                        │                               │   │
│  │          ┌─────────────┼─────────────┐                │   │
│  │          ↓             ↓             ↓                │   │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────────┐       │   │
│  │  │hilog_lite │ │bounds_chk │ │ graphic_config│       │   │
│  │  │  (日志)   │ │ (内存安全)│ │  (包含目录)    │       │   │
│  │  └───────────┘ └───────────┘ └───────────────┘       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                   │
│                            ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              graphic_hals (仅非 liteos_m)               │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ Sources: gfx_engines.cpp, hi_fbdev.cpp          │   │   │
│  │  │                                               │   │   │
│  │  │ Deps: utils_lite, hdi_display                  │   │   │
│  │  │ Ldflags: -ldisplay_gfx, -ldisplay_gralloc      │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 构建配置参数

### 可配置参数

| 参数 | 类型 | 默认值 | 用途 |
|------|------|--------|------|
| `graphic_utils_lite_enable_ohos_product_config` | bool | false | 启用产品配置 |
| `enable_graphic_dualcore` | bool | false | 启用双核支持（定义 `HAL_CPU_NUM=2`） |

### Feature Flags 控制

通过修改 `BUILD.gn` 第 111-125 行的 defines 可禁用不需要的功能以减小 ROM 占用：

```
# 禁用模糊和阴影效果
# defines -= [ "GRAPHIC_ENABLE_BLUR_EFFECT_FLAG" ]
# defines -= [ "GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG" ]
```

---

*最后更新时间：2026-02-06*
