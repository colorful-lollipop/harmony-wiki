# 关键配置项

## Kconfig 配置

**证据**: `Kconfig`

### 架构配置 (ARCH)

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_ARCH_ARM` | bool | y | ARM 架构支持 |
| `LOSCFG_ARCH_ARM_AARCH32` | bool | - | 32位 ARM (ARM32) |
| `LOSCFG_ARCH_ARM_AARCH64` | bool | - | 64位 ARM (ARM64) |
| `LOSCFG_ARCH_CPU` | string | - | CPU 类型 (如 cortex-a7) |
| `LOSCFG_ARCH_FPU` | string | - | FPU 类型 |
| `LOSCFG_ARCH_FPU_DISABLE` | bool | - | 禁用 FPU |
| `LOSCFG_COMPILER_CLANG_LLVM` | bool | - | 使用 Clang/LLVM 编译器 |

### 核心配置 (CORE)

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_KERNEL_CORE` | bool | y | 核心内核 |
| `LOSCFG_KERNEL_TASK` | bool | y | 任务管理 |
| `LOSCFG_KERNEL_PROCESS` | bool | y | 进程管理 |
| `LOSCFG_KERNEL_CPUP` | bool | y | CPU 使用率统计 |
| `LOSCFG_KERNEL_DYNLOAD` | bool | - | 动态加载 (ELF) |
| `LOSCFG_KERNEL_CONTAINER` | bool | - | 容器支持 |
| `LOSCFG_KERNEL_SYSCALL` | bool | y | 系统调用 |

### 内存管理配置 (MEM)

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_KERNEL_MEM` | bool | y | 内存管理 |
| `LOSCFG_KERNEL_MEM_TLSF` | bool | y | TLSF 分配器 |
| `LOSCFG_KERNEL_MEMBOX` | bool | y | MemBox 分配器 |
| `LOSCFG_KERNEL_VM` | bool | y | 虚拟内存 |
| `LOSCFG_KERNEL_SHM` | bool | - | 共享内存 |
| `LOSCFG_SRAM_BASE` | hex | - | SRAM 起始地址 |
| `LOS_SRAM_SIZE` | hex | - | SRAM 大小 |

### IPC 配置

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_KERNEL_IPC` | bool | y | IPC 机制 |
| `LOSCFG_KERNEL_MUX` | bool | y | 互斥锁 |
| `LOSCFG_KERNEL_SEM` | bool | y | 信号量 |
| `LOSCFG_KERNEL_QUEUE` | bool | y | 消息队列 |
| `LOSCFG_KERNEL_EVENT` | bool | y | 事件 |
| `LOSCFG_KERNEL_FUTEX` | bool | - | Futex |
| `LOSCFG_KERNEL_LITEIPC` | bool | - | LiteIPC |
| `LOSCFG_KERNEL_PIPE` | bool | - | 管道 |

### 文件系统配置 (FS)

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_FS_VFS` | bool | y | VFS 层 |
| `LOSCFG_FS_FAT` | bool | y | FAT 文件系统 |
| `LOSCFG_FS_JFFS2` | bool | - | JFFS2 文件系统 |
| `LOSCFG_FS_NFS` | bool | - | NFS 客户端 |
| `LOSCFG_FS_RAMFS` | bool | y | RAMFS |
| `LOSCFG_FS_PROC` | bool | - | ProcFS |

### 网络配置 (NET)

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_NET_LWIP_SACK` | bool | y | lwIP 协议栈 |
| `LOSCFG_NET_LWIP_IPV4` | bool | y | IPv4 支持 |
| `LOSCFG_NET_LWIP_IPV6` | bool | - | IPv6 支持 |

### 驱动配置

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_DRIVERS` | bool | y | 驱动框架 |
| `LOSCFG_DRIVERS_CHAR` | bool | y | 字符设备 |
| `LOSCFG_DRIVERS_RANDOM` | bool | y | 随机数驱动 |

### Shell 配置

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_SHELL` | bool | y | Shell 支持 |
| `LOSCFG_SHELL_LK` | bool | y | 命令行 |
| `LOSCFG_SHELL_CMD_DEBUG` | bool | - | 调试命令 |
| `LOSCFG_SHELL_CMD_MEM` | bool | - | 内存命令 |
| `LOSCFG_SHELL_CMD_TASK` | bool | - | 任务命令 |

### 安全配置

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_SECURITY_CAPABILITY` | bool | y | 能力机制 |
| `LOSCFG_SECURITY_VID` | bool | y | 虚拟 ID 映射 |

### 编译选项

| 配置项 | 类型 | 默认 | 说明 |
|--------|------|------|------|
| `LOSCFG_COMPILE_DEBUG` | bool | - | 调试编译 |
| `LOSCFG_COMPILE_OPTIMIZE` | bool | y | 优化编译 |
| `LOSCFG_COMPILE_OPTIMIZE_SIZE` | bool | - | 体积优化 |
| `LOSCFG_COMPILE_LTO` | bool | - | LTO 链接时优化 |
| `LOSCFG_CC_STACKPROTECTOR` | bool | - | 栈保护 |
| `LOSCFG_CC_STACKPROTECTOR_STRONG` | bool | - | 强栈保护 |
| `LOSCFG_CC_STACKPROTECTOR_ALL` | bool | - | 全局栈保护 |

## GN 构建参数

**证据**: `BUILD.gn`

### 可配置参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `liteos_name` | string | "OHOS_Image" | 产物名称 |
| `liteos_container_enable` | bool | false | 启用容器 |
| `liteos_skip_make` | bool | false | 跳过 Make |
| `liteos_is_mini` | bool | false | Mini 模式 |
| `tee_enable` | bool | false | 启用 TEE |

### 编译器标志

| 标志 | 示例值 | 说明 |
|------|--------|------|
| `-mcpu` | cortex-a7 | CPU 类型 |
| `-mfloat-abi` | softfp | 浮点 ABI |
| `-mfpu` | fpv4-sp-d16 | FPU 类型 |
| `-mthumb` | - | Thumb 指令集 |
| `-O0/-O2/-Os` | -O2 | 优化级别 |
| `-g` | - | 调试信息 |
| `-Wall` | - | 警告启用 |
| `-Werror` | - | 警告变错误 |

## 宏定义

**证据**: `BUILD.gn:218-244`

### 预定义宏

| 宏 | 说明 |
|----|------|
| `__LITEOS__` | LiteOS 标识 |
| `__LITEOS_A__` | LiteOS-A 标识 |
| `NDEBUG` | 非调试模式 |

### 条件编译宏

| 宏 | 触发条件 | 说明 |
|----|----------|------|
| `LOSCFG_KERNEL_CONTAINER` | 启用容器 | 容器相关代码 |
| `LOSCFG_KERNEL_DYNLOAD` | 启用动态加载 | dlopen/dlsym |
| `LOSCFG_FS_FAT` | 启用 FAT | FAT 文件系统 |
| `LOSCFG_SECURITY_CAPABILITY` | 启用能力 | capability 检查 |

## 链接脚本符号

**证据**: `tools/build/liteos.ld`, `liteos_llvm.ld`

| 符号 | 类型 | 说明 |
|------|------|------|
| `_etext` | 地址 | 代码段结束 |
| `_edata` | 地址 | 数据段结束 |
| `_end` | 地址 | BSS 结束 |
| `__heap_start` | 地址 | 堆起始 |
| `__heap_limit` | 地址 | 堆结束 |

## 相关文档

- [构建系统](/03_Build_System.md)
- [内核模块详解](/04_Kernel_Modules.md)
