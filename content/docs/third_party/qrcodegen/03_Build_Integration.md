# 03_构建适配

## 3.1 构建系统概述

qrcodegen 库使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统，配置文件为 `BUILD.gn`。

### 支持的系统类型

| 系统类型 | 支持状态 | 说明 |
|----------|----------|------|
| **standard** | ✅ 支持 | 标准系统（手机、平板等） |
| **small** | ✅ 支持 | 小型系统（智能手表等） |
| **mini** | ✅ 支持 | 轻量系统（智能家居等） |

### 架构支持

- **ARM64** (默认)
- **ARM32**
- **x86/x64** (模拟器)

## 3.2 构建目标

本库提供 **两个**构建目标：

### 目标一：qrcodegen_static

**用途**：通用静态库，不包含 OH 特定适配

**定义位置**：`BUILD.gn:51-65`

```gn
ohos_static_library("qrcodegen_static") {
  visibility = [
    ":*",
    "//vendor/huawei/domains/iotdev/home_host_service/services/touchui/business:touchui",
    "//foundation/arkui/ui_ext_lite/tools/ide:graphic_lite",
    "//foundation/arkui/ui_lite:ui",
    "//foundation/arkui/ui_lite/ext/ide:ui_ide",
    "//out/*",
    "//vendor/hisi/confidential/contexthub/src/framework/hisi/ui/third_party/*",
  ]
  sources = [ "cpp/qrcodegen.cpp" ]
  include_dirs = [ "//third_party/qrcodegen/cpp" ]
  configs = [ ":qrcodegen_config" ]
  public_configs = [ ":libqrcodegen_config" ]
}
```

**可见性限制**：仅对指定模块可见，不对全局开放。

### 目标二：ace_engine_qrcode

**用途**：AceEngine 专用静态库，包含 `ACE_ENGINE_QRCODE_ABLE` 适配

**定义位置**：`BUILD.gn:66-83`

```gn
if (qrcodegen_feature_ace_engine_qrcode_able) {
  config("ace_engine_qrcode_config") {
    include_dirs = [ "//third_party/qrcodegen/cpp" ]
    defines = [ "ACE_ENGINE_QRCODE_ABLE" ]
    cflags = [
      "-Wall",
      "-Wno-reorder",
    ]
    cflags_cc = cflags
  }

  ohos_static_library("ace_engine_qrcode") {
    sources = [ "cpp/qrcodegen.cpp" ]
    public_configs = [ ":ace_engine_qrcode_config" ]
    subsystem_name = "thirdparty"
    part_name = "qrcodegen"
  }
}
```

**编译定义**：`ACE_ENGINE_QRCODE_ABLE`

**包含的适配**：所有 OH 特定代码分支（见 [02_适配分析](02_Patches.md)）

## 3.3 配置详解

### libqrcodegen_config

**用途**：公共头文件搜索路径配置

```gn
config("libqrcodegen_config") {
  include_dirs = [ "cpp" ]
}
```

### qrcodegen_config

**用途**：通用构建选项

```gn
config("qrcodegen_config") {
  cflags = [
    "-Wall",           # 启用所有警告
    "-fexceptions",    # 启用 C++ 异常（仅 qrcodegen_static）
  ]
  cflags_cc = cflags
}
```

### ace_engine_qrcode_config

**用途**：AceEngine 专用构建选项

```gn
config("ace_engine_qrcode_config") {
  include_dirs = [ "//third_party/qrcodegen/cpp" ]
  defines = [ "ACE_ENGINE_QRCODE_ABLE" ]  # 关键：启用 OH 适配
  cflags = [
    "-Wall",
    "-Wno-reorder",    # 抑制特定警告
  ]
  cflags_cc = cflags
}
```

## 3.4 特性开关

### qrcodegen_feature_ace_engine_qrcode_able

**类型**：Boolean

**默认值**：`true`

**定义位置**：`BUILD.gn:4`

```gn
declare_args() {
  qrcodegen_feature_ace_engine_qrcode_able = true
}
```

**控制**：是否构建 `ace_engine_qrcode` 目标

