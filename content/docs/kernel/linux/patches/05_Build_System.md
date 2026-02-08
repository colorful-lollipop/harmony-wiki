# 构建系统

本文档描述 `kernel_linux_patches` 仓库的构建系统，包括 GN 构建配置、构建流程和产物说明。

## 构建系统概述

### 技术栈

| 组件 | 版本要求 | 说明 |
|------|----------|------|
| GN | 任意稳定版本 | 生成构建文件 |
| Make | 3.82+ | 内核构建工具 |
| GCC/Clang | 支持目标架构 | 交叉编译工具链 |
| Python | 3.8+ | 构建脚本 |

### 构建流程

```
┌─────────────────────────────────────────────────────────────┐
│                    构建流程                                  │
├─────────────────────────────────────────────────────────────┤
│  1. 准备内核源码                                             │
│     ├── 克隆 Linux 内核源码                                  │
│     ├── 切换到对应版本分支                                    │
│     └── 应用 patches 仓库的补丁                               │
│                                                             │
│  2. 配置内核                                                 │
│     ├── 复制 defconfig 到 .config                            │
│     ├── 执行 make oldconfig                                  │
│     └── 配置内核选项                                         │
│                                                             │
│  3. 编译内核                                                 │
│     ├── 执行 make ARCH=arm64 Image                           │
│     ├── 打包为 uImage                                        │
│     └── 生成设备树文件 (.dtb)                                │
│                                                             │
│  4. 输出产物                                                 │
│     ├── 内核镜像: uImage/Image                               │
│     ├── 设备树: *.dtb                                        │
│     └── 模块: *.ko                                          │
└─────────────────────────────────────────────────────────────┘
```

## GN 构建配置

### 位置

构建配置位于构建系统的主仓库（`kernel/linux/build`）：

```
kernel/linux/build/
├── BUILD.gn          # GN 构建入口
├── kernel.mk         # Make 构建脚本
└── ohos.build        # OpenHarmony 构建配置
```

### BUILD.gn 配置示例

```gn
# 关键配置参数
ohos_kernel_version = "linux-5.10"     # 内核版本
ohos_build_type = "standard"           # 构建类型
ohos_device_name = "hispark_taurus"    # 设备名称
```

### kernel.mk 关键变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `OHOS_BUILD_HOME` | OpenHarmony 构建根目录 | - |
| `KERNEL_VERSION` | 内核版本 | linux-5.10 |
| `DEVICE_NAME` | 目标设备名称 | hispark_taurus |
| `BUILD_TYPE` | 构建类型 | standard |
| `KERNEL_SRC_TMP_PATH` | 内核源码临时目录 | - |
| `KERNEL_PATCH_PATH` | 补丁路径 | - |

## 构建配置选项

### 常用参数

```bash
./build.sh \
    --product-name Hi3516DV300 \         # 产品名称
    --build-target build_kernel \        # 构建目标
    --gn-args linux_kernel_version="linux-5.10"  # 内核版本
```

### GN 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `linux_kernel_version` | string | 内核版本 |
| `device_name` | string | 设备名称 |
| `build_type` | string | 构建类型 |

### 构建类型

| 类型 | 说明 | 配置文件 |
|------|------|----------|
| small | 小型系统 | `{board}_small_defconfig` |
| standard | 标准系统 | `{board}_standard_defconfig` |

## 配置文件

### 配置文件位置

```
config/
├── linux-4.19/
│   └── arch/
│       └── arm/
│           └── configs/
│               ├── hispark_taurus_small_defconfig
│               ├── hispark_taurus_standard_defconfig
│               ├── small_common_defconfig
│               └── standard_common_defconfig
└── linux-5.10/
    └── arch/
        └── arm/
            └── configs/
                └── ... (同上结构)
```

### defconfig 组成

典型 defconfig 包含：

| 配置项 | 说明 |
|--------|------|
| `CONFIG_ARM64` | ARM64 架构支持 |
| `CONFIG_SOC_HI3516DV300` | SOC 支持 |
| `CONFIG_HDF` | HDF 驱动框架 |
| `CONFIG_NET` | 网络支持 |
| `CONFIG_MODULES` | 模块支持 |

## 构建产物

### 产物清单

| 产物 | 格式 | 位置 | 说明 |
|------|------|------|------|
| uImage | 镜像 | out/kernel/ | 启动镜像 |
| Image | 镜像 | out/kernel/ | 内核镜像 |
| *.dtb | 设备树 | out/kernel/ | 设备描述 |
| *.ko | 模块 | out/kernel/ | 内核模块 |

### 产物路径

```
out/
└── kernel/
    ├── uImage              # ARM 启动镜像
    ├── Image               # 原始内核镜像
    ├── arch/arm64/boot/
    │   ├── dts/
    │   │   ├── hisi/
    │   │   │   └── hi3516d-v300.dtb
    │   │   └── rockchip/
    │   │       └── rk3568.dtb
    │   └── Image
    └── modules/
        └── *.ko            # 内核模块
```

## 常见构建问题

### 问题 1: 补丁应用失败

**现象**: `patch` 命令失败

**解决**:
```bash
# 检查内核版本匹配
cat Makefile | grep VERSION

# 手动应用补丁查看详细错误
patch -p1 --dry-run < patchfile.patch
```

### 问题 2: 配置错误

**现象**: 内核配置缺失

**解决**:
```bash
# 使用正确的 defconfig
cp ${CONFIG_PATH}/${BOARD}_${TYPE}_defconfig .config
make oldconfig
```

### 问题 3: 交叉编译工具链

**现象**: 编译架构错误

**解决**:
```bash
# 设置交叉编译工具链
export CROSS_COMPILE=aarch64-linux-gnu-
export ARCH=arm64
```

## 相关文档

- [支持的板卡](./03_Supported_Boards.md)
- [补丁分析](./04_Patches_Analysis.md)
- [配置详情](./appendix/Config_Details.md)
