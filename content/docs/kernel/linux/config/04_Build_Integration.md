# 构建集成说明

## 构建系统集成

OpenHarmony 使用 GN（Generate Ninja）作为主要的构建系统，内核配置仓库与构建系统的集成通过 `kernel/linux/build` 仓库中的构建脚本实现。本节详细说明配置仓库如何被构建系统使用，以及构建过程中的关键环节。

### 构建脚本依赖

构建系统对本仓库的引用主要通过 kernel.mk 文件中的变量定义实现。根据 README_zh.md 中的描述，以下是与构建系统集成的关键路径变量。

`KERNEL_CONFIG_PATH` 定义了配置仓库的根路径，构建系统通过该路径定位内核配置文件。标准值为 `$(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}`，其中 `${KERNEL_VERSION}` 由 `linux_kernel_version` 参数指定，如 `linux-5.10`。

`DEFCONFIG_FILE` 定义了具体的 defconfig 文件名，由芯片名称（`DEVICE_NAME`）和构建类型（`BUILD_TYPE`）组合而成。标准命名模式为 `${DEVICE_NAME}_${BUILD_TYPE}_defconfig`，例如 `hi3516dv300_standard_defconfig`。

`DEVICE_PATCH_DIR` 和 `DEVICE_PATCH_FILE` 定义了芯片驱动补丁的路径。虽然这些补丁不存储在本仓库中（存储在 `kernel/linux/patches`），但它们的查找规则与配置文件密切相关，需要在构建时配合使用。

### 构建命令示例

以下命令展示了如何从 OpenHarmony 构建系统触发本仓库的配置使用。

```bash
./build.sh --product-name Hi3516DV300 \
    --build-target build_kernel \
    --gn-args linux_kernel_version="linux-5.10"
```

这个命令的参数含义如下。`--product-name Hi3516DV300` 指定目标产品名称，构建系统据此确定 DEVICE_NAME，进而确定要使用的配置文件名。`--build-target build_kernel` 指定构建目标为内核镜像，这将触发内核配置和编译流程。`--gn-args linux_kernel_version="linux-5.10"` 指定使用的内核版本，构建系统据此确定从本仓库的哪个子目录读取配置。

### 构建流程概述

基于本仓库的配置进行内核构建的完整流程可以分为以下五个阶段。

**阶段一：配置准备**。构建系统根据指定的 `linux_kernel_version`、`DEVICE_NAME` 和 `BUILD_TYPE` 参数，计算出完整的配置路径。然后将对应路径的 defconfig 文件复制到内核源码目录的 `.config` 文件位置。对于多层配置（base → type → chip → product），构建系统会按顺序加载并合并各层配置。

**阶段二：补丁合入**。在配置准备好之后，构建系统会执行 HDF 内核补丁和芯片驱动补丁的合入。补丁合入在配置复制之后进行，确保补丁能够正确修改基于 defconfig 生成的配置和源码。这一步对应 README_zh.md 中描述的 `patch_hdf.sh` 脚本调用。

**阶段三：配置生成**。执行 `make olddefconfig` 命令，基于合并后的 defconfig 生成完整的 .config 文件。这一步会解析配置依赖关系，填充所有未显式设置的配置项为默认值。

**阶段四：内核编译**。执行 `make` 命令编译 Linux 内核，生成最终的 uImage 或其他格式的内核镜像。编译过程中会用到构建系统传递的交叉编译工具链参数。

**阶段五：产物输出**。编译生成的内核镜像被复制到输出目录（`out/${product}/kernel/`），供后续的系统镜像打包使用。

## 配置查找规则

### 基于内核版本的查找

构建系统首先根据 `linux_kernel_version` 参数确定搜索的根目录。可选值包括 `linux-4.19`、`linux-5.10` 和 `linux-6.6`。不同版本对应的目录结构略有差异。

对于 Linux 4.19 和 5.10，配置文件主要位于 `${VERSION}/arch/arm/configs/` 目录下。对于 Linux 6.6，配置文件采用新的分层结构，包含 `${VERSION}/base_defconfig`、`${VERSION}/type/` 和 `${VERSION}/chip/` 等目录。

### 基于产品名称的查找

在确定版本根目录后，构建系统根据 `DEVICE_NAME` 参数进一步定位具体的配置文件。命名模式为 `${DEVICE_NAME}_${BUILD_TYPE}_defconfig`，其中 `BUILD_TYPE` 通常为 `standard` 或 `small`。

例如，对于 `DEVICE_NAME=hi3516dv300` 和 `BUILD_TYPE=standard`，构建系统会在以下路径查找配置文件：`kernel/linux/config/linux-5.10/arch/arm/configs/hi3516dv300_standard_defconfig`。

### 基于芯片平台的查找

