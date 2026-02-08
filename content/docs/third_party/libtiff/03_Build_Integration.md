# OH 构建适配

## 概述

libtiff 在 OpenHarmony 中通过 **BUILD.gn 构建系统**完成集成，无需修改源代码。适配包括：

1. **BUILD.gn 构建配置** - GN 构建系统集成
2. **install.sh 构建脚本** - Autotool + CMake 混合构建
3. **port/ 跨平台适配** - 平台兼容层
4. **条件编译开关** - 压缩算法功能控制

---

## BUILD.gn 结构说明

### 完整路径

```
third_party/libtiff/BUILD.gn
```

### 构建目标

| 目标名称 | 类型 | 输出 |
|---------|------|------|
| `libtiff` | `ohos_shared_library` | `libtiff.so` |
| `libtiff_configure` | `action` | 执行 `install.sh` |

### 子系统和部件

```gn
subsystem_name = "thirdparty"
part_name = "libtiff"
output_name = "libtiff"
output_extension = "so"
install_images = [ "system" ]
```

**说明**:
- 安装到 `system` 镜像
- 输出为共享库 `libtiff.so`

---

## 关键编译选项

### 1. 压缩算法条件编译

BUILD.gn 通过 `declare_args()` 定义了丰富的压缩算法开关：

```gn
declare_args() {
    enable_lzw = true          # LZW 压缩
    enable_packbits = true      # PackBits 压缩
    enable_thunder = true       # Thunder 压缩
    enable_next = true         # NeXT 压缩
    enable_jpeg = true         # JPEG 压缩
    enable_ojpeg = true        # Old JPEG 压缩
    enable_ccitt = true        # CCITT G3/G4 压缩
    enable_jbig = false        # JBIG 压缩（禁用）
    enable_zip = true          # Deflate 压缩
    enable_pixarlog = true     # PixarLog 压缩
    enable_logluv = true       # LogLuv 压缩
    enable_lerc = false        # LERC 压缩（禁用）
    enable_lzma = false        # LZMA 压缩（禁用）
    enable_zstd = false        # Zstd 压缩（禁用）
    enable_webp = false        # WebP 压缩（禁用）
}
```

**启用状态汇总**:

| 压缩算法 | OH 状态 | 依赖 |
|---------|---------|------|
| LZW | ✅ 启用 | 无 |
| PackBits | ✅ 启用 | 无 |
| Thunder | ✅ 启用 | 无 |
| Next | ✅ 启用 | 无 |
| JPEG | ✅ 启用 | libjpeg-turbo |
| Old JPEG | ✅ 启用 | libjpeg-turbo |
| CCITT G3/G4 | ✅ 启用 | 无 |
| Deflate | ✅ 启用 | zlib |
| PixarLog | ✅ 启用 | zlib |
| LogLuv | ✅ 启用 | 无 |
| JBIG | ❌ 禁用 | - |
| LERC | ❌ 禁用 | - |
| LZMA | ❌ 禁用 | lzma（依赖可用但未启用） |
| Zstd | ❌ 禁用 | - |
| WebP | ❌ 禁用 | - |

### 2. 条件编译宏定义

根据 `enable_*` 变量，BUILD.gn 自动添加对应的 `#define`:

```gn
if (enable_lzw) {
    defines += [ "LZW_SUPPORT" ]
}
if (enable_packbits) {
    defines += [ "PACKBITS_SUPPORT" ]
}
if (enable_jpeg) {
    defines += [ "JPEG_SUPPORT" ]
    external_deps += [ "libjpeg-turbo:turbojpeg" ]
}
if (enable_zip) {
    defines += [ "ZIP_SUPPORT" ]
    external_deps += [ "zlib:libz" ]
}
# ... 其他压缩算法类似
```

**源文件条件编译**:

```gn
if (enable_lzw) {
    all_libtiff_sources += [ "libtiff/tif_lzw.c" ]
}
if (enable_jpeg) {
    all_libtiff_sources += [ "libtiff/tif_jpeg.c" ]
}
# ... 其他压缩算法类似
```

**说明**: 只有启用的压缩算法对应的源文件才会被编译。

### 3. 外部依赖

根据启用的压缩算法，BUILD.gn 自动添加外部依赖：

```gn
external_deps = []

if (enable_jpeg) {
    external_deps += [ "libjpeg-turbo:turbojpeg" ]
}
if (enable_ojpeg) {
    external_deps += [ "libjpeg-turbo:turbojpeg" ]
}
if (enable_zip) {
    external_deps += [ "zlib:libz" ]
}
if (enable_pixarlog) {
    external_deps += [ "zlib:libz" ]
}
if (enable_lzma) {
    external_deps += [ "lzma:lzma_shared" ]
}
```

**依赖汇总**:

