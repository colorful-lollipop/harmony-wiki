# 05 - API/接口差异

## 结论

**bindgen 在 OpenHarmony 中未修改上游 API。**

所有 API 与上游 bindgen 0.70.1 完全一致。OH 的适配完全通过构建系统层完成，不涉及源码修改。

---

## 上游 API 概览

### Library API (bindgen crate)

```rust
use bindgen::Builder;

// 基础用法
let bindings = Builder::default()
    .header("input.h")
    .parse_callbacks(Box::new(bindgen::CargoCallbacks))
    .generate()
    .expect("Unable to generate bindings");

bindings
    .write_to_file("output.rs")
    .expect("Couldn't write bindings!");
```

### 常用配置方法

| 方法 | 说明 |
|-----|-----|
| `header(path)` | 指定输入头文件 |
| `clang_arg(arg)` | 添加 clang 参数 |
| `whitelist_type(pattern)` | 白名单类型 |
| `whitelist_function(pattern)` | 白名单函数 |
| `blacklist_type(pattern)` | 黑名单类型 |
| `derive_debug(enable)` | 控制 Debug derive |
| `layout_tests(enable)` | 控制布局测试生成 |
| `rust_target(target)` | 指定 Rust 目标版本 |

### CLI API (bindgen CLI)

```bash
bindgen [OPTIONS] [HEADER] -- [CLANG_ARGS]

# 常用选项
--output <FILE>          # 输出文件路径
--no-layout-tests        # 禁用布局测试
--rust-target <VERSION>  # Rust 目标版本
--whitelist-type <TYPE>  # 类型白名单
--blacklist-type <TYPE>  # 类型黑名单
```

---

## OH 特有的使用模式

虽然 API 本身未变，但 OH 构建系统封装了特定的使用模式：

### 1. 默认参数差异

| 参数 | 上游默认值 | OH 强制值 | 说明 |
|-----|-----------|----------|-----|
| `--no-layout-tests` | 未设置 | 启用 | 禁用布局测试，减少生成代码量 |
| `--rust-target` | 1.47 | nightly | 使用 nightly 特性 |

**实现位置**: `//build/templates/rust/rust_bindgen.py`

```python
ohos_genargs = []
ohos_genargs.append('--no-layout-tests')
ohos_genargs += ['--rust-target', 'nightly']
```

### 2. 环境变量配置

| 环境变量 | OH 设置 | 说明 |
|---------|---------|-----|
| `LLVM_CONFIG_PATH` | `$clang_base/bin/llvm-config` | 显式指定 LLVM 配置 |
| `CLANG_PATH` | `$clang_base/bin/clang` | 显式指定 clang 路径 |
| `LD_LIBRARY_PATH` | `$clang_base/lib` | 确保找到 libclang.so |

**实现位置**: `//build/templates/rust/rust_bindgen.py`

```python
env["LLVM_CONFIG_PATH"] = args.llvm_config_path
env["CLANG_PATH"] = args.clang_path
env["LD_LIBRARY_PATH"] = args.ld_library_path
```

### 3. clang 参数过滤

OH 特有的参数被过滤：

| 参数模式 | 处理 | 说明 |
|---------|-----|-----|
| `-Xclang` | 过滤 | clang 插件参数 |
| `-plugin-arg-*` | 过滤 | 插件参数 |
| `-add-plugin` | 过滤 | 插件加载 |

**实现位置**: `//build/templates/rust/rust_bindgen.py`

```python
def remove_args_of_clang(ohos_clangargs):
    # 过滤 OH 特有的 clang 插件参数
    for i, j in enumerate(args):
        if args[i] == '-Xclang':
            i += 1
            # ...
```

---

## 与上游行为对比

### 上游直接使用

```bash
# 环境自动检测
$ bindgen input.h -o output.rs

# 手动指定 clang
$ BINDGEN_EXTRA_CLANG_ARGS="--sysroot=/path" bindgen input.h -o output.rs
```

### OH 构建系统使用

```gn
rust_bindgen("bindings") {
  header = "input.h"
}
# 自动处理所有环境配置
```

**差异总结**:

| 方面 | 上游 | OH |
|-----|------|-----|
| clang 发现 | 自动检测 libclang | 显式配置 LLVM_CONFIG_PATH/CLANG_PATH |
| 参数传递 | 直接传递 | 过滤 OH 特有参数 |
| 输出控制 | 手动指定 | 自动生成到 `$target_gen_dir` |
| 依赖跟踪 | 手动维护 | 自动生成 depfile |

---

## 新增/变更的行为

### 无新增 API

OH 未向 bindgen 添加任何新的 API 方法或配置选项。

### 封装的构建行为

OH 添加的是构建系统层的封装，而非 API 层：

1. **rust_bindgen.gni 模板**: 提供声明式接口，底层仍调用标准 bindgen
2. **rust_bindgen.py 包装**: 参数预处理，不改变 bindgen 行为

---

## 废弃或禁用的功能

### 布局测试 (Layout Tests)

在 OH 中默认禁用：

```python
# rust_bindgen.py
ohos_genargs.append('--no-layout-tests')
```

**原因**:
- 布局测试需要运行生成的 Rust 测试代码
- 在交叉编译环境下执行复杂
- OH 通过其他方式保证 ABI 兼容性

**如需启用**:
当前 rust_bindgen 模板不支持自定义此选项。如需启用，需修改模板或直接使用 bindgen CLI。

---

## 迁移指南

### 从上游 bindgen 迁移到 OH rust_bindgen

#### 上游代码 (Cargo 构建)

```rust
// build.rs
use bindgen::Builder;

fn main() {
    Builder::default()
        .header("wrapper.h")
        .clang_arg("-I/some/path")
        .generate()
        .unwrap()
        .write_to_file("src/bindings.rs")
        .unwrap();
}
```

#### OH 代码 (GN 构建)

```gn
# BUILD.gn
rust_bindgen("bindings") {
  header = "wrapper.h"
  # include_dirs 通过 deps 和 configs 传递
}

ohos_rust_static_library("my_lib") {
  deps = [ ":bindings" ]
  sources = [ "src/lib.rs" ]
  rustenv = [ "BINDINGS_RS_FILE=" + rebase_path(...) ]
}
```

```rust
// src/lib.rs
include!(env!("BINDINGS_RS_FILE"));
```

### 注意事项

1. **头文件路径**: 使用绝对路径 `//path/to/header.h`
2. **Include 路径**: 通过 GN 依赖关系自动传递，而非手动 clang_arg
3. **生成文件位置**: 通过 `get_target_outputs()` 获取，不固定
