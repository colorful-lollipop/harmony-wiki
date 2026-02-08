# libbpf OpenHarmony 构建适配详解

> libbpf 在 OpenHarmony 中通过 **构建系统适配** 集成，无源代码 Patch。

---

## 1. 构建系统概述

### 1.1 OH 构建框架

OpenHarmony 使用 **GN (Generate Ninja)** 作为构建系统，而 libbpf 原生使用 **Makefile**。

**适配方式**:
- ✅ 创建 `BUILD.gn` 替代 Makefile
- ✅ 配置 OH 特定的编译选项和依赖
- ✅ 保留上游源代码不变

### 1.2 BUILD.gn 位置

```
/Volumes/lexar/code/d/work/oh/third_party/libbpf/
├── BUILD.gn                    # OH 构建配置（唯一）
└── src/
    └── Makefile              # 上游构建脚本（参考用）
```

---

## 2. BUILD.gn 配置详解

### 2.1 完整配置文件

```gn
# Copyright (C) 2021 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0

import("//build/ohos.gni")
import("//build/ohos/ndk/ndk.gni")

THIRDPARTY_LIBBPF_SUBSYS_NAME = "thirdparty"
THIRDPARTY_LIBBPF_PART_NAME = "libbpf"

config("libbpf_config") {
  cflags = [
    "-Wno-incompatible-pointer-types",
    "-Wimplicit-function-declaration",
    "-Wno-tautological-constant-out-of-range-compare",
    "-Wno-constant-conversion",
    "-Wno-unknown-attributes",
    "-Wno-bitwise-op-parentheses",
    "-Wno-shift-op-parentheses",
    "-Wno-sign-compare",
    "-Wno-unused-function",
    "-fno-omit-frame-pointer",
    "-mno-omit-leaf-frame-pointer",
    "-fno-inline",
    "-fno-optimize-sibling-calls",
    "-ferror-limit=0",
    "-Wno-unused-variable",
    "-Wno-uninitialized",
  ]
  defines = [ "HAVE_ELFIO" ]
}

config("libbpf_public_config") {
  include_dirs = [
    "./src",
    "./include",
    "./include/uapi",
  ]
}

ohos_shared_library("libbpf") {
  branch_protector_ret = "pac_ret"
  external_deps = [
    "elfio:elfio",
    "zlib:libz"
  ]
  sources = [
    "./src/bpf.c",
    "./src/bpf.h",
    "./src/bpf_core_read.h",
    "./src/bpf_endian.h",
    "./src/bpf_gen_internal.h",
    "./src/bpf_helper_defs.h",
    "./src/bpf_helpers.h",
    "./src/bpf_prog_linfo.c",
    "./src/bpf_tracing.h",
    "./src/btf.c",
    "./src/btf.h",
    "./src/btf_dump.c",
    "./src/elf.c",
    "./src/gen_loader.c",
    "./src/hashmap.c",
    "./src/hashmap.h",
    "./src/libbpf.c",
    "./src/libbpf.h",
    "./src/libbpf_common.h",
    "./src/libbpf_errno.c",
    "./src/libbpf_internal.h",
    "./src/libbpf_legacy.h",
    "./src/libbpf_probes.c",
    "./src/libbpf_version.h",
    "./src/netlink.c",
    "./src/nlattr.c",
    "./src/nlattr.h",
    "./src/relo_core.c",
    "./src/relo_core.h",
    "./src/ringbuf.c",
    "./src/skel_internal.h",
    "./src/str_error.c",
    "./src/str_error.h",
    "./src/strset.c",
    "./src/strset.h",
    "./src/zip.c",
  ]
  configs = [ ":libbpf_config" ]
  public_configs = [ ":libbpf_public_config" ]
  output_extension = "so"
  subsystem_name = "${THIRDPARTY_LIBBPF_SUBSYS_NAME}"
  part_name = "${THIRDPARTY_LIBBPF_PART_NAME}"
  install_enable = true
  license_file = "LICENSE.BSD-2-Clause"
}
```

### 2.2 配置分解

#### A. 子系统和部件定义