| 依赖库 | 用途 | 条件 |
|-------|------|------|
| **libjpeg-turbo** | JPEG/Old JPEG 压缩 | enable_jpeg / enable_ojpeg |
| **zlib** | Deflate/PixarLog 压缩 | enable_zip / enable_pixarlog |
| **lzma** | LZMA 压缩 | enable_lzma（当前禁用） |

### 4. 编译配置

**输出映射文件**:

```gn
ldflags = [ "-Wl,-Map=libtiff.map" ]
```

**说明**: 生成符号映射文件 `libtiff.map`，用于链接优化。

---

## 构建流程

### 1. libtiff_configure Action

BUILD.gn 定义了一个自定义 action `libtiff_configure`:

```gn
action("libtiff_configure") {
   script = "install.sh"
   args = [
        rebase_path(src_path),
        rebase_path(out_dir),
   ]

   inputs = []
   foreach(src, all_libtiff_sources) {
        inputs += [ rebase_path(src_path) + "/" + src ]
   }

   outputs = []
   foreach(src, all_libtiff_sources) {
        outputs += ["$code_dir/" + src]
   }

   outputs += [ "$code_dir/config/config.h" ]
}
```

**说明**:
- **输入**: 所有源文件
- **输出**: 复制后的源文件 + `config/config.h`
- **脚本**: `install.sh`

### 2. install.sh 脚本

**完整内容**:

```bash
#!/bin/bash

SRC_DIR="$1"
CODE_DIR="$2"

set -e
if [ "$SRC_DIR" == "" ] || [ "$CODE_DIR" == "" ]; then
    exit 1
fi

mkdir -p $CODE_DIR
cp -r $SRC_DIR/* $CODE_DIR
rm -rf $CODE_DIR/.git

CURRENT_DIR=$(pwd)
cd $CODE_DIR
sh ./autogen.sh
sh ./configure
cmake . -DCMAKE_BUILD_TYPE=Release
cd "$CURRENT_DIR"
```

**执行步骤**:

1. **复制源码**:
   ```bash
   cp -r $SRC_DIR/* $CODE_DIR
   ```

2. **执行 autogen.sh**:
   ```bash
   sh ./autogen.sh
   ```
   生成 `configure` 脚本和 `aclocal.m4` 等文件。

3. **执行 configure**:
   ```bash
   sh ./configure
   ```
   配置构建环境，生成 `Makefile` 和 `config.h`。

4. **执行 CMake**:
   ```bash
   cmake . -DCMAKE_BUILD_TYPE=Release
   ```
   生成 CMake 构建文件。

**输出目录**:
- `root_out_dir/third_party_libtiff`
- 最终生成 `config/config.h`

### 3. libtiff 共享库目标

```gn
ohos_shared_library("libtiff") {
    public_configs = [ ":libtiff_config" ]
    sources = []
    defines = []
    external_deps = []

    # 添加压缩算法 defines 和 external_deps
    # ... (见上文)

    foreach(src, all_libtiff_sources) {
        sources += [ "$code_dir/" + src ]
    }

    deps = [ ":libtiff_configure" ]

    subsystem_name = "thirdparty"
    part_name = "libtiff"
    output_name = "libtiff"
    output_extension = "so"
    install_images = [ "system" ]
    ldflags = [ "-Wl,-Map=libtiff.map" ]
}
```

**说明**:
- **依赖**: `libtiff_configure`（先执行 install.sh）
- **源文件**: 从 `$code_dir/libtiff` 读取（install.sh 复制后的文件）
- **输出**: `libtiff.so`

---

## 头文件配置

### libtiff_config 公共配置

```gn
config("libtiff_config") {
    visibility = [ ":*" ]
    include_dirs = [
        "$code_dir/libtiff",
        "$code_dir/config",
    ]
}
```

**说明**:
- 添加 `libtiff/` 头文件路径
- 添加 `config/` 头文件路径（包含 `config.h`）

### bundle.json 头文件导出

```json
{
    "name": "@ohos/libtiff",
    "component": {
        "inner_kits": [
            {
                "name": "//third_party/libtiff:libtiff",
                "header": {
                    "header_files": [
                        "tiff.h",
                        "tiffio.h"
                    ],
                    "header_base": "//third_party/libtiff/libtiff"
                }
            }
        ]
    }
}
```

**导出头文件**:
- `tiff.h` - TIFF 基础定义和常量
- `tiffio.h` - TIFF I/O 接口

**头文件路径**: `//third_party/libtiff/libtiff`

---

## port/ 跨平台适配

### 目录结构

