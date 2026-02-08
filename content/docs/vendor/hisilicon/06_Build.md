# 构建指南

本文档介绍 Hisilicon Vendor 仓库的构建方法和构建产物。

---

## 6.1 构建概述

### 6.1.1 构建系统

Hisilicon Vendor 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统。

### 6.1.2 构建入口

| 文件 | 用途 |
|-----|------|
| `BUILD.gn` | 定义构建目标 [证据：hispark_pegasus_mini_system/BUILD.gn] |
| `ohos.build` | OpenHarmony 产品构建入口 |
| `product.gni` | 产品配置变量 |
| `config.json` | 产品配置 |

---

## 6.2 构建命令

### 6.2.1 构建产品

```bash
# 编译 wifiiot 产品 (Pegasus)
python build.py --product-name wifiiot

# 编译 taurus 产品
python build.py --product-name hispark_taurus

# 编译 taurus 标准系统
python build.py --product-name hispark_taurus_standard
```

### 6.2.2 构建选项

| 选项 | 说明 |
|-----|------|
| `--product-name` | 指定产品名称 |
| `--build-type` | 构建类型 (debug/release) |
| `--target-cpu` | 目标 CPU |

---

## 6.3 构建文件说明

### 6.3.1 BUILD.gn 示例

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

### 6.3.2 HDF 构建

**uhdf/BUILD.gn** [证据：hispark_taurus_standard/hdf_config/uhdf/BUILD.gn]

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

## 6.4 构建产物

### 6.4.1 产物类型

| 产物类型 | 说明 | 位置 |
|---------|------|------|
| .bin | 固件镜像 | out/product/ |
| .so | 动态库 | out/product/ |
| .a | 静态库 | out/product/ |
| .hap | 应用包 | out/product/ |
| .hcb | HDF 配置编译产物 | out/ |
| .cfg | 设备配置文件 | out/ |

### 6.4.2 产物位置

```
out/
├── {product_name}/
│   ├── bin/                          # 可执行文件
│   ├── lib/                          # 库文件 (.so, .a)
│   ├── etc/                          # 配置文件
│   ├── system/                       # 系统文件
│   ├── vendor/                       # 厂商文件
│   └── modules/                       # 模块文件
└── {product_name}/drivers/           # 驱动文件
```

---

## 6.5 编译配置

### 6.5.1 产品配置

**config.json** [证据：hispark_pegasus/config.json]

```json
{
  "product_name": "wifiiot_hispark_pegasus",
  "subsystems": [
    {
      "subsystem": "applications",
      "components": [
        { "component": "wifi_iot_sample_app" }
      ]
    },
    {
      "subsystem": "security",
      "components": [
        { "component": "huks", "features": [...] }
      ]
    }
  ]
}
```

---

## 6.6 常见问题

### Q1: 构建失败怎么办？

1. 检查依赖是否完整
2. 确认 Python 环境正确
3. 查看错误日志定位问题

### Q2: 如何添加新的组件？

1. 在 `config.json` 中添加子系统组件配置
2. 创建对应的构建文件 BUILD.gn

---

## 6.7 相关文档

- [GN 构建系统](./06a_GN_Build.md)
- [配置体系](./04_Configuration.md)
- [产品系列](./03_Products.md)
