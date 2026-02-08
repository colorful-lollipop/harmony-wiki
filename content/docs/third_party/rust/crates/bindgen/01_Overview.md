# 01 - 原始库简介

## bindgen 概述

bindgen 是 Rust 生态系统中广泛使用的 FFI (Foreign Function Interface) 绑定生成工具。它通过解析 C/C++ 头文件，自动生成对应的 Rust FFI 声明代码。

### 核心功能

```c
// 输入: C 头文件 (example.h)
typedef struct MyStruct {
    int x;
    char* name;
} MyStruct;

void my_function(MyStruct* s);
```

```rust
// 输出: Rust 绑定 (example.rs)
#[repr(C)]
pub struct MyStruct {
    pub x: ::std::os::raw::c_int,
    pub name: *mut ::std::os::raw::c_char,
}

extern "C" {
    pub fn my_function(s: *mut MyStruct);
}
```

### 技术特点

| 特性 | 说明 |
|-----|-----|
| **libclang 驱动** | 使用 clang 解析头文件，支持完整 C/C++ 语法 |
| **可定制化** | 支持黑名单、类型替换、回调函数等多种配置 |
| **布局测试** | 可生成测试验证 Rust 和 C 结构体布局一致 |
| **宏处理** | 支持将 C 宏转换为 Rust const |

## 上游信息

| 属性 | 值 |
|-----|-----|
| **官方仓库** | https://github.com/rust-lang/rust-bindgen |
| **官方文档** | https://rust-lang.github.io/rust-bindgen |
| **API 文档** | https://docs.rs/bindgen |
| **crate 地址** | https://crates.io/crates/bindgen |
| **MSRV** | Rust 1.70.0 |

## OpenHarmony 中的定位

### 在 OH 架构中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Applications)                    │
├─────────────────────────────────────────────────────────────┤
│                   ArkTS / ArkUI (ArkUI-X)                   │
├─────────────────────────────────────────────────────────────┤
│              ArkCompiler (ArkTS 运行时)                      │
├─────────────────────────────────────────────────────────────┤
│    ANI (Ark Native Interface) ←──── bindgen 生成绑定        │
├─────────────────────────────────────────────────────────────┤
│   Rust 组件 (data_share, netmanager_base...)                │
├─────────────────────────────────────────────────────────────┤
│              C/C++ 系统服务 / 第三方库                        │
└─────────────────────────────────────────────────────────────┘
```

### OH 特定使用场景

1. **ANI 绑定生成**
   - 为 `arkcompiler/runtime_core/static_core/plugins/ets/runtime/ani/ani.h` 生成 Rust 绑定
   - 使 Rust 组件能够调用 ArkTS 运行时接口
   - 主要使用者: `data_share`, `netmanager_base`

2. **Rust/C++ 互操作**
   - Rust 组件需要调用 C/C++ 库时，通过 bindgen 生成绑定
   - 避免手动编写繁琐且不安全的 FFI 声明

3. **构建时工具链**
   - bindgen 作为 host 工具编译，不参与 target 镜像构建
   - 在编译阶段生成绑定文件，编译后产物为普通 Rust 代码

### 为什么不直接手写 FFI？

| 方式 | 优点 | 缺点 |
|-----|------|-----|
| **手写 FFI** | 完全可控，无额外依赖 | 繁琐、易出错、难以维护、头文件更新需手动同步 |
| **bindgen** | 自动化、准确、可复现、支持复杂 C++ | 需要配置、依赖 libclang |

在 OpenHarmony 这种大型系统中，bindgen 是维护 Rust/C++ 互操作层的标准做法。

## 版本信息

| 版本 | 说明 |
|-----|-----|
| **上游最新** | 0.70.1 (2024-08-20) |
| **OH 声明** | 0.70.1 (README.OpenSource) |
| **OH BUILD.gn** | 0.64.0 ⚠️ **需更新** |

### 升级历史 (从 git log)

- `7ad92f7d` - merge master into master
- `d6ca6fb5` - 升级 rust 版本到 0.70.1
- `01c87a7b` - bindgen 升级 0.70.1 版本
- `ad75cef3` - 适配 syn 升级至 2.0
- `72caa0ad` - bitflags 升级，配套修改

## 许可证

BSD-3-Clause-LBNL (Lawrence Berkeley National Laboratory)

- 允许商业使用
- 允许修改和分发
- 需保留版权声明和免责声明
