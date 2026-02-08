# 03 - OH 构建适配

## 概览

OpenHarmony 使用 **GN + Ninja** 构建系统替代了 LZMA SDK 原有的 Makefile 构建方式。

**构建文件**:
- `BUILD.gn` - 主构建配置
- `lzma.gni` - 源文件列表定义

---

## BUILD.gn 结构说明

### 文件位置

```
third_party/lzma/
├── BUILD.gn          # 主构建配置
├── lzma.gni          # 源文件列表
└── ...
```

### 构建目标层次

```
lzma (group)
├── lzma_shared (ohos_shared_library)      # 设备端共享库
│   ├── lzma_source_arm (设备 ARM)
│   ├── lzma_source_arm64 (设备 ARM64)
│   └── lzma_source_riscv64 (设备 RISC-V)
│
└── lzma_static (ohos_static_library)      # 主机端静态库
    ├── lzma_source_arm64_host (主机 ARM64)
    ├── lzma_source_x86_host (主机 X86_64)
    └── lzma_source_riscv64_host (主机 RISC-V)
```

---

## 配置详解

### 1. 公共配置 (`lzma_config_common`)

```gn
config("lzma_config_common") {
  include_dirs = [ "C" ]
  cflags = [
    "-DZ7_AFFINITY_DISABLE",          # OH 特定：禁用 CPU 亲和性
    "-Wall",                           # 启用所有警告
    "-Werror",                         # 警告视为错误
    "-Wno-empty-body",                 # 忽略空语句体警告
    "-Wno-enum-conversion",            # 忽略枚举转换警告
    "-Wno-logical-op-parentheses",     # 忽略逻辑操作符优先级警告
    "-Wno-self-assign",                # 忽略自赋值警告
    "-Wno-implicit-function-declaration", # 忽略隐式函数声明警告
  ]
  visibility = [ ":*" ]
}
```

#### 关键配置说明

| 配置项 | 值 | 说明 |
|--------|----|----|
| `include_dirs` | `"C"` | 头文件搜索路径指向 C 目录 |
| `-DZ7_AFFINITY_DISABLE` | 定义 | 禁用 CPU 亲和性设置，适配 OH 调度策略 |
| `-Wall -Werror` | 编译选项 | 严格编译，代码质量要求高 |

#### 警告抑制说明

由于 LZMA SDK 是第三方代码，部分代码风格与 OH 编译规则冲突，因此抑制了以下警告：

- `-Wno-empty-body`: 允许 `if (cond);` 这种空语句体
- `-Wno-enum-conversion`: 允许枚举类型隐式转换
- `-Wno-logical-op-parentheses`: 允许 `a && b || c` 不加括号
- `-Wno-self-assign`: 允许 `a = a` 自赋值
- `-Wno-implicit-function-declaration`: 允许隐式函数声明（遗留代码）

### 2. 主机端配置 (`lzma_config_host`)

```gn
config("lzma_config_host") {
  defines = []
  defines += [ "target_cpu=${target_cpu}" ]
  defines += [ "host_toolchain=${host_toolchain}" ]
  defines += [ "current_toolchain=${current_toolchain}" ]
  defines += [ "default_toolchain=${default_toolchain}" ]
}
```

**用途**: 为主机端工具传递构建系统变量，用于条件编译。

---

## 架构特定配置

### 设备端配置

#### ARM (`lzma_source_arm`)

```gn
ohos_source_set("lzma_source_arm") {
  configs = [ ":lzma_config_common" ]
  public_configs = [ ":lzma_config_common" ]
  
  include_dirs = [ "Asm/arm" ]
  cflags = [ "-march=armv7-a" ]
  ldflags = [ "-lpthread" ]
  
  sources = common_c_source
}
```

| 配置项 | 说明 |
|--------|------|
| 架构 | ARMv7-A |
| 线程支持 | 使用 pthread |
| 汇编优化 | Asm/arm 目录（未启用具体文件） |