```gn
THIRDPARTY_LIBBPF_SUBSYS_NAME = "thirdparty"
THIRDPARTY_LIBBPF_PART_NAME = "libbpf"
```

**说明**:
- **子系统**: thirdparty（第三方库子系统）
- **部件名**: libbpf（在 OH 包管理中的标识）

#### B. 编译器配置 (libbpf_config)

**警告抑制** (18 个):

| 警告选项 | 说明 |
|---------|------|
| `-Wno-incompatible-pointer-types` | 忽略不兼容指针类型警告 |
| `-Wimplicit-function-declaration` | 忽略隐式函数声明警告 |
| `-Wno-tautological-constant-out-of-range-compare` | 忽略常量越界比较警告 |
| `-Wno-constant-conversion` | 忽略常量转换警告 |
| `-Wno-unknown-attributes` | 忽略未知属性警告 |
| `-Wno-bitwise-op-parentheses` | 忽略位运算括号警告 |
| `-Wno-shift-op-parentheses` | 忽略移位运算括号警告 |
| `-Wno-sign-compare` | 忽略有符号比较警告 |
| `-Wno-unused-function` | 忽略未使用函数警告 |
| `-Wno-unused-variable` | 忽略未使用变量警告 |
| `-Wno-uninitialized` | 忽略未初始化变量警告 |

**编译选项**:

| 选项 | 说明 |
|------|------|
| `-fno-omit-frame-pointer` | 保留帧指针（便于调试和性能分析） |
| `-mno-omit-leaf-frame-pointer` | ARM64 叶函数保留帧指针 |
| `-fno-inline` | 禁用内联 |
| `-fno-optimize-sibling-calls` | 禁用尾调用优化 |
| `-ferror-limit=0` | 不限制错误数量 |

**为什么需要这些选项？**

libbpf 源代码使用 **内核编程模式**，与标准用户态代码有差异：
- 大量使用内核宏和类型
- 依赖编译器扩展
- 使用特殊的内存和指针操作

#### C. 条件编译定义

```gn
defines = [ "HAVE_ELFIO" ]
```

**作用**: 启用 **ELFIO 库** 替代标准的 libelf。

**原因**: OpenHarmony 使用 elfio 作为 ELF 文件解析库，而非 Linux 标准的 libelf。

**代码示例** (src/elf.c):
```c
#ifdef HAVE_ELFIO
    // 使用 ELFIO 库的代码
#else
    // 使用 libelf 的代码
#endif
```

#### D. 公共配置 (libbpf_public_config)

```gn
include_dirs = [
  "./src",
  "./include",
  "./include/uapi",
]
```

**说明**: 定义公共头文件路径，依赖者可以直接引用 libbpf 的头文件。

#### E. 外部依赖

```gn
external_deps = [
  "elfio:elfio",
  "zlib:libz"
]
```

**依赖说明**:

| 依赖 | 库名 | 用途 |
|------|------|------|
| elfio | elfio | ELF 文件解析（替代 libelf） |
| zlib | libz | 压缩和解压缩 |

**配置文件** (bundle.json):
```json
{
  "deps": {
    "components": [ "elfio", "zlib" ]
  }
}
```

#### F. 构建目标配置

```gn
ohos_shared_library("libbpf") {
  branch_protector_ret = "pac_ret"
  output_extension = "so"
  subsystem_name = "${THIRDPARTY_LIBBPF_SUBSYS_NAME}"
  part_name = "${THIRDPARTY_LIBBPF_PART_NAME}"
  install_enable = true
  license_file = "LICENSE.BSD-2-Clause"
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **构建目标** | `ohos_shared_library` | 动态共享库 |
| **输出扩展名** | `.so` | Linux/Unix 共享库 |
| **分支保护** | `pac_ret` | ARM64 PAC-RET（指针认证） |
| **子系统** | thirdparty | 所属子系统 |
| **部件名** | libbpf | 组件标识 |
| **安装启用** | true | 安装到 OH 系统 |
| **许可证文件** | LICENSE.BSD-2-Clause | OAT 审计使用的许可证 |

---

## 3. 与上游构建系统的差异

### 3.1 源文件对比

**上游 Makefile (src/Makefile)**:

```makefile
OBJS = bpf.o btf.o libbpf.o libbpf_errno.o netlink.o nlattr.o \
       str_error.o libbpf_probes.o bpf_prog_linfo.o btf_dump.o \
       hashmap.o ringbuf.o strset.o linker.o gen_loader.o \
       relo_core.o usdt.o zip.o elf.o

