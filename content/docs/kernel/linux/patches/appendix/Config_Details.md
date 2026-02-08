# 配置详情

本文档提供 OpenHarmony 内核配置文件的详细说明，包括配置选项的含义和建议值。

## 配置文件位置

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
                └── ... (同上)
```

## 配置分类

### 系统架构配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_ARM64` | y | 启用 ARM64 架构支持 |
| `CONFIG_ARCH_BCM2835` | n | 禁用非目标架构 |
| `CONFIG_ARCH_HISI` | y | 启用 HiSilicon SOC 支持 |
| `CONFIG_ARCH_ROCKCHIP` | y | 启用 Rockchip SOC 支持 |

### HDF 驱动框架配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_HDF` | y | 启用 HDF 驱动框架 |
| `CONFIG_HDF_CORE` | y | 启用 HDF 核心功能 |
| `CONFIG_HDF_PLATFORM` | y | 启用平台适配层 |
| `CONFIG_HDF_DRIVER` | y | 启用驱动接口 |

### 内存管理配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_ARM64_4K_PAGES` | y | 使用 4KB 页大小 |
| `CONFIG_NEED_PER_CPU_KM` | y | 每 CPU 内存 |
| `CONFIG_SPARSEMEM` | y | 稀疏内存模型 |
| `CONFIG_NUMA` | y | 启用 NUMA 支持 |

### 安全配置

| 配置项 | 值 | 说明 | 优先级 |
|--------|-----|------|--------|
| `CONFIG_STRICT_DEVMEM` | y | 限制 /dev/mem 访问 | 高 |
| `CONFIG_IO_STRICT_DEVMEM` | y | 严格 I/O 内存访问 | 高 |
| `CONFIG_HARDENED_USERCOPY` | y | 用户空间拷贝检查 | 高 |
| `CONFIG_RANDOMIZE_BASE` | y | KASLR 基础随机化 | 高 |
| `CONFIG_MODULE_SIG` | y | 模块签名验证 | 高 |
| `CONFIG_STACKPROTECTOR` | y | 栈保护 | 中 |
| `CONFIG_REFCOUNT_FULL` | y | 引用计数检查 | 中 |

### 网络配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_NET` | y | 启用网络支持 |
| `CONFIG_NETDEVICES` | y | 启用网络设备 |
| `CONFIG_ETHERNET` | y | 启用以太网 |
| `CONFIG_WLAN` | y | 启用无线网络 |

### 文件系统配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_FILESYSTEM` | y | 启用文件系统支持 |
| `CONFIG_EXT4_FS` | y | 启用 ext4 文件系统 |
| `CONFIG_FAT_FS` | y | 启用 FAT 文件系统 |
| `CONFIG_PROC_FS` | y | 启用 proc 文件系统 |
| `CONFIG_SYSFS` | y | 启用 sys 文件系统 |

### 设备驱动配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CONFIG_SOUND` | y | 启用音频支持 |
| `CONFIG_SND` | y | 启用 ALSA 声音系统 |
| `CONFIG_USB` | y | 启用 USB 支持 |
| `CONFIG_MMC` | y | 启用 MMC/SD 卡支持 |

## 小型系统配置特点

小型系统（Small System）针对资源受限设备优化：

```kconfig
# 资源优化
CONFIG_EMBEDDED=n                # 禁用嵌入式选项
CONFIG_EXPERT=n                  # 禁用专家选项
CONFIG_SLOB=y                    # 使用简单分配器
# 禁用不需要的功能
CONFIG_BLK_DEV_THROTTLING=n
CONFIG_CIFS_UPCALL=n
CONFIG_FTRACE=n
```

## 标准系统配置特点

标准系统（Standard System）提供完整功能：

```kconfig
# 完整功能
CONFIG_EMBEDDED=y
CONFIG_EXPERT=y
CONFIG_SLUB=y                    # 使用 SLUB 分配器
# 启用完整功能
CONFIG_KALLSYMS=y
CONFIG_KALLSYMS_ALL=y
CONFIG_DEBUG_INFO=y
```

## 设备特定配置

### Hi3516D V300 配置

```kconfig
# SOC 特定
CONFIG_ARCH_HI3516DV300=y
CONFIG_SOC_HI3516DV300=y

# 驱动支持
CONFIG_HISI_SDMMC=y              # SD/MMC 控制器
CONFIG_HISI_I2C=y                # I2C 控制器
CONFIG_HISI_SPI=y                # SPI 控制器
```

### RK3568 配置

```kconfig
# SOC 特定
CONFIG_ARCH_ROCKCHIP=y
CONFIG_ROCKCHIP_PM_DOMAINS=y

# 驱动支持
CONFIG_ROCKCHIP_IODOMAIN=y
CONFIG_ROCKCHIP_VOP=y            # 显示控制器
CONFIG_ROCKCHIP_VDEC=y           # 视频解码器
```

## 配置验证

### 检查关键配置

```bash
# 检查安全配置
grep -E "CONFIG_STRICT_DEVMEM|CONFIG_HARDENED_USERCOPY|CONFIG_MODULE_SIG" .config

# 检查 HDF 配置
grep -E "CONFIG_HDF" .config

# 检查架构配置
grep -E "CONFIG_ARM64|CONFIG_ARCH_" .config
```

### 配置冲突检测

```bash
# 检查互斥配置
if [ "$CONFIG_A" = "y" ] && [ "$CONFIG_B" = "y" ]; then
    echo "Warning: $CONFIG_A and $CONFIG_B may conflict"
fi
```

## 常见配置问题

### 问题 1: HDF 无法启动

**原因**: 未启用 HDF 相关配置

**解决**: 启用以下配置：
```kconfig
CONFIG_HDF=y
CONFIG_HDF_CORE=y
```

### 问题 2: 内存不足

**原因**: 启用了过多功能

**解决**: 针对小型系统优化配置：
```kconfig
CONFIG_SLOB=y
CONFIG_EMBEDDED=n
```

### 问题 3: 构建时间过长

**原因**: 启用了调试选项

**解决**: 禁用调试选项（生产环境）：
```kconfig
CONFIG_DEBUG_INFO=n
CONFIG_KALLSYMS=n
```

## 相关文档

- [构建系统](../05_Build_System.md)
- [支持的板卡](../03_Supported_Boards.md)
- [补丁分析](../04_Patches_Analysis.md)
