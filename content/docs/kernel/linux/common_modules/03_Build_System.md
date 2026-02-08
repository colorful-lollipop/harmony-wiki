# 构建系统

## 1. GN 构建概述

### 1.1 构建系统类型

本仓库使用 **GN (Generate Ninja)** 构建系统，配合 OpenHarmony 的构建框架。

| 构建类型 | 用途 |
|----------|------|
| **GN** | ko 模块构建（用户态/编译时） |
| **Makefile** | 内核模块源码编译 |
| **Kconfig** | 内核配置选项 |

### 1.2 构建入口

**根目录 BUILD.gn**: `/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/BUILD.gn`

```gn
# Copyright (C) 2023 Huawei Device Co., Ltd.

group("ko_build") {
  deps = [ "module_sample:ko_sample" ]
}
```

**证据来源**: `BUILD.gn:14-16`

---

## 2. ko 模块构建

### 2.1 构建模板

**模板文件**: `//build/templates/kernel/ohos_kernel_build.gni`

```gn
import("//build/templates/kernel/ohos_kernel_build.gni")

ohos_build_ko("ko_sample") {
  sources = [
    "ko_sample.c",
    "sample_fun.c",
  ]
  target_ko_name = "kosample"     # 内核模块名（不含 .ko 后缀）
  device_name = device_name       # 设备名（从构建参数继承）
  device_arch = "arm64"           # 架构配置
}
```

**证据来源**: `README.md:99-111`

### 2.2 配置参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `sources` | list | 源码文件列表 |
| `target_ko_name` | string | 输出 ko 文件名 |
| `device_name` | string | 目标设备名 |
| `device_arch` | string | 目标架构 |
| `defines` | list | 预定义宏（可选） |
| `include_dirs` | list | 包含目录（可选） |
| `deps` | list | 依赖其他 target（可选） |

---

## 3. KBuild 构建

### 3.1 Makefile 配置

#### A. 简单 Makefile

```makefile
obj-$(CONFIG_TZDRIVER) += core/
obj-$(CONFIG_TZDRIVER) += auth/
obj-$(CONFIG_TZDRIVER) += ion/
obj-$(CONFIG_TZDRIVER) += whitelist/
obj-$(CONFIG_TZDRIVER) += tlogger/
```

#### B. 条件编译 Makefile

**memory_security/Makefile**:

```makefile
# HIDEADDR objects
obj-$(CONFIG_HIDE_MEM_ADDRESS) += src/hideaddr.o module.o

# JIT_MEMORY objects
obj-$(CONFIG_JIT_MEM_CONTROL) += src/jit_space_list.o src/jit_process.o \
                                  src/jit_memory.o src/jit_memory_module.o module.o

# SELinux dependencies
ccflags-$(CONFIG_HIDE_MEM_ADDRESS) += -I$(srctree)/security/selinux/include
```

### 3.2 Kconfig 选项

#### A. 通用配置

| 配置项 | 位置 | 功能 |
|--------|------|------|
| `CONFIG_MEMORY_SECURITY` | memory_security/Kconfig | 内存安全主开关 |
| `CONFIG_TZDRIVER` | tzdriver/Kconfig | TrustZone 驱动 |
| `CONFIG_ARM64_PTR_AUTH` | pac | PAC 主开关 |

#### B. 子模块配置

| 父配置项 | 子配置项 | 功能 |
|----------|----------|------|
| `CONFIG_MEMORY_SECURITY` | `CONFIG_HIDE_MEM_ADDRESS` | 地址隐藏 |
| `CONFIG_MEMORY_SECURITY` | `CONFIG_JIT_MEM_CTRL` | JIT 内存控制 |
| `CONFIG_ARM64_PTR_AUTH` | `CONFIG_ARM64_PTR_AUTH_EXT` | 密钥管理 |
| `CONFIG_ARM64_PTR_AUTH` | `CONFIG_ARM64_PTR_AUTH_DATA_FIELD` | 数据字段保护 |

