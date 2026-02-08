# OH 构建适配

## 3.1 BUILD.gn 结构

OpenHarmony 使用 GN (Generate Ninja) 构建系统，RE2 通过 `BUILD.gn` 文件适配 OH 构建环境。

### 文件位置

```
third_party/re2/BUILD.gn
```

### 完整配置

```gn
# Copyright (c) 2021-2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# ...

import("//build/ohos.gni")

THIRDPARTY_RE2_SUBSYS_NAME = "thirdparty"
THIRDPARTY_RE2_PART_NAME = "re2"

RE2_DIR = rebase_path("//third_party/re2")

config("re2_public_config") {
  include_dirs = [ "${RE2_DIR}/" ]
}

ohos_shared_library("re2") {
  sources = [
    "${RE2_DIR}/re2/bitmap256.cc",
    "${RE2_DIR}/re2/bitmap256.h",
    "${RE2_DIR}/re2/bitstate.cc",
    "${RE2_DIR}/re2/compile.cc",
    "${RE2_DIR}/re2/dfa.cc",
    "${RE2_DIR}/re2/filtered_re2.cc",
    "${RE2_DIR}/re2/mimics_pcre.cc",
    "${RE2_DIR}/re2/nfa.cc",
    "${RE2_DIR}/re2/onepass.cc",
    "${RE2_DIR}/re2/parse.cc",
    "${RE2_DIR}/re2/perl_groups.cc",
    "${RE2_DIR}/re2/pod_array.h",
    "${RE2_DIR}/re2/prefilter.cc",
    "${RE2_DIR}/re2/prefilter.h",
    "${RE2_DIR}/re2/prefilter_tree.cc",
    "${RE2_DIR}/re2/prefilter_tree.h",
    "${RE2_DIR}/re2/prog.cc",
    "${RE2_DIR}/re2/prog.h",
    "${RE2_DIR}/re2/re2.cc",
    "${RE2_DIR}/re2/regexp.cc",
    "${RE2_DIR}/re2/regexp.h",
    "${RE2_DIR}/re2/set.cc",
    "${RE2_DIR}/re2/simplify.cc",
    "${RE2_DIR}/re2/sparse_array.h",
    "${RE2_DIR}/re2/sparse_set.h",
    "${RE2_DIR}/re2/tostring.cc",
    "${RE2_DIR}/re2/unicode_casefold.cc",
    "${RE2_DIR}/re2/unicode_casefold.h",
    "${RE2_DIR}/re2/unicode_groups.cc",
    "${RE2_DIR}/re2/unicode_groups.h",
    "${RE2_DIR}/re2/walker-inl.h",
    "${RE2_DIR}/util/logging.h",
    "${RE2_DIR}/util/mix.h",
    "${RE2_DIR}/util/mutex.h",
    "${RE2_DIR}/util/rune.cc",
    "${RE2_DIR}/util/strutil.cc",
    "${RE2_DIR}/util/strutil.h",
    "${RE2_DIR}/util/utf.h",
    "${RE2_DIR}/util/util.h",
  ]
  include_dirs = [ "re2" ]
  if (!is_asan && !is_debug) {
    version_script = "libre2.map"
  }
  external_deps = [
    "abseil-cpp:absl_base",
    "abseil-cpp:absl_container",
    "abseil-cpp:absl_cord",
    "abseil-cpp:absl_hash",
    "abseil-cpp:absl_log",
    "abseil-cpp:absl_raw_logging_internal",
    "abseil-cpp:absl_spinlock_wait",
    "abseil-cpp:absl_str_format_internal",
    "abseil-cpp:absl_strings",
  ]
  public_configs = [ ":re2_public_config" ]
  install_enable = true
  subsystem_name = "${THIRDPARTY_RE2_SUBSYS_NAME}"
  part_name = "${THIRDPARTY_RE2_PART_NAME}"
}
```

## 3.2 关键配置解析

### 3.2.1 构建目标类型

```gn
ohos_shared_library("re2")
```

**说明**: 使用共享库（动态链接库）而非静态库

**考虑**:
- 共享库可被多个模块共享，减少内存占用
- 便于独立更新（如果 ABI 兼容）
- 与 gRPC 等依赖模块的期望一致

### 3.2.2 源文件列表

```gn
sources = [
  "${RE2_DIR}/re2/bitmap256.cc",
  "${RE2_DIR}/re2/bitmap256.h",
  // ... 共 40+ 个文件
]
```

**特点**:
- 显式列出所有源文件，不使用通配符
- 包含 `.cc` 实现文件和 `.h` 头文件
- 按目录组织（`re2/` 和 `util/`）

**维护**: 升级时需要检查是否有新增/删除/重命名的文件

### 3.2.3 符号导出控制

```gn
if (!is_asan && !is_debug) {
  version_script = "libre2.map"
}
```

**说明**: 使用版本脚本控制共享库导出的符号

