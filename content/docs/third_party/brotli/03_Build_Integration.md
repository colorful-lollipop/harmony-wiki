# OpenHarmony 构建适配

## 构建配置概述

Brotli 在 OpenHarmony 中通过 `BUILD.gn` 文件适配 OH 构建系统（GN 构建系统）。

### 构建产物

| 产物名称 | 类型 | 说明 |
|---------|------|------|
| `brotli_shared` | 共享库 | 主要构建产物，提供 Brotli 编解码功能 |

---

## BUILD.gn 文件结构

```
//third_party/brotli/BUILD.gn
├── 版权声明
├── 导入语句
├── 源文件列表
├── 构建配置
├── 共享库定义
└── 聚合目标
```

### 版权和导入

```gn
# Copyright (c) 2020-2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ...

import("//build/ohos.gni")  # 导入 OH 构建模板
```

---

## 源文件配置

### 源文件列表

```gn
brotli_source = [
  # 公共组件
  "c/common/constants.c",
  "c/common/context.c",
  "c/common/dictionary.c",
  "c/common/platform.c",
  "c/common/shared_dictionary.c",
  "c/common/transform.c",

  # 解码器
  "c/dec/bit_reader.c",
  "c/dec/decode.c",
  "c/dec/huffman.c",
  "c/dec/state.c",

  # 编码器
  "c/enc/backward_references.c",
  "c/enc/backward_references_hq.c",
  "c/enc/bit_cost.c",
  "c/enc/block_splitter.c",
  "c/enc/brotli_bit_stream.c",
  "c/enc/cluster.c",
  "c/enc/command.c",
  "c/enc/compound_dictionary.c",
  "c/enc/compress_fragment.c",
  "c/enc/compress_fragment_two_pass.c",
  "c/enc/dictionary_hash.c",
  "c/enc/encode.c",
  "c/enc/encoder_dict.c",
  "c/enc/entropy_encode.c",
  "c/enc/fast_log.c",
  "c/enc/histogram.c",
  "c/enc/literal_cost.c",
  "c/enc/memory.c",
  "c/enc/metablock.c",
  "c/enc/static_dict.c",
  "c/enc/utf8_util.c",
]
```

**统计**：共 46 个源文件

---

## 编译配置

### 编译器配置

```gn
config("brotli_config") {
  include_dirs = [ "c/include" ]  # 头文件搜索路径

  cflags = [
    # 警告抑制
    "-Wno-deprecated-declarations",  # 禁用废弃 API 警告

    # 平台配置
    "-D_GNU_SOURCE",                  # 启用 GNU 扩展

    # 异常控制
    "-D_HAS_EXCEPTIONS=0",           # 禁用 C++ 异常（C 代码不需要）

    # 构建检测
    "-DHAVE_CONFIG_H",               # 假设存在 config.h（实际可能被移除）

    # 警告管理
    "-Wno-macro-redefined",          # 允许宏重定义（避免与系统宏冲突）
  ]
}
```

### 配置说明

| 配置项 | 值 | 作用 |
|-------|-----|------|
| `-Wno-deprecated-declarations` | 警告抑制 | 避免上游代码中的废弃 API 产生警告 |
| `-D_GNU_SOURCE` | 预定义宏 | 启用 POSIX/GNU 扩展功能 |
| `-D_HAS_EXCEPTIONS=0` | 预定义宏 | 禁用异常支持，减小二进制体积 |
| `-DHAVE_CONFIG_H` | 预定义宏 | 适配需要 config.h 的代码路径 |
| `-Wno-macro-redefined` | 警告抑制 | 避免与系统定义的宏冲突 |
| `include_dirs` | 头文件路径 | 指定 `c/include/brotli` 为头文件基目录 |

---

## 共享库定义

### brotli_shared

```gn
ohos_shared_library("brotli_shared") {
  # ARM 安全特性
  branch_protector_ret = "pac_ret"

  # iOS 特定配置
  if (current_os == "ios") {
    ldflags = [
      "-Wl",
      "-install_name",
      "@rpath/libbrotli_shared.framework/libbrotli_shared",
    ]
  }

  # 公开配置
  public_configs = [ ":brotli_config" ]

  # 源文件
  sources = brotli_source

  # 安装配置
  install_images = [
    "updater",   # 更新分区
    "system",    # 系统分区
  ]

  # 内部 API 标签
  innerapi_tags = [ "chipsetsdk_indirect" ]

  # OH 构建系统字段
  subsystem_name = "thirdparty"
  part_name = "brotli"
}
```

