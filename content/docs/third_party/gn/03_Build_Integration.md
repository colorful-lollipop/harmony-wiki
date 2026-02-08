# 03 - OpenHarmony 构建适配

## 3.1 构建系统概述

GN 在 OpenHarmony 中作为**构建工具链**使用，而非通过源码编译。这意味着：

1. **无根目录 BUILD.gn**: third_party/gn 目录下没有 OH 风格的 BUILD.gn
2. **Prebuilt 交付**: 使用预编译的 gn 二进制文件
3. **独立构建**: 如果需要从源码构建 GN，使用上游的 build/gen.py

## 3.2 上游构建方式

GN 原始库的构建流程：

```bash
# 1. 生成 Ninja 构建文件
python build/gen.py

# 2. 编译
ninja -C out

# 3. 运行测试
out/gn_unittests
```

### build/gen.py 关键参数

| 参数 | 说明 |
|-----|------|
| `--allow-warning` | 允许警告时构建 |
| `--zoslib-dir` | z/OS 平台指定 ZOSLIB 路径 |

### 构建模板文件

GN 使用模板文件生成平台特定的 Ninja 配置：

| 模板文件 | 平台 |
|---------|------|
| `build/build_linux.ninja.template` | Linux |
| `build/build_mac.ninja.template` | macOS |
| `build/build_win.ninja.template` | Windows |
| `build/build_zos.ninja.template` | z/OS |
| `build/build_haiku.ninja.template` | Haiku |
| `build/build_aix.ninja.template` | AIX |
| `build/build_openbsd.ninja.template` | OpenBSD |

## 3.3 OpenHarmony 构建集成

### Prebuilt GN 位置

在 OH 源码树中，gn 工具可能位于：

```
prebuilts/build-tools/linux-x64/bin/gn
developtools/integration_verification/tools/precise_build/gn
```

### OH 构建流程中的 GN

OH 构建系统调用 GN 的典型流程：

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony Build Flow                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│   │ 开发者   │───→│ BUILD.gn │───→│   gn     │             │
│   │ 编写代码 │    │ 定义规则 │    │ 生成 Ninja │             │
│   └──────────┘    └──────────┘    └────┬─────┘             │
│                                         │                   │
│                                         ↓                   │
│                                   ┌──────────┐             │
│                                   │ build.ninja │          │
│                                   └────┬─────┘             │
│                                         │                   │
│                                         ↓                   │
│                                   ┌──────────┐             │
│                                   │  ninja   │             │
│                                   │ 执行构建  │             │
│                                   └────┬─────┘             │
│                                         │                   │
│                                         ↓                   │
│                                   ┌──────────┐             │
│                                   │ 构建输出 │             │
│                                   │ .so/.bin │             │
│                                   └──────────┘             │
└─────────────────────────────────────────────────────────────┘
```

### 典型 BUILD.gn 结构

OH 组件的 BUILD.gn 示例：

```gn
# Copyright (c) 2021-2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0.

import("//build/ohos.gni")  # 导入 OH 标准构建模板

# 定义编译目标
ohos_shared_library("my_component") {
  sources = [
    "src/foo.cpp",
    "src/bar.cpp",
  ]
  
  include_dirs = [
    "include",
    "//foundation/some_other_component/include",
  ]
  
  deps = [
    "//foundation/other_component:lib",
  ]
  
  external_deps = [
    "hilog:libhilog",
    "ipc:ipc_core",
  ]
}

# 定义组件分组
ohos_component("my_component_group") {
  deps = [ ":my_component" ]
}
```

## 3.4 OH 构建配置关键文件

| 文件 | 作用 |
|-----|------|
| `//build/ohos.gni` | OH 标准构建模板入口 |
| `//build/ohos_var.gni` | OH 构建变量定义 |
| `//build/test.gni` | 测试组件模板 |
| `//build/lite/config/component/lite_component.gni` | Lite 系统组件模板 |
| `//build/config/features.gni` | 特性开关配置 |
| `//.gn` | GN 根配置文件 |
| `//BUILD.gn` | 根构建文件 |
| `//build/config/BUILDCONFIG.gn` | 构建配置 |

## 3.5 Patch 对构建的影响

### 路径表示变化

由于 Patch 移除了 BUILD_DIR 占位符，构建输出路径发生变化：

| 场景 | 上游 GN | OH GN (Patch 后) |
|-----|--------|-----------------|
| 生成文件对象路径 | `obj/BUILD_DIR/gen/foo.o` | `obj/out/Debug/gen/foo.o` |
| 跨工具链依赖 | `toolchain2/obj/BUILD_DIR/toolchain1/gen/foo.o` | `toolchain2/obj/out/Debug/toolchain1/gen/foo.o` |

### 构建行为

- **功能无变化**: Patch 不影响 GN 的功能，只影响输出路径表示
- **Ninja 文件**: 生成的 build.ninja 中路径使用实际值而非占位符
- **兼容性**: 不影响构建结果，仅路径字符串不同

## 3.6 构建系统对比

| 特性 | 上游 GN | OH GN |
|-----|--------|-------|
| 配置文件 | 项目自定义 | 统一 //build/ohos.gni |
| 工具链配置 | 项目定义 | OH 标准工具链 |
| 构建目录处理 | BUILD_DIR 占位符 | 实际路径 |
| 交付形式 | 源码 | prebuilt 二进制 |
| 版本管理 | 项目自行管理 | OH 统一版本 |

## 3.7 维护建议

### 版本升级检查清单

升级 GN 版本时需验证：

- [ ] 新版本的 gen.py 在 OH 环境中可正常运行
- [ ] Patch 可应用到新版本（检查 filesystem_utils.cc 冲突）
- [ ] 生成的 Ninja 文件格式正确
- [ ] OH 构建系统使用新版本 GN 能成功构建
- [ ] 关键组件构建测试通过

### 调试技巧

```bash
# 查看 GN 版本
gn --version

# 生成构建文件（详细模式）
gn gen out --args='...' -v

# 检查 BUILD.gn 语法
gn check //path/to/component

# 查看目标依赖图
gn desc out //path/to:target --tree
```
