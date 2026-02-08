# GN 构建配置

## 构建系统概述

本仓库使用 **GN (Generate Ninja)** 作为构建系统，配合 OpenHarmony 的 `build/ohos.gni` 和内核的 `liteos.gni` 使用。

### GN 文件分布

| 类型 | 路径模式 | 用途 |
|------|----------|------|
| 根构建 | `/BUILD.gn` | 顶层模块组 |
| 板级配置 | `*/config.gni` | 芯片/工具链定义 |
| 设备配置 | `*/device.gni` | SoC/产品配置 |
| 子模块 | `*/BUILD.gn` | 各模块 target 定义 |

---

## 根构建配置

**文件**: `/BUILD.gn`

```gn
if (ohos_kernel_type == "liteos_m") {
  import("//kernel/liteos_m/liteos.gni")
  module_name = get_path_info(rebase_path("."), "name")
  module_group(module_name) {
    modules = [
      "neptune100",
      "shields",
    ]
  }
}
```

**证据**: `/BUILD.gn:14-22`

**说明**:
- 仅在 `liteos_m` 内核时加载根模块组
- 包含 `neptune100` 和 `shields` 两个模块

---

## Neptune100 构建 (LiteOS-M)

### 目录构建

**文件**: `/neptune100/BUILD.gn`

```gn
if (ohos_kernel_type == "liteos_m") {
  import("//kernel/liteos_m/liteos.gni")
  module_name = get_path_info(rebase_path("."), "name")
  module_group(module_name) {
    modules = [ "liteos_m" ]
  }
}
```

**证据**: `/neptune100/BUILD.gn:14-20`

### LiteOS-M 模块

**文件**: `/neptune100/liteos_m/BUILD.gn`

```gn
import("//kernel/liteos_m/liteos.gni")
module_name = get_path_info(rebase_path("."), "name")
module_switch = defined(LOSCFG_BOARD_NEPTUNE100)
kernel_module(module_name) {
  # deps = [ "$product_path/hdf_config:hdf_hcs" ]
}
```

**证据**: `/neptune100/liteos_m/BUILD.gn:14-20`

### 板级配置

**文件**: `/neptune100/liteos_m/config.gni`

```gn
board_arch = "ck803"
board_cpu = "ck804ef"
board_toolchain_type = "gcc"
```

**证据**: `/neptune100/liteos_m/config.gni:15-18`

---

## RK3568 构建 (Linux)

### 目录构建

**文件**: `/rk3568/BUILD.gn`

```gn
import("//build/ohos.gni")
import("device.gni")

print("rk3568_group in")
group("rk3568_group") {
  deps = [
    "cfg:init_configs",
    "distributedhardware:distributedhardware",
    "kernel:kernel",
    "updater:updater_files",
    "//device/soc/rockchip/rk3568/hardware:hardware_group",
  ]
  if (is_support_graphic) {
    deps += [
      "//device/soc/rockchip/${device_name}/hardware/display:display_buffer_model",
      "//device/soc/rockchip/${device_name}/hardware/display:display_composer_model",
    ]
  }
  if (is_support_codec) {
    deps += [
      "//device/soc/rockchip/rk3568/hardware/codec:codec_oem_interface",
      "//device/soc/rockchip/rk3568/hardware/omx_il:lib_omx",
    ]
  }
  if (is_support_boot_animation) {
    deps += [ "bootanimation:bootanimation" ]
  }
}
```

**证据**: `/rk3568/BUILD.gn:14-42`

### 设备配置

**文件**: `/rk3568/device.gni`

```gn
soc_company = "rockchip"
soc_name = "rk3568"

import("//device/soc/${soc_company}/${soc_name}/soc.gni")
import("//build/ohos.gni")

declare_args() {
  is_support_boot_animation = true
  is_support_graphic = true
  is_support_codec = true
}

is_support_v4l2 = true
if (is_support_v4l2) {
  is_support_mpi = false
  defines += [ "SUPPORT_V4L2" ]
}
```

**证据**: `/rk3568/device.gni:14-45`

### 板级配置

**文件**: `/rk3568/config.gni`

```gn
board_arch = "armv8-a"
board_cpu = "cortex-a55"
board_toolchain_type = "clang"
board_fpu = "neon-fp-armv8"
```

**证据**: `/rk3568/config.gni:15-18`

---

## DAYU210 构建 (RK3588)

### 目录构建

**文件**: `/dayu210/BUILD.gn`