# 总计: 19 个 C 文件
```

**OH BUILD.gn**:

```gn
sources = [
  "./src/bpf.c",
  "./src/bpf_prog_linfo.c",
  "./src/btf.c",
  "./src/btf_dump.c",
  "./src/elf.c",
  "./src/gen_loader.c",
  "./src/hashmap.c",
  "./src/libbpf.c",
  "./src/libbpf_errno.c",
  "./src/libbpf_probes.c",
  "./src/netlink.c",
  "./src/nlattr.c",
  "./src/relo_core.c",
  "./src/ringbuf.c",
  "./src/str_error.c",
  "./src/strset.c",
  "./src/zip.c",
  # ... (包含头文件)
]

# 总计: 17 个 C 文件
```

### 3.2 禁用的功能

| 功能 | 源文件 | BUILD.gn 状态 | 影响分析 |
|------|---------|--------------|----------|
| **BPF 静态链接器** | `src/linker.c` | ❌ 未包含 | 无法使用 BPF 静态链接功能 |
| **USDT 支持** | `src/usdt.c` | ❌ 未包含 | 无法使用用户态动态追踪（DTrace 风格） |

**为什么禁用？**

1. **linker.c**: OH 网络和性能分析场景不需要静态链接 BPF 对象
2. **usdt.c**: OH 当前不使用 USDT 探针（主要是 DTrace 风格）

**如果需要启用**:

```gn
sources = [
  # ... 现有源文件
  "./src/linker.c",   # 添加 linker.c
  "./src/usdt.c",     # 添加 usdt.c
]
```

### 3.3 构建产物对比

| 构建系统 | 静态库 | 动态库 |
|---------|---------|--------|
| **上游 Makefile** | ✅ `libbpf.a` | ✅ `libbpf.so` |
| **OH BUILD.gn** | ❌ | ✅ `libbpf.so` |

**OH 策略**: 仅构建动态共享库，满足 OH 所有使用场景。

### 3.4 ELF 库差异

| 项目 | 上游 | OH |
|------|------|-----|
| **ELF 库** | libelf | elfio |
| **条件编译** | 默认 libelf | `HAVE_ELFIO` |
| **pkg-config** | 使用 libelf | 使用 elfio |

**适配代码** (src/elf.c):
```c
#ifdef HAVE_ELFIO
    // OH: 使用 ELFIO 库
    #include <elfio/elfio.hpp>
#else
    // 上游: 使用 libelf
    #include <libelf.h>
#endif
```

---

## 4. 构建流程

### 4.1 OH 构建步骤

```bash
# 1. GN 生成构建文件
gn gen out/ohos

# 2. Ninja 编译
ninja -C out/ohos //third_party/libbpf:libbpf

# 3. 输出产物
# out/ohos/third_party/libbpf/libbpf.so
```

### 4.2 构建产物

| 文件 | 路径 | 说明 |
|------|------|------|
| **libbpf.so** | `out/ohos/.../libbpf.so` | 动态共享库 |
| **头文件** | `src/`, `include/`, `include/uapi/` | 公共 API 头文件 |

### 4.3 安装路径

```bash
# 安装到 OH 系统路径
install_enable = true

# 实际安装路径（标准系统）
# /system/lib64/libbpf.so
# /usr/include/libbpf.h
```

---

## 5. 依赖者集成指南

### 5.1 引入依赖

在模块的 `BUILD.gn` 中添加：

```gn
import("//build/ohos.gni")

