# 构建系统

## 构建系统概述

LiteOS-A 内核使用 **GN + HB + Make** 的三层构建系统：

| 层级 | 工具 | 用途 |
|------|------|------|
| **1** | GN (Generate Ninja) | 生成 Ninja 构建文件 |
| **2** | HB (OpenHarmony Builder) | 产品构建系统 |
| **3** | Make + 编译器 | 实际编译链接 |

## 根构建文件 (BUILD.gn)

**证据**: `BUILD.gn:30-464`

### 关键 Targets

| target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `modules` | group | - | 所有内核模块 |
| `apps` | group | - | 用户态应用 |
| `tests` | group | - | 测试套件 |
| `liteos_a` | group | - | 完整内核产物 |
| `liteos` | executable | `liteos` | 可执行文件 |
| `copy_liteos` | copy | `OHOS_Image` | 复制/重命名 |
| `build_kernel_image` | build_ext | `OHOS_Image.bin` | 生成二进制镜像 |

### 主要配置 (BUILD.gn)

```gn
# 配置开关
liteos_name = "OHOS_Image"
liteos_container_enable = false
liteos_is_mini = false
tee_enable = false

# 架构配置
if (defined(LOSCFG_ARCH_ARM)) {
  mcpu = LOSCFG_ARCH_CPU  # 如 cortex-a7
}
```

## GN 模板 (liteos.gni)

**证据**: `liteos.gni`

### 内核模块模板

```gn
template("kernel_module") {
  # 自动检测配置
  source_set(target_name) {
    sources = invoker.sources
    configs += [ ":los_config" ]
  }
}
```

### 使用方式

```gn
kernel_module("my_module") {
  sources = [ "src/my_module.c" ]
  deps = [ ":other_module" ]
}
```

## 模块构建文件

### kernel/BUILD.gn
**证据**: `kernel/BUILD.gn:32-49`

```gn
group("kernel") {
  deps = [
    "base",       # 基础模块
    "common",     # 公共组件
    "extended",   # 扩展模块
    "user",       # Init
  ]
}

config("public") {
  include_dirs = [ "include" ]
  configs = ["base:public", "common:public", "extended:public", "user:public"]
}
```

### 其他模块 BUILD.gn
| 目录 | 关键 target |
|------|-------------|
| `syscall/` | `syscall` (通过 `kernel_module` 模板) |
| `security/` | `security` (cap/vid) |
| `fs/` | `fs` (FAT/JFFS2/VFS) |
| `net/` | `net` (lwIP) |

## Kconfig 配置

**证据**: `Kconfig`

内核功能开关，通过 `menuconfig` 配置：

```bash
# 启用/禁用模块
LOSCFG_KERNEL_XXX=y/n

# 示例
LOSCFG_KERNEL_TASK=y
LOSCFG_KERNEL_SEM=y
LOSCFG_KERNEL_QUEUE=y
```

### 关键配置项

| 配置项 | 默认 | 说明 |
|--------|------|------|
| `LOSCFG_KERNEL_CORE` | y | 核心调度 |
| `LOSCFG_KERNEL_IPC` | y | IPC 机制 |
| `LOSCFG_KERNEL_MEM` | y | 内存管理 |
| `LOSCFG_KERNEL_FS` | y | 文件系统 |
| `LOSCFG_KERNEL_NET` | y | 网络栈 |
| `LOSCFG_KERNEL_SYSCALL` | y | 系统调用 |

## 编译产物

### 产物清单

| 产物 | 路径 | 说明 |
|------|------|------|
| `liteos` | `out/xxx/kernel/liteos` | ELF 可执行文件 |
| `OHOS_Image` | `out/xxx/OHOS_Image` | 符号表完整镜像 |
| `OHOS_Image.bin` | `out/xxx/OHOS_Image.bin` | 纯二进制镜像 |
| `OHOS_Image.map` | `out/xxx/OHOS_Image.map` | 链接映射文件 |
| `OHOS_Image.sym.sorted` | - | 符号排序表 |
| `OHOS_Image.asm` | - | 反汇编文件 |

### 链接脚本

| 文件 | 说明 |
|------|------|
| `tools/build/liteos.ld` | GCC 链接脚本 |
| `tools/build/liteos_llvm.ld` | LLVM 链接脚本 |

## 构建流程

### GN 阶段
```bash
hb set kernel_liteos_a    # 选择内核
hb build                  # GN 生成 Ninja
```

### 编译阶段
```bash
./build.sh <board> <compiler> <out_dir> ...
```
**证据**: `build.sh` - 调用 Make 完成实际编译

### 产物生成
```bash
# objcopy 生成二进制
arm-linux-ohoself-objcopy -O binary liteos OHOS_Image.bin

# objdump 生成符号和反汇编
arm-linux-ohoself-objdump -t liteos > OHOS_Image.sym.sorted
arm-linux-ohoself-objdump -d liteos > OHOS_Image.asm
```

## 编译配置

### 架构配置
**证据**: `BUILD.gn:86-122`

```gn
liteos_arch_cflags = []
liteos_arch_cflags += [ "-mcpu=$mcpu" ]  # 如 cortex-a7
liteos_arch_cflags += [ "-mfloat-abi=softfp" ]
liteos_arch_cflags += [ "-mfpu=$LOSCFG_ARCH_FPU" ]
```

### 优化配置
**证据**: `BUILD.gn:157-185`

| 模式 | 优化级别 |
|------|----------|
| Debug | `-O0 -g` |
| Optimize | `-O2` |
| Size | `-Oz` (LLVM) / `-Os` (GCC) |

### 安全配置
**证据**: `BUILD.gn:139-155`

```gn
config("ssp_config") {
  # 栈保护
  if (defined(LOSCFG_CC_STACKPROTECTOR_ALL)) {
    cflags += [ "-fstack-protector-all" ]
  }
}
```

## 相关文档

- [目录结构](/01_Directory_Structure.md)
- [内核模块详解](/04_Kernel_Modules.md)
- [附录：配置项](/appendix/Config_Flags.md)
