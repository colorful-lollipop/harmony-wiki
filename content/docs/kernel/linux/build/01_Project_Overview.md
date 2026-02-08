# 项目概览

> **更新时间**: 2026-02-06

---

## 文档目的

本文档介绍 OpenHarmony Linux Kernel 构建项目的基本情况、核心能力和技术边界，帮助新成员快速理解项目定位。

---

## 项目定位

### 项目身份

本项目是 **OpenHarmony Linux Kernel 构建系统** (`kernel/linux/build`)，位于 OpenHarmony 生态系统的构建基础设施层。

**核心特征**:
- ❌ **不是** Linux 内核源码本身
- ✅ **是** 内核构建的编排包装器
- ✅ 提供多架构内核编译支持
- ✅ 管理 HDF 驱动框架补丁
- ✅ 管理设备特定驱动补丁
- ✅ 集成到 OpenHarmony GN 构建系统

**证据**:
- `kernel.mk:28` - 定义了 `KERNEL_SRC_PATH = $(OHOS_BUILD_HOME)/kernel/linux/${KERNEL_VERSION}`，引用外部内核源码
- `BUILD.gn:46-47` - 定义了 `kernel_build_script_dir = "//kernel/linux/build"` 和 `kernel_source_dir = "//kernel/linux/$linux_kernel_version"`

---

## 核心能力

### 1. 多架构支持

本项目支持以下 CPU 架构的内核编译：

| 架构 | 工具链 | 内核镜像 | 代码证据 |
|--------|--------|---------|-----------|
| arm | gcc-linaro-7.5.0-arm-linux-gnueabi | uImage | `kernel.mk:36-39` |
| arm64 | gcc-linaro-7.5.0-aarch64-linux-gnu | Image | `kernel.mk:40-42` |
| riscv64 | LLVM/Clang | Image | `kernel.mk:43-48` |
| x86_64 | gcc | bzImage | `kernel.mk:49-51` |
| loongarch64 | gcc-loongarch64-linux-gnu | vmlinuz.efi | `kernel.mk:52-56` |

**代码证据**:
```makefile
# kernel.mk:36-56
ifeq ($(KERNEL_ARCH), arm)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin
else ifeq ($(KERNEL_ARCH), arm64)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin
else ifeq ($(KERNEL_ARCH), riscv64)
    PATH := $(CLANG_HOST_TOOLCHAIN):$(PATH)
    KERNEL_ARCH := riscv
    KERNEL_TARGET_TOOLCHAIN_PREFIX :=
    KERNEL_CROSS_COMPILE += LLVM=1
    KERNEL_CROSS_COMPILE += LLVM_IAS=1
else ifeq ($(KERNEL_ARCH), x86_64)
    KERNEL_TARGET_TOOLCHAIN := gcc
else ifeq ($(KERNEL_ARCH), loongarch64)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/loongarch64/gcc-loongarch64-linux-gnu/bin
endif
```

### 2. 内核版本管理

本项目支持两个 Linux 内核版本基线：

- **Linux 4.19.y LTS** - 较老设备支持
- **Linux 5.10.y LTS** - 主流设备支持

**代码证据**:
- `README.md:12-16` - 提及基于 Linux LTS 4.19.y 和 5.10.y 演进

### 3. HDF 驱动框架集成

HDF (Hardware Driver Foundation) 补丁通过独立脚本集成到内核构建流程中。

**代码证据**:
```bash
# kernel.mk:95
$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(KERNEL_PATCH_PATH) $(DEVICE_NAME)
```

### 4. 补丁管理

支持多种补丁类型的按序应用：

1. **HDF 通用补丁** - HDF 框架适配
2. **设备特定补丁** - 芯片平台驱动支持
3. **产品补丁** - 产品定制化
4. **Small 系统补丁** - 小型系统优化
5. **统一集合补丁** - 公共模块集成

**代码证据**:
```makefile
# kernel.mk:75-108
DEVICE_PATCH_DIR := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}/$(DEVICE_NAME)_patch
DEVICE_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME).patch
PRODUCT_PATCH_FILE := $(OHOS_BUILD_HOME)/vendor/hisilicon/watchos/patches/$(DEVICE_NAME).patch
SMALL_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME)_$(BUILD_TYPE).patch

# 补丁应用顺序
$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh ...
$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(DEVICE_PATCH_FILE) || true
$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(SMALL_PATCH_FILE)
```

---

## 运行环境

### 构建依赖