ohos_shared_library("my_module") {
  sources = [ "my_module.c" ]

  # 引入 libbpf
  external_deps = [ "libbpf:libbpf" ]

  public_configs = [ ":my_module_config" ]
}
```

### 5.2 头文件引用

```c
#include "libbpf.h"  // 主 API
#include "bpf.h"     // 底层 BPF 操作
#include "btf.h"     // BTF 类型系统
```

### 5.3 链接方式

**动态链接** (OH 默认):

```c
// 运行时动态链接
// 不需要静态链接 libbpf
#include "libbpf.h"

struct bpf_object *obj = bpf_object__open("prog.o");
// ...
```

---

## 6. 版本升级指南

### 6.1 同步上游版本

```bash
cd /Volumes/lexar/code/d/work/oh/third_party/libbpf

# 1. 使用上游同步脚本
./scripts/sync-kernel.sh

# 2. 检查 BUILD.gn 源文件列表
# 对比 src/Makefile 中的 OBJS

# 3. 更新版本信息
# 更新 bundle.json 中的 version
# 更新 README.OpenSource 中的 Version Number

# 4. 更新 checkpoint commits
# 更新 CHECKPOINT-COMMIT 和 BPF-CHECKPOINT-COMMIT

# 5. 测试
# 编译 OH 系统
# 测试 netmanager_base 和 hiebpf 功能
```

### 6.2 必须保留的配置

升级 libbpf 版本时，必须保留以下配置：

| 配置 | 位置 | 说明 |
|------|------|------|
| `HAVE_ELFIO` | `defines` | 使用 ELFIO 库 |
| 源文件排除 | `sources` | 不包含 linker.c 和 usdt.c |
| 警告抑制 | `cflags` | 所有 `-Wno-*` 选项 |
| 编译选项 | `cflags` | `-fno-omit-frame-pointer` 等 |
| PAC-RET | `branch_protector_ret` | ARM64 分支保护 |
| 动态库 | `ohos_shared_library` | 不构建静态库 |

### 6.3 升级验证清单

- [ ] 编译 libbpf.so 成功
- [ ] 所有依赖者编译成功
- [ ] netmanager_base 网络功能正常
- [ ] hiebpf 性能追踪功能正常
- [ ] 无新的编译警告或错误

---

## 7. 常见构建问题

### 7.1 编译错误: 找不到 elfio

**错误信息**:
```
fatal error: elfio/elfio.hpp: No such file or directory
```

**解决方案**:
- 确认 elfio 组件已正确配置
- 检查 bundle.json 中的依赖

### 7.2 链接错误: undefined reference

**错误信息**:
```
undefined reference to 'bpf_some_function'
```

**解决方案**:
- 确认所有必要的源文件在 BUILD.gn 中
- 检查 linker.c 和 usdt.c 是否被需要

### 7.3 警告: implicit function declaration

**警告信息**:
```
warning: implicit declaration of function 'xxx'
```

**解决方案**:
- 检查 libbpf 内部头文件是否正确包含
- 确认 BUILD.gn 中所有头文件路径正确

---

## 8. 总结

### 8.1 核心适配点

| 适配点 | OH 方案 |
|--------|---------|
| **构建系统** | GN (BUILD.gn) |
| **ELF 库** | ELFIO (HAVE_ELFIO) |
| **编译产物** | 仅动态库 (libbpf.so) |
| **功能禁用** | linker.c, usdt.c |
| **编译选项** | 大量警告抑制 + 保留帧指针 |
| **安全特性** | ARM64 PAC-RET |

### 8.2 适配优势

- ✅ **无源代码 Patch**: 简化升级和维护
- ✅ **构建层适配**: 保留上游代码纯净性
- ✅ **功能裁剪**: 禁用不需要的功能（linker, usdt）
- ✅ **OH 兼容**: 使用 ELFIO 替代 libelf

### 8.3 维护建议

1. **升级时**: 保留 BUILD.gn 所有配置
2. **添加功能**: 优先通过 BUILD.gn 添加源文件
3. **调试问题**: 保留帧指针（-fno-omit-frame-pointer）便于调试
4. **性能优化**: 谨慎移除编译选项，需充分测试

---

**下一步**: 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md) 了解依赖关系与使用场景
