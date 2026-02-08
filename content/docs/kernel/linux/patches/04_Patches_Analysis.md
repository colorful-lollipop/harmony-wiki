# 补丁分析

本文档详细分析 `kernel_linux_patches` 仓库中的补丁类型、组织方式和应用流程。

## 补丁类型概述

仓库中的补丁主要分为以下几类：

| 类型 | 说明 | 优先级 |
|------|------|--------|
| HDF 补丁 | Hardware Driver Foundation 驱动框架 | 高 |
| SOC 补丁 | 芯片特定功能驱动 | 高 |
| 配置补丁 | 内核配置文件 | 中 |
| 安全补丁 | CVE 修复和安全加固 | 高 |

## HDF 补丁详解

### 位置

```
{version}/common_patch/hdf.patch
{version}/{board}_patch/hdf.patch
```

### 作用范围

HDF 补丁实现以下功能：

1. **驱动框架核心**
   - 驱动生命周期管理
   - 驱动接口标准化
   - 设备节点管理

2. **平台适配**
   - 芯片相关驱动适配
   - 板级资源配置
   - 中断和时钟配置

### 补丁内容结构

典型 HDF 补丁包含：

```
hdf.patch
├── 驱动框架修改        # 核心驱动代码
├── 设备树修改          # 平台设备配置
├── Kconfig 修改        # 内核配置选项
└── Makefile 修改       # 构建配置
```

### 关键修改点

| 修改类型 | 说明 | 影响 |
|----------|------|------|
| 驱动注册 | 添加新驱动支持 | 扩展硬件兼容性 |
| 资源管理 | 内存/中断/时钟 | 系统稳定性 |
| 接口适配 | POSIX 兼容层 | API 一致性 |

## SOC 补丁详解

### 位置

```
{version}/{board}_patch/kernel.patch
{version}/{board}_patch/{board}.patch
```

### 芯片特定功能

SOC 补丁包含以下内容：

| 功能类型 | 说明 | 示例 |
|----------|------|------|
| 驱动支持 | 芯片特有外设驱动 | GPU/NPU/VPU |
| 电源管理 | DVFS/休眠/唤醒 | CPU 频率调节 |
| 安全特性 | TrustZone/TEE | 安全启动 |
| 调试支持 | JTAG/性能分析 | 系统诊断 |

### imx8mm 补丁示例

`linux-5.10/imx8mm_patch/` 采用模块化组织：

```
imx8mm_patch/
├── patches/
│   ├── 0001_linux_arch.patch      # 架构相关
│   ├── 0002_linux_block.patch     # 块设备
│   ├── 0003_linux_crypto.patch    # 加密模块
│   ├── 0004_linux_fs.patch        # 文件系统
│   ├── 0005_linux_include.patch   # 头文件
│   ├── 0006_linux_init.patch      # 初始化
│   ├── 0007_linux_kernel.patch    # 内核核心
│   ├── 0008_linux_net.patch       # 网络
│   ├── 0009_linux_sound.patch     # 音频
│   ├── 0010_linux_tools.patch     # 工具
│   └── drivers/                   # 驱动分类
│       ├── 0011_linux_drivers_acpi.patch
│       ├── 0012_linux_drivers_ata.patch
│       └── ... (按类别分组)
└── hdf.patch                      # HDF 适配
```

## 补丁应用流程

### 手动应用

```bash
# 1. 进入内核源码目录
cd ${KERNEL_SRC_PATH}

# 2. 应用 HDF 补丁
patch -p1 < ${OHOS_BUILD_HOME}/kernel/linux/patches/${KERNEL_VERSION}/common_patch/hdf.patch

# 3. 应用 SOC 补丁
patch -p1 < ${OHOS_BUILD_HOME}/kernel/linux/patches/${KERNEL_VERSION}/${DEVICE_NAME}_patch/${DEVICE_NAME}.patch
```

### 脚本自动应用

通过 `kernel.mk` 中的脚本自动应用：

```makefile
# kernel.mk 中的应用逻辑
$(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) \
    $(KERNEL_SRC_TMP_PATH) \
    $(KERNEL_PATCH_PATH) \
    $(DEVICE_NAME)
```

### 应用顺序

```
1. common_patch/hdf.patch    # 必须首先应用
2. {board}_patch/hdf.patch   # 板卡 HDF 适配
3. {board}_patch/kernel.patch # SOC 功能补丁
```

## 补丁冲突处理

### 常见冲突类型

1. **行号冲突**: 内核版本差异导致
2. **功能冲突**: 多补丁修改同一文件
3. **依赖冲突**: 补丁依赖关系不满足

### 解决策略

| 策略 | 适用场景 | 方法 |
|------|----------|------|
| 版本匹配 | 内核版本差异 | 使用对应版本补丁 |
| 顺序调整 | 依赖顺序问题 | 按正确顺序应用 |
| 手动合并 | 复杂冲突 | 手动解决冲突 |

## 补丁管理规范

### 命名规范

- `{feature}.patch`: 功能补丁
- `{version}_{description}.patch`: 带版本描述
- `{date}_{issue}_{description}.patch`: 带日期和问题号

### 版本控制

- 主版本号: 内核版本变化时递增
- 次版本号: 重大功能变更
- 修订号: 小修复和 CVE 补丁

## 相关文档

- [支持的板卡](./03_Supported_Boards.md)
- [构建系统](./05_Build_System.md)
- [安全评审](./06_Security_Review.md)
