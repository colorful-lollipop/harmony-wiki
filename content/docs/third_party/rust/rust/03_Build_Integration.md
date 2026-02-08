# OH 构建适配

本文档详细说明 Rust 工具链在 OpenHarmony 构建系统中的集成方式。

## 构建系统概览

### 架构层次

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 构建系统                      │
├─────────────────────────────────────────────────────────────┤
│  GN (Generate Ninja) 构建配置                               │
│  ├── build/rust/rustc_toolchain.gni                        │
│  ├── build/templates/rust/*.gni                           │
│  └── 各子系统的 BUILD.gn                                   │
├─────────────────────────────────────────────────────────────┤
│  Rust 工具链 (prebuilts)                                    │
│  ├── prebuilts/rustc/linux-x86_64/                        │
│  │   ├── current/                                         │
│  │   │   ├── bin/                                         │
│  │   │   ├── lib/                                         │
│  │   │   └── libexec/                                     │
│  │   └── nightly/                                         │
│  └── third_party/rust/crates/                              │
│       └── 56 个第三方 crate                                │
├─────────────────────────────────────────────────────────────┤
│  Rust 源码构建                                              │
│  └── third_party/rust/rust/                                │
│       └── rust-build/ohos_ci_build.sh                      │
└─────────────────────────────────────────────────────────────┘
```

## 关键 GN 配置文件

### rustc_toolchain.gni

**路径**: `//build/rust/rustc_toolchain.gni`

定义 Rust 工具链的核心路径和配置：

```gn
# 工具链路径配置
rust_sysroot = "//prebuilts/rustc/linux-x86_64/current"
rustc_path = rust_sysroot + "/bin/rustc"
cargo_path = rust_sysroot + "/bin/cargo"

# OHOS 目标配置
ohos_targets = [
  "aarch64-unknown-linux-ohos",
  "armv7-unknown-linux-ohos",
  "x86_64-unknown-linux-ohos",
]

# 默认构建配置
rust_version = "nightly"  # 或 "stable"
```

### rust_template.gni

**路径**: `//build/templates/rust/rust_template.gni`

提供所有 Rust 构建模板的基础：

```gn
# 导入模板
import("//build/templates/rust/rust_template.gni")

# 可用的构建模板
# ohos_rust_executable     - Rust 可执行文件
# ohos_rust_static_library - Rust 静态库 (.rlib)
# ohos_rust_shared_library - Rust 共享库 (.so)
# ohos_rust_shared_ffi     - FFI 兼容共享库
# ohos_rust_static_ffi     - FFI 兼容静态库
# ohos_rust_proc_macro     - 过程宏
```

### ohos_cargo_crate.gni

**路径**: `//build/templates/rust/ohos_cargo_crate.gni`

用于将第三方 Cargo crate 转换为 GN 构建规则：

```gn
# 示例：添加第三方 crate
ohos_cargo_crate("serde") {
  crate_name = "serde"
  version = "1.0.195"
  cargo_toml = "//third_party/rust/crates/serde/Cargo.toml"
  deps = [
    "//third_party/rust/crates:serde_json",
    "//third_party/rust/crates:serde_derive",
  ]
}
```

## OHOS 特定构建配置

### 目标三元组注册

在 `third_party/rust/rust/rust-build/config.toml` 中定义：

```toml
[build]
target = [
  "x86_64-unknown-linux-gnu",       # 主机工具链
  "x86_64-pc-windows-gnullvm",       # Windows 交叉编译
  "armv7-unknown-linux-ohos",        # 32位 ARM OHOS
  "x86_64-unknown-linux-ohos",      # 64位 x86 OHOS
  "aarch64-unknown-linux-ohos",      # 64位 ARM OHOS
]

[target.aarch64-unknown-linux-ohos]
cc = "aarch64-unknown-linux-ohos-clang"
cxx = "aarch64-unknown-linux-ohos-clang++"
linker = "aarch64-unknown-linux-ohos-clang"
ar = "llvm-ar"
```

### Clang 包装器

为每个 OHOS 目标创建了 Clang 包装器脚本：

| 脚本 | 路径 |
|------|------|
| aarch64 OHOS | `rust-build/tools/aarch64-unknown-linux-ohos-clang` |
| armv7 OHOS | `rust-build/tools/armv7-unknown-linux-ohos-clang` |
| x86_64 OHOS | `rust-build/tools/x86_64-unknown-linux-ohos-clang` |

**关键编译标志**:

```bash
# 来自 rust-build/tools/aarch64-unknown-linux-ohos-clang
FLAGS="$FLAGS -Imusl"
FLAGS="$FLAGS -fstack-protector-all"
exec clang -target aarch64-linux-ohos $FLAGS "$@"
```

### LLVM 构建配置

```toml
[llvm]
download-ci-llvm = false
targets = "X86;AArch64;ARM"
cflags = "-fstack-protector-all"
cxxflags = "-fstack-protector-all"
```

## 构建产物

### 输出目录结构

```
output/
├── rust-nightly-x86_64-unknown-linux-gnu.tar.gz    # 主机工具链
├── rust-std-nightly-aarch64-unknown-linux-ohos.tar.gz  # ARM64 std
├── rust-std-nightly-armv7-unknown-linux-ohos.tar.gz    # ARMv7 std
├── rust-std-nightly-x86_64-unknown-linux-ohos.tar.gz  # x86_64 std
└── rust-std-nightly-x86_64-pc-windows-gnullvm.tar.gz  # Windows std
```

### 标准库组件

每个 `rust-std-*.tar.gz` 包含：

```
lib/
├── libstd-*.rlib           # 标准库
├── libcore-*.rlib          # 核心库
├── liballoc-*.rlib         # 分配库
├── libproc_macro-*.rlib    # 过程宏库
└── libtest-*.rlib          # 测试框架
```

## 编译器选项

### 全局编译配置

```toml
[rust]
optimize = true              # 优化开启
codegen-units = 1            # 单个 codegen unit (最大优化)
lto = "thin-local"           # 链接时优化
stack-protector = "strong"    # 强栈保护
channel = "nightly"          # nightly 通道
strip = true                 # 符号剥离
```

### per-target 配置

```toml
[target.x86_64-unknown-linux-gnu]
cc = "clang"
cxx = "clang++"
linker = "clang"
ar = "llvm-ar"
```

## Rust 运行库构建

### 运行时库位置

```
//build/rust/
├── libstd/                  # 标准库
│   ├── BUILD.gn            # 构建配置
│   └── src/               # 标准库源码
├── libproc_macro/          # 过程宏
└── tests/                  # 测试配置
```

### 库清单 (BUILD.gn)

```gn
rust_libraries = [
  "libstd",
  "libtest",
  "libpanic_abort",
  "libpanic_unwind",
  "libprocb_hooks",
  "libunwind",
]
```

## 第三方 Crate 管理

### Crate 仓库结构

```
//third_party/rust/crates/
├── bindgen/                # FFI 绑定生成器
├── clap/                   # CLI 解析器
├── serde/                  # 序列化框架
├── regex/                  # 正则表达式
├── cxx/                    # C++ 互操作
├── rust-openssl/           # OpenSSL 绑定
├── nix/                    # Unix 系统绑定
├── libc/                   # C 库绑定
├── rand/                   # 随机数生成
├── getrandom/              # 随机数源
└── ... (45 more crates)
```

### 添加新 Crate

1. 创建目录: `mkdir //third_party/rust/crates/new_crate`
2. 添加 `Cargo.toml`
3. 运行 `cargo2gn.py` 生成 BUILD.gn
4. 添加到 `//third_party/rust/crates/BUILD.gn`

## 测试配置

### Rust 测试类型

| 类型 | GN 模板 | 用途 |
|------|--------|------|
| 单元测试 | `ohos_rust_unittest` | 模块级测试 |
| 系统测试 | `ohos_rust_systemtest` | 集成测试 |
| 冒烟测试 | `ohos_rust_sanitytest` | 快速验证 |

### 测试配置示例

```gn
ohos_rust_unittest("my_rust_test") {
  sources = [ "src/lib.rs" ]
  deps = [
    "//third_party/rust/crates:testing_framework",
  ]
  test_only = true
}
```

## 与上游构建系统的差异

| 特性 | 上游 Rust | OH 适配 |
|------|----------|---------|
| 构建系统 | Cargo | GN + Cargo |
| 包管理器 | Cargo | cargo2gn 转换 |
| 目标平台 | 30+ | 3 (OHOS) + 2 (主机) |
| 标准库 | 完整构建 | 选择性构建 |
| LLVM 集成 | 系统 LLVM | 自定义 LLVM 15 |
| 测试框架 | cargo test | ohos_rust_unittest |

## 常见问题

### Q1: 如何添加新的 OHOS 目标?

1. 在 `third_party/rust/rust/compiler/rustc_target/src/spec/` 添加目标定义
2. 更新 `rust-build/config.toml` 的 target 列表
3. 创建对应的 Clang 包装器
4. 更新 `build/rust/rustc_toolchain.gni`

### Q2: 如何更新 Rust 版本?

1. 修改 `rust-build/config.toml` 中的版本配置
2. 更新 `rust-static-source` 版本号
3. 重新运行 `ohos_ci_build.sh`
4. 更新 prebuilts 软链接

### Q3: 为什么使用 nightly 而不是 stable?

- **历史原因**: 项目启动时 nightly 功能更完善
- **特性需求**: 某些 OH 功能依赖 nightly 特性
- **建议**: 评估迁移到 stable 的可行性

## 参考资料

- [OpenHarmony Rust 工具链文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-rust-toolchain.md)
- [Cargo2GN 指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-cargo2gn-guide.md)
- [上游 Rust 构建文档](https://doc.rust-lang.org/nightly/rustc/platform-support.html)
