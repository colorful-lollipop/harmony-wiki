# 03 OH 构建适配

> 详细分析 gptfdisk 的 OpenHarmony 构建配置

---

## 3.1 BUILD.gn 完整配置

### 文件位置

`third_party/gptfdisk/BUILD.gn`

### 完整内容

```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.

import("//build/ohos.gni")
import("//build/ohos/ndk/ndk.gni")

config("gptdisk_config") {
  cflags_cc = [
    "-Wall",
    "-D_FILE_OFFSET_BITS=64",
    "-Wno-unused-parameter",
    "-Wno-pragma-pack",
    "-Wno-error=header-hygiene",
    "-Wno-register",
    "-Wno-unused-but-set-variable",
  ]
}

ohos_executable("sgdisk") {
  install_enable = true
  sources = [
    "attributes.cc",
    "basicmbr.cc",
    "bsd.cc",
    "crc32.cc",
    "diskio-unix.cc",
    "diskio.cc",
    "gpt.cc",
    "gptcl.cc",
    "gptpart.cc",
    "guid.cc",
    "mbr.cc",
    "mbrpart.cc",
    "parttypes.cc",
    "sgdisk.cc",
    "support.cc",
  ]
  public_configs = [ ":gptdisk_config" ]
  external_deps = [
    "e2fsprogs:libext2_uuid",
    "popt:popt_static",
  ]
  subsystem_name = "thirdparty"
  part_name = "gptfdisk"
  install_images = [ "system" ]
}
```

---

## 3.2 配置项详解

### config("gptdisk_config")

#### 编译器标志 (cflags_cc)

| 标志 | 用途 | 说明 |
|------|------|------|
| `-Wall` | 启用所有警告 | GCC 标准警告 |
| `-D_FILE_OFFSET_BITS=64` | 大文件支持 | 支持 >2TB 磁盘 |
| `-Wno-unused-parameter` | 抑制未使用参数警告 | 代码风格兼容 |
| `-Wno-pragma-pack` | 抑制 pragma pack 警告 | 字节对齐相关 |
| `-Wno-error=header-hygiene` | 头文件卫生检查降级 | 第三方库兼容 |
| `-Wno-register` | 抑制 register 关键字警告 | C++17 兼容 |
| `-Wno-unused-but-set-variable` | 抑制变量设置未使用警告 | 代码风格兼容 |

**为什么需要这些警告抑制?**

gptfdisk 是成熟的 C++ 项目，使用了一些传统 C++ 特性：
- `register` 关键字 (C++17 已弃用)
- 某些参数/变量在特定配置下未使用
- 字节对齐 pragma 的使用

### ohos_executable("sgdisk")

#### 源文件 (sources)

| 文件 | 类别 | 功能 |
|------|------|------|
| `sgdisk.cc` | 入口 | 命令行解析，含 `--ohos-dump` |
| `gptcl.cc` | 核心 | GPT 命令行处理逻辑 |
| `gpt.cc` | 核心 | GPT 数据结构操作 |
| `gptpart.cc` | 核心 | GPT 分区操作 |
| `guid.cc` | 核心 | GUID 处理 |
| `parttypes.cc` | 数据 | 分区类型代码表 |
| `diskio.cc` | I/O | 磁盘 I/O 抽象层 |
| `diskio-unix.cc` | I/O | Unix 平台 I/O 实现 |
| `basicmbr.cc` | MBR | MBR 数据结构 |
| `mbr.cc` | MBR | MBR 操作 |
| `mbrpart.cc` | MBR | MBR 分区操作 |
| `bsd.cc` | 兼容 | BSD disklabel 支持 |
| `attributes.cc` | 属性 | GPT 属性处理 |
| `crc32.cc` | 工具 | CRC32 校验 |
| `support.cc` | 工具 | 辅助函数 |

**未包含的文件**:

