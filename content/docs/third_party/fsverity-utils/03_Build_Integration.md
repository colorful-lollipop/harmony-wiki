# fsverity-utils 构建适配

## 3.1 构建系统概述

### 构建系统

fsverity-utils 在 OpenHarmony 中使用 **GN (Generate Ninja)** 构建系统。

### 构建产物

| 目标 | 产物类型 | 输出路径 | 说明 |
|------|----------|----------|------|
| `libfsverity_utils` | 共享库 (.so) | system/lib64/ | 动态链接版本 |
| `libfsverity_utils_static` | 静态库 (.a) | - | 静态链接版本 |

---

## 3.2 BUILD.gn 完整配置

### 文件位置

```
third_party/fsverity-utils/BUILD.gn
```

### 完整内容

```gn
# Copyright (c) 2023-2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

config("common_config") {
  cflags = [
    "-Wall",
    "-Wundef",
    "-Wdeclaration-after-statement",
    "-Wimplicit-fallthrough",
    "-Wmissing-field-initializers",
    "-Wmissing-prototypes",
    "-Wstrict-prototypes",
    "-Wunused-parameter",
    "-Wvla",
    "-Wno-deprecated-declarations",
  ]
}

config("libfsverity_public_config") {
  include_dirs = [
    "include",
    "common",
  ]
}

ohos_shared_library("libfsverity_utils") {
  sources = [
    "lib/compute_digest.c",
    "lib/enable.c",
    "lib/hash_algs.c",
    "lib/sign_digest.c",
    "lib/utils.c",
  ]
  external_deps = [ "openssl:libcrypto_shared" ]
  configs = [ ":common_config" ]
  public_configs = [ ":libfsverity_public_config" ]
  install_enable = true

  output_name = "libfsverity_utils"
  part_name = "fsverity-utils"
  subsystem_name = "thirdparty"
  install_images = [ "system" ]
}

ohos_static_library("libfsverity_utils_static") {
  sources = [
    "lib/compute_digest.c",
    "lib/enable.c",
    "lib/hash_algs.c",
    "lib/sign_digest.c",
    "lib/utils.c",
  ]
  include_dirs = [
    "include",
    "common",
  ]
  configs = [ ":common_config" ]
  external_deps = [ "openssl:libcrypto_static" ]
}
```

---

## 3.3 配置详解

### 编译选项配置 (`common_config`)

```gn
config("common_config") {
  cflags = [
    "-Wall",                    # 开启所有警告
    "-Wundef",                  # 未定义宏警告
    "-Wdeclaration-after-statement",  # 语句后声明警告
    "-Wimplicit-fallthrough",   # 隐式 fallthrough 警告
    "-Wmissing-field-initializers",  # 缺失字段初始化警告
    "-Wmissing-prototypes",     # 缺失函数原型警告
    "-Wstrict-prototypes",      # 严格函数原型警告
    "-Wunused-parameter",       # 未使用参数警告
    "-Wvla",                    # 变长数组警告
    "-Wno-deprecated-declarations",  # 禁用弃用声明警告
  ]
}
```

**说明**：这些是标准的 GCC 警告选项，确保代码质量和可移植性。

### 公共头文件配置 (`libfsverity_public_config`)

```gn
config("libfsverity_public_config") {
  include_dirs = [
    "include",      # libfsverity.h 所在目录
    "common",       # 公共定义目录
  ]
}
```

**用途**：当其他模块依赖此库时，自动包含这些头文件目录。

---

## 3.4 库目标定义

### 共享库 (`libfsverity_utils`)

```gn
ohos_shared_library("libfsverity_utils") {
  sources = [
    "lib/compute_digest.c",  # 摘要计算
    "lib/enable.c",          # fs-verity 启用
    "lib/hash_algs.c",       # 哈希算法
    "lib/sign_digest.c",     # 签名功能
    "lib/utils.c",           # 工具函数
  ]

  external_deps = [ "openssl:libcrypto_shared" ]

  configs = [ ":common_config" ]
  public_configs = [ ":libfsverity_public_config" ]

  install_enable = true
  output_name = "libfsverity_utils"
  part_name = "fsverity-utils"
  subsystem_name = "thirdparty"
  install_images = [ "system" ]
}
```

**关键属性**：

| 属性 | 值 | 说明 |
|------|-----|------|
| `external_deps` | `openssl:libcrypto_shared` | 链接 OpenSSL 共享库 |
| `install_enable` | `true` | 允许安装到系统分区 |
| `part_name` | `fsverity-utils` | 所属组件名称 |
| `subsystem_name` | `thirdparty` | 所属子系统 |
| `install_images` | `["system"]` | 安装到 system 分区 |

