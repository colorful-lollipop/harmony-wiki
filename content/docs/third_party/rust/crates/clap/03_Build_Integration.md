# 03 - OH 构建适配

## 概述

clap 在 OH 中的适配完全通过 **BUILD.gn** 文件完成，无任何源码修改。本章详细说明 GN 构建配置及其与上游 Cargo 构建的差异。

## BUILD.gn 文件结构

```
clap/
├── BUILD.gn              # 主 crate 配置
├── clap_derive/
│   └── BUILD.gn          # 派生宏 crate
└── clap_lex/
    └── BUILD.gn          # 词法分析 crate
```

## 主 BUILD.gn 详解

### 文件位置
`third_party/rust/crates/clap/BUILD.gn`

### 完整配置

```gn
import("//build/ohos.gni")

if (host_os != "linux" || host_cpu != "arm64") {
  ohos_cargo_crate("lib") {
    crate_name = "clap"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = [ "src/lib.rs" ]
    edition = "2021"
    cargo_pkg_version = "4.1.13"
    cargo_pkg_name = "clap"
    cargo_pkg_description = "A simple to use, efficient, and full-featured Command Line Argument Parser"
    
    deps = [
      "//third_party/rust/crates/bitflags:lib",
      "//third_party/rust/crates/clap/clap_derive:lib(${host_toolchain})",
      "//third_party/rust/crates/clap/clap_lex:lib",
      "//third_party/rust/crates/is-terminal:lib",
      "//third_party/rust/crates/once_cell:lib",
      "//third_party/rust/crates/strsim-rs:lib",
      "//third_party/rust/crates/termcolor:lib",
    ]
    
    features = [
      "color",
      "error-context",
      "help",
      "std",
      "suggestions",
      "usage",
      "derive",
    ]
    
    module_output_extension = ".rlib"
    part_name = "rust_clap"
    subsystem_name = "thirdparty"
  }

  ohos_cargo_crate("stdio_fixture") {
    crate_type = "bin"
    crate_root = "src/bin/stdio-fixture.rs"
    # ... 配置与 lib 类似
  }
}
```

### 关键配置项解析

#### 1. 平台条件
```gn
if (host_os != "linux" || host_cpu != "arm64") {
```
**作用**: 在 Linux ARM64 平台上跳过构建

**原因**: 
- clap 仅用于宿主机构建工具
- Linux ARM64 交叉编译环境下无需重复构建
- 避免构建循环依赖

#### 2. crate 类型
```gn
crate_type = "rlib"  # Rust 静态库
```
**说明**: OH 中全部使用静态链接 (`rlib`)，不使用动态库 (`dylib`)

#### 3. Rust Edition
```gn
edition = "2021"
```
**说明**: 与上游 `Cargo.toml` 保持一致

#### 4. 依赖声明
```gn
deps = [
  "//third_party/rust/crates/bitflags:lib",
  "//third_party/rust/crates/clap/clap_derive:lib(${host_toolchain})",
  # ...
]
```
**关键依赖**:
| 依赖 | 用途 |
|------|------|
| bitflags | 位标志宏支持 |
| clap_derive | 派生宏（proc-macro）|
| clap_lex | 内部词法分析 |
| is-terminal | 终端检测 |
| once_cell | 懒初始化 |
| strsim-rs | 字符串相似度（错误建议）|
| termcolor | 终端颜色输出 |

**注意**: `clap_derive` 是 proc-macro，需要 `${host_toolchain}` 标记

#### 5. 特性控制
```gn
features = [
  "color",           # 彩色输出
  "error-context",   # 错误上下文
  "help",            # 帮助生成
  "std",             # 标准库
  "suggestions",     # 拼写建议
  "usage",           # 用法信息
  "derive",          # 派生宏
]
```

**与上游对比**:

| 特性 | OH 配置 | Cargo.toml 默认 | 说明 |
|------|---------|-----------------|------|
| color | ✓ | ✓ | 一致 |
| error-context | ✓ | ✓ | 一致 |
| help | ✓ | ✓ | 一致 |
| std | ✓ | ✓ | 一致 |
| suggestions | ✓ | ✓ | 一致 |
| usage | ✓ | ✓ | 一致 |
| derive | ✓ | ✗ | OH 启用 |

