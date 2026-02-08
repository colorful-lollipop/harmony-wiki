# 编译产物

> **更新时间**: 2026-02-06

---

## 文档目的

本文档说明 OpenHarmony Linux Kernel 构建系统产生的所有编译产物，包括内核镜像类型、输出路径和运行时加载关系。

---

## 内核镜像类型

### 按架构分类

| 架构 | 内核镜像 | 格式说明 | 用途 |
|--------|----------|---------|--------|
| arm | `uImage` | U-Boot 封装的 Linux 内核镜像 | ARM 32位设备 |
| arm | `zImage-dtb` | 压缩镜像 + 设备树合并 | hispark_taurus 设备 |
| arm64 | `Image` | 未压缩的 Linux 内核镜像 | ARM 64位设备 |
| riscv64 | `Image` | 未压缩的 Linux 内核镜像 | RISC-V 64位设备 |
| x86_64 | `bzImage` | 压缩的 Linux 内核镜像（bzip2） | x86_64 设备 |
| loongarch64 | `vmlinuz.efi` | 压缩 + EFI 可执行文件格式 | LoongArch 64位设备 |

**代码证据**:
```makefile
# kernel.mk:80
KERNEL_IMAGE_FILE := $(KERNEL_SRC_TMP_PATH)/arch/$(KERNEL_ARCH)/boot/$(KERNEL_IMAGE)
```

### 按格式分类

| 格式 | 扩展名 | 压缩方式 | 引导加载器 |
|------|--------|---------|------------|
| uImage | - | 无 | U-Boot |
| Image | - | 无 | U-Boot/GRUB/EFI |
| bzImage | - | bzip2 | U-Boot/GRUB/EFI |
| zImage | - | gzip/zImage | U-Boot/GRUB |
| vmlinuz.efi | .efi | gzip + EFI 格式 | EFI |
| dtbo.img | .img | 设备树 blob | U-Boot |

---

## 输出路径

### 构建输出目录

```
out/
├── kernel/
│   ├── src_tmp/${KERNEL_VERSION}/          # 标准系统：内核源码临时目录
│   └── OBJ/${KERNEL_VERSION}/           # 编译对象目录
│       └── arch/${KERNEL_ARCH}/
│           └── boot/
│               ├── uImage
│               ├── Image
│               ├── bzImage
│               ├── zImage
│               └── dts/
│                   └── *.dtb
└── packages/phone/images/              # 最终镜像输出目录
    ├── uImage
    ├── Image
    ├── bzImage
    ├── vmlinuz.efi
    ├── zImage-dtb
    └── dtbo.img
```

**代码证据**:
```makefile
# kernel.mk:19-24
KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/${KERNEL_VERSION}
KERNEL_OBJ_TMP_PATH := $(OUT_DIR)/kernel/OBJ/${KERNEL_VERSION}

ifeq ($(BUILD_TYPE), standard)
    KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/src_tmp/${KERNEL_VERSION}
    export KERNEL_SRC_DIR=out/KERNEL_OBJ/kernel/src_tmp/${KERNEL_VERSION}
endif
```

### 镜像输出路径映射

| Target | 源路径 | 输出路径 |
|--------|--------|----------|
| `:build_kernel` (arm) | `kernel/OBJ/linux-5.10/arch/arm/boot/uImage` | `packages/phone/images/uImage` |
| `:build_kernel` (arm) | `kernel/OBJ/linux-5.10/arch/arm/boot/zImage-dtb` | `packages/phone/images/zImage-dtb` |
| `:build_kernel` (arm64) | `kernel/OBJ/linux-5.10/arch/arm64/boot/Image` | `packages/phone/images/Image` |
| `:build_kernel` (riscv64) | `kernel/OBJ/linux-5.10/arch/riscv/boot/Image` | `packages/phone/images/Image` |
| `:build_kernel` (x86_64) | `kernel/OBJ/linux-5.10/arch/x86/boot/bzImage` | `packages/phone/images/bzImage` |
| `:build_kernel` (loongarch64) | `kernel/OBJ/linux-5.10/vmlinuz.efi` | `packages/phone/images/vmlinuz.efi` |
| `:build_kernel` (hispark_phoenix) | `kernel/OBJ/linux-5.10/arch/arm/boot/dts/hi3751v350.dtb` | `packages/phone/images/dtbo.img` |

**代码证据**:
```bash
# build_kernel.sh:29-49
if [ "$5" == "arm" ];then
    cp ${2}/kernel/OBJ/${8}/arch/arm/boot/uImage ${3}/uImage
    if [ "$7" == "hispark_phoenix"  ];then
        cp ${2}/kernel/OBJ/${8}/arch/arm/boot/dts/hi3751v350.dtb ${3}/dtbo.img
        cat ${2}/kernel/OBJ/${8}/arch/arm/boot/zImage ${3}/dtbo.img > ${3}/zImage-dtb
    fi
elif [ "$5" == "arm64" ];then
    cp ${2}/kernel/OBJ/${8}/arch/arm64/boot/Image ${3}/Image
fi
```

---

## 镜像详细信息

### uImage (ARM 32位）

**特点**:
- U-Boot 兼容格式
- 包含 64 字节头部（魔数、加载地址、镜像大小、入口点）
- 可包含设备树

**结构**:
```
+------------------+
|  U-Boot Header | 64 bytes
|------------------+
|  Kernel Image   |
|------------------+
|  Device Tree    | (可选）
|------------------+
```

