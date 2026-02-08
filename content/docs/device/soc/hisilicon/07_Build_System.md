# 构建系统

本文档描述 `device_soc_hisilicon` 仓库使用的 GN 构建系统配置。

## 构建系统概览

| 项目 | 说明 |
|------|------|
| 构建工具 | GN (Generate Ninja) + Ninja |
| 根配置文件 | `ohos.build` |
| 构建入口 | `//build.sh` 或 `hb` (OpenHarmony Build) |
| 主要语言 | C / C++ |

## 根构建配置

### ohos.build

**路径**: `ohos.build`

**代码证据**:

```json
{
  "subsystem": "vendor",
  "parts": {
    "hardware": {
      "module_list": [
        "//device/soc/hisilicon/common/hal/media:hardware_media_sdk"
      ]
    },
    "hi3861_sdk": {
      "module_list": [
        "//device/soc/hisilicon/hi3861v100/sdk_liteos:wifiiot_sdk"
      ]
    },
    "middleware": {
      "module_list": [
        "//device/soc/hisilicon/common/hal/middleware:middleware_source_sdk"
      ]
    }
  }
}
```

## 芯片级构建配置

### Hi3861V100 构建

**路径**: `hi3861v100/BUILD.gn`

```gn
import("//build/ohos.gni")

ohos_module("hisilicon_hi3861v100") {
  features = [
    ":sdk_liteos",
    ":hi3861_adapter",
  ]
}
```

**代码证据**: `hi3861v100/BUILD.gn`

### SDK LiteOS 构建

**路径**: `hi3861v100/sdk_liteos/BUILD.gn`

```gn
import("//build/ohos.gni")

ohos_executable("wifiiot_sdk") {
  sources = [
    # 源文件列表
  ]
  
  include_dirs = [
    "include/",
    "//device/soc/hisilicon/hi3861v100/hi3861_adapter/hals/iot_hardware/wifiiot_lite/inc",
    "//device/soc/hisilicon/hi3861v100/hi3861_adapter/kal/cmsis",
    "//device/soc/hisilicon/hi3861v100/sdk_liteos/boot/flashboot/include",
  ]
  
  deps = [
    "//device/soc/hisilicon/hi3861v100/hi3861_adapter:hals",
    "//device/soc/hisilicon/hi3861v100/hi3861_adapter:kal",
    "//third_party/mbedtls:mbedcrypto",
  ]
  
  cflags = [
    "-Wall",
    "-Werror",
  ]
  
  configs = [
    ":sdk_config",
  ]
}

config("sdk_config") {
  defines = [
    "LOSCFG_PLATFORM_HI3861V100",
    "LOSCFG_KERNEL_CPUP",
  ]
  
  include_dirs = [
    "//device/soc/hisilicon/hi3861v100/sdk_liteos/include",
    "//device/soc/hisilicon/hi3861v100/sdk_liteos/config",
  ]
}
```

### WS63V100 构建

**路径**: `ws63v100/BUILD.gn`

```gn
import("//build/ohos.gni")

ohos_module("hisilicon_ws63v100") {
  features = [
    ":sdk",
    ":adapter",
  ]
}
```

## HAL 构建配置

### 通用 HAL 构建

**路径**: `common/hal/BUILD.gn`

```gn
hal_subsystem("hisilicon_hal") {
  parts = [
    "ai",
    "display",
    "media",
    "middleware",
    "multimedia",
    "update",
    "usb",
  ]
}
```

### Media HAL 构建

**路径**: `common/hal/media/BUILD.gn`

```gn
import("//build/ohos.gni")

ohos_shared_library("hdi_media") {
  sources = [
    "src/**/*.cpp",
  ]
  
  include_dirs = [
    "inc/",
    "//drivers/framework/ability/hdi_adapter/include",
    "//third_party/cjson",
  ]
  
  deps = [
    "//drivers/framework/ability/hdi_adapter:hdi_header",
    "//third_party/cjson:cjson",
  ]
  
  external_shared_libs = [
    "libhilog.so",
    "libhdi_drm.so",
  ]
  
  visibility = [
    "//device/soc/hisilicon/*",
  ]
}
```

## 平台驱动构建

### 平台顶层构建

**路径**: `common/platform/BUILD.gn`

```gn
import("//build/ohos.gni")

config("platform_config") {
  include_dirs = [
    "inc/",
    "include/",
  ]
}

# 静态库定义
static_library("platform_gpio") {
  sources = [
    "gpio/src/hi_gpio.c",
  ]
  
  public_configs = [
    ":platform_config",
  ]
}

# 其他驱动模块...
```

### Kconfig

**路径**: `common/platform/Kconfig`

提供内核配置菜单：

```kconfig
config DRIVER_GPIO
    bool "GPIO Support"
    default y
    help
      Enable GPIO driver support

config DRIVER_I2C
    bool "I2C Support"
    default y
    help
      Enable I2C driver support
```

## 构建产物配置

### 目标类型

| 类型 | GN 模板 | 输出 |
|------|---------|------|
| 可执行文件 | `ohos_executable` | `.elf`, `.bin` |
| 静态库 | `static_library` | `.a` |
| 动态库 | `ohos_shared_library` | `.so` |
| 内核模块 | `ohos_kernel_module` | `.ko` |

### 配置选项

| 配置项 | 说明 |
|--------|------|
| `sources` | 源文件列表 |
| `include_dirs` | 包含路径 |
| `defines` | 宏定义 |
| `cflags` / `cflags_cc` | 编译选项 |
| `deps` | 内部依赖 |
| `external_deps` | 外部依赖 |
| `configs` | 配置集合 |
| `public_configs` | 公开配置 |
| `libs` | 链接库 |
| `lib_dirs` | 库搜索路径 |

## 构建示例

### 构建命令

```bash
# 使用 hb 构建
hb set
hb build

# 使用 gn + ninja
gn gen out/hispark_pegasus
ninja -C out/hispark_pegasus
```

### 指定芯片

```bash
# Hi3861V100
hb set --product hispark_pegasus

# Hi3516DV300
hb set --product hispark_taurus
```

## 配置文件说明

### config.gni

**路径**: `{chip}/sdk_linux/config.gni`

```gn
# 芯片配置
chip_family = "hi35xx"
chip_name = "hi3516dv300"

# 编译配置
target_cpu = "arm"
target_os = "linux"

# 工具链配置
Clang_prefix = "arm-linux-gnueabi-"
```

### soc.gni

**路径**: `{chip}/soc.gni`

```gn
# SoC 配置
import("//vendor/hisilicon/{board}/config.gni")

# Include 路径
include_dirs = [
  "//device/soc/hisilicon/{chip}/sdk_linux/include",
  "//device/soc/hisilicon/common/platform/include",
]
```

## 相关文档

- 编译产物: [08_Products.md](08_Products.md)
- SDK 架构: [06_SDK_Architecture.md](06_SDK_Architecture.md)