### 关键配置说明

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `branch_protector_ret` | "pac_ret" | ARM PAC 指针认证，防止 ROP 攻击 |
| `install_images` | ["updater", "system"] | 指定安装到更新分区和系统分区 |
| `innerapi_tags` | ["chipsetsdk_indirect"] | 标记为芯片 SDK 间接依赖 |
| `subsystem_name` | "thirdparty" | 所属子系统 |
| `part_name` | "brotli" | 组件名称 |

---

## 与上游构建系统的差异

### 构建系统对比

| 特性 | OH (GN) | 上游 (CMake/Bazel) |
|-----|---------|-------------------|
| 构建系统 | GN | CMake, Bazel |
| 产物类型 | 共享库 | 静态库、共享库、头文件库 |
| 安全加固 | PAC 指针认证 | 无（上游默认不启用） |
| 安装目标 | updater, system | 自定义 |
| 异常支持 | 禁用 | 可选 |

### 主要差异

1. **安全加固**
   - OH：启用 ARM PAC 指针认证
   - 上游：无此配置

2. **安装路径**
   - OH：固定到 `updater` 和 `system` 分区
   - 上游：可配置的安装前缀

3. **头文件配置**
   - OH：使用 `c/include` 作为头文件基目录
   - 上游：可能使用不同的路径布局

---

## 依赖关系

### 该库依赖

```json
"deps": {
  "components": [],
  "third_party": []
}
```

**说明**：Brotli 是自包含的压缩库，不依赖其他第三方库

### 依赖该库的模块

参见 [04_Usage_in_OH.md](04_Usage_in_OH.md)

---

## 编译选项自定义

### 常用编译配置

如果需要在依赖模块中自定义 Brotli 编译选项：

```gn
# 在依赖模块的 BUILD.gn 中
config("my_brotli_config") {
  # 添加自定义编译选项
  cflags = [
    "-O3",  # 优化级别
    "-flto",  # 链接时优化
  ]
}

my_component() {
  # ...
  deps += [ "//third_party/brotli:brotli_shared" ]
  # 不能直接修改 brotli 的配置
}
```

### 性能优化建议

| 场景 | 建议 |
|-----|------|
| **性能优先** | 使用 `-O3` 优化级别 |
| **大小优先** | 使用 `-Os` 优化级别 |
| **调试模式** | 添加 `-g` 调试符号 |

---

## 构建验证

### 验证构建成功

```bash
# 在 OH 构建环境中
hb build -p //third_party/brotli:brotli_shared
```

### 检查构建产物

```bash
# 确认生成的库文件
ls -la out/.../system/lib/libbrotli_shared.so
```

### 运行测试

```bash
# 如果有测试目标
hb build -p //third_party/brotli:brotli
hb test -p //third_party/brotli
```

---

## 常见问题

### Q1: 如何静态链接 Brotli？

当前 OH 配置只提供共享库。如需静态链接，需要：
1. 修改 `BUILD.gn` 添加 `ohos_static_library` 目标
2. 更新 `bundle.json` 的 `build.sub_component`

### Q2: 如何启用调试符号？

```gn
config("brotli_debug") {
  # 继承基础配置
  configs = [ ":brotli_config" ]
  cflags = [ "-g" ]
}

ohos_shared_library("brotli_shared") {
  # ...
  configs += [ ":brotli_debug" ]  # 添加调试配置
}
```

### Q3: PAC 配置是否兼容所有设备？

`branch_protector_ret = "pac_ret"` 主要针对 ARM 架构设备。对于非 ARM 设备，此配置会被忽略或产生警告。

---

## 相关资源

- **BUILD.gn 文件**：`//third_party/brotli/BUILD.gn`
- **bundle.json**：`//third_party/brotli/bundle.json`
- **OH 构建文档**：[构建系统文档](placeholder)
- **上游构建**：http://github.com/google/brotli
