# libfuse OpenHarmony 构建系统集成

本文档说明 libfuse 在 OpenHarmony 中的构建系统适配、关键编译选项以及与上游构建系统的差异。

---

## 概述

libfuse 原生使用 **Meson + Ninja** 构建系统，在 OpenHarmony 中适配为使用 **GN (Generate Ninja)** 构建系统。

### 构建系统对比

| 特性 | 上游 (Meson) | OpenHarmony (GN) |
|------|-------------|-----------------|
| 配置文件 | `meson.build` | `BUILD.gn` |
| 构建目标 | `shared_library` | `ohos_shared_library` |
| 选项配置 | `meson_options.txt` | 直接在 cflags 中定义 |
| 源文件选择 | 自动收集 | 手动列表 |
| 部署信息 | 无 | install_images, part_name 等 |

---

## BUILD.gn 结构说明

### 文件位置

`/third_party/libfuse/BUILD.gn`

### 整体结构

```gn
import("//build/ohos.gni")

# 源文件列表
libfuse_source = [
    "//third_party/libfuse/lib/fuse.c",
    "//third_party/libfuse/lib/fuse_loop.c",
    # ... 共 15 个源文件
]

# 公共配置
config("libfuse_public_config") {
    include_dirs = [
        "//third_party/libfuse/include",
        "//third_party/libfuse/lib",
        "//third_party/libfuse/util",
        "//third_party/libfuse/example",
    ]
}

# 私有配置
config("libfuse_config") {
    cflags = [
        "-DFUSE_USE_VERSION=317",
        # ... 其他编译选项
    ]
}

# 构建目标
ohos_shared_library("libfuse") {
    configs = [ ":libfuse_config" ]
    public_configs = [ ":libfuse_public_config" ]
    sources = libfuse_source
    # ... 其他配置
}
```

---

## 关键编译选项

### C 编译选项 (cflags)

```gn
cflags = [
    # 基础选项
    "-fdiagnostics-color=always",    # 彩色诊断输出
    "-pipe",                        # 使用管道加速编译
    "-O2",                          # 优化级别 2
    "-g",                           # 包含调试信息

    # 文件系统选项
    "-D_FILE_OFFSET_BITS=64",       # 大文件支持（64 位偏移量）
    "-D_REENTRANT",                 # 线程安全

    # FUSE 特定选项
    "-DFUSE_USE_VERSION=317",       # FUSE API 版本 3.17
    "-DFUSERMOUNT_DIR=\"/usr/local/bin\"",  # fusermount 目录

    # 警告选项
    "-Wextra",
    "-Wno-sign-compare",
    "-Wmissing-declarations",
    "-Wwrite-strings",
    "-Wno-unused-result",

    # 其他选项
    "-Winvalid-pch",
    "-U_GNU_SOURCE",                # 取消 GNU_SOURCE
    "-pthread",                     # 多线程支持
]
```

### 链接选项 (ldflags)

```gn
ldflags = [
    # 基础选项
    "-ldl",                         # 动态链接库
    "-Wl,--version-script",          # 版本脚本
    rebase_path("//third_party/libfuse/lib/fuse_versionscript", root_build_dir),

    # SONAME
    "-Wl,-soname,libfuse3.so.4",    # 共享库 SONAME

    # 链接优化
    "-Wl,--no-undefined",           # 未定义符号检查
    "-Wl,--as-needed",              # 按需链接

    # 共享库选项
    "-shared",
    "-fPIC",

    # ⭐ OpenHarmony 特有：禁用编译器优化
    "-Wl,-mllvm,-import-instr-limit=0",  # 解决构建失败
]
```

### 外部依赖

```gn
external_deps = [
    "c_utils:utils",                 # OH 基础工具库
]
```

---

## 源文件选择

### 编译的源文件 (共 15 个)

