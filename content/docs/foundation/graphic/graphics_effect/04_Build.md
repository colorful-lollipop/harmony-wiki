# graphics_effect 构建指南

## 构建系统

graphics_effect 使用 **GN (Generate Ninja)** 作为构建系统，是 OpenHarmony 项目的标准构建工具。

**证据来源**: `CLAUDE.md:9-11` - "This project uses GN (Generate Ninja) as its build system"

---

## 关键配置文件

| 文件 | 用途 |
|-----|------|
| `BUILD.gn` | 主要构建入口，定义所有 targets |
| `config.gni` | 配置参数和平台标志 |
| `bundle.json` | 部件配置，定义对外接口 |

---

## GN Targets 详解

### 主要 Target: graphics_effect_core

**定义位置**: `BUILD.gn:191-210`

```gn
ohos_shared_library("graphics_effect_core") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  public_configs = [ ":export_config" ]
  deps = [ ":graphics_effect_src" ]
  external_deps = [
    "bounds_checking_function:libsec_shared",
    "hilog:libhilog",
  ]
  output_name = "graphics_effect"
  part_name = "graphics_effect"
  subsystem_name = "graphic"
}
```

| 属性 | 值 | 说明 |
|-----|------|------|
| 类型 | shared_library | 共享库 (.so) |
| 输出名 | libgraphics_effect.so | 产物名称 |
| CFI | 启用 | Control Flow Integrity |
| PAC/RET | 启用 | Pointer Authentication |

### Source Set: graphics_effect_src

**定义位置**: `BUILD.gn:22-170`

**编译选项**:

| 选项 | 值 | 说明 |
|-----|------|------|
| cflags | `-Wall`, `-O2`, `-ftrapv`, `-D_FORTIFY_SOURCE=2` | C 编译标志 |
| cflags_cc | `-std=c++17`, `-fvisibility=hidden` | C++ 编译标志 |
| include_dirs | `["include"]` | 头文件搜索路径 |
| outputs | 静态库 | intermediate 产物 |

**源文件列表** (约 70 个 .cpp 文件):

```gn
sources = [
  # 核心模块
  "src/ge_render.cpp",
  "src/ge_visual_effect.cpp",
  "src/ge_visual_effect_container.cpp",
  # 模糊效果
  "src/ge_kawase_blur_shader_filter.cpp",
  "src/ge_mesa_blur_shader_filter.cpp",
  "src/ge_linear_gradient_blur_shader_filter.cpp",
  # ... 更多文件
]
```

**证据来源**: `BUILD.gn:56-128`

### Feature Target: libgraphics_effect (ArkUI X)

**定义位置**: `BUILD.gn:173-189`

当 `is_arkui_x = true` 时，构建为 `libgraphics_effect`（静态库）。

---

## 依赖配置

### 外部依赖 (external_deps)

| 依赖 | 用途 | 条件 |
|-----|------|------|
| `graphic_2d:2d_graphics` | 2D 绘图 API | 默认 |
| `hilog:libhilog` | 日志 | 默认 |
| `bounds_checking_function:libsec_shared` | 安全函数 | 默认 |
| `c_utils:utils` | 工具库 | OHOS |
| `init:libbegetutil` | 系统初始化 | OHOS && !build_ohos_sdk |
| `hitrace:hitrace_meter` | 性能追踪 | !build_ohos_sdk |

**证据来源**: `BUILD.gn:132-151`

### 内部依赖 (deps)

| 依赖 | 类型 | 用途 |
|-----|------|------|
| `:utils` | group | 工具库适配 |
| `:graphics_effect_src` | source_set | 源码编译 |

---

## 平台配置

### config.gni 参数

```gn
declare_args() {
  graphics_effect_feature_upgrade_skia = false  # Skia 版本升级
}
```

### 平台标志

| 标志 | 定义 | 用途 |
|-----|------|------|
| `ge_is_ohos` | `current_os == "ohos"` | OpenHarmony 平台 |
| `ge_is_linux` | `current_os == "linux"` | Linux 平台 |
| `ge_is_mac` | `current_os == "mac"` | macOS 平台 |
| `ge_is_win` | `current_os == "win"\|"mingw"` | Windows 平台 |
| `ge_is_ios` | `current_os == "ios"\|"tvos"` | iOS 平台 |

