# 开发板移植指南

本指南旨在帮助开发者将 OpenHarmony 移植到新的开发板或芯片平台，涵盖从环境准备到配置调整的完整流程。

## 移植前准备

### 硬件需求评估

在开始移植工作之前，需要确认目标开发板具备以下基本条件。

**芯片架构支持**：目标芯片的 CPU 架构必须在 OpenHarmony 支持范围内。目前 OpenHarmony Linux 内核主要支持 ARM32（arm）和 ARM64（arm64/aarch64）架构。对于其他架构（如 x86、RISC-V），需要确认对应的内核版本和构建系统的支持情况。

**最小资源要求**：根据选择的系统形态（标准系统或小系统），目标设备需要满足不同的最小资源要求。标准系统通常需要 512MB 以上的内存和 4GB 以上的存储；小系统可以在 256MB 内存和 32MB 存储的条件下运行。

**外设兼容性**：确认目标开发板的主要外设（串口、GPIO、I2C、SPI、网络等）有对应的 Linux 驱动支持。OpenHarmony 的 HDF 框架需要这些驱动以框架化的方式工作。

### 软件环境准备

**构建环境**：推荐使用 Ubuntu 18.04/20.04/22.04 LTS 作为构建主机。确保安装了必要的构建工具，包括 make、gcc、perl、python3、libncurses-dev 等。

**交叉编译工具链**：根据目标芯片架构安装对应的交叉编译工具链。ARM32 平台使用 arm-linux-gnueabihf-gcc，ARM64 平台使用 aarch64-linux-gnu-gcc。建议使用 GCC 9.x 或 10.x 系列以获得最佳兼容性。

**OpenHarmony 构建工具**：安装 hb（OpenHarmony 构建工具）和必要的依赖。具体安装方法请参考 OpenHarmony 官方文档。

### 代码仓库准备

移植工作需要以下 OpenHarmony 代码仓库的本地副本：`kernel/linux/linux-*.*`（目标内核版本的源码）、`kernel/linux/build`（构建脚本）、`kernel/linux/config`（本仓库，配置文件）、`kernel/linux/patches`（补丁目录）以及 `drivers/hdf_core`（HDF 驱动框架）。

## 移植步骤

### 第一步：评估和选择内核版本

根据目标芯片的特性和 OpenHarmony 版本支持情况，选择合适的 Linux 内核版本。

**推荐选择**：对于大多数新项目，推荐使用 Linux 5.10 版本，该版本经过充分测试，社区支持完善。如果目标芯片较新且需要 6.x 版本的内核特性，可以考虑 Linux 6.6 版本。

**版本降级**：如果目标芯片的 BSP（Board Support Package）基于较旧的内核版本，可能需要降级移植。这种情况下需要额外关注驱动兼容性问题。

### 第二步：准备内核源码

获取目标内核版本的源码，并确保内核源码树的初始状态干净。

```bash
# 克隆内核源码（以 5.10 为例）
git clone -b linux-5.10.y https://gitee.com/openharmony/kernel_linux.git kernel/linux/linux-5.10

# 进入源码目录
cd kernel/linux/linux-5.10

# 确保源码干净
make mrproper
```

### 第三步：创建芯片配置目录

在本仓库（`kernel/linux/config`）中创建芯片平台配置目录。

```bash
# 进入配置仓库根目录
cd kernel/linux/config

# 创建芯片配置目录
mkdir -p ${KERNEL_VERSION}/${CHIP_NAME}/arch

# 例如，为新芯片 "newchip" 创建配置目录
mkdir -p linux-5.10/newchip/arch
```

### 第四步：编写基础配置文件

基于现有开发板的配置文件创建新的芯片配置。

**复制参考配置**：选择与目标芯片架构相近的现有配置作为起点。

```bash
# 复制参考配置（以 ARM64 为例）
cp linux-5.10/arch/arm/configs/hispark_phoenix_standard_defconfig \
   linux-5.10/newchip/newchip_standard_defconfig
```

**修改配置**：根据目标芯片的规格修改配置项。主要需要关注以下配置：

```bash
# 打开配置文件进行编辑
vim linux-5.10/newchip/newchip_newchip_standard_defconfig

# 关键配置项修改示例
CONFIG_LOCALVERSION="-openharmony"        # 内核版本后缀
CONFIG_SERIAL_8250=y                      # 串口驱动
CONFIG_SERIAL_8250_CONSOLE=y              # 串口控制台
CONFIG_CMDLINE="console=ttyS0,115200"     # 引导参数
```

### 第五步：添加设备树

设备树描述了目标硬件的拓扑结构，需要为新开发板创建设备树文件。

设备树源文件通常位于内核源码的 `arch/arm/boot/dts/`（ARM32）或 `arch/arm64/boot/dts/`（ARM64）目录下。具体路径取决于芯片厂商的 BSP。