| 文件 | 大小 | 功能 |
|------|------|------|
| `lib/buffer.c` | 6.6 KB | 缓冲区管理 |
| `lib/compat.c` | 2.8 KB | 兼容性代码 |
| `lib/cuse_lowlevel.c` | 9.0 KB | CUSE 字符设备 |
| `lib/fuse.c` | 119 KB | 主 FUSE 实现 |
| `lib/fuse_log.c` | 1.8 KB | 日志实现 |
| `lib/fuse_loop.c` | 0.9 KB | 单线程事件循环 |
| `lib/fuse_loop_mt.c` | 12 KB | 多线程事件循环 |
| `lib/fuse_lowlevel.c` | 92 KB | 低级 API 实现 |
| `lib/fuse_opt.c` | 8.9 KB | 选项解析 |
| `lib/fuse_signals.c` | 4.8 KB | 信号处理 |
| `lib/helper.c` | 14 KB | 辅助函数 |
| `lib/mount_util.c` | 8.1 KB | 挂载工具函数 |
| `lib/mount.c` | 18 KB | 挂载实现 |
| `lib/util.c` | 0.5 KB | 通用工具函数 |
| `lib/modules/iconv.c` | - | 字符集转换模块 |
| `lib/modules/subdir.c` | - | 子目录模块 |

### 未编译的源文件

| 文件 | 原因 |
|------|------|
| `lib/mount_bsd.c` | BSD 特定，OH 不使用 |
| `util/fusermount.c` | 用户态挂载工具，OH 不编译 |
| `util/mount.fuse.c` | 挂载工具，OH 不编译 |
| `example/` | 示例代码，OH 不编译 |

---

## 安装配置

### 安装目标

```gn
ohos_shared_library("libfuse") {
    install_enable = true
    install_images = [
        "system",     # 系统分区
        "updater",    # 更新器分区
    ]
}
```

### 说明

- **system 分区**: 正常运行的系统使用
- **updater 分区**: 系统更新时使用

### 部件信息

```gn
subsystem_name = "thirdparty"
part_name = "libfuse"
innerapi_tags = [ "platformsdk" ]
```

- **subsystem_name**: 所属子系统为 "thirdparty"
- **part_name**: 部件名称为 "libfuse"
- **innerapi_tags**: 标记为 "platformsdk"，表示平台 SDK 接口

---

## 头文件包含路径

### 公共头文件路径 (public_configs)

```gn
config("libfuse_public_config") {
    include_dirs = [
        "//third_party/libfuse/include",    # 公共头文件
        "//third_party/libfuse/lib",        # 内部头文件
        "//third_party/libfuse/util",       # 工具头文件
        "//third_party/libfuse/example",    # 示例头文件
    ]
}
```

### 私有头文件路径 (config)

```gn
config("libfuse_config") {
    include_dirs = [
        "//third_party/libfuse/include",
        "//third_party/libfuse/lib",
        "//third_party/libfuse",           # 根目录
    ]
}
```

---

## 与上游构建系统的差异

### 1. 构建工具差异

| 项目 | 上游 (Meson) | OH (GN) |
|------|-------------|---------|
| 配置语言 | Python | GN DSL |
| 命令 | `meson setup` | 无（直接构建） |
| 命令 | `meson compile` | `hb build` / `ninja` |
| 选项 | `meson configure` | 手动修改 BUILD.gn |

### 2. 源文件收集方式

**上游**: 自动收集，在 meson.build 中配置：

```meson
libfuse_sources = files(
    'lib/fuse.c',
    'lib/fuse_lowlevel.c',
    # ...
)
```

**OH**: 手动列出完整路径：

```gn
libfuse_source = [
    "//third_party/libfuse/lib/fuse.c",
    "//third_party/libfuse/lib/fuse_lowlevel.c",
    # ...
]
```

### 3. 特殊处理

#### 3.1 禁用 util/ 工具

上游编译 `util/` 下的工具程序（fusermount, mount.fuse 等），OH 不编译这些工具。

**原因**:
- OpenHarmony 的文件系统挂载方式不同
- 不需要用户态挂载工具
- 减少系统镜像大小

#### 3.2 禁用 example/ 示例