### 静态库 (`libfsverity_utils_static`)

```gn
ohos_static_library("libfsverity_utils_static") {
  sources = [
    "lib/compute_digest.c",
    "lib/enable.c",
    "lib/hash_algs.c",
    "lib/sign_digest.c",
    "lib/utils.c",
  ]

  include_dirs = [
    "include",
    "common",
  ]

  configs = [ ":common_config" ]
  external_deps = [ "openssl:libcrypto_static" ]
}
```

**与共享库的区别**：

| 特性 | 共享库 | 静态库 |
|------|--------|--------|
| `external_deps` | `libcrypto_shared` | `libcrypto_static` |
| `public_configs` | 有 | 无 |
| `install_enable` | true | false |

---

## 3.5 与上游 Makefile 的比较

### 上游 Makefile 关键配置

```makefile
# 上游 Makefile 片段
LIBSSRCS = compute_digest.c enable.c hash_algs.c sign_digest.c utils.c
LIBOBJS = $(LIBSSRCS:.c=.o)

LIB = libfsverity.so
STATICLIB = libfsverity.a

# 链接 OpenSSL
LDLIBS += -lcrypto
```

### 差异对比

| 配置项 | 上游 Makefile | OH BUILD.gn |
|--------|---------------|-------------|
| 源文件列表 | `LIBSSRCS` | `sources` |
| OpenSSL 依赖 | `-lcrypto` | `openssl:libcrypto_shared` |
| 编译选项 | CFLAGS | `common_config` cflags |
| 头文件目录 | -I | `public_configs` include_dirs |
| 安装路径 | PREFIX/lib | system/lib64/ |

### 关键差异说明

#### 1. OpenSSL 依赖处理

**上游**：
```makefile
LDLIBS += -lcrypto  # 简单链接标志
```

**OH**：
```gn
external_deps = [ "openssl:libcrypto_shared" ]  # GN 依赖声明
```

**优势**：OH 方式确保正确的链接顺序和依赖传递。

#### 2. 头文件暴露

**上游**：使用者在 Makefile 中手动添加 `-Iinclude`

**OH**：
```gn
public_configs = [ ":libfsverity_public_config" ]
```

**优势**：自动包含头文件路径，减少使用者配置。

---

## 3.6 依赖关系

### 外部依赖

| 依赖 | 最小版本 | 用途 | 必需性 |
|------|----------|------|--------|
| OpenSSL | 1.0.0+ | 哈希算法、签名、X.509 | 必需 |

### OpenSSL 子模块

```
openssl
├── libcrypto_shared    # 共享库链接
├── libcrypto_static   # 静态库链接
└── headers            # 头文件
```

### 内部依赖

```
fsverity-utils
├── lib/compute_digest.c  → hash_algs.c
├── lib/enable.c          → 无内部依赖
├── lib/hash_algs.c       → 无内部依赖
├── lib/sign_digest.c     → hash_algs.c
└── lib/utils.c           → 无内部依赖
```

---

## 3.7 如何在其他模块中使用

### 方式一：共享库依赖

```gn
# 其他模块的 BUILD.gn
ohos_executable("my_module") {
  sources = [ "src/my_module.cpp" ]

  # 依赖 fsverity-utils 共享库
  external_deps = [ "fsverity-utils:libfsverity_utils" ]

  # 自动包含头文件路径 (来自 public_configs)
}

# 或使用配置变量
fsverity_utils_dir = "//third_party/fsverity-utils"
```

### 方式二：静态库依赖

```gn
ohos_executable("my_module") {
  sources = [ "src/my_module.cpp" ]

  # 依赖 fsverity-utils 静态库
  external_deps = [ "fsverity-utils:libfsverity_utils_static" ]

  # 手动添加头文件路径
  include_dirs = [
    "$fsverity_utils_dir/include",
    "$fsverity_utils_dir/common",
  ]
}
```

### 方式三：内联源码

```gn
# 如果需要修改源码，可以内联到使用模块
source_set("my_fsverity") {
  sources = [
    "$fsverity_utils_dir/lib/compute_digest.c",
    "$fsverity_utils_dir/lib/enable.c",
    "$fsverity_utils_dir/lib/hash_algs.c",
    "$fsverity_utils_dir/lib/sign_digest.c",
    "$fsverity_utils_dir/lib/utils.c",
  ]

  include_dirs = [
    "$fsverity_utils_dir/include",
    "$fsverity_utils_dir/common",
  ]

  external_deps = [ "openssl:libcrypto_shared" ]
}
```

