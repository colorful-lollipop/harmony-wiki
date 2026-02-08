# 03 - OpenHarmony 构建适配

## 3.1 BUILD.gn 结构概览

OpenHarmony 使用 GN (Generate Ninja) 构建系统管理 e2fsprogs 的编译。BUILD.gn 文件定义了所有构建目标及其依赖关系。

### 构建流程图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         构建流程                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  1. e2fsprogs_action (action)                                 │   │
│  │     - 解压 e2fsprogs.tar.xz                                   │   │
│  │     - 应用 8 个 Patch                                         │   │
│  │     - 生成 ${target_gen_dir}/e2fsprogs                        │   │
│  └───────────────────────┬──────────────────────────────────────┘   │
│                          │                                           │
│                          ▼                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  2. 编译静态/共享库                                            │   │
│  │     - libext2_uuid                                            │   │
│  │     - libext2_com_err                                         │   │
│  │     - libext2fs                                               │   │
│  │     - libext2_blkid                                           │   │
│  │     - ...                                                     │   │
│  └───────────────────────┬──────────────────────────────────────┘   │
│                          │                                           │
│                          ▼                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  3. 编译可执行工具                                             │   │
│  │     - e2fsck                                                  │   │
│  │     - mke2fs                                                  │   │
│  │     - resize2fs                                               │   │
│  │     - e2fsdroid                                               │   │
│  │     - blkid                                                   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 3.2 构建目标详解

### 3.2.1 Action 目标: e2fsprogs_action

这是整个构建流程的起点，负责准备源代码。

```gn
action("e2fsprogs_action") {
  script = "//third_party/e2fsprogs/install.sh"
  inputs = [
    "//third_party/e2fsprogs/e2fsprogs.tar.xz",
    "//third_party/e2fsprogs/1001-image-make.patch",
    "//third_party/e2fsprogs/1002-add-header-file-to-musl-compile-mk2efs.patch",
    "//third_party/e2fsprogs/1003-add-dac-config.patch",
    "//third_party/e2fsprogs/1004-modify-code-to-compile.patch",
    "//third_party/e2fsprogs/1005-read-vfat-chinese-label.patch",
    "//third_party/e2fsprogs/1006-add-hmfs-for-blkid.patch",
    "//third_party/e2fsprogs/1007-blkid-support-skip-specified-filesystem.patch",
    "//third_party/e2fsprogs/1008-blkid-enlarge-cluster-for-ntfs.patch"
  ]
  outputs = [
    "${target_gen_dir}/e2fsprogs",
    # ... 列出所有生成的源文件
  ]
  args = ["$e2fsprogs_gen_path", "$e2fsprogs_src_path"]
}
```

**关键点**:
- 所有 Patch 文件作为输入依赖
- 生成的源代码位于 `${target_gen_dir}/e2fsprogs`
- 所有后续目标依赖此 action

### 3.2.2 共享库目标

#### libext2_uuid
```gn
ohos_shared_library("libext2_uuid") {
  sources = [
    "${E2FSPROGS_DIR}/lib/uuid/clear.c",
    "${E2FSPROGS_DIR}/lib/uuid/compare.c",
    # ... 共 11 个源文件
  ]
  deps = [":e2fsprogs_action"]
  include_dirs = ["${E2FSPROGS_DIR}/lib/uuid"]
  innerapi_tags = ["platformsdk"]  # 提供给平台 SDK
  install_images = ["system", "updater"]
}
```

#### libext2_blkid
```gn
ohos_shared_library("libext2_blkid") {
  configs = [
    ":libext2_blkid-defaults",
    ":libext2-headers",
  ]
  public_configs = [":libext2_blkid_public_config"]
  sources = [
    "${E2FSPROGS_DIR}/lib/blkid/cache.c",
    "${E2FSPROGS_DIR}/lib/blkid/probe.c",
    # ... 共 14 个源文件
  ]
  deps = [":libext2_uuid", ":e2fsprogs_action"]
  cflags = [
    "-Wno-error=attributes",
    "-Wno-error=pointer-sign",
    "-fno-strict-aliasing",
  ]
  install_images = ["system", "updater"]
}
```

**关键配置**:
- `libext2_blkid_public_config`: 公开头文件路径给其他模块
- `fno-strict-aliasing`: 禁用严格别名优化，避免类型双关问题

#### libext2fs (核心库)
```gn
ohos_shared_library("libext2fs") {
  sources = [
    "${E2FSPROGS_DIR}/lib/ext2fs/alloc.c",
    "${E2FSPROGS_DIR}/lib/ext2fs/extent.c",
    "${E2FSPROGS_DIR}/lib/ext2fs/inode.c",
    # ... 共 90+ 个源文件
  ]
  configs = [":libext2fs-defaults"]
  deps = [":libext2_com_err", ":e2fsprogs_action"]
  include_dirs = [
    "${E2FSPROGS_DIR}/lib/ext2fs",
    "${E2FSPROGS_DIR}/lib",
  ]
  install_images = ["system", "updater"]
}
```

### 3.2.3 可执行工具目标

#### e2fsck
```gn
ohos_executable("e2fsck") {
  configs = [":e2fsck-defaults"]
  sources = [
    "${E2FSPROGS_DIR}/e2fsck/e2fsck.c",
    "${E2FSPROGS_DIR}/e2fsck/pass1.c",
    "${E2FSPROGS_DIR}/e2fsck/pass2.c",
    # ... 共 30+ 个源文件
  ]
  include_dirs = [
    "${E2FSPROGS_DIR}/e2fsck",
    "${E2FSPROGS_DIR}/lib",
    "${E2FSPROGS_DIR}/lib/ext2fs",
  ]
  deps = [
    ":libext2_blkid",
    ":libext2_com_err",
    ":libext2_e2p",
    ":libext2_quota",
    ":libext2_uuid",
    ":libext2fs",
    ":e2fsprogs_action"
  ]
  install_images = ["system", "updater"]
}
```