| 文件 | 排除原因 |
|------|---------|
| `gdisk.cc` | 交互式 TUI，不需要 |
| `cgdisk.cc` | curses GUI，不需要 |
| `gptcurses.cc` | curses 库依赖，不需要 |
| `fixparts.cc` | MBR 修复工具，不需要 |
| `diskio-windows.cc` | Windows 平台，不需要 |

#### 依赖 (external_deps)

| 依赖 | 目标 | 用途 |
|------|------|------|
| `e2fsprogs` | `libext2_uuid` | UUID 生成与解析 |
| `popt` | `popt_static` | 命令行参数解析 |

**为什么不使用系统库?**

- **e2fsprogs**: OH 使用自己维护的 e2fsprogs 版本
- **popt**: 使用静态链接，避免运行时依赖

#### 安装配置

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `install_enable` | `true` | 安装到系统镜像 |
| `install_images` | `["system"]` | 安装到 system 分区 |
| `subsystem_name` | `"thirdparty"` | 子系统归属 |
| `part_name` | `"gptfdisk"` | 组件名 |

**安装路径**: `/system/bin/sgdisk`

---

## 3.3 与上游构建对比

### 上游 Makefile (关键部分)

```makefile
# 上游 Makefile 结构
CXXFLAGS += -D_FILE_OFFSET_BITS=64
LIBS = -luuid -lpopt

TARGETS = gdisk cgdisk sgdisk fixparts

sgdisk: sgdisk.o gptcl.o ...
    $(CXX) $(CXXFLAGS) $(LDFLAGS) -o sgdisk sgdisk.o ... $(LIBS)
```

### OH BUILD.gn 对比

| 对比项 | 上游 | OH |
|-------|------|-----|
| **构建系统** | GNU Make | GN |
| **目标数量** | 4 (gdisk, cgdisk, sgdisk, fixparts) | 1 (仅 sgdisk) |
| **链接方式** | 动态链接 | 静态链接 (popt) |
| **依赖声明** | 系统包管理器 | GN external_deps |
| **安装控制** | `make install` | `install_enable` |
| **交叉编译** | 手动配置 TARGET | GN 工具链自动处理 |

### 功能裁剪

```
上游构建:
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐
│  gdisk  │  │ cgdisk  │  │ sgdisk  │  │ fixparts │
└────┬────┘  └────┬────┘  └────┬────┘  └────┬─────┘
     │            │            │            │
     └────────────┴────────────┴────────────┘
                  │
          共享核心代码

OH 构建:
               ┌─────────┐
               │ sgdisk  │
               └────┬────┘
                    │
            静态链接核心代码
```

---

## 3.4 关键编译选项

### `-D_FILE_OFFSET_BITS=64`

**重要性**: ⭐⭐⭐⭐⭐ (关键)

**作用**: 启用大文件支持 (Large File Support)

**背景**:
- 32 位系统上，`off_t` 默认为 32 位，最大支持 2GB
- GPT 分区表支持最大 8 ZiB (9.4 ZB)
- 现代磁盘已超过 2TB

**代码影响**:
```cpp
// 启用后，以下函数支持 64 位偏移:
// open(), lseek(), pread(), pwrite(), ftruncate(), etc.
```

### 警告抑制选项

**使用场景**:

| 警告 | 触发原因 | 处理策略 |
|------|---------|---------|
| `unused-parameter` | 某些回调函数参数在特定配置下未使用 | 抑制，非错误 |
| `pragma-pack` | 使用 `#pragma pack` 进行字节对齐 | 抑制，有意为之 |
| `header-hygiene` | 头文件包含顺序或重复包含 | 抑制，第三方库兼容 |
| `register` | 代码使用 `register` 关键字 | 抑制，C++17 兼容 |
| `unused-but-set-variable` | 变量被赋值但未使用 | 抑制，代码风格 |

---

## 3.5 依赖分析

### e2fsprogs:libext2_uuid

**功能**: UUID (通用唯一识别码) 生成和解析

