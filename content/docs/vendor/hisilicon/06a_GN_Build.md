# GN 构建系统

本文档详细说明 Hisilicon Vendor 仓库中 GN 构建文件的使用方法。

---

## 6.a.1 GN 概述

**GN** (Generate Ninja) 是 Google 开发的元构建系统，用于生成 Ninja 构建文件。

### 6.a.1.1 构建流程

```
*.gn → gn gen → build.ninja → ninja → 可执行文件/库
```

### 6.a.1.2 文件类型

| 文件 | 用途 |
|-----|------|
| `BUILD.gn` | 构建定义文件 |
| `*.gni` | 构建配置变量文件 |

---

## 6.a.2 构建文件说明

### 6.a.2.1 BUILD.gn 示例

**LiteOS-M 构建** [证据：hispark_pegasus_mini_system/BUILD.gn]

```gn
# Copyright (C) 2020 Hisilicon (Shanghai) Technologies Co., Ltd. All rights reserved.

group("hispark_pegasus_mini_system") {
}
```

**标准系统构建** [证据：hispark_taurus_standard/BUILD.gn]

```gn
# Copyright (C) 2023 Hisilicon (Shanghai) Technologies Co., Ltd. All rights reserved.

group("hispark_taurus_standard") {
  deps = [ "preinstall-config:preinstall-config" ]
}
```

### 6.a.2.2 常用目标类型

| 目标类型 | 说明 |
|---------|------|
| `executable` | 可执行程序 |
| `static_library` | 静态库 (.a) |
| `shared_library` | 动态库 (.so) |
| `group` | 目标组 |
| `copy` | 复制文件 |

### 6.a.2.3 常用函数

| 函数 | 说明 |
|-----|------|
| `group()` | 定义目标组 |
| `executable()` | 定义可执行目标 |
| `static_library()` | 定义静态库 |
| `shared_library()` | 定义动态库 |
| `config()` | 定义配置 |
| `template()` | 定义模板 |

---

## 6.a.3 GN 构建配置

### 6.a.3.1 导入配置

```gn
import("//drivers/hdf_core/adapter/uhdf2/hcs/hcs.gni")
```

### 6.a.3.2 HDF 构建配置

**uhdf/BUILD.gn** [证据]

```gn
import("//drivers/hdf_core/adapter/uhdf2/hcs/hcs.gni")

hdf_hcb("hdf_default.hcb") {
  source = "./hdf.hcs"
  part_name = "product_hispark_taurus_standard"
  subsystem_name = "product_hisilicon"
}

hdf_cfg("hdf_devhost.cfg") {
  source = "./hdf.hcs"
  part_name = "product_hispark_taurus_standard"
  subsystem_name = "product_hisilicon"
}

group("hdf_config") {
  deps = [
    ":hdf_default.hcb",
    ":hdf_devhost.cfg",
  ]
}
```

---

## 6.a.4 构建目标

### 6.a.4.1 定义构建目标

```gn
executable("app") {
  sources = [
    "main.c",
    "utils.c",
  ]
  deps = [
    "//base/foo",
  ]
  public_deps = [
    "//third_party/bar",
  ]
}
```

### 6.a.4.2 条件依赖

```gn
if (is_linux) {
  deps += [ "//platform/linux:foo" ]
} else {
  deps += [ "//platform/liteos:bar" ]
}
```

---

## 6.a.5 常用变量

| 变量 | 说明 |
|-----|------|
| `sources` | 源文件列表 |
| `deps` | 依赖目标 |
| `public_deps` | 公开依赖 |
| `include_dirs` | 包含目录 |
| `defines` | 宏定义 |
| `configs` | 配置列表 |

---

## 6.a.6 构建命令

### 6.a.6.1 生成构建文件

```bash
gn gen out/product --args='target_os="ohos"'
```

### 6.a.6.2 执行构建

```bash
ninja -C out/product
```

---

## 6.a.7 相关文档

- [构建指南](./06_Build.md)
- [配置体系](./04_Configuration.md)
- [HDF 配置详解](./04a_HDF_Configuration.md)