上游编译 `example/` 下的示例程序，OH 不编译。

**原因**:
- 示例代码仅用于学习和测试
- 不适合作为系统组件
- 减少系统镜像大小

#### 3.3 禁用 test/ 测试

上游支持 `test/` 下的测试，OH 在单独的测试构建中处理。

**原因**:
- 测试与生产代码分离
- 符合 OH 的构建规范

---

## 在模块中使用 libfuse

### 基本配置

```gn
ohos_shared_library("my_module") {
    sources = [
        "src/my_filesystem.c",
        # ... 其他源文件
    ]

    include_dirs = [
        "//third_party/libfuse/include",
        "//third_party/libfuse/lib",
        # ... 其他头文件路径
    ]

    external_deps = [
        "libfuse:libfuse",  # 依赖 libfuse
        # ... 其他依赖
    ]

    cflags = [
        "-DFUSE_USE_VERSION=317",  # 设置 FUSE API 版本
        # ... 其他编译选项
    ]

    part_name = "my_part"
    subsystem_name = "my_subsystem"
}
```

### 完整示例

参考 `cloudfiledaemon` 的配置：

```gn
# foundation/filemanagement/dfs_service/services/cloudfiledaemon/BUILD.gn
ohos_shared_library("cloudfiledaemon") {
    sources = [
        "src/cloud_disk/fuse_operations.cpp",
        # ... 其他源文件
    ]

    include_dirs = [
        "//third_party/libfuse/include",
        "//third_party/libfuse/lib",
        # ... 其他头文件路径
    ]

    external_deps = [
        "libfuse:libfuse",
        # ... 其他依赖
    ]

    cflags_cc = [
        "-DFUSE_USE_VERSION=35",  # 注意：使用 35 而非 317
        # ... 其他编译选项
    ]

    part_name = "dfs_service"
    subsystem_name = "filemanagement"
}
```

**注意**: `cloudfiledaemon` 使用 `-DFUSE_USE_VERSION=35`，与 BUILD.gn 中的 `317` 不同。这表明不同模块可能使用不同的 FUSE API 版本。

---

## FUSE API 版本

### 版本说明

libfuse 支持多个 API 版本，通过 `FUSE_USE_VERSION` 宏指定。

### 当前使用情况

| 模块 | 版本 | 说明 |
|------|------|------|
| libfuse 本身 | 317 | 主要版本 |
| cloudfiledaemon | 35 | 较新版本 |
| libdlp_fuse | 35 | 较新版本 |

### 版本差异

不同版本的 API 可能存在不兼容，升级时需要注意：

1. **检查 API 变化**: 查看上游 ChangeLog
2. **测试兼容性**: 确保所有模块正常工作
3. **统一版本**（推荐）: 尽量使用相同的版本

---

## 编译验证

### 编译命令

```bash
# 在 OH 根目录
hb build -f

# 或使用 ninja 直接编译
ninja -C out/ohos-arm64-release
```

### 验证结果

编译成功后，生成的文件：
- `out/.../libfuse.so` - 共享库
- `out/.../libfuse.so.4` - SONAME 链接

---

## 常见问题

### Q: 为什么不编译 util/ 下的工具？

A: OpenHarmony 使用不同的文件系统挂载机制，不需要 fusermount 等工具。这些工具主要用于 Linux 系统。

### Q: FUSE_USE_VERSION 应该使用哪个版本？

A: 建议使用 libfuse BUILD.gn 中定义的版本（当前为 317）。如需使用其他版本，请确保兼容性。

### Q: 如何调试编译问题？

A:
1. 查看 `hb build` 的详细输出
2. 检查编译选项是否正确
3. 使用 `hb clean` 清理后重新编译

### Q: 为什么需要禁用 `-mllvm,-import-instr-limit=0`？

A: 这是 OpenHarmony 构建系统特有的问题，禁用此优化可以避免构建失败。

---

**相关文档**:
- [02_Patches.md](02_Patches.md) - OH 特有修改
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中的使用方式
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 完整评估报告