---

## 4. 构建产物

### 4.1 ko 文件产物

| 产物类型 | 位置 | 说明 |
|----------|------|------|
| .ko 文件 | `out/{device}/packages/phone/chip_ckm/` | 内核模块文件 |
| .ko 文件 | `out/{device}/packages/phone/images/` | 打包前位置 |

**证据来源**: `README.md:124-133`

```
README.md:124-128
所有构建后ko生成在新增的chip_ckm目录下，以rk3568为例：
out/rk3568/packages/phone/chip_ckm/
                           └ ─ ─  *.ko
```

### 4.2 镜像打包

| 产物 | 位置 | 说明 |
|------|------|------|
| chip_ckm.img | `out/{device}/packages/phone/images/` | ko 打包镜像 |

**证据来源**: `README.md:129-133`

```
README.md:129-133
ko全部打包到独立镜像chip_ckm.img镜像中，镜像位置以rk3568为例：
out/rk3568/packages/phone/images/
                           └ ─ ─  chip_ckm.img
```

---

## 5. 构建命令

### 5.1 完整构建

```bash
./build.sh --product-name rk3568 --build-target mk_chip_ckm_img --ccache --jobs 4
```

**证据来源**: `README.md:117-119`

### 5.2 内核构建

```bash
./build.sh --product-name rk3568 --ccache --build-target kernel \
  --gn-args linux_kernel_version="linux-5.10"
```

**证据来源**: `tzdriver/README_zh.md:73-74`

---

## 6. 模块构建清单

### 6.1 当前已配置模块

| 模块 | 构建系统 | BUILD.gn | Makefile | Kconfig |
|------|----------|-----------|----------|----------|
| module_sample | ✅ | ✅ | ❌ | ❌ |
| tzdriver | ❌ | ❌ | ✅ | ✅ |
| memory_security | ❌ | ❌ | ✅ | ✅ |
| pac | ❌ | ❌ | ✅ | ✅ |
| newip | ❌ | ❌ | ✅ | ❌ |
| qos_auth | ❌ | ❌ | ✅ | ✅ |
| xpm | ❌ | ❌ | ✅ | ✅ |
| code_sign | ❌ | ❌ | ✅ | ✅ |
| container_escape_detection | ❌ | ❌ | ✅ | ✅ |
| ucollection | ❌ | ❌ | ✅ | ✅ |

### 6.2 待配置模块

以下模块暂无 `BUILD.gn`，需要适配 GN 构建系统：

- [ ] tzdriver
- [ ] memory_security
- [ ] pac
- [ ] newip
- [ ] qos_auth
- [ ] xpm
- [ ] code_sign
- [ ] container_escape_detection
- [ ] ucollection

---

## 7. 集成脚本

### 7.1 apply_*.sh 脚本

| 脚本 | 功能 |
|------|------|
| `apply_tzdriver.sh` | 创建 tzdriver 内核符号链接 |
| `apply_hideaddr.sh` | 创建 memory_security 内核符号链接 |
| `apply_pac.sh` | 创建 pac 内核符号链接 |
| `apply_qos_auth.sh` | 创建 qos_auth 内核符号链接 |
| `apply_xpm.sh` | 创建 xpm 内核符号链接 |
| `apply_ced.sh` | 创建 container_escape_detection 内核符号链接 |
| `apply_code_sign.sh` | 创建 code_sign 内核符号链接 |
| `apply_ucollection.sh` | 创建 ucollection 内核符号链接 |

### 7.2 脚本使用

```bash
# 示例：集成 tzdriver
./apply_tzdriver.sh <ohos_source_root> <kernel_build_root> <product_name> <kernel_version>
```

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [modules/module_sample.md](modules/module_sample.md) | BUILD.gn 示例 |
| [README.md](../README.md#ko模块指导) | ko 模块构建详细指南 |
| [OpenHarmony 构建指南](https://gitee.com/openharmony/docs/) | 官方构建文档 |