**libre2.map 内容**:
```
RE2_0.0.0 {
  global:
    *RE2*;
    *FilteredRE2*;
    *StringPiece*;
    *operator*;
  local:
    *;
};
```

**作用**:
1. **ABI 稳定性**: 控制哪些符号对外可见
2. **版本管理**: 支持符号版本控制
3. **避免冲突**: 隐藏内部符号

**例外**: ASAN 和调试模式不应用版本脚本，便于调试

### 3.2.4 头文件包含路径

```gn
config("re2_public_config") {
  include_dirs = [ "${RE2_DIR}/" ]
}
```

**说明**: 设置公共包含目录为库根目录

**使用方式**:
```cpp
#include "re2/re2.h"
#include "util/strutil.h"
```

### 3.2.5 Abseil 依赖

```gn
external_deps = [
  "abseil-cpp:absl_base",
  "abseil-cpp:absl_container",
  "abseil-cpp:absl_cord",
  "abseil-cpp:absl_hash",
  "abseil-cpp:absl_log",
  "abseil-cpp:absl_raw_logging_internal",
  "abseil-cpp:absl_spinlock_wait",
  "abseil-cpp:absl_str_format_internal",
  "abseil-cpp:absl_strings",
]
```

**依赖组件说明**:

| 组件 | 用途 |
|------|------|
| `absl_base` | 基础类型和工具 |
| `absl_container` | 容器类型 |
| `absl_cord` | 大型字符串处理 |
| `absl_hash` | 哈希函数 |
| `absl_log` | 日志记录 |
| `absl_raw_logging_internal` | 底层日志（新增） |
| `absl_spinlock_wait` | 自旋锁等待 |
| `absl_str_format_internal` | 字符串格式化内部实现 |
| `absl_strings` | 字符串工具 |

**历史变更**: `absl_raw_logging_internal` 是后期添加的依赖（commit 891c14b）

## 3.3 与上游构建系统对比

### 上游构建选项

| 构建系统 | 文件 | RE2 上游使用 |
|----------|------|--------------|
| Bazel | BUILD.bazel | Google 内部首选 |
| CMake | CMakeLists.txt | 开源社区常用 |
| Makefile | Makefile | 简单构建 |
| GN | BUILD.gn | **OH 特有** |

### 差异分析

| 方面 | 上游 | OH BUILD.gn |
|------|------|-------------|
| **源文件扫描** | 自动（glob） | 显式列表 |
| **库类型** | 可选静态/动态 | 强制共享库 |
| **符号导出** | 默认全部 | 受控导出 |
| **依赖声明** | 自动处理 | 显式声明 |
| **安装规则** | 标准安装 | install_enable |

## 3.4 特殊配置说明

### 3.4.1 安装启用

```gn
install_enable = true
```

**说明**: 允许将此库安装到系统镜像中

**影响**:
- 库文件会包含在系统镜像
- 其他模块可在运行时动态链接

### 3.4.2 子系统/部件标识

```gn
subsystem_name = "${THIRDPARTY_RE2_SUBSYS_NAME}"  # "thirdparty"
part_name = "${THIRDPARTY_RE2_PART_NAME}"          # "re2"
```

**作用**:
- 参与 OH 组件化构建
- 用于编译白名单检查
- 在 `build/component_compilation_whitelist.json` 中注册

### 3.4.3 白名单配置

文件: `build/component_compilation_whitelist.json`
```json
{
  "component": [
    "//third_party/re2:re2"
  ]
}
```

**说明**: 允许在组件编译期间构建 RE2

## 3.5 构建问题排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 找不到头文件 | include_dirs 配置错误 | 检查 `public_configs` 是否被依赖 |
| 符号未定义 | 符号未导出或链接顺序问题 | 检查 `libre2.map` 或调整依赖顺序 |
| Abseil 链接错误 | 版本不匹配 | 同步更新 abseil-cpp 版本 |
| 缺少源文件 | 升级后文件变更 | 更新 BUILD.gn 源文件列表 |

### 调试技巧

```bash
# 查看构建配置
gn args out/your_target

# 检查依赖关系
gn desc out/your_target //third_party/re2:re2

# 查看编译命令
ninja -v -d keeprsp //third_party/re2:re2
```

## 3.6 升级维护指南

### 升级检查清单

- [ ] 检查上游新增/删除的源文件
- [ ] 验证 abseil-cpp 版本兼容性
- [ ] 检查 libre2.map 是否需要更新
- [ ] 测试 gRPC 等依赖模块功能
- [ ] 更新 README.OpenSource 版本号

### 源文件列表更新

如果上游有文件变更，修改 BUILD.gn 的 `sources` 列表：

```bash
# 获取上游文件列表
ls re2/*.cc re2/*.h util/*.cc util/*.h

# 对比当前 BUILD.gn 中的列表
# 添加新增文件，删除移除的文件
```

---

**上一篇**: [02_Patches.md](./02_Patches.md) | **下一篇**: [04_Usage_in_OH.md](./04_Usage_in_OH.md)