| 组件 | 作用 | 路径 | 代码证据 |
|--------|------|--------|----------|
| GCC 工具链 | 交叉编译目标代码 | `prebuilts/gcc/` | `kernel.mk:30` |
| Clang 工具链 | 主机编译和 RISC-V 支持 | `prebuilts/clang/` | `kernel.mk:31-32` |
| 内核源码 | 实际编译对象 | `kernel/linux/linux-4.19` 或 `linux-5.10` | `kernel.mk:27` |
| 内核补丁 | 设备驱动和 HDF | `kernel/linux/patches/` | `kernel.mk:28` |
| 内核配置 | defconfig 文件 | `kernel/linux/config/` | `kernel.mk:29` |
| HDF 驱动框架 | 驱动支持层 | `drivers/hdf_core/adapter/khdf/linux/` | `kernel.mk:95` |

**代码证据**:
```makefile
# kernel.mk:17-29
PRODUCT_NAME=$(TARGET_PRODUCT)
OHOS_BUILD_HOME := $(realpath $(shell pwd)/../../../)
KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/${KERNEL_VERSION}
KERNEL_OBJ_TMP_PATH := $(OUT_DIR)/kernel/OBJ/${KERNEL_VERSION}
KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux/${KERNEL_VERSION}
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}
KERNEL_CONFIG_PATH := $(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}
PREBUILTS_GCC_DIR := $(OHOS_BUILD_HOME)/prebuilts/gcc
CLANG_HOST_TOOLCHAIN := $(OHOS_BUILD_HOME)/prebuilts/clang/ohos/linux-x86_64/llvm/bin
```

### 构建类型

支持两种构建变体：

| 构建类型 | 用途 | 代码证据 |
|---------|------|----------|
| standard | 标准系统，完整功能 | `BUILD.gn:14-18`, `kernel.mk:21` |
| small | 小型系统，资源受限 | `BUILD.gn:14-15`, `kernel.mk:22` |

**代码证据**:
```makefile
# BUILD.gn:14-18
if (os_level == "mini" || os_level == "small") {
  import("//build/lite/config/component/lite_component.gni")
} else {
  import("//build/config/clang/clang.gni")
  import("//build/ohos.gni")
}

# kernel.mk:21-24
ifeq ($(BUILD_TYPE), standard)
    BOOT_IMAGE_PATH = $(OHOS_BUILD_HOME)/device/board/hisilicon/hispark_taurus/uboot/prebuilts
    KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/src_tmp/${KERNEL_VERSION}
endif
```

---

## 关键概念

### GN 构建系统

OpenHarmony 使用 GN (Generate Ninja) 作为主构建系统。本项目通过 `BUILD.gn` 集成到 OpenHarmony 构建流程中。

**关键 Target**:
- `linux_kernel` - 入口 group 目标
- `build_kernel` - 核心构建 action
- `check_build` - 增量构建检查

**代码证据**:
```gn
# BUILD.gn:79-81
group("linux_kernel") {
  deps = [ ":build_kernel" ]
}

action("build_kernel") {
  script = "build_kernel.sh"
  sources = [ kernel_source_dir ]
  deps = [ ":check_build" ]
  outputs = [ "$root_build_dir/packages/phone/images/$kernel_image" ]
}
```

### Makefile

内核编译通过传统的 GNU Make 进行，支持多并行编译（`make -j64`）。

**代码证据**:
```makefile
# kernel.mk:115
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) -j64 $(KERNEL_IMAGE)
```

### Shell 脚本

Shell 脚本负责环境设置、文件操作和构建编排。

**关键脚本**:
- `kernel_module_build.sh` - 构建入口，环境设置
- `build_kernel.sh` - 镜像复制
- `check_build.sh` - 增量构建检查

### Kernel Configuration (defconfig)

内核通过 defconfig 文件进行配置，不同设备和构建类型有独立的配置。

**代码证据**:
```makefile
# kernel.mk:81
DEFCONFIG_FILE := $(DEVICE_NAME)_$(BUILD_TYPE)_defconfig

# kernel.mk:111
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) $(DEFCONFIG_FILE)
```

---

## 技术边界

### 本项目不包含

❌ Linux 内核源码 - 位于 `kernel/linux/linux-4.19` 或 `kernel/linux/linux-5.10`
❌ HDF 驱动框架实现 - 位于 `drivers/hdf_core/adapter/khdf/linux`
❌ 芯片平台驱动源码 - 位于各个 vendor 仓库
❌ 工具链二进制 - 位于 `prebuilts/` 目录

### 本项目负责

✅ 内核构建流程编排
✅ 补丁应用管理
✅ 多架构编译支持
✅ 内核镜像打包
✅ OpenHarmony 构建系统集成

---

## 相关文档

- [目录结构](02_Directory_Structure.md) - 文件组织详情
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [GN Targets](04_GN_Targets.md) - GN 构建目标清单
- [构建脚本](05_Build_Scripts.md) - 脚本详细分析
- [内核配置](06_Kernel_Configuration.md) - defconfig 管理机制

---

## 参考资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Linux 4.19 LTS](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-4.19.y)
- [Linux 5.10 LTS](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-5.10.y)
