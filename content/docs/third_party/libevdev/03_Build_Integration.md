# 03 OH 构建适配

## 3.1 构建系统概述

libevdev 在 OpenHarmony 中使用 **GN (Generate Ninja)** 构建系统，通过 `BUILD.gn` 文件进行配置。与上游的 **Autotools (autoconf/automake)** 构建系统不同，OH 使用 GN 进行更精细的构建控制。

### 构建文件结构

```
third_party/libevdev/
├── BUILD.gn                          # 主构建配置
├── patch/
│   ├── BUILD.gn                      # Patch 应用配置
│   ├── apply_patch.sh                # Patch 应用脚本
│   └── diff_libevdev_mmi/
│       └── libevdev/
│           └── libevdev_0000.diff    # 核心 Patch
```

### 构建流程

```
1. 解析 BUILD.gn
   ↓
2. 执行 patch:apply_patch action
   ↓
3. 应用 libevdev_0000.diff 到输出目录
   ↓
4. 编译生成的源文件 (patch_gen_libevdev-third-mmi)
   ↓
5. 链接为共享库 (libevdev.so)
```

---

## 3.2 BUILD.gn 详细分析

### 主构建配置 (BUILD.gn)

#### 整体结构

```gn
import("//build/ohos.gni")

# 全局变量
gen_dst_dir = root_out_dir + "/diff_libevdev_mmi"

## Build libevdev.so {{{
config("libevdev_config") { ... }
config("libevdev_public_config") { ... }

ohos_source_set("patch_gen_libevdev-third-mmi") { ... }
ohos_shared_library("libevdev") { ... }
## Build libevdev.so }}}
```

#### 编译配置 (config)

##### libevdev_config

```gn
config("libevdev_config") {
  visibility = [ ":*" ]  # 对所有目标可见

  include_dirs = [
    "$gen_dst_dir/libevdev",      # 生成的头文件目录
    "$gen_dst_dir/include",       # 编译产生的 include 目录
  ]

  cflags = [
    "-Wno-unused-parameter",      # 忽略未使用参数警告
    "-Wno-missing-braces",        # 忽略缺少大括号警告
  ]
}
```

**配置说明**：

| 选项 | 值 | 说明 |
|------|-----|------|
| visibility | `":*"` | 配置对所有目标可见 |
| include_dirs | 2 个目录 | 指定头文件搜索路径 |
| cflags | 2 个标志 | 编译器警告抑制 |

##### libevdev_public_config

```gn
config("libevdev_public_config") {
  include_dirs = [
    "$gen_dst_dir/export_include",
    "$gen_dst_dir/libevdev",
  ]

  cflags = []  # 无额外编译标志
}
```

**配置说明**：

| 选项 | 值 | 说明 |
|------|-----|------|
| include_dirs | 2 个目录 | 导出头文件目录 |
| cflags | 空 | 无额外标志 |

---

#### 源文件集合 (ohos_source_set)

```gn
ohos_source_set("patch_gen_libevdev-third-mmi") {
  part_name = "libevdev"
  subsystem_name = "thirdparty"

  sources = [
    root_out_dir + "/diff_libevdev_mmi/libevdev/libevdev-names.c",
    root_out_dir + "/diff_libevdev_mmi/libevdev/libevdev-uinput.c",
    root_out_dir + "/diff_libevdev_mmi/libevdev/libevdev.c",
  ]

  branch_protector_ret = "pac_ret"
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }

  configs = [ ":libevdev_config" ]
  public_configs = [ ":libevdev_public_config" ]

  deps = [ "patch:apply_patch" ]
}
```

**配置说明**：

| 选项 | 值 | 说明 |
|------|-----|------|
| part_name | `libevdev` | 所属组件名 |
| subsystem_name | `thirdparty` | 所属子系统 |
| sources | 3 个文件 | 经过 Patch 处理的源文件 |
| branch_protector_ret | `pac_ret` | PAC (Pointer Authentication) 返回地址保护 |
| sanitize.cfi | `true` | 启用 CFI (Control Flow Integrity) 检查 |
| sanitize.cf_cross_dso | `true` | 跨 DSO 的 CFI 检查 |
| deps | `patch:apply_patch` | 依赖 Patch 应用 |

**安全加固说明**：