#### e2fsdroid (OH 特有工具)
```gn
ohos_executable("e2fsdroid") {
  configs = [":e2fsdroid-defaults"]
  defines = ["HAVE_SYS_TYPES_H"]
  sources = [
    "${E2FSPROGS_DIR}/contrib/android/base_fs.c",
    "${E2FSPROGS_DIR}/contrib/android/e2fsdroid.c",
    "${E2FSPROGS_DIR}/contrib/android/perms.c",
    # ... 共 7 个源文件
  ]
  include_dirs = [
    "${E2FSPROGS_DIR}/contrib/android/",
    "${E2FSPROGS_DIR}/lib",
    "${E2FSPROGS_DIR}/lib/ext2fs",
    "${E2FSPROGS_DIR}/misc",
  ]
  deps = [
    ":libdacconfig",
    ":libext2_com_err",
    ":libext2_misc",
    ":libext2fs",
    ":e2fsprogs_action"
  ]
  external_deps = ["selinux:libselinux"]
  install_images = ["system", "updater"]
}
```

**特点**:
- 依赖 SELinux 库
- 包含 DAC 配置支持
- 用于镜像制作

## 3.3 配置定义详解

### 3.3.1 编译警告抑制

```gn
config("libext2fs-defaults") {
  cflags = [
    "-Wno-sign-compare",
    "-Wno-pointer-sign",
    "-Wno-implicit-function-declaration",
    "-Wno-int-conversion",
  ]
  defines = ["secure_getenv=getenv"]
}
```

**说明**:
- 禁用特定类型的编译警告
- `secure_getenv=getenv`: 替换 glibc 的 secure_getenv 为标准 getenv

### 3.3.2 包含路径配置

```gn
config("libext2-headers") {
  include_dirs = ["${E2FSPROGS_DIR}/lib"]
}

config("libext2_blkid_public_config") {
  include_dirs = ["${E2FSPROGS_DIR}/lib"]
}
```

**说明**:
- `include_dirs`: 仅当前目标使用
- `public_configs`: 依赖该目标的其他目标也会继承

### 3.3.3 e2fsdroid 专用配置

```gn
config("e2fsdroid-defaults") {
  cflags = [
    "-Wno-incompatible-pointer-types",
    "-Wno-tautological-constant-out-of-range-compare",
  ]
}
```

**说明**:
- 针对 Android 迁移代码的特殊警告抑制

## 3.4 与上游构建系统的差异

### 上游构建系统

e2fsprogs 上游使用 autotools 构建系统:

```bash
# 上游构建流程
./configure --prefix=/usr --enable-elf-shlibs
make
make install
```

**特点**:
- 使用 `configure` 脚本检测系统特性
- 自动生成 `config.h`
- 支持多种配置选项

### OpenHarmony 适配

| 方面 | 上游 | OpenHarmony |
|------|------|-------------|
| **构建工具** | autotools | GN + Ninja |
| **配置方式** | configure 脚本 | 预定义 config.h (Patch 1002) |
| **依赖管理** | 系统包管理 | GN deps |
| **安装路径** | /usr/{bin,lib} | system/updater 分区 |
| **交叉编译** | 配置工具链 | GN 工具链配置 |

### 关键差异说明

#### 1. 配置头文件
- **上游**: 由 `configure` 自动生成
- **OH**: 预定义在 Patch 1002 中，针对 musl 和 OH 内核定制

#### 2. 库类型
- **上游**: 支持静态/动态库选择
- **OH**: 主要使用共享库，静态库用于内部工具

#### 3. 安装目标
- **上游**: 安装到系统目录
- **OH**: 安装到 system/updater 分区镜像

#### 4. 工具选择
- **上游**: 编译所有工具
- **OH**: 仅编译必要工具 (e2fsck, mke2fs, resize2fs, blkid, e2fsdroid)

## 3.5 添加新的编译单元

### 示例：添加新的文件系统探测支持

假设需要添加对 NewFS 的支持：

#### 1. 修改 Patch (probe.c)

```diff
// 在 probe.c 中添加
+static int probe_newfs(struct blkid_probe *probe, ...) { ... }

 static struct blkid_magic type_array[] = {
   // ...
+  { "newfs", 1, 0, 4, "\xAB\xCD\xEF\x01", probe_newfs },
   { NULL, 0, 0, 0, NULL, NULL }
 };
```

#### 2. 更新 BUILD.gn (如需要)

通常不需要修改 BUILD.gn，因为：
- libext2_blkid 使用通配符包含所有 probe.c 中的代码
- 只要 patch 应用成功，新功能自动包含

#### 3. 验证编译

```bash
# 在 OH 源码根目录
./build.sh --product {product_name} --target //third_party/e2fsprogs:libext2_blkid
```

## 3.6 常见问题与解决

### 问题 1: Patch 应用失败

**症状**: 编译时提示找不到生成的文件

**解决**:
```bash
# 1. 清除生成目录
rm -rf out/{board}/gen/third_party/e2fsprogs

# 2. 重新编译
./build.sh --product {product_name} --target //third_party/e2fsprogs:e2fsprogs_action
```

### 问题 2: 头文件找不到

**症状**: `fatal error: 'xxx.h' file not found`

**解决**:
- 检查 Patch 1002 是否正确应用
- 确认 `include_dirs` 配置包含正确路径

### 问题 3: 符号未定义

**症状**: `undefined reference to 'xxx'`

**解决**:
- 检查 `deps` 是否包含提供该符号的目标
- 确认链接顺序（GN 通常自动处理）

---

## 下一章

- **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - OpenHarmony 中的依赖与使用
