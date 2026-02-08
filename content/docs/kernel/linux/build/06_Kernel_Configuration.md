# 内核配置

> **更新时间**: 2026-02-06

---

## 文档目的

本文档说明 OpenHarmony Linux Kernel 的配置管理机制，包括 defconfig 文件组织、配置规则和不同架构/设备的配置差异。

---

## 配置文件组织

### 目录结构

```
kernel/linux/config/
├── linux-4.19/                    # Linux 4.19 内核配置
│   └── arch/
│       ├── arm/configs/
│       │   ├── hispark_taurus_small_defconfig
│       │   ├── hispark_taurus_standard_defconfig
│       │   ├── small_common_defconfig
│       │   └── standard_common_defconfig
│       └── arm64/configs/
│           ├── hispark_taurus_small_defconfig
│           ├── hispark_taurus_standard_defconfig
│           ├── small_common_defconfig
│           └── standard_common_defconfig
└── linux-5.10/                    # Linux 5.10 内核配置
    └── arch/
        ├── arm/configs/
        │   ├── hispark_taurus_small_defconfig
        │   ├── hispark_taurus_standard_defconfig
        │   ├── small_common_defconfig
        │   └── standard_common_defconfig
        ├── arm64/configs/
        │   ├── rk3568_standard_defconfig
        │   ├── small_common_defconfig
        │   └── standard_common_defconfig
        └── riscv/configs/
            ├── small_common_defconfig
            └── standard_common_defconfig
```

**代码证据**:
```makefile
# kernel.mk:29
KERNEL_CONFIG_PATH := $(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}
```

---

## 配置文件命名规则

### 命名模式

`${DEVICE_NAME}_${BUILD_TYPE}_defconfig`

| 部分 | 说明 | 示例 |
|------|------|--------|
| `DEVICE_NAME` | 设备/板名称 | `hispark_taurus`, `rk3568` |
| `BUILD_TYPE` | 构建类型 | `small`, `standard` |

**代码证据**:
```makefile
# kernel.mk:81
DEFCONFIG_FILE := $(DEVICE_NAME)_$(BUILD_TYPE)_defconfig
```

### 示例配置文件

| 设备 | 内核版本 | 架构 | 构建类型 | 配置文件 |
|------|---------|--------|---------|---------|
| hispark_taurus | linux-4.19 | arm | small | `hispark_taurus_small_defconfig` |
| hispark_taurus | linux-4.19 | arm | standard | `hispark_taurus_standard_defconfig` |
| hispark_taurus | linux-5.10 | arm | small | `hispark_taurus_small_defconfig` |
| hispark_taurus | linux-5.10 | arm | standard | `hispark_taurus_standard_defconfig` |
| rk3568 | linux-5.10 | arm64 | standard | `rk3568_standard_defconfig` |

---

## 配置机制

### 配置应用流程

```
kernel.mk 构建过程
    │
    ▼
1. 拷贝配置文件
    │   从 kernel/linux/config/${KERNEL_VERSION}/
    │   拷贝到 ${KERNEL_SRC_TMP_PATH}/
    │
    ▼
2. 清理旧的配置
    │   make distclean
    │
    ▼
3. 应用 defconfig
    │   make ${DEFCONFIG_FILE}
    │   生成 .config 文件
    │
    ▼
4. 准备模块（仅 linux-5.10）
    │   make modules_prepare
    │
    ▼
5. 编译内核
    │   make -j64 ${KERNEL_IMAGE}
```

**代码证据**:
```makefile
# kernel.mk:109-115
$(hide) cp -rf $(KERNEL_CONFIG_PATH)/. $(KERNEL_SRC_TMP_PATH)/
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) distclean
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) $(DEFCONFIG_FILE)
ifeq ($(KERNEL_VERSION), linux-5.10)
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) modules_prepare
endif
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) -j64 $(KERNEL_IMAGE)
```

---

## 关键配置选项

### HDF 相关配置

HDF (Hardware Driver Foundation) 是 OpenHarmony 的驱动框架，需要在内核中启用相应支持。

**典型配置项**:
```kconfig
CONFIG_DRIVERS_HDF=y
CONFIG_DRIVERS_HDF_WIFI=y
CONFIG_DRIVERS_HDF_INPUT=y
CONFIG_DRIVERS_HDF_SENSOR=y
CONFIG_DRIVERS_HDF_DISPLAY=y
```

### 设备树支持

不同设备需要启用对应的设备树配置：

```kconfig
# ARM 设备树
CONFIG_USE_OF=y
CONFIG_ARM_DT=y

# 设备树编译
CONFIG_DTC=y
```

### 安全特性配置

OpenHarmony Linux 内核包含多种安全特性：

**代码证据** (来自 kernel_linux_common_modules):
```kconfig
# 代码签名
CONFIG_CODE_SIGN=y

# 容器逃逸检测
CONFIG_CONTAINER_ESCAPE_DETECTION=y

# 内存安全
CONFIG_MEMORY_SECURITY=y

# 指针认证
CONFIG_PAC=y

# 权限管理
CONFIG_XPM=y
```

---

## 不同架构的配置差异

### ARM (32位) 配置