**差异说明**:
- OH 显式启用了 `derive` 特性（bindgen、cxxbridge 需要）
- 未启用的特性：`cargo`, `env`, `unicode`, `wrap_help`

#### 6. 部件化配置
```gn
part_name = "rust_clap"
subsystem_name = "thirdparty"
```
**说明**: 符合 OH 部件化要求，与 `bundle.json` 一致

## clap_derive BUILD.gn

### 文件位置
`third_party/rust/crates/clap/clap_derive/BUILD.gn`

### 关键配置

```gn
ohos_cargo_crate("lib") {
  crate_name = "clap_derive"
  crate_type = "proc-macro"    # 关键：指定为过程宏
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2021"
  cargo_pkg_version = "4.1.12"   # 注意：与 clap 版本不同
  
  deps = [
    "//third_party/rust/crates/heck:lib",
    "//third_party/rust/crates/proc-macro2:lib",
    "//third_party/rust/crates/quote:lib",
    "//third_party/rust/crates/syn:lib",
  ]
}
```

### 过程宏特殊处理

**特性**:
1. `crate_type = "proc-macro"` - 必须显式声明
2. 无 `features` 字段 - proc-macro crate 通常不使用特性标志
3. 版本号独立 - clap_derive 4.1.12 对应 clap 4.1.13

## clap_lex BUILD.gn

### 文件位置
`third_party/rust/crates/clap/clap_lex/BUILD.gn`

### 关键配置

```gn
ohos_cargo_crate("lib") {
  crate_name = "clap_lex"
  crate_type = "rlib"
  crate_root = "src/lib.rs"

  sources = [ "src/lib.rs" ]
  edition = "2021"
  cargo_pkg_version = "0.3.3"    # 独立版本号
  
  deps = [ "//third_party/rust/crates/os_str_bytes:lib" ]
}
```

## stdio_fixture 二进制目标

### 用途
辅助测试工具，位于 `src/bin/stdio-fixture.rs`

### 配置说明
```gn
ohos_cargo_crate("stdio_fixture") {
  crate_type = "bin"           # 二进制可执行文件
  crate_root = "src/bin/stdio-fixture.rs"
  # ...
}
```

**注意**: 二进制目标不参与库的链接，仅用于测试。

## GN 与 Cargo 差异对比

| 方面 | Cargo | GN (OH) |
|------|-------|---------|
| 构建工具 | cargo | gn + ninja |
| 依赖管理 | Cargo.toml | BUILD.gn 手动声明 |
| 特性控制 | 自动解析 | 手动列出 |
| 版本管理 | Semver 范围 | 精确版本 |
| 平台条件 | target triple | `host_os` / `host_cpu` |
| 缓存策略 | 增量编译 | 增量编译 |

## 构建流程

### 依赖构建顺序

```
1. heck, proc-macro2, quote, syn  (clap_derive 依赖)
2. clap_derive (proc-macro)
3. os_str_bytes
4. clap_lex
5. bitflags, is-terminal, once_cell, strsim-rs, termcolor
6. clap (主库)
```

### 链接关系

```
最终工具 (bindgen-cli / cxxbridge-cmd)
    ├── clap:rlib
    │   ├── clap_derive:proc-macro
    │   ├── clap_lex:rlib
    │   └── 其他依赖:rlib
    └── ...
```

## 升级操作指南

### 升级步骤

1. **获取新版本代码**
   ```bash
   git fetch upstream
   git merge upstream/v4.x.x
   ```

2. **更新 BUILD.gn 版本号**
   ```gn
   cargo_pkg_version = "新版本号"
   ```

3. **检查依赖变更**
   - 对比 `Cargo.toml` 和现有 BUILD.gn 的 `deps`
   - 如有新增/删除依赖，同步更新 BUILD.gn

4. **检查特性变更**
   - 对比 `Cargo.toml` 的 `features`
   - 更新 BUILD.gn 的 `features` 列表

5. **验证构建**
   ```bash
   gn gen out
   ninja -C out third_party/rust/crates/clap:lib
   ```

### 升级检查清单

- [ ] 版本号已更新
- [ ] 依赖列表已同步
- [ ] 特性列表已同步
- [ ] 构建成功
- [ ] 依赖者 (bindgen-cli, cxxbridge-cmd) 构建成功

---

*文档版本: v1.0*