```c
/* 
 * CFI (Control Flow Integrity) 安全机制
 * 
 * - cfi = true
 *   启用控制流完整性检查，防止 ROP (Return-Oriented Programming) 攻击
 * 
 * - cfi_cross_dso = true
 *   跨共享库的 CFI 检查，增强安全性
 * 
 * - branch_protector_ret = "pac_ret"
 *   PAC (Pointer Authentication Code) 返回地址保护
 *   验证返回地址的完整性
 */
```

---

#### 共享库目标 (ohos_shared_library)

```gn
ohos_shared_library("libevdev") {
  sources = []  # 无直接源文件，从 deps 获取

  configs = [ ":libevdev_config" ]
  public_configs = [ ":libevdev_public_config" ]

  deps = [":patch_gen_libevdev-third-mmi"]

  public_deps = []

  part_name = "libevdev"
  subsystem_name = "thirdparty"
}
```

**配置说明**：

| 选项 | 值 | 说明 |
|------|-----|------|
| sources | 空 | 通过 deps 引入源文件 |
| deps | `patch_gen_libevdev-third-mmi` | 依赖源文件集合 |
| public_deps | 空 | 无公共依赖 |
| part_name | `libevdev` | 组件名 |

---

## 3.3 Patch 应用机制

### patch/BUILD.gn

```gn
import("//build/ohos.gni")

gen_src_dir = "//third_party/libevdev"
gen_dst_dir = root_out_dir + "/diff_libevdev_mmi"
patches_root_dir = gen_src_dir + "/patch"
build_gn_dir = "$patches_root_dir/diff_libevdev_mmi/libevdev"

action("apply_patch") {
  visibility = [ "*" ]
  script = "${gen_src_dir}/patch/apply_patch.sh"
  inputs = [ "$gen_src_dir" ]
  outputs = [
    "$gen_dst_dir/libevdev/libevdev-names.c",
    "$gen_dst_dir/libevdev/libevdev-uinput.c",
    "$gen_dst_dir/libevdev/libevdev.c",
  ]

  args = [
    rebase_path(gen_src_dir, root_build_dir),
    rebase_path(gen_dst_dir, root_build_dir),
    rebase_path(build_gn_dir, root_build_dir),
  ]
}
```

### apply_patch.sh 脚本

```bash
#!/bin/bash
# Patch 应用脚本

set -e

source_dir=$1      # 源目录
out_dir=$2         # 输出目录
path_file_dir=$3   # Patch 目录

# 1. 创建输出目录
mkdir -p $out_dir

# 2. 复制源文件到输出目录
cp -fra $source_dir/* $out_dir/

# 3. 如果有 install.sh，处理 tarball
if [ -e "$out_dir/install.sh" ]; then
    cd $out_dir
    tar -xvJf libevdev-1.13.0.tar.xz
    cp -rf libevdev-1.13.0/* ./
    ./configure
    cd -
fi

# 4. 应用最新的 .diff 文件
PATCH_FILE=$(realpath $(ls $path_file_dir/*.diff | tail -n 1))
cd $out_dir
patch -p1 -i $PATCH_FILE
```

**脚本执行流程**：

```
apply_patch.sh 执行流程：
1. 检查参数有效性
2. 创建/清理输出目录
3. 复制源文件到输出目录
4. 处理特殊安装脚本（如果存在）
5. 查找并应用最新的 .diff Patch
6. 返回状态码
```

---

## 3.4 与上游构建系统的差异

### 构建系统对比

| 方面 | 上游 (Autotools) | OpenHarmony (GN) |
|------|-----------------|------------------|
| **构建工具** | configure/make | gn/ninja |
| **配置方式** | configure.ac/Makefile.am | BUILD.gn |
| **Patch 处理** | 手动或 quilt | 构建时自动应用 |
| **安全加固** | 可选 | CFI/PAC 强制启用 |
| **输出格式** | .so/.a | .so + 内置构建产物 |

### 编译选项对比

| 选项 | 上游 | OH |
|------|------|-----|
| **警告抑制** | 无/可选 | `-Wno-unused-parameter`, `-Wno-missing-braces` |
| **安全检查** | 无 | CFI, PAC, sanitizers |
| **头文件路径** | PKGCONFIG | include_dirs 配置 |
| **符号导出** | 默认全部 | GN 自动处理 |