**证据来源**: `config.gni:21-25`

### 编译宏定义

| 宏 | 条件 | 用途 |
|-----|------|------|
| `GE_OHOS` | `current_os == "ohos"` | OpenHarmony 平台 |
| `GE_PLATFORM_UNIX` | `ge_is_ohos \|\| ge_is_linux` | Unix-like 平台 |
| `USE_M133_SKIA` | `is_arkui_x && graphics_effect_feature_upgrade_skia` | M133+ Skia |

**证据来源**: `BUILD.gn:153-165`

---

## 编译产物

### 产物清单

| 产物 | 类型 | 位置 | 说明 |
|-----|------|------|------|
| `libgraphics_effect.so` | 共享库 | 系统库目录 | 主要产物 |
| `libgraphics_effect.a` | 静态库 | 中间产物 | ArkUI X 构建 |
| 头文件 | - | `include/` | 对外接口 |

### 产物规格

| 属性 | 值 |
|-----|------|
| 架构 | arm64-v8a, armeabi-v7a, x86_64, etc. |
| 链接方式 | 动态链接 |
| 依赖 | libc++, libhilog, libsec_shared |

### 安装路径

**OpenHarmony 系统**:

```
/system/lib64/module/libgraphics_effect.z.so
```

或

```
/system/lib/module/libgraphics_effect.z.so
```

**证据来源**: `bundle.json:36-47` - inner_kits 配置

---

## 构建命令

### 标准构建

```bash
# 构建 graphics_effect 核心库
./build.sh --product-name <product> --build-target graphics_effect:graphics_effect_core
```

### 带测试构建

```bash
# 构建并运行单元测试
./build.sh --product-name <product> --build-target graphics_effect:GraphicsEffectTest
```

### 构建 Fuzz 测试

```bash
./build.sh --product-name <product> --build-target graphics_effect:fuzztest
```

**证据来源**: `CLAUDE.md:13-31`

---

## 构建变体

### 按平台构建

```bash
# Linux 平台
./build.sh --product-name linux --build-target graphics_effect:graphics_effect_core

# OHOS 平台
./build.sh --product-name ohos --build-target graphics_effect:graphics_effect_core
```

### 按配置构建

```bash
# 启用 Skia M133+
./build.sh ... --enable-feature graphics_effect_feature_upgrade_skia
```

---

## 编译选项详解

### 安全加固选项

| 选项 | 值 | 作用 |
|-----|------|------|
| `-D_FORTIFY_SOURCE=2` | 启用 | 运行时缓冲区溢出检测 |
| `-ftrapv` | 启用 | 整数溢出陷阱 |
| `-fvisibility=hidden` | 启用 | 隐藏符号导出 |
| `-fdata-sections` | 启用 | 数据段分离 |
| `-ffunction-sections` | 启用 | 函数段分离 |
| `branch_protector_ret` | `"pac_ret"` | PAC 返回地址保护 |
| `cfi` | true | Control Flow Integrity |

**证据来源**: `BUILD.gn:24-51`

### 优化选项

| 选项 | 值 | 作用 |
|-----|------|------|
| `-O2` | 启用 | 级别 2 优化 |
| `-FPIC` | 启用 | 位置无关代码 |
| `-FS` | 启用 | 直接链接 |

---

## 部件配置 (bundle.json)

```json
{
  "name": "@ohos/graphics_effect",
  "component": {
    "name": "graphics_effect",
    "subsystem": "graphic",
    "features": ["graphics_effect_feature_upgrade_skia"],
    "build": {
      "sub_component": [
        "//foundation/graphic/graphics_effect:graphics_effect_core"
      ],
      "inner_kits": [
        {
          "type": "so",
          "name": "//foundation/graphic/graphics_effect:graphics_effect_core",
          "header": {
            "header_base": "//foundation/graphic/graphics_effect/include"
          }
        }
      ]
    }
  }
}
```

**证据来源**: `bundle.json`

---

## 相关文档

- [项目概述](01_Overview.md) - 定位和核心能力
- [架构说明](02_Architecture.md) - 详细架构设计
- [问题排查](06_Troubleshooting.md) - 构建问题
