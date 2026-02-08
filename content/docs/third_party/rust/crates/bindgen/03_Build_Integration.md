# 03 - OH 构建适配

## 概述

OpenHarmony 使用 GN (Generate Ninja) 构建系统，而 bindgen 原生使用 Cargo。OH 通过 `ohos_cargo_crate` 模板和配套配置，将 Cargo 项目集成到 GN 构建系统中。

## BUILD.gn 结构

### bindgen/BUILD.gn (库)

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "bindgen"
    crate_type = "rlib"
    crate_root = "./lib.rs"

    sources = ["./lib.rs"]
    edition = "2018"
    cargo_pkg_version = "0.64.0"  # ⚠️ 应为 0.70.1
    
    deps = [
        "//third_party/rust/crates/bitflags:lib",
        "//third_party/rust/crates/rust-cexpr:lib",
        "//third_party/rust/crates/clang-sys:lib",
        "//third_party/rust/crates/lazy-static.rs:lib",
        "//third_party/rust/crates/lazycell:lib",
        "//third_party/rust/crates/log:lib",
        "//third_party/rust/crates/peeking_take_while:lib",
        "//third_party/rust/crates/proc-macro2:lib",
        "//third_party/rust/crates/quote:lib",
        "//third_party/rust/crates/regex:lib",
        "//third_party/rust/crates/rustc-hash:lib",
        "//third_party/rust/crates/shlex:lib",
        "//third_party/rust/crates/syn:lib",
        "//third_party/rust/crates/which-rs:lib",
    ]
    
    features = [
        "cli",
        "experimental",
        "log",
        "logging",
        "static",
        "which",
        "which-rustfmt",
    ]
    
    build_root = "build.rs"
    build_sources = ["build.rs"]
    build_script_outputs = ["host-target.txt"]
}
```

### bindgen-cli/BUILD.gn (CLI)

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
import("//build/ohos.gni")

ohos_cargo_crate("bindgen") {
    crate_type = "bin"
    crate_root = "main.rs"

    sources = [ "main.rs" ]
    edition = "2018"
    cargo_pkg_version = "0.64.0"  # ⚠️ 应为 0.70.1
    
    deps = [
        "//third_party/rust/crates/bindgen/bindgen:lib",
        "//third_party/rust/crates/clap:lib",
        "//third_party/rust/crates/env_logger:lib",
        "//third_party/rust/crates/log:lib",
        "//third_party/rust/crates/shlex:lib",
    ]
    
    features = [
        "env_logger",
        "log",
        "logging",
        "static",
        "which-rustfmt",
    ]
    
    part_name = "rust_bindgen"
    subsystem_name = "thirdparty"
}
```

## 关键编译选项

### Features 对比

| Feature | bindgen 库 | bindgen-cli | 说明 |
|---------|-----------|-------------|-----|
| `cli` | ✅ | - | 启用 CLI 支持 |
| `experimental` | ✅ | - | 实验性功能 |
| `log` | ✅ | ✅ | 日志支持 |
| `logging` | ✅ | ✅ | 详细日志 |
| `static` | ✅ | ✅ | 静态链接 libclang |
| `which` | ✅ | - | 查找 rustfmt |
| `which-rustfmt` | ✅ | ✅ | rustfmt 支持 |
| `env_logger` | - | ✅ | 环境日志 |

### 与上游的差异

#### 1. 版本号不一致

```
上游 Cargo.toml:    version = "0.70.1"
OH BUILD.gn:        cargo_pkg_version = "0.64.0"
```

**影响**: 可能导致构建缓存问题或版本混淆。

**修复**: 更新 BUILD.gn 中的 `cargo_pkg_version`。

#### 2. 依赖映射

OH 将 Cargo 依赖映射到 GN 目标：

```toml
# Cargo.toml (上游)
[dependencies]
bitflags = "2.2.1"
clang-sys = { version = "1", features = ["clang_6_0"] }
```

```gn
# BUILD.gn (OH)
deps = [
    "//third_party/rust/crates/bitflags:lib",
    "//third_party/rust/crates/clang-sys:lib",
]
features = [
    "static",  # 对应 clang-sys/static
]
```

#### 3. 构建脚本处理

bindgen 使用 `build.rs` 检测目标平台：

```gn
build_root = "build.rs"
build_sources = ["build.rs"]
build_script_outputs = ["host-target.txt"]
```

## rust_bindgen 模板

OH 提供 `rust_bindgen` GN 模板，封装 bindgen 调用：

### 模板定义

**文件**: `//build/templates/rust/rust_bindgen.gni`