**特点**:
- 使用 `uImage` 镜像格式
- 适用于资源受限设备
- 支持小型系统 (small) 构建类型

**典型配置**:
```kconfig
CONFIG_ARM=y
CONFIG_ARM_LPAE=n
CONFIG_VMSPLIT_3G=n
CONFIG_AEABI=y

# 内核镜像
CONFIG_ZBOOT_ROM=y
CONFIG_UIMAGE=y
```

**代码证据**:
```makefile
# kernel.mk:36-39
ifeq ($(KERNEL_ARCH), arm)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/arm-linux-gnueabi-
endif
```

### ARM64 配置

**特点**:
- 使用 `Image` 镜像格式
- 支持 64 位地址空间
- 主要用于标准系统

**典型配置**:
```kconfig
CONFIG_ARM64=y
CONFIG_ARM64_VA_BITS_48=y
CONFIG_ARM64_LPAE=n

# 内核镜像
CONFIG_ARM64_PAGE_OFFSET_BITS=14
```

**代码证据**:
```makefile
# kernel.mk:40-42
else ifeq ($(KERNEL_ARCH), arm64)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/aarch64-linux-gnu-
endif
```

### RISC-V 64 位配置

**特点**:
- 使用 LLVM 工具链
- 启用 LLVM 汇编器
- 使用 `Image` 镜像格式

**典型配置**:
```kconfig
CONFIG_RISCV=y
CONFIG_RISCV_ISA_C=y
CONFIG_RISCV_ISA_SVPB=y
CONFIG_64BIT=y
```

**代码证据**:
```makefile
# kernel.mk:43-48
else ifeq ($(KERNEL_ARCH), riscv64)
    PATH := $(CLANG_HOST_TOOLCHAIN):$(PATH)
    KERNEL_ARCH := riscv
    KERNEL_TARGET_TOOLCHAIN_PREFIX :=
    KERNEL_CROSS_COMPILE += LLVM=1
    KERNEL_CROSS_COMPILE += LLVM_IAS=1
endif
```

### LoongArch 64 位配置

**特点**:
- 使用 `vmlinuz.efi` 镜像格式
- 支持 EFI 引导
- 使用 GCC 工具链

**典型配置**:
```kconfig
CONFIG_LOONGARCH=y
CONFIG_64BIT=y
CONFIG_EFI=y
CONFIG_EFI_STUB=y
```

**代码证据**:
```makefile
# kernel.mk:52-56
else ifeq ($(KERNEL_ARCH), loongarch64)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/loongarch64/gcc-loongarch64-linux-gnu/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/loongarch64-unknown-linux-gnu-
    KERNEL_ARCH := loongarch
endif
```

### x86_64 配置

**特点**:
- 使用 `bzImage` 镜像格式
- 使用原生 GCC（非交叉编译）
- 适用于 x86 模拟或开发

**典型配置**:
```kconfig
CONFIG_X86_64=y
CONFIG_X86=y
CONFIG_HYPERVISOR_GUEST=y
```

**代码证据**:
```makefile
# kernel.mk:49-51
else ifeq ($(KERNEL_ARCH), x86_64)
    KERNEL_TARGET_TOOLCHAIN := gcc
    KERNEL_TARGET_TOOLCHAIN_PREFIX :=
endif
```

---

## 配置文件管理

### 查看当前配置

在内核源码目录中查看配置：

```bash
cd ${KERNEL_SRC_TMP_PATH}
make ARCH=${KERNEL_ARCH} menuconfig
```

### 生成新配置

编译后导出当前配置：

```bash
cd ${KERNEL_SRC_TMP_PATH}
make ARCH=${KERNEL_ARCH} savedefconfig
# 生成的配置在 arch/${KERNEL_ARCH}/configs/${KERNEL_ARCH}_defconfig
```

### 修改现有配置

编辑 defconfig 文件后重新编译：

1. 修改 `kernel/linux/config/${KERNEL_VERSION}/arch/${KERNEL_ARCH}/configs/${DEVICE_NAME}_${BUILD_TYPE}_defconfig`
2. 重新构建内核
3. 配置自动应用到编译中

**代码证据**:
```makefile
# kernel.mk:109-110
$(hide) cp -rf $(KERNEL_CONFIG_PATH)/. $(KERNEL_SRC_TMP_PATH)/
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) $(DEFCONFIG_FILE)
```

---

## 配置常见问题

### 配置文件不存在

**问题**: `make: *** No rule to make target 'xxx_defconfig'`

**原因**: 配置文件路径或名称错误

**解决方案**:
1. 检查 `KERNEL_CONFIG_PATH` 是否正确
2. 确认 `DEVICE_NAME` 和 `BUILD_TYPE` 变量
3. 验证配置文件存在于 `kernel/linux/config/${KERNEL_VERSION}/arch/${KERNEL_ARCH}/configs/`

### 配置冲突

**问题**: 应用补丁后配置失效

**原因**: 补丁修改了配置，但未同步 defconfig

**解决方案**:
1. 重新编译内核
2. 使用 `make savedefconfig` 导出新配置
3. 更新对应的 defconfig 文件

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [构建脚本](05_Build_Scripts.md) - Shell 脚本详细分析
- [补丁管理](07_Patch_Management.md) - 补丁应用机制详解