对于 Linux 6.6 版本或部分新平台的配置，构建系统采用基于 `target_cpu` 参数的芯片平台查找方式。芯片平台的配置文件通常位于 `${chip_name}/arch/` 目录下。

例如，对于 `target_cpu=rk3568`，构建系统会在以下路径查找配置文件：`kernel/linux/config/linux-6.6/rk3568/arch/arm64_defconfig`。

## 配置文件来源

### 配置文件来源渠道

本仓库中的配置文件可以来自以下几个渠道。

**OpenHarmony 官方提供**：标准系统配置（standard_common_defconfig）和小系统配置（small_common_defconfig）由 OpenHarmony 内核团队维护。这些配置经过充分测试，确保满足 OpenHarmony 框架的运行需求。

**芯片厂商贡献**：Hi3516DV300、RK3568 等平台的配置主要由对应的芯片厂商（如海思、瑞芯微）或社区贡献者维护。这些配置需要与厂商提供的驱动补丁配合使用。

**社区贡献**：QEMU 模拟器配置、部分开发板配置由 OpenHarmony 社区贡献和维护。这些配置主要用于开发和测试目的。

### 配置文件的来源验证

在构建日志中可以观察到配置文件的实际来源路径。构建系统在复制配置文件时会输出类似以下的信息。

```
[CONFIG] Using defconfig: kernel/linux/config/linux-5.10/arch/arm/configs/hi3516dv300_standard_defconfig
[CONFIG] Patching HDF...
```

## 与补丁的配合使用

### HDF 补丁集成

HDF（Hardware Driver Foundation）是 OpenHarmony 的硬件驱动框架，其内核部分以补丁形式与内核源码配合使用。HDF 内核补丁的合入由 `patch_hdf.sh` 脚本自动完成。

根据 README_zh.md，HDF 补丁合入命令的基本格式如下。

```bash
$(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) \
    $(KERNEL_SRC_TMP_PATH) \
    $(KERNEL_PATCH_PATH) \
    $(DEVICE_NAME)
```

其中，`$(OHOS_BUILD_HOME)` 是 OpenHarmony 构建根目录，`$(KERNEL_SRC_TMP_PATH)` 是临时内核源码目录，`$(KERNEL_PATCH_PATH)` 是 HDF 补丁目录，`$(DEVICE_NAME)` 是设备名称。

HDF 补丁与 defconfig 配置的关系是互补的：defconfig 文件启用了 HDF 框架相关的内核配置选项（如配置管理器、驱动抽象层等），而补丁则提供了 HDF 框架的具体实现代码。

### 芯片驱动补丁

除了 HDF 补丁外，各芯片平台通常还需要额外的驱动补丁。这些补丁由芯片厂商提供，存储在 `kernel/linux/patches/${KERNEL_VERSION}/${DEVICE_NAME}_patch/` 目录下。

芯片补丁的命名规则为 `${DEVICE_NAME}.patch`，合入时机在 HDF 补丁之后。补丁与 defconfig 的配合方式与 HDF 类似：defconfig 启用驱动相关的配置选项，补丁提供具体的驱动代码。

## 产物与输出

### 编译产物

基于本仓库配置编译生成的直接产物是 Linux 内核镜像文件。根据目标平台的不同，可能生成以下格式的内核镜像。

**uImage**：传统的 ARM 内核镜像格式，包含 U-Boot 引导头。适用于 Hi3516DV300 等老平台。输出路径为 `out/${product}/kernel/uImage`。

**Image** 或 **Image.gz**：现代 ARM64 平台使用的内核镜像格式。适用于 RK3568 等新平台。输出路径为 `out/${product}/kernel/Image` 或 `Image.gz`。

**zImage**：压缩的内核镜像，适用于多种 ARM 平台。输出路径为 `out/${product}/kernel/zImage`。

### 运行时加载关系

内核镜像被 boot loader（如 U-Boot、Fastboot）加载到内存后开始执行。内核启动过程中会根据 defconfig 中的配置初始化各项子系统。

与本仓库配置相关的运行时组件包括：设备树 blob（.dtb），其配置应与 defconfig 中的驱动配置保持一致；内核模块（.ko 文件），由 defconfig 中的 `CONFIG_MODULES` 配置决定是否编译；内核参数（cmdline），虽然不存储在 defconfig 中，但部分配置会影响默认参数。

### 产物验证

构建完成后，可以通过以下方式验证配置是否正确应用。

检查配置项是否生效：运行 `zcat /proc/config.gz`（如果 `CONFIG_IKCONFIG_PROC=y`）或查看 `/boot/config-*` 文件，确认关键配置项的取值。

检查内核版本信息：运行 `uname -a` 或 `cat /proc/version`，确认内核版本与构建参数一致。

检查内核参数：运行 `cat /proc/cmdline`，查看启动参数是否符合预期配置。
