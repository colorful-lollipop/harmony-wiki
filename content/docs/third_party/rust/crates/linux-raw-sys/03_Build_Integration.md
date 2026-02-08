# OH 构建适配

## 一、构建系统概述

### 1.1 构建系统选择

OpenHarmony 使用 **GN**（Generate Ninja）作为构建系统，并为 Rust 第三方库提供了专门的构建模板 `ohos_cargo_crate`。这种设计使得将上游 Rust 库集成到 OH 系统中变得标准化和简单。

### 1.2 构建模板说明

`ohos_cargo_crate` 是 OH 为 Rust crates 提供的构建模板，具有以下特点：

- **自动元数据读取**：从 Cargo.toml 读取 crate 名称、版本、作者等信息
- **特性管理**：支持启用/禁用 Rust features
- **依赖管理**：自动处理 Rust 依赖关系
- **输出格式**：生成 rlib（静态库）格式

## 二、BUILD.gn 详细分析

### 2.1 完整配置

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
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

ohos_cargo_crate("lib") {
  crate_name = "linux_raw_sys"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2018"
  cargo_pkg_version = "0.1.4"
  cargo_pkg_authors = "Dan Gohman <dev@sunfishcode.online>"
  cargo_pkg_name = "linux-raw-sys"
  cargo_pkg_description = "Generated bindings for Linux's userspace API"
  features = [
    "errno",
    "general",
    "std",
    "ioctl",
  ]
  module_output_extension = ".rlib"
  part_name = "rust_linux_raw_sys"
  subsystem_name = "thirdparty"
}
```

### 2.2 配置项详解

#### 2.2.1 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| crate_name | linux_raw_sys | GN 构建目标名称 |
| crate_type | rlib | Rust 静态库格式 |
| crate_root | src/lib.rs | crate 入口文件 |
| sources | [src/lib.rs] | 源文件列表 |
| edition | 2018 | Rust Edition 版本 |

**说明**：crate_name 使用下划线是因为 GN 不允许目标名称中出现连字符。

#### 2.2.2 包元数据

| 配置项 | 值 | 来源 |
|--------|-----|------|
| cargo_pkg_version | 0.1.4 | Cargo.toml [package] |
| cargo_pkg_authors | Dan Gohman... | Cargo.toml [package] |
| cargo_pkg_name | linux-raw-sys | Cargo.toml [package] |
| cargo_pkg_description | Generated bindings... | Cargo.toml [package] |

**说明**：这些元数据直接从上游 Cargo.toml 同步，确保版本一致性。

#### 2.2.3 功能特性

| 配置项 | 值 | 说明 |
|--------|-----|------|
| features | errno | 错误码绑定（上游默认） |
| | general | 通用绑定（上游默认） |
| | std | 标准库支持（上游默认） |
| | ioctl | IOCTL 绑定（OH 额外启用） |

**特性启用策略**：OH 启用了比上游默认更多的特性，这提供了更完整的 API 覆盖。

#### 2.2.4 OH 特有配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| module_output_extension | .rlib | Rust 库文件扩展名 |
| part_name | rust_linux_raw_sys | OH 组件名称 |
| subsystem_name | thirdparty | OH 子系统名称 |

**说明**：part_name 和 subsystem_name 是 OH 特有的组件管理配置。

### 2.3 与上游构建的对比

| 方面 | 上游 Cargo | OH BUILD.gn |
|------|------------|-------------|
| 构建工具 | cargo | gn + ninja |
| 配置文件 | Cargo.toml | BUILD.gn |
| 特性声明 | [features] | features = [...] |
| 版本管理 | version字段 | cargo_pkg_version |
| 元数据 | [package] | cargo_pkg_* |

## 三、编译选项分析

### 3.1 启用的功能特性

#### 3.1.1 errno 特性

**上游定义**：
```toml
errno = []
```

**OH 配置**：
```gn
features = [ "errno", ... ]
```

**功能说明**：启用 Linux 错误码定义，包括：
- 标准错误码常量（E2BIG、EACCES 等）
- errno 类型定义
- 错误码处理辅助定义

#### 3.1.2 general 特性

**上游定义**：
```toml
general = []
```

**OH 配置**：
```gn
features = [ ..., "general", ... ]
```

**功能说明**：启用通用 Linux API 绑定，包括：
- 系统调用结构体
- 时间相关类型
- 文件系统类型
- 网络编程类型

#### 3.1.3 std 特性

**上游定义**：
```toml
std = []
```

**OH 配置**：
```gn
features = [ ..., "std", ... ]
```

**功能说明**：启用标准库支持，允许使用 std::os::raw 中的类型定义。

#### 3.1.4 ioctl 特性

**上游定义**：
```toml
ioctl = []
```

**OH 配置**：
```gn
features = [ ..., "ioctl", ... ]
```

**功能说明**：启用设备控制接口绑定，这是 OH 额外启用的特性。

**启用原因**：许多 OH 系统模块需要进行设备控制操作，ioctl 绑定提供了必要的接口定义。

### 3.2 条件编译配置

#### 3.2.1 架构条件

```rust
#[cfg(target_arch = "aarch64")]
#[path = "aarch64/general.rs"]
pub mod general;
```

这些条件编译指令在 lib.rs 中定义，自动根据目标架构选择对应的绑定文件。

#### 3.2.2 特性条件

```rust
#[cfg(feature = "errno")]
pub mod errno;
```

根据启用的功能特性包含对应的模块。

## 四、组件依赖关系

### 4.1 被依赖情况

linux-raw-sys 被以下 OH 模块依赖：

| 依赖模块 | 依赖路径 | 依赖类型 |
|----------|----------|----------|
| rustix | //third_party/rust/crates/rustix:BUILD.gn | 直接依赖 |

### 4.2 依赖声明方式

**rustix/BUILD.gn 中的声明**：
```gn
deps = [
  "//third_party/rust/crates/linux-raw-sys:lib",
]
```

**说明**：rustix 依赖 linux-raw-sys 提供的底层绑定来实现安全的系统调用封装。

## 五、构建产物

### 5.1 输出产物

| 产物类型 | 文件格式 | 用途 |
|----------|----------|------|
| 静态库 | librust_linux_raw_sys.rlib | 链接到 rustix |
| 组件清单 | rust_linux_raw_sys.config | OH 组件管理 |

### 5.2 输出路径

构建产物位于：
```
out/.../obj/thirdparty/rust/crates/linux-raw-sys/
```

## 六、构建验证

### 6.1 验证方法

#### 方法一：编译测试

```bash
# 在 OH 构建环境中
hb build -p rust_linux_raw_sys
```

#### 方法二：依赖检查

```bash
# 检查依赖关系
gn desc out/... rust_linux_raw_sys deps
```

### 6.2 预期结果

| 检查项 | 预期结果 |
|--------|----------|
| 编译状态 | 成功，无错误 |
| 链接状态 | 可被 rustix 正常链接 |
| 产物存在 | librust_linux_raw_sys.rlib 存在 |

## 七、维护指南

### 7.1 版本更新流程

1. **检查上游版本**：访问 crates.io 或 GitHub 查看新版本
2. **验证兼容性**：确认新版本与当前 OH 环境兼容
3. **更新配置**：修改 BUILD.gn 中的 cargo_pkg_version
4. **测试验证**：执行构建测试
5. **提交变更**：创建 OH PR

### 7.2 配置修改场景

#### 场景一：启用新特性

如果需要启用 `netlink` 特性：

```gn
features = [
  "errno",
  "general",
  "std",
  "ioctl",
  "netlink",  # 新增
]
```

#### 场景二：禁用特性

如果某个特性存在问题，可以禁用：

```gn
features = [
  "errno",
  "general",
  # "std",  # 注释掉以禁用
  "ioctl",
]
```

### 7.3 故障排查

| 问题现象 | 可能原因 | 解决方案 |
|----------|----------|----------|
| 编译错误 | Rust 版本不匹配 | 确保 Rust 版本 >= 1.48 |
| 链接错误 | 依赖缺失 | 检查 rustix 的依赖声明 |
| 特性冲突 | features 配置错误 | 检查 features 列表 |

---

**文档版本**：1.0
**最后更新**：2024年