---

## 3.8 构建配置变量

### 在 code_signature 中的使用

```gn
# base/security/code_signature/code_signature.gni
fsverity_utils_dir = "//third_party/fsverity-utils"

# 使用示例
source_set("fsverity_sign_src_set") {
  sources = [
    "$fsverity_utils_dir/lib/compute_digest.c",
    "$fsverity_utils_dir/lib/hash_algs.c",
    "$fsverity_utils_dir/lib/sign_digest.c",
    "$fsverity_utils_dir/lib/utils.c",
  ]

  include_dirs = [ "$fsverity_utils_dir/include" ]

  if (is_standard_system) {
    external_deps = [ "openssl:libcrypto_shared" ]
  }
}
```

---

## 3.9 构建产物

### 共享库信息

```bash
# 查看库信息
$ file out/release/system/lib64/libfsverity_utils.so
libfsverity_utils.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV)

# 依赖检查
$ readelf -d out/release/system/lib64/libfsverity_utils.so
(NEEDED)  Shared library: [libcrypto.so.1.1]
```

### 符号导出

```bash
# 导出符号
$ nm -D out/release/system/lib64/libfsverity_utils.so | grep " T "
000000000000XXXX T libfsverity_compute_digest
000000000000XXXX T libfsverity_sign_digest
000000000000XXXX T libfsverity_enable
000000000000XXXX T libfsverity_enable_with_sig
000000000000XXXX T libfsverity_find_hash_alg_by_name
000000000000XXXX T libfsverity_get_digest_size
000000000000XXXX T libfsverity_get_hash_name
000000000000XXXX T libfsverity_set_error_callback
```

---

## 3.10 构建问题排查

### 常见问题

#### 1. 链接错误：找不到 libcrypto

**症状**：
```
error: cannot find -lcrypto
```

**解决**：
```gn
# 确保正确声明 OpenSSL 依赖
external_deps = [ "openssl:libcrypto_shared" ]
```

#### 2. 头文件找不到

**症状**：
```
fatal error: libfsverity.h: No such file or directory
```

**解决**：
```gn
# 确保使用 public_configs 或手动添加 include_dirs
public_configs = [ "//third_party/fsverity-utils:libfsverity_public_config" ]
```

#### 3. 符号未定义

**症状**：
```
undefined reference to `libfsverity_compute_digest'
```

**解决**：
```gn
# 确保链接了正确的库
external_deps = [ "fsverity-utils:libfsverity_utils" ]
```

---

## 3.11 构建最佳实践

### 1. 使用共享库还是静态库？

| 场景 | 推荐 | 原因 |
|------|------|------|
| 一般应用 | 共享库 | 减少 ROM/RAM 占用 |
| 系统组件 | 共享库 | 便于安全更新 |
| 性能敏感 | 静态库 | 减少函数调用开销 |
| 独立部署 | 静态库 | 无外部依赖 |

### 2. 依赖声明规范

```gn
# ✅ 正确：使用 external_deps
external_deps = [ "fsverity-utils:libfsverity_utils" ]

# ❌ 错误：直接使用 sources（除非需要内联）
sources = [ "$fsverity_utils_dir/lib/compute_digest.c" ]
```

### 3. 配置变量管理

```gn
# ✅ 良好实践：集中管理路径
fsverity_utils_dir = "//third_party/fsverity-utils"

# ❌ 不良实践：硬编码路径
sources = [ "//third_party/fsverity-utils/lib/compute_digest.c" ]
```

---

## 3.12 总结

### 构建适配要点

| 要点 | 状态 |
|------|------|
| BUILD.gn 配置完整 | ✅ |
| 共享库和静态库都支持 | ✅ |
| 正确依赖 OpenSSL | ✅ |
| 头文件正确暴露 | ✅ |
| 安装路径正确 | ✅ |

### 与上游差异

| 维度 | 差异 |
|------|------|
| 构建系统 | Makefile → GN |
| 依赖管理 | 手动 → external_deps |
| 头文件 | 手动添加 → public_configs |
| 安装 | 手动 → install_images |

---

## 参考文档

- [README.md](README.md) - 项目概述
- [01_Overview.md](01_Overview.md) - 原始库介绍
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景
- [上游 Makefile](https://git.kernel.org/pub/scm/fs/fsverity/fsverity-utils.git)