#### ARM64 (`lzma_source_arm64`)

```gn
ohos_source_set("lzma_source_arm64") {
  branch_protector_ret = "pac_ret"    # 返回地址保护
  
  configs = [ ":lzma_config_common" ]
  public_configs = [ ":lzma_config_common" ]
  
  include_dirs = [ "Asm/arm64" ]
  cflags = [ "-march=armv8-a+crc" ]   # ARMv8 + CRC 指令集
  
  sources = common_c_source
  sources += arm64_asm_source          # 添加汇编优化
}
```

| 配置项 | 说明 |
|--------|------|
| 架构 | ARMv8-A + CRC |
| 安全特性 | PAC-RET (返回地址保护) |
| 汇编优化 | LzmaDecOpt.S, 7zAsm.S |

**PAC-RET 保护**:
- `branch_protector_ret = "pac_ret"`
- 防止 ROP (Return-Oriented Programming) 攻击
- 使用 ARM64 的 Pointer Authentication 特性

#### RISC-V 64 (`lzma_source_riscv64`)

```gn
ohos_source_set("lzma_source_riscv64") {
  configs = [ ":lzma_config_common" ]
  public_configs = [ ":lzma_config_common" ]
  
  cflags = [ "-march=rv64gc" ]        # RV64G + C 压缩指令
  
  sources = common_c_source
}
```

| 配置项 | 说明 |
|--------|------|
| 架构 | RV64GC |
| 汇编优化 | 无（上游未提供） |

### 主机端配置

主机端配置与设备端类似，但增加了 `lzma_config_host` 配置：

#### X86_64 Host (`lzma_source_x86_host`)

```gn
ohos_source_set("lzma_source_x86_host") {
  configs = [
    ":lzma_config_common",
    ":lzma_config_host",
  ]
  public_configs = [
    ":lzma_config_common",
    ":lzma_config_host",
  ]
  
  include_dirs = [ "Asm/x86" ]
  
  sources = common_c_source
}
```

---

## 构建目标

### 共享库 (`lzma_shared`)

**用途**: 设备端运行时库

```gn
ohos_shared_library("lzma_shared") {
  branch_protector_ret = "pac_ret"
  public_configs = [ ":lzma_config_common" ]
  
  if (target_cpu == "arm") {
    deps = [ ":lzma_source_arm" ]
  } else if (target_cpu == "arm64") {
    deps = [ ":lzma_source_arm64" ]
  } else if (target_cpu == "riscv64") {
    deps = [ ":lzma_source_riscv64" ]
  }
  
  innerapi_tags = [
    "chipsetsdk_sp_indirect",
    "platformsdk_indirect",
  ]
  
  output_name = "lzma"
  
  install_images = [
    "system",
    "updater",
  ]
  
  part_name = "lzma"
  subsystem_name = "thirdparty"
}
```

#### 关键属性

| 属性 | 值 | 说明 |
|------|----|----|
| `output_name` | `"lzma"` | 输出 `liblzma.so` |
| `install_images` | `["system", "updater"]` | 安装到 system 和 updater 分区 |
| `innerapi_tags` | `chipsetsdk_sp_indirect`, `platformsdk_indirect` | 内部 API 标签 |

### 静态库 (`lzma_static`)

**用途**: 主机端工具链

```gn
ohos_static_library("lzma_static") {
  public_configs = [
    ":lzma_config_common",
    ":lzma_config_host",
  ]
  
  if (current_cpu == "arm64") {
    deps = [ ":lzma_source_arm64_host" ]
  } else if (current_cpu == "x86_64" || current_cpu == "x64") {
    deps = [ ":lzma_source_x86_host" ]
  } else if (current_cpu == "riscv64") {
    deps = [ ":lzma_source_riscv64_host" ]
  }
  
  part_name = "lzma"
  subsystem_name = "thirdparty"
}
```