设备树文件命名遵循 `SOC-BOARD.dts` 模式，例如 `rk3568-evb.dts` 是 RK3568 EVB 开发板的设备树文件。设备树文件需要描述以下内容：CPU 核心和时钟配置、内存布局、串口配置、外设节点（GPIO、I2C、SPI 等），以及引导参数。

### 第六步：准备补丁文件

根据 README_zh.md 中的说明，准备芯片驱动补丁。

```bash
# 创建补丁目录
mkdir -p kernel/linux/patches/linux-5.10/newchip_patch

# 放置补丁文件
cp /path/to/newchip.patch \
   kernel/linux/patches/linux-5.10/newchip_patch/newchip.patch
```

补丁文件应包含目标芯片所需的所有内核驱动修改。补丁的命名和放置规则遵循 kernel.mk 中的定义，确保构建系统能够正确识别和合入。

### 第七步：测试编译

使用 OpenHarmony 构建系统测试配置是否正确。

```bash
# 进入 OpenHarmony 构建根目录
cd /path/to/openharmony

# 执行构建
./build.sh --product-name NewChipDevBoard \
    --build-target build_kernel \
    --gn-args linux_kernel_version="linux-5.10"
```

编译过程中需要关注以下要点。

**配置错误**：如果出现配置相关的编译错误，需要回到第四步调整配置项。常见的配置错误包括缺少必要的驱动配置、配置依赖未满足，以及配置项拼写错误。

**链接错误**：链接阶段的错误通常与设备树或驱动代码相关。检查设备树中的外设节点配置是否与驱动代码匹配。

**运行时错误**：编译成功后需要进行实际测试。由于这属于移植工作的范畴，本指南不再详细展开。

## 常见问题与解决方案

### 问题一：串口无输出

**现象**：内核编译成功但串口无任何输出。

**可能原因**：串口配置不正确、波特率不匹配、设备树中串口节点配置错误。

**排查步骤**：首先确认硬件串口连接正确，检查使用的串口号是否正确（ttyS0/ttyS1 等）。然后核对设备树中的串口配置，确保时钟和引脚配置正确。最后检查内核配置中的 `CONFIG_SERIAL_8250_CONSOLE=y` 和 `CONFIG_CMDLINE` 参数。

### 问题二：内核panic：无法挂载根文件系统

**现象**：内核启动后提示无法挂载根文件系统（Kernel panic - unable to mount root fs）。

**可能原因**：根文件系统类型配置错误、设备树中存储控制器配置错误、根文件系统镜像未正确生成。

**排查步骤**：首先确认内核配置中的根文件系统类型与实际使用的类型匹配（如 CONFIG_EXT4_FS=y）。然后检查设备树中存储控制器（eMMC/SD/NAND）的配置是否正确。最后确认根文件系统镜像是否正确生成并放置在正确位置。

### 问题三：内核编译时间过长

**现象**：首次编译耗时过长（超过数小时）。

**可能原因**：编译未使用并行模式、交换空间不足、存储 I/O 性能差。

**解决方案**：确保使用 `-j` 参数启用并行编译（`make -j$(nproc)`）。如果内存不足，增加交换空间或使用分页策略。对于虚拟机环境，建议使用物理机或高性能存储。

## 配置优化建议

### 编译优化

为了加速开发和调试阶段，可以采用以下优化策略。

**增量编译**：首次完整编译后，后续修改可以通过增量编译大幅减少编译时间。使用 `make olddefconfig && make zImage` 而非每次都执行 `make defconfig`。

**ccache**：安装并配置 ccache 缓存编译结果，可以显著加速重复编译。

```bash
# 安装 ccache
sudo apt install ccache

# 配置
export PATH="/usr/lib/ccache:$PATH"

# 设置缓存大小
ccache -M 10G
```

### 运行时优化

对于生产环境，建议进行以下运行时配置优化。

**禁用调试选项**：发布版本中应禁用不必要的调试选项，包括 `CONFIG_DEBUG_INFO=n`、`CONFIG_DEBUG_KMEMLEAK=n`、`CONFIG_DEBUG_BUGVERBOSE=n` 等。

**优化 IO 调度器**：根据存储设备类型选择合适的 IO 调度器。eMMC/SSD 使用 `deadline`，机械硬盘使用 `bfq`。

```bash
# 在内核 cmdline 中添加
# 对于 eMMC/SSD
root=... elevator=deadline

# 对于机械硬盘
root=... elevator=bfq
```

## 参考资源

以下资源对移植工作有重要参考价值。

**OpenHarmony 官方文档**：https://docs.openharmony.cn 提供详细的系统架构说明和移植指南。

**Linux 内核文档**：内核源码中的 `Documentation/` 目录包含丰富的配置和使用说明。

**芯片厂商 BSP**：芯片厂商通常提供完整的 BSP 文档和参考设计，其中包含详细的硬件接口说明和驱动源码。

**开发板参考实现**：本仓库中已有的开发板配置（如 hispark_taurus、rk3568）可作为移植的重要参考。