```gn
template("rust_bindgen") {
  assert(defined(invoker.header),
         "Must specify the C header file to make bindings for.")
  
  action(target_name) {
    # ...
    
    ohos_bindgen_target = "rust_bindgen:bindgen($host_toolchain)"
    
    # 可执行文件路径
    if (ohos_indep_compiler_enable) {
      ohos_bindgen_executable =
          "${ohos_bindgen_obj_dir}/clang_x64/libs/bindgen"
    } else {
      ohos_bindgen_executable =
          "${ohos_bindgen_obj_dir}/thirdparty/rust_bindgen/bindgen"
    }
    
    # clang 工具链配置
    llvm_config_path = "$default_clang_base_path/bin/llvm-config"
    clang_path = "$default_clang_base_path/bin/clang"
    
    outputs = [ "$target_gen_dir/${target_name}.rs" ]
    
    args = [
      "--exe", rebase_path(ohos_bindgen_executable),
      "--llvm-config-path", rebase_path(llvm_config_path),
      "--clang-path", rebase_path(clang_path),
      "--header", rebase_path(invoker.header),
      # ...
    ]
  }
}
```

### 模板参数

| 参数 | 类型 | 必需 | 说明 |
|-----|------|-----|-----|
| `header` | string | ✅ | 要生成绑定的 C/C++ 头文件路径 |
| `enable_c_plus_plus` | bool | ❌ | 启用 C++ 模式 (默认 false) |
| `configs` | list | ❌ | 额外的编译器配置 |
| `deps` | list | ❌ | 依赖的其他 GN 目标 |
| `visibility` | list | ❌ | 可见性控制 |
| `testonly` | bool | ❌ | 是否仅用于测试 |
| `part_name` | string | ❌ | OH 部件名称 |
| `subsystem_name` | string | ❌ | OH 子系统名称 |

### 使用示例

```gn
import("//build/ohos.gni")

rust_bindgen("ani_bindgen_h") {
  header = "//arkcompiler/runtime_core/static_core/plugins/ets/runtime/ani/ani.h"
}

ohos_rust_static_library("ani_sys") {
  deps = [ ":ani_bindgen_h" ]
  sources = [ "src/lib.rs" ]
  
  bindgen_output = get_target_outputs(":ani_bindgen_h")
  inputs = bindgen_output
  rustenv = [ "ANI_BINDGEN_RS_FILE=" + rebase_path(bindgen_output[0]) ]
}
```

## rust_bindgen.py 包装脚本

**文件**: `//build/templates/rust/rust_bindgen.py`

### 功能

1. **参数过滤**: 移除 OH 特有的 clang 插件参数
2. **环境设置**: 配置 LLVM_CONFIG_PATH 和 CLANG_PATH
3. **默认参数**: 注入 OH 推荐的 bindgen 参数
4. **错误处理**: 失败时清理临时文件

### 关键代码

```python
def remove_args_of_clang(ohos_clangargs):
    """过滤 OH 特有的 -Xclang 插件参数"""
    def filter_args(args):
        for i, j in enumerate(args):
            if args[i] == '-Xclang':
                i += 1
                if args[i].startswith('-plugin-arg'):
                    i += 2
                elif args[i] == '-add-plugin':
                    pass
            else:
                yield args[i]
    return list(filter_args(args))

def main():
    # OH 默认参数
    ohos_genargs = []
    ohos_genargs.append('--no-layout-tests')  # 禁用布局测试
    ohos_genargs += ['--rust-target', 'nightly']  # 使用 nightly Rust
    
    # 环境变量
    env["LLVM_CONFIG_PATH"] = args.llvm_config_path
    env["CLANG_PATH"] = args.clang_path
```

### 与上游 bindgen 的差异

| 方面 | 上游 bindgen | OH rust_bindgen |
|-----|-------------|-----------------|
| 布局测试 | 可选启用 | 默认禁用 (`--no-layout-tests`) |
| Rust 目标 | 默认 stable | nightly |
| clang 参数 | 直接传递 | 过滤 OH 插件参数 |
| 环境变量 | 自动检测 | 显式配置 |

## 升级建议

### BUILD.gn 版本号修复

```gn
# 修改前
cargo_pkg_version = "0.64.0"

# 修改后
cargo_pkg_version = "0.70.1"
```

两个文件需同步修改:
- `bindgen/BUILD.gn`
- `bindgen-cli/BUILD.gn`

### 依赖版本检查

升级后验证:
1. 依赖的 Rust crates 版本兼容性
2. `clang-sys` 与 OH LLVM 版本兼容性
3. 生成的绑定代码是否能正常编译
