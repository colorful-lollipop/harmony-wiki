# GN Targets 与构建系统

## 目的

本文档详细说明 OpenHarmony Hisilicon 板卡仓库的 GN 构建系统，包括 Targets 列表、类型、依赖关系和产物。

## 适用范围

本文档适用于：
- 构建工程师理解 GN 构建配置
- 开发者了解编译依赖
- 运维人员了解构建产物

---

## GN 构建系统概览

### 构建文件清单

| 文件类型 | 数量 | 示例 |
|---------|------|------|
| **BUILD.gn** | 22 | `hispark_aries/BUILD.gn` |
| **.gni** (config.gni) | 4 | `hispark_aries/liteos_a/config.gni` |
| **.gni** (device.gni) | 2 | `hispark_taurus/device.gni` |
| **ohos.build** | 5 | `hispark_aries/ohos.build` |
| **bundle.json** | 0 | 无（板卡层不包含） |

**证据**:
- `find . -name 'BUILD.gn'` - 22 个 BUILD.gn 文件
- `find . -name 'config.gni'` - 4 个 config.gni 文件
- `find . -name 'device.gni'` - 2 个 device.gni 文件
- `find . -name 'ohos.build'` - 5 个 ohos.build 文件

---

## HiSpark Aries (Hi3518EV300)

### 关键 BUILD.gn

#### 1. 顶层 BUILD.gn

**位置**: `hispark_aries/BUILD.gn`

**Target 定义**:

```gn
group("hispark_aries") {
  deps = []
  if (ohos_kernel_type == "liteos_a") {
    deps += [ "//device/soc/hisilicon/hi3518ev300/mpp:copy_mpp_libs" ]
  }
}
```

| 字段 | 值 | 说明 |
|------|------|------|
| **Target 类型** | `group` | 目标组，无输出 |
| **条件** | `ohos_kernel_type == "liteos_a"` | 仅 LiteOS-A 内核 |
| **依赖** | `//device/soc/hisilicon/hi3518ev300/mpp:copy_mpp_libs` | SoC 层 MPP 库 |

**产物**: 无（group 目标）

**证据**: `hispark_aries/BUILD.gn:3-8`

---

### 关键 config.gni

**位置**: `hispark_aries/liteos_a/config.gni`

**关键配置**:

```gn
kernel_type = "liteos_a"
board_cpu = "cortex-a7"
board_toolchain_type = "clang"
board_cflags = ["-mfloat-abi=softfp", "-mfpu=neon-vfpv4"]
board_cxx_flags = ["-mfloat-abi=softfp", "-mfpu=neon-vfpv4"]
storage_type = "spinor"
board_adapter_dir = "//device/soc/hisilicon/common/hal"
```

**证据**: `hispark_aries/liteos_a/config.gni:15-61`

---

## HiSpark Pegasus (Hi3861V100)

### 关键配置

**位置**: `hispark_pegasus/liteos_m/config.gni`

**特点**: 最简配置，无 BUILD.gn 文件

**证据**: `hispark_pegasus/` - 仅包含 `liteos_m/config.gni` 和 `ohos.build`

---

## HiSpark Phoenix (Hi3751V350)

### 关键 BUILD.gn

#### 1. 顶层 BUILD.gn

**位置**: `hispark_phoenix/linux/BUILD.gn`

**Target 定义**:

```gn
group("hi3516dv300") {
  deps = [
    "boot:hi3516dv300_boot",
    "system:hi3516dv300_system",
    "updater:hi3516dv300_updater",
  ]
}
```