```gn
import("//build/ohos.gni")
import("device.gni")

print("dayu210_group in")
group("dayu210_group") {
  deps = [
    "cfg:init_configs",
    "distributedhardware:distributedhardware",
    "kernel:kernel",
    "updater:updater_files",
    "//device/soc/rockchip/rk3588/hardware:hardware_group",
  ]
  # ... 类似的条件依赖
}
```

**证据**: `/dayu210/BUILD.gn:14-42`

### 设备配置

**文件**: `/dayu210/device.gni`

```gn
soc_company = "rockchip"
soc_name = "rk3588"
```

**证据**: `/dayu210/device.gni:14-15`

### 板级配置

**文件**: `/dayu210/config.gni`

```gn
board_arch = "armv8-a"
board_cpu = "cortex-a55"
board_toolchain_type = "clang"
board_fpu = "neon-fp-armv8"
```

**证据**: `/dayu210/config.gni:15-18`

---

## HDF 驱动构建

### neptune100 shields

**文件**: `/shields/neptune100/BUILD.gn`

```gn
if (ohos_kernel_type == "liteos_m") {
  import("//drivers/hdf_core/adapter/khdf/liteos_m/hdf.gni")
  module_name = get_path_info(rebase_path("."), "name")
  hdf_driver(module_name) {
    hcs_sources = [ "neptune100.hcs" ]
    visibility = [ "$device_path", "." ]
  }
}
```

**证据**: `/shields/neptune100/BUILD.gn:14-25`

### hcs 配置

**文件**: `/hcs/BUILD.gn`

```gn
if (ohos_kernel_type == "liteos_m") {
  import("//drivers/hdf_core/adapter/khdf/liteos_m/hdf.gni")
  module_name = get_path_info(rebase_path("."), "name")
  hdf_driver(module_name) {
    hcs_sources = [ "neptune100.hcs" ]
  }
}
```

**证据**: `/hcs/BUILD.gn:14-25`

---

## Target 类型汇总

| Target 类型 | 模块 | 用途 | 示例 |
|------------|------|------|------|
| `module_group` | neptune100 | LiteOS-M 模块分组 | `module_group("neptune100")` |
| `kernel_module` | neptune100/liteos_m | 内核模块 | `kernel_module("liteos_m")` |
| `hdf_driver` | shields/hcs | HDF 驱动 | `hdf_driver("neptune100")` |
| `group` | rk3568/dayu210 | 标准系统模块分组 | `group("rk3568_group")` |
| `action` | kernel | 自定义构建动作 | `build_kernel.sh` |
| `ohos_prebuilt_etc` | cfg/updater | 预配置文件 | `init_configs`, `updater_files` |

---

## Feature Flags

### 条件编译变量

| 变量 | 默认值 | 用途 |
|------|--------|------|
| `is_support_boot_animation` | true | 开机动画 |
| `is_support_graphic` | true | 图形显示 |
| `is_support_codec` | true | 硬件编解码 |
| `is_support_v4l2` | true | V4L2 摄像头支持 |

### 宏定义

| 宏 | 定义位置 | 用途 |
|-----|----------|------|
| `SUPPORT_V4L2` | `device.gni:44` | V4L2 摄像头支持 |

---

## 依赖关系图

```
rk3568_group / dayu210_group
├── cfg:init_configs                    # 初始化配置
├── distributedhardware:distributedhardware  # 分布式硬件
├── kernel:kernel                       # Linux 内核
├── updater:updater_files               # 升级器
├── bootanimation:bootanimation         # (条件: is_support_boot_animation)
├── //device/soc/rockchip/.../hardware  # SoC 硬件
├── display:display_buffer_model        # (条件: is_support_graphic)
├── display:display_composer_model     # 显示合成
└── codec:codec_oem_interface          # (条件: is_support_codec)
```

---

## 编译命令

### DAYU210
```bash
./build.sh --product-name dayu210
```

### RK3568 (DAYU200)
```bash
hb set    # 选择 hihope -> rk3568
hb build -f
```

### 产物路径
```
out/rk3588/packages/phone/images/   # DAYU210
out/rk3568/packages/phone/images/  # RK3568
```

---

## 相关文档

- [03_Architecture](03_Architecture.md) - 架构说明
- [05_Board_Configurations](05_Board_Configurations.md) - 开发板配置
- [appendix/Config_Flags](appendix/Config_Flags.md) - 配置参数速查