**用途**: ARM 32位设备引导加载器

### Image (ARM 64位 / RISC-V)

**特点**:
- 纯内核镜像，无额外封装
- 直接由引导加载器加载到内存
- 支持大型内核（无大小限制）

**用途**: ARM 64位和 RISC-V 设备

### bzImage (x86_64)

**特点**:
- 使用 bzip2 压缩
- 自解压引导头
- 包含 16 位实模式代码和 32 位保护模式代码

**用途**: x86_64 设备，支持传统 BIOS 引导

### vmlinuz.efi (LoongArch)

**特点**:
- 压缩镜像 + EFI 可执行文件格式
- 兼容 UEFI 固件
- 可直接从 EFI Shell 引导

**用途**: LoongArch 64位设备，支持现代 UEFI 引导

---

## 设备树镜像

### dtbo.img

**用途**: Device Tree Blob Overlay

**特点**:
- 设备树二进制格式
- 可叠加在基础设备树上
- 支持设备特定配置

**代码证据**:
```bash
# build_kernel.sh:31-32
if [ "$7" == "hispark_phoenix"  ];then
    cp ${2}/kernel/OBJ/${8}/arch/arm/boot/dts/hi3751v350.dtb ${3}/dtbo.img
fi
```

### zImage-dtb

**用途**: 压缩内核镜像 + 设备树合并

**特点**:
- zImage（压缩内核）和 DTB（设备树）的简单拼接
- 便于一次性加载内核和设备树

**代码证据**:
```bash
# build_kernel.sh:33-35
cat ${2}/kernel/OBJ/${8}/arch/arm/boot/zImage ${3}/dtbo.img > ${3}/zImage-dtb
```

---

## 运行时加载关系

### U-Boot 引导流程 (ARM 设备）

```
U-Boot 启动
    │
    ▼
1. 加载 uImage 到内存
    │   解析 U-Boot 头部
    │   获取内核大小、加载地址、入口点
    │
    ▼
2. 可选：加载设备树 (DTB)
    │   解析设备树
    │   传递硬件配置给内核
    │
    ▼
3. 跳转到内核入口点
    │   start_kernel()
    │
    ▼
4. 内核初始化
    │   early_param
    │   early_console
    │   setup_arch
    │
    ▼
5. 设备树解析
    │   of_fdt_unflatten_tree()
    │   early_init_dt_scan()
    │
    ▼
6. 驱动初始化
    │   HDF 初始化
    │   设备驱动加载
    │
    ▼
7. 用户空间启动
    │   init 进程
    │   加载 OpenHarmony 系统
```

### EFI 引导流程 (x86_64 / LoongArch)

```
UEFI 固件启动
    │
    ▼
1. 加载 EFI 应用程序
    │   执行 EFI Boot Manager
    │   加载 vmlinuz.efi
    │
    ▼
2. 解析 PE/COFF 格式
    │   获取内核入口点
    │   解析设备树（ACPI/DSDT）
    │
    ▼
3. 调用内核入口
    │   efi_main()
    │   切换到保护模式
    │
    ▼
4. 内核初始化
    │   (同 U-Boot 流程步骤 4-7)
```

---

## 安装路径

### 系统分区布局

| 分区 | 内容 | 文件 |
|------|------|------|
| boot | 内核镜像、设备树 | `uImage`, `Image`, `dtbo.img` |
| system | OpenHarmony 系统 | `system.img` |
| vendor | 厂商特定文件 | `vendor.img` |
| userdata | 用户数据 | `userdata.img` |
| recovery | 恢复系统 | `recovery.img` |

### 内核镜像安装

```bash
# 通过 fastboot 刷写（Android/OpenHarmony 设备）
fastboot flash boot uImage

# 或通过 parted + cp（Linux 环境）
parted /dev/sda --script
cp uImage /boot/
```

---

## 镜像验证

### 检查镜像格式

```bash
# 检查 uImage 头部
mkimage -l uImage

# 检查设备树
dtc -I dtb -o dts hi3751v350.dtb

# 检查 EFI 镜像
file vmlinuz.efi
```

### 提取内核符号表

```bash
# 从 Image 中提取符号
extract-ikconfig Image > /boot/config-$(uname -r)
```

---

## 大小限制

### U-Boot 限制

| 镜像格式 | 最大大小 | 说明 |
|---------|--------|--------|
| uImage | 16 MB | U-Boot 头部字段限制 |
| uImage (自定义） | 32 MB 或更大 | 需要修改 U-Boot |

### 文件系统限制

| 分区类型 | 典型大小 | 说明 |
|---------|--------|--------|
| boot 分区 | 32-64 MB | 仅存放内核和设备树 |
| system 分区 | 1-4 GB | 存放完整系统 |

---

## 常见问题

### 镜像过大

**问题**: `uImage: Image too big`

**原因**: 内核配置过多，导致镜像超过 16 MB 限制

**解决方案**:
1. 减少启用的内核模块
2. 使用 zImage 格式（压缩）
3. 配置 U-Boot 支持更大的镜像

### 设备树加载失败

**问题**: `dtb: FDT_ERR_NOTFOUND`

**原因**: 设备树文件路径错误或格式错误

**解决方案**:
1. 验证 DTB 文件格式
2. 检查设备树加载地址
3. 确认 DTB 与内核配置匹配

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [GN Targets](04_GN_Targets.md) - GN 构建目标清单
- [构建脚本](05_Build_Scripts.md) - Shell 脚本详细分析