```
port/
├── CMakeLists.txt          # CMake 构建配置
├── Makefile.am            # Autotools 构建配置
├── README                # 说明文档
├── dummy.c              # CMake 占位源文件
├── getopt.c            # getopt 函数实现
├── libport.h           # 跨平台接口头文件
├── libport_config.h.in           # CMake 配置模板
├── libport_config.h.cmake.in     # CMake 配置模板
└── libport_config.vc.h         # Visual Studio 配置
```

### libport.h 跨平台接口

**关键代码**:

```c
#ifndef _LIBPORT_
#define _LIBPORT_

#include <libport_config.h>

#if HAVE_GETOPT
#if HAVE_UNISTD_H
#include <unistd.h>
#endif
#else

int getopt(int argc, char *const argv[], const char *optstring);
extern char *optarg;
extern int opterr;
extern int optind;
extern int optopt;

#endif

#endif /* ndef _LIBPORT_ */
```

**说明**:
- 如果系统有 `getopt`，使用系统实现
- 如果没有，使用 `getopt.c` 中的实现

### getopt.c 实现

**用途**: 提供 BSD 风格的 getopt 函数实现

**适用场景**: 缺少 getopt 函数的平台（如某些嵌入式系统）

**注意**: port/ 目录属于 libtiff 原有的跨平台适配，**非 OH 特有**。

---

## 与上游构建系统的差异

### 上游构建系统

libtiff 上游支持多种构建系统：

1. **Autotools**: `./configure && make`
2. **CMake**: `cmake && make`
3. **nmake**: Windows Visual Studio

### OH 构建系统

OpenHarmony 使用 **GN + Ninja** 构建系统：

1. **BUILD.gn**: GN 构建配置
2. **install.sh**: 调用 autotools + cmake
3. **Ninja**: 实际编译

**混合构建流程**:

```
GN (BUILD.gn)
  ↓
install.sh (调用 autogen.sh, configure, cmake)
  ↓
CMake (生成 CMakeLists.txt)
  ↓
Ninja (实际编译)
```

**说明**: OH 使用混合构建流程，利用了上游的 autotools 和 cmake 配置。

---

## 输出文件和安装位置

### 输出文件

| 文件 | 路径 | 说明 |
|-----|------|------|
| `libtiff.so` | `root_out_dir/lib/libtiff.so` | 共享库 |
| `libtiff.map` | 构建输出目录 | 符号映射文件 |

### 安装位置

```gn
install_images = [ "system" ]
```

**说明**: libtiff.so 安装到 `system` 镜像中，系统启动时自动加载。

### 中间输出

```
root_out_dir/third_party_libtiff/
├── libtiff/          # 源代码（从 install.sh 复制）
├── config/           # 配置文件（包含 config.h）
└── ...               # 其他文件
```

---

## 构建配置示例

### 默认配置

使用默认的压缩算法配置：

```gn
# 无需额外配置，使用默认值
```

### 禁用 JPEG 压缩

```gn
declare_args() {
    enable_jpeg = false    # 禁用 JPEG
    # 其他压缩算法保持默认
}
```

### 启用 LZMA 压缩

```gn
declare_args() {
    enable_lzma = true    # 启用 LZMA
    # LZMA 需要外部依赖: lzma:lzma_shared
}
```

**注意**: 需确保 `lzma` 组件已集成到 OH。

---

## 常见问题排查

### 问题 1: 编译错误 "undefined reference to TIFF..."

**可能原因**:
- 压缩算法未启用，但使用了相关函数
- 外部依赖未正确链接

**解决方法**:
- 检查 `enable_*` 配置是否正确
- 检查 `external_deps` 是否包含所需依赖

### 问题 2: 头文件找不到

**可能原因**:
- 头文件路径配置错误
- 未正确引用 libtiff 组件

**解决方法**:
- 检查 `include_dirs` 配置
- 在 BUILD.gn 中添加 `external_deps += [ "libtiff:libtiff" ]`

### 问题 3: install.sh 执行失败

**可能原因**:
- 缺少 autotools 或 cmake
- 权限不足

**解决方法**:
- 确保构建环境包含 autotools 和 cmake
- 检查文件权限

---

## 总结

### BUILD.gn 关键特性

| 特性 | 说明 |
|-----|------|
| **条件编译** | 通过 `enable_*` 变量控制压缩算法 |
| **自动依赖** | 根据压缩算法自动添加外部依赖 |
| **混合构建** | GN + install.sh (autotools + cmake) |
| **输出配置** | 生成 libtiff.so 和符号映射文件 |
| **跨平台** | port/ 目录提供兼容层 |

### 构建流程总结

```
1. GN 解析 BUILD.gn
2. 执行 install.sh
   - 复制源码
   - 运行 autogen.sh
   - 运行 configure
   - 运行 cmake
3. 编译 libtiff 源文件
4. 链接生成 libtiff.so
5. 安装到 system 镜像
```

---

**文档版本**: 1.0
**最后更新**: 2026年2月8日