**使用场景**：禁用该特性可减少固件大小（如果不需要 QRCode 组件）

## 3.5 轻量级系统支持

### ohos_lite 分支

**用途**：支持 liteos_m 内核的轻量级设备

```gn
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")

  lite_library("qrcodegen") {
    if (ohos_kernel_type == "liteos_m") {
      target_type = "static_library"
    } else {
      target_type = "shared_library"
    }
    sources = [ "cpp/qrcodegen.cpp" ]
    include_dirs = [ "//third_party/qrcodegen/cpp" ]
    # ...
  }
}
```

### ICCARM 工具链支持

```gn
if (defined(board_toolchain_type) && board_toolchain_type == "iccarm") {
  cflags = [
    "--diag_suppress",
    "Pe366",  # 抑制特定警告
  ]
  cflags_cc = cflags
} else {
  cflags = [ "-Wall" ]
  cflags_cc = cflags
}
```

## 3.6 依赖关系

### 输入依赖

| 依赖类型 | 依赖项 | 说明 |
|----------|--------|------|
| 源文件 | `cpp/qrcodegen.cpp` | 唯一源文件 |
| 头文件 | `cpp/qrcodegen.hpp` | 随源文件自动包含 |

### 输出依赖者

| 依赖模块 | 构建目标 | 用途 |
|----------|----------|------|
| ace_engine | `//foundation/arkui/ace_engine/...` | QRCode 组件 |
| ui_lite | `//foundation/arkui/ui_lite/...` | 轻量级 UI |

## 3.7 构建示例

### 构建整个库

```bash
# 标准系统
hb build -f

# 查看构建产物
ls out/ohos-arm64/obj/third_party/qrcodegen/
```

### 仅构建 qrcodegen

```bash
# 单独构建
hdc shell
hilog
```

### 查看构建配置

```bash
# 查看可用的 GN 目标
gn desc //third_party/qrcodegen targets

# 查看配置详情
gn desc //third_party/qrcodegen:ace_engine_qrcode config ace_engine_qrcode_config
```

## 3.8 常见问题

### Q1: 编译报错 "undefined reference to ..."

**可能原因**：链接了错误的构建目标

**解决方案**：
- AceEngine QRCode 组件应链接 `//third_party/qrcodegen:ace_engine_qrcode`
- 其他模块应链接 `//third_party/qrcodegen:qrcodegen_static`

### Q2: 运行时异常崩溃

**可能原因**：使用了 `qrcodegen_static` 但代码期望 `ACE_ENGINE_QRCODE_ABLE`

**解决方案**：确认依赖关系正确，QRCode 组件必须依赖 `ace_engine_qrcode`

### Q3: 如何禁用 ace_engine_qrcode 目标

**方法**：设置 build.gn.args

```bash
# 在 product 配置中
enable_feature = false
```

或修改 `BUILD.gn`：

```gn
declare_args() {
  qrcodegen_feature_ace_engine_qrcode_able = false
}
```

## 3.9 构建产物

### 静态库文件

| 系统类型 | 文件路径 | 说明 |
|----------|----------|------|
| standard | `out/ohos-arm64/obj/third_party/qrcodegen/libace_engine_qrcode.a` | AceEngine 专用 |
| standard | `out/ohos-arm64/obj/third_party/qrcodegen/libqrcodegen_static.a` | 通用版本 |

### 符号差异

```
# ace_engine_qrcode (ACE_ENGINE_QRCODE_ABLE 定义)
$ nm libace_engine_qrcode.a | grep -E "(QrCode|QrSegment|BitBuffer)"
00000000 T QrCode::QrCode(int, QrCode::Ecc, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, int)
00000000 T QrCode::QrCode(int, QrCode::Ecc, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, int)
...

# qrcodegen_static (无 ACE_ENGINE_QRCODE_ABLE)
$ nm libqrcodegen_static.a | grep -E "(QrCode|QrSegment|BitBuffer)"
00000000 T QrCode::encodeText(char const*, QrCode::Ecc)
00000000 T QrCode::encodeSegments(...)
...
```

**注意**：可见性差异导致符号表不同。