---

## 源文件列表 (lzma.gni)

### ARM64 汇编源文件

```gn
arm64_asm_source = [
  "Asm/arm64/7zAsm.S",      # ARM64 汇编基础宏
  "Asm/arm64/LzmaDecOpt.S", # 优化的 LZMA 解码器
]
```

### C 语言源文件

共 60 个文件，按功能分类：

#### 7z 格式支持
- `7zAlloc.c` - 内存分配
- `7zArcIn.c` - 7z 归档解析
- `7zBuf.c`, `7zBuf2.c` - 缓冲区管理
- `7zCrc.c`, `7zCrcOpt.c` - CRC 计算
- `7zDec.c` - 7z 解码
- `7zFile.c` - 文件 I/O
- `7zStream.c` - 流处理

#### LZMA/LZMA2 压缩
- `LzmaDec.c` - LZMA 解码（核心）
- `LzmaEnc.c` - LZMA 编码
- `Lzma2Dec.c` - LZMA2 解码
- `Lzma2Enc.c` - LZMA2 编码
- `Lzma86Dec.c`, `Lzma86Enc.c` - LZMA86 变体
- `LzmaLib.c` - 库接口封装

#### XZ 格式支持
- `Xz.c` - XZ 基础
- `XzCrc64.c`, `XzCrc64Opt.c` - CRC64
- `XzDec.c` - XZ 解码
- `XzEnc.c` - XZ 编码
- `XzIn.c` - XZ 输入处理

#### 多线程支持
- `LzFindMt.c` - 多线程查找
- `MtCoder.c` - 多线程编码器
- `MtDec.c` - 多线程解码器
- `Threads.c` - 线程抽象

#### 其他算法
- `Bcj2.c` - BCJ2 过滤器
- `Bra.c`, `Bra86.c`, `BraIA64.c` - 分支重定位
- `Delta.c` - Delta 过滤器
- `Ppmd7.c`, `Ppmd7Dec.c`, `Ppmd7Enc.c` - PPMd 压缩
- `Sha256.c`, `Sha256Opt.c` - SHA-256 哈希
- `Sort.c` - 排序算法

#### 基础设施
- `Alloc.c` - 内存分配器
- `CpuArch.c` - CPU 架构检测
- `LzFind.c`, `LzFindOpt.c` - 查找算法

---

## 与上游构建系统对比

| 特性 | 上游 Makefile | OH BUILD.gn |
|------|--------------|-------------|
| 构建工具 | make | gn + ninja |
| 架构检测 | 手动配置 | 自动检测 |
| 交叉编译 | 手动设置 CC/CXX | 内置支持 |
| 安装 | 手动复制 | 自动安装到分区 |
| 安全特性 | 基础 | PAC-RET (ARM64) |
| 多架构 | 支持 | 支持 + RISC-V |

---

## 使用指南

### 依赖 lzma 的模块

在 BUILD.gn 中添加：

```gn
# 设备端使用共享库
external_deps += [ "lzma:lzma_shared" ]

# 主机端工具使用静态库
external_deps += [ "lzma:lzma_static" ]
```

### 头文件引用

```c
// 在源文件中
#include "LzmaDec.h"  // 直接从 C/ 目录包含
```

在 BUILD.gn 中配置：

```gn
config("lzma_config") {
  include_dirs = [ "//third_party/lzma/C" ]
}
```

---

## 修改建议

### 如需添加新架构支持

1. 在 `lzma.gni` 中添加新的汇编源文件列表（如有）
2. 在 `BUILD.gn` 中添加新的 `ohos_source_set`
3. 在 `lzma_shared` 和 `lzma_static` 中添加对应条件分支

### 如需修改编译选项

1. 修改 `lzma_config_common` 影响所有架构
2. 修改特定架构的 `ohos_source_set` 仅影响该架构

---

*最后更新: 2025-02-08*