---

## 3.5 Inner Kit 导出

### bundle.json 配置

```json
{
  "build": {
    "sub_component": [],
    "inner_kits": [
      {
        "name": "//third_party/libevdev:libevdev",
        "header": {
          "header_files": [
            "libevdev.h",
            "libevdev-uinput.h",
            "libevdev-util.h",
            "libevdev-int.h",
            "libevdev-uinput-int.h"
          ],
          "header_base": "//third_party/libevdev/libevdev"
        }
      }
    ],
    "test": []
  }
}
```

### 导出头文件说明

| 头文件 | 用途 | 稳定性 |
|--------|------|--------|
| `libevdev.h` | 核心公共 API | 稳定 |
| `libevdev-uinput.h` | uinput 虚拟设备 API | 稳定 |
| `libevdev-util.h` | 工具函数 | 稳定 |
| `libevdev-int.h` | 内部接口 | 不稳定 |
| `libevdev-uinput-int.h` | uinput 内部接口 | 不稳定 |

---

## 3.6 编译标志详解

### C 编译器标志

```gn
cflags = [
  "-Wno-unused-parameter",
  "-Wno-missing-braces",
]
```

| 标志 | 作用 | 抑制原因 |
|------|------|---------|
| `-Wno-unused-parameter` | 忽略未使用参数警告 | libevdev API 设计包含可选参数 |
| `-Wno-missing-braces` | 忽略初始化列表缺少大括号警告 | 某些初始化符合风格 |

### 安全加固标志

```gn
sanitize = {
  cfi = true              # Control Flow Integrity
  cfi_cross_dso = true    # 跨 DSO 的 CFI
  debug = false          # 关闭调试模式
}

branch_protector_ret = "pac_ret"  # PAC 返回地址保护
```

| 安全机制 | 作用 | 性能影响 |
|---------|------|---------|
| CFI | 防止控制流劫持 | ~5-10% |
| PAC | 指针认证 | ~2-5% |

---

## 3.7 构建产物

### 输出目录结构

```
out/ohos-arm-release/
└── libevdev/
    ├── libevdev.so                    # 最终共享库
    └── diff_libevdev_mmi/              # 中间构建产物
        └── libevdev/
            ├── libevdev.c              # (已应用 Patch)
            ├── libevdev-uinput.c       # (已应用 Patch)
            ├── libevdev-names.c
            └── ...
```

### 库依赖关系

```bash
# 检查 libevdev.so 的依赖
ldd libevdev.so

# 预期输出（可能包含）：
# libc.so -> C 标准库
# libm.so -> 数学库
# (其他系统库)
```

---

## 3.8 常见构建问题

### 问题 1：Patch 应用失败

**症状**：
```
patch fail. path_file_dir=...
```

**排查步骤**：

```bash
# 1. 检查 Patch 文件是否存在
ls patch/diff_libevdev_mmi/libevdev/*.diff

# 2. 手动应用 Patch 调试
cd out/ohos-arm-release/diff_libevdev_mmi
patch -p1 -i /path/to/libevdev_0000.diff --verbose
```

**解决方案**：
- 确认源文件版本与 Patch 兼容
- 检查 Patch 文件权限

### 问题 2：头文件找不到

**症状**：
```
fatal error: 'libevdev.h' file not found
```

**排查步骤**：

```bash
# 1. 检查 include_dirs 配置
# 2. 确认 gen_dst_dir 生成正确
# 3. 检查头文件是否在正确位置
```

**解决方案**：
- 确认 `apply_patch` action 正确执行
- 验证 include_dirs 路径

---

## 3.9 构建最佳实践

### 开发时构建

```bash
# 仅构建 libevdev
hb set
hb build libevdev

# 查看构建日志
cat out/ohos-arm-release/logs/libevdev.log
```

### 完整系统构建

```bash
# 构建整个 thirdparty 子系统
hb build subsystem:thirdparty
```

### 清理构建

```bash
# 清理 libevdev 构建产物
rm -rf out/ohos-arm-release/diff_libevdev_mmi
rm -rf out/ohos-arm-release/libevdev
hb build libevdev
```

---

## 下一章

下一章将介绍 **libevdev 在 OpenHarmony 中的使用场景**，包括依赖关系图和使用示例。

👉 **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** →