**在 gptfdisk 中的作用**:
- 生成新的 GPT GUID
- 解析 GUID 字符串
- 验证 GUID 格式

**BUILD.gn 中的依赖**:
```gn
external_deps = [
    "e2fsprogs:libext2_uuid",
    ...
]
```

**对应源码**:
```cpp
// guid.cc
#include <uuid/uuid.h>  // 来自 e2fsprogs

void GUIDData::GenerateGUID() {
    uuid_generate(uuidData);  // 使用 libext2_uuid
}
```

### popt:popt_static

**功能**: 命令行选项解析库

**在 ggdisk 中的作用**:
- 解析 `--new`, `--delete`, `--typecode` 等选项
- 处理短选项和长选项
- 参数验证和转换

**BUILD.gn 中的依赖**:
```gn
external_deps = [
    "popt:popt_static",  // 静态链接
    ...
]
```

**对应源码**:
```cpp
// gptcl.cc
#include <popt.h>

// 选项定义
struct poptOption optionsTable[] = {
    {"new", 'n', POPT_ARG_STRING, ...},
    {"delete", 'd', POPT_ARG_STRING, ...},
    ...
};
```

---

## 3.6 构建产物

### 输出文件

| 文件 | 路径 | 大小 (估算) |
|------|------|------------|
| `sgdisk` | `out/.../system/bin/sgdisk` | ~200KB |

### 依赖库 (静态链接)

| 库 | 大小 (估算) | 来源 |
|---|------------|------|
| libext2_uuid | ~30KB | e2fsprogs |
| libpopt | ~50KB | popt |

**总二进制大小**: ~280KB (含静态链接库)

### 安装验证

构建完成后，验证 sgdisk 是否正确安装:

```bash
# 检查文件存在
ls out/.../system/bin/sgdisk

# 检查文件类型
file out/.../system/bin/sgdisk
# 输出: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, ...

# 检查符号 (确认 --ohos-dump 相关代码包含)
nm out/.../system/bin/sgdisk | grep ohos_dump
```

---

## 3.7 构建命令参考

### 完整构建

```bash
# 在 OH 源码根目录
./build.sh --product {product_name} --build-target //third_party/gptfdisk:sgdisk
```

### 仅构建 gptfdisk

```bash
# 使用 GN
./prebuilts/build-tools/linux-x64/bin/gn gen out/{target}
./prebuilts/build-tools/linux-x64/bin/ninja -C out/{target} //third_party/gptfdisk:sgdisk
```

### 清理构建

```bash
# 删除输出
rm -rf out/{target}/obj/third_party/gptfdisk/

# 重新构建
./build.sh ...
```

---

## 3.8 常见问题

### Q: 编译错误 "uuid.h not found"

**原因**: e2fsprogs 未先编译

**解决**:
```bash
# 确保 e2fsprogs 在依赖中声明
# 检查 bundle.json deps
```

### Q: 链接错误 "undefined reference to popt..."

**原因**: popt 静态库未找到

**解决**:
```bash
# 确认 popt 已编译
./build.sh --build-target //third_party/popt:popt_static
```

### Q: 警告过多导致编译失败

**原因**: 某些警告被当作错误

**解决**: BUILD.gn 中已添加 `-Wno-error=...` 抑制

---

## 3.9 与上游同步建议

### 升级流程

1. **备份 BUILD.gn**
   ```bash
   cp BUILD.gn BUILD.gn.bak
   ```

2. **检查上游变更**
   - 查看上游 Makefile 是否有新源文件
   - 检查新编译选项

3. **更新 BUILD.gn**
   - 添加/删除源文件
   - 更新编译选项

4. **保留 OH 特有配置**
   - `ohos_executable()`
   - `external_deps`
   - `install_images`

5. **测试构建**
   ```bash
   ./build.sh --build-target //third_party/gptfdisk:sgdisk
   ```