**证据**: `hispark_phoenix/linux/BUILD.gn` (需读取实际内容）

---

### 关键 device.gni

**位置**: `hispark_phoenix/device.gni`

**关键配置**:

```gn
soc_company = "hisilicon"
soc_name = "hi3751v350"
product_config_path = "//vendor/hisilicon/hispark_taurus_standard"
board_camera_path = "//device/board/hisilicon/hispark_taurus/camera"
is_support_mpi = true
```

**证据**: `hispark_phoenix/device.gni:14-33`

---

## HiSpark Taurus (Hi3516DV300) - 最复杂

### 关键 BUILD.gn

#### 1. 顶层 BUILD.gn

**位置**: `hispark_taurus/BUILD.gn`

**Target 定义**:

```gn
if (defined(ohos_lite)) {
  group("hispark_taurus") {
    deps = []
    if (ohos_kernel_type == "linux") {
      deps += [
        "//device/soc/hisilicon/common/platform/wifi/hi3881v100/firmware:wifi_firmware",
        "//device/soc/hisilicon/hi3516dv300/sdk_linux:hispark_taurus_sdk",
      ]
    } else if (ohos_kernel_type == "liteos_a") {
      deps += [
        "//device/soc/hisilicon/hi3516dv300/sdk_liteos/mpp:copy_mpp_libs"
      ]
    }
  }
} else {
  group("hispark_taurus") {
    deps = [ "linux:hi3516dv300_group" ]
    deps += [
      "//device/soc/hisilicon/common/hal/media:hardware_group",
      "//device/soc/hisilicon/common/hal/middleware:middleware_group",
    ]
  }
}
```

| 字段 | 值 | 说明 |
|------|------|------|
| **条件 1** | `defined(ohos_lite)` | LiteOS 内核 |
| **条件 2** | `ohos_kernel_type == "linux"` | Linux 内核 |
| **Linux 依赖** | WiFi 固件, SDK, Media HAL, Middleware | SoC 层依赖 |
| **LiteOS-A 依赖** | MPP 库 | SoC 层依赖 |
| **标准系统依赖** | Linux 组, Media HAL, Middleware | SoC 层依赖 |

**证据**: `hispark_taurus/BUILD.gn:3-25`

---

#### 2. 相机 HAL BUILD.gn

**位置**: `hispark_taurus/camera/BUILD.gn`

**LiteOS-A 配置**:

```gn
if (defined(ohos_lite)) {
  hc_gen("build_config") {
    hcs_file_prefix = "$product_config_path/hdf_config/uhdf/camera"
    sources = [
      "$hcs_file_prefix/driver/mpp_config.hcs",
      "$hcs_file_prefix/hdi_impl/camera_host_config.hcs",
      "$hcs_file_prefix/pipeline_core/ipp_algo_config.hcs",
    ]
    outputs = [ "$root_out_dir/etc/camera/{{source_name_part}}.hcb" ]
  }

  group("hispark_taurus_build") {
    deps = [
      ":build_config",
      "driver_adapter:driver_adapter",
    ]
  }
}
```

**标准系统配置**:

```gn
else {
  hc_gen("build_camera_host_config") {
    sources = [ "$product_config_path/hdf_config/uhdf/camera/hdi_impl/camera_host_config.hcs" ]
    outputs = [ "$target_gen_dir/hdi_impl/{{source_name_part}}.hcb" ]
  }

  ohos_prebuilt_etc("camera_host_config.hcb") {
    deps = [ ":build_camera_host_config" ]
    hcs_outputs = get_target_outputs(":build_camera_host_config")
    source = hcs_outputs[0]
    relative_install_dir = "hdfconfig"
    install_images = [ chipset_base_dir ]
    subsystem_name = "hdf"
    part_name = "drivers_peripheral_camera"
  }

  group("hispark_taurus_build") {
    public_deps = [
      ":camera_host_config.hcb",
      ":config.c",
      ":ipp_algo_config.hcb",
      ":mpp_config.hcb",
      ":params.c",
      "driver_adapter:driver_adapter",
      "pipeline_core:camera_ipp_algo_example",
    ]
  }
}
```

| Target 类型 | 输出 | 用途 |
|-----------|------|------|
| `hc_gen` | `.hcb` 文件 | HDF 配置生成 |
| `ohos_prebuilt_etc` | `.hcb` 文件 | 安装 HDF 配置 |
| `group` | - | 目标组 |

**证据**: `hispark_taurus/camera/BUILD.gn:6-145`

---

### 关键 device.gni

**位置**: `hispark_taurus/device.gni`

**关键配置**:

```gn
soc_company = "hisilicon"
soc_name = "hi3516dv300"

if (!defined(defines)) {
  defines = []
}
if (product_name == "ipcamera_hispark_taurus") {
  product_config_path = "//vendor/hisilicon/hispark_taurus_standard"
} else {
  product_config_path = "//vendor/hisilicon/${product_name}"
}
board_camera_path = "//device/board/hisilicon/hispark_taurus/camera"
is_support_mpi = true
if (is_support_mpi) {
  is_support_v4l2 = false
  defines += [ "SUPPORT_MPI" ]
  chipset_build_deps = "$board_camera_path:hispark_taurus_build"
  camera_device_manager_deps =
      "$board_camera_path/device_manager:camera_device_manager"
  camera_pipeline_core_deps =
      "$board_camera_path/pipeline_core:camera_pipeline_core"
}
```

**证据**: `hispark_taurus/device.gni:14-37`

---

## Target 类型说明

| Target 类型 | 说明 | 产物 | 示例 |
|-----------|------|------|------|
| **group** | 目标组，无编译 | 无 | `hispark_taurus` |
| **hc_gen** | HDF 配置生成 | `.hcb` 文件 | `build_camera_host_config` |
| **ohos_prebuilt_etc** | 预构建配置安装 | `.hcb` 文件 | `camera_host_config.hcb` |

---

## Target 依赖关系

### HiSpark Taurus (LiteOS-A)

```
hispark_taurus (group)
    ↓ deps
hispark_taurus_build (group)
    ↓ deps
build_config (hc_gen)
    ↓ outputs
camera_host_config.hcb (ohos_prebuilt_etc)
    ↓ deps
driver_adapter (group)
    ↓ 依赖
device_manager (group)
    ↓ 依赖
pipeline_core (group)
    ↓ 依赖
ipp_algo_example (source)
```

**证据**: `hispark_taurus/camera/BUILD.gn:20-35`

---

## Target 与最终产物映射

### 产物清单

| Target | 输出产物 | 安装位置 |
|-------|----------|---------|
| `build_config` (hc_gen) | `mpp_config.hcb`<br/>`camera_host_config.hcb`<br/>`ipp_algo_config.hcb` | `/etc/camera/` (LiteOS)<br/>`/vendor/chipsets/.../hdfconfig/` (Linux) |
| `copy_mpp_libs` (SoC 层) | MPP 动态库 (`.so`) | `/vendor/lib/` |
| `hispark_taurus_sdk` (SoC 层) | SDK 库和头文件 | `/vendor/...` |
| `wifi_firmware` (SoC 层) | WiFi 固件 (`.bin`) | `/vendor/firmware/` |

**证据**:
- `hispark_taurus/camera/BUILD.gn:17` - `outputs = ["$root_out_dir/etc/camera/..."]`
- `hispark_taurus/camera/BUILD.gn:50-51` - `install_images = [ chipset_base_dir ]`

---

## 关键 GN 变量

### 板卡级变量

| 变量 | 定义位置 | 说明 |
|------|---------|------|
| `ohos_kernel_type` | 全局 | 内核类型: `"liteos_a"`, `"linux"`, `"liteos_m"` |
| `ohos_lite` | 全局 | 是否为 LiteOS 内核 |
| `product_name` | 命令行/配置 | 产品名称 |
| `chipset_base_dir` | SoC 层 | 芯片基目录 |

**证据**:
- `hispark_taurus/BUILD.gn:5` - `if (defined(ohos_lite))`
- `hispark_taurus/BUILD.gn:6` - `if (ohos_kernel_type == "linux")`
- `hispark_taurus/device.gni:22-26` - `if (product_name == "ipcamera_hispark_taurus")`

---

## 编译产物详解

### HDF 配置文件

**产物类型**: `.hcb` (HCS compiled binary)

**生成方式**: `hc_gen` 工具编译 `.hcs` → `.hcb`

**配置来源**:
- `$product_config_path/hdf_config/uhdf/camera/*.hcs` (Vendor 层）

**安装位置**:
- LiteOS: `$root_out_dir/etc/camera/`
- Linux: `/vendor/chipsets/.../hdfconfig/`

**证据**:
- `hispark_taurus/camera/BUILD.gn:10-18` - `hc_gen` 配置

---

### 相机 HAL 产物

**产物类型**:
- 配置文件: `config.c`, `params.c` (编译生成）
- 库文件: `libcamera_*.so` (从 SoC 层链接）

**证据**:
- `hispark_taurus/camera/BUILD.gn:76-77` - `config.c`, `params.c` 生成

---

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 详细的目录组织
- [内部 API](03_Inner_API.md) - 模块接口和依赖
- [编译产物](05_Build_Artifacts.md) - 详细的产物和安装路径
- [附录: 构建配置标志](appendix/Build_Config_Flags.md) - 关键编译选项
