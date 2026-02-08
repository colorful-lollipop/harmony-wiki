# 构建配置标志

## 目的

本文档说明 OpenHarmony Hisilicon 板卡仓库的关键编译选项和 Feature flags。

## 适用范围

本文档适用于：
- 构建工程师理解编译配置
- 开发者启用/禁用特定功能
- 系统工程师优化构建参数

---

## 关键编译变量

### 内核类型

**变量**: `ohos_kernel_type`

**可选值**:
- `"liteos_m"` - 轻量系统（HiSpark Pegasus）
- `"liteos_a"` - 小型系统（HiSpark Aries, HiSpark Taurus）
- `"linux"` - 标准系统（HiSpark Phoenix, HiSpark Taurus）

**定义位置**: GN 全局配置

**使用示例**:
```gn
if (ohos_kernel_type == "linux") {
  deps += [ "//device/soc/hisilicon/hi3516dv300/sdk_linux:hispark_taurus_sdk" ]
}
```

**证据**:
- `hispark_taurus/BUILD.gn:6-14` - `if (ohos_kernel_type == "linux")`

---

### CPU 架构

**变量**: `board_cpu`

**可选值**:
- `"cortex-a7"` - HiSpark Aries, HiSpark Taurus (LiteOS-A)
- `"riscv32"` - HiSpark Pegasus (LiteOS-M)
- `"cortex-a53"` - HiSpark Phoenix (Linux)

**定义位置**: `liteos_a/config.gni`, `linux/config.gni`

**使用示例**:
```gn
board_cpu = "cortex-a7"
board_cflags = [
  "-mfloat-abi=softfp",
  "-mfpu=neon-vfpv4",
]
```

**证据**:
- `hispark_aries/liteos_a/config.gni:21` - `board_cpu = "cortex-a7"`

---

### 工具链类型

**变量**: `board_toolchain_type`

**可选值**:
- `"clang"` - Clang 编译器（默认）
- `"gcc"` - GCC 编译器

**定义位置**: `liteos_a/config.gni`

**使用示例**:
```gn
board_toolchain_type = "clang"
```

**证据**:
- `hispark_aries/liteos_a/config.gni:38` - `board_toolchain_type = "clang"`

---

### 编译标志

#### C 编译标志

**变量**: `board_cflags`

**常用选项**:
- `-mfloat-abi=softfp` - 浮点 ABI
- `-mfpu=neon-vfpv4` - FPU 类型
- `-march=armv7-a` - ARMv7 架构

**定义位置**: `liteos_a/config.gni`

**使用示例**:
```gn
board_cflags = [
  "-mfloat-abi=softfp",
  "-mfpu=neon-vfpv4",
]
```

**证据**:
- `hispark_aries/liteos_a/config.gni:41-48` - `board_cflags` 定义

---

#### C++ 编译标志

**变量**: `board_cxx_flags`

**常用选项**: 与 C 编译标志类似

**定义位置**: `liteos_a/config.gni`

**使用示例**:
```gn
board_cxx_flags = [
  "-mfloat-abi=softfp",
  "-mfpu=neon-vfpv4",
]
```

**证据**:
- `hispark_aries/liteos_a/config.gni:45-48` - `board_cxx_flags` 定义

---

### 存储类型

**变量**: `storage_type`

**可选值**:
- `"spinor"` - SPI NOR Flash（HiSpark Aries）
- `"nand"` - NAND Flash
- `"emmc"` - eMMC

**定义位置**: `liteos_a/config.gni`

**使用示例**:
```gn
storage_type = "spinor"
```

**证据**:
- `hispark_aries/liteos_a/config.gni:61` - `storage_type = "spinor"`

---

## Feature Flags

### MPI 支持

**变量**: `is_support_mpi`

**说明**: 是否支持 Media Processing Interface（相机功能）

**类型**: `bool`

**默认值**: `true` (HiSpark Taurus)

**使用示例**:
```gn
is_support_mpi = true
if (is_support_mpi) {
  is_support_v4l2 = false
  defines += [ "SUPPORT_MPI" ]
  chipset_build_deps = "$board_camera_path:hispark_taurus_build"
}
```

**影响**:
- `true`: 启用相机 HAL，编译 MPP 相关代码
- `false`: 禁用相机 HAL

**证据**:
- `hispark_taurus/device.gni:28` - `is_support_mpi = true`

---

### V4L2 支持

**变量**: `is_support_v4l2`

**说明**: 是否支持 Video4Linux2 接口

**类型**: `bool`

**默认值**: `false` (HiSpark Taurus)

**使用示例**:
```gn
if (is_support_mpi) {
  is_support_v4l2 = false
}
```

**影响**:
- `true`: 使用 V4L2 接口（Linux 标准）
- `false`: 使用 MPI 接口（HiSilicon 私有）

**证据**:
- `hispark_taurus/device.gni:30` - `is_support_v4l2 = false`

---

## 产品配置

### 产品名称

**变量**: `product_name`

**说明**: 当前编译的产品名称

**使用示例**:
```gn
if (product_name == "ipcamera_hispark_taurus") {
  product_config_path = "//vendor/hisilicon/hispark_taurus_standard"
} else {
  product_config_path = "//vendor/hisilicon/${product_name}"
}
```

**证据**:
- `hispark_taurus/device.gni:22-26` - `product_name` 条件判断

---

### 产品配置路径

**变量**: `product_config_path`

**说明**: Vendor 层产品配置路径

**默认值**: `//vendor/hisilicon/hispark_taurus_standard` (HiSpark Taurus)

**使用示例**:
```gn
product_config_path = "//vendor/hisilicon/hispark_taurus_standard"
```

**影响**:
- 决定 HDF 配置文件来源
- 决定产品特定参数

**证据**:
- `hispark_taurus/device.gni:23` - `product_config_path` 定义

---

## 相关跳转

- [GN Targets](04_GN_Targets.md) - 构建系统和目标依赖
- [目录结构](01_Directory_Structure.md) - 详细的目录组织
- [内部 API](03_Inner_API.md) - 模块接口和配置
