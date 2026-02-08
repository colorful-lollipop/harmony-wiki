# 原始库简介

## 项目概述

**cxx** 是由 David Tolnay 开发并维护的开源 Rust 库，旨在提供 Rust 与 C++ 之间安全、便捷的外部函数接口（FFI）。该项目起源于解决 Rust 生态系统中混合语言编程的痛点：传统的 `bindgen` 和 `cbindgen` 工具生成的 C 风格绑定存在安全隐患，且无法处理 C++ 的高级特性（如模板、智能指针、异常处理等）。

**核心设计理念**：让审计 C++ 代码足以保证整个 FFI 边界的正确性，即 Rust 侧可以做到 100% 安全。

## 核心特性

### 1. 类型安全的 FFI 定义

cxx 通过 `#[cxx::bridge]` 属性宏统一声明 FFI 边界两端的类型和函数：

```rust
#[cxx::bridge]
mod ffi {
    struct SharedData {
        id: i32,
        name: String,
    }

    extern "Rust" {
        type RustHandle;
        fn create_rust_handle() -> Box<RustHandle>;
        fn process_data(handle: &mut RustHandle, data: &SharedData);
    }

    unsafe extern "C++" {
        include!("demo.h");
        type CppHandle;
        fn create_cpp_handle() -> UniquePtr<CppHandle>;
        fn get_data(handle: &CppHandle) -> SharedData;
    }
}
```

### 2. 双向类型映射

cxx 自动处理 Rust 与 C++ 标准库类型之间的转换：

| Rust 类型 | C++ 类型 | 说明 |
|----------|---------|------|
| `String` | `rust::String` | UTF-8 字符串 |
| `&str` | `rust::Str` | 字符串切片 |
| `Box<T>` | `rust::Box<T>` | 堆分配指针 |
| `Vec<T>` | `rust::Vec<T>` | 动态数组 |
| `Result<T, E>` | throw/catch | 错误传播 |
| `*const T` / `*mut T` | `T*` / `const T*` | 原始指针 |
| `std::string` | `CxxString` | C++ 字符串 |
| `std::unique_ptr<T>` | `UniquePtr<T>` | 独占指针 |
| `std::shared_ptr<T>` | `SharedPtr<T>` | 共享指针 |
| `std::vector<T>` | `CxxVector<T>` | C++ 向量 |

### 3. 零开销设计

cxx 生成的 FFI 桥接代码在运行时几乎没有额外开销：
- 不进行数据拷贝（共享内存布局）
- 不进行序列化/反序列化
- 不进行动态内存分配（除非必要）
- 运行时类型检查最小化

### 4. 静态分析保障

在编译时，cxx 会执行以下检查：
- **ABI 兼容性验证**：确保 Rust 和 C++ 端的类型布局一致
- **不可拷贝类型检测**：防止按值传递不应拷贝的类型
- **所有权转移验证**：确保 `Box`/`unique_ptr` 的正确转移

## 工作原理

### 代码生成流程

1. **宏展开阶段**：Rust 编译器调用 `cxxbridge-macro` 过程宏
2. **类型分析**：分析 `#[cxx::bridge]` 模块中的类型和函数声明
3. **静态断言生成**：生成 C++ 静态断言验证类型兼容性
4. **桥接代码生成**：生成 Rust 端和 C++ 端的胶水代码
5. **编译集成**：C++ 代码通过 `cc` crate 编译，Rust 代码正常编译

### 架构组件

```
用户代码
    |
    v
┌───────────────────────────────┐
│   #[cxx::bridge] 模块定义      │
└───────────────────────────────┘
    |
    +──► Rust 宏 (cxxbridge-macro)
    |         │
    │         +──► Rust FFI 桥接代码
    │         └──► C++ 头文件模板
    │
    +──► C++ 构建脚本 (cxx-build)
              │
              +──► C++ 静态断言
              └──► C++ 桥接实现 (cxx.cc)
```

## 使用示例

### Rust 调用 C++

```rust
#[cxx::bridge]
mod ffi {
    unsafe extern "C++" {
        include!("math.h");
        fn add(a: i32, b: i32) -> i32;
        fn factorial(n: i32) -> i32;
    }
}

fn main() {
    let result = unsafe { ffi::add(10, 20) };
    println!("10 + 20 = {}", result);
}
```

### C++ 调用 Rust

```rust
#[cxx::bridge]
mod ffi {
    extern "Rust" {
        fn rust_greeting(name: &str) -> String;
        fn compute_sum(values: &[i32]) -> i32;
    }
}

fn rust_greeting(name: &str) -> String {
    format!("Hello, {}!", name)
}

fn compute_sum(values: &[i32]) -> i32 {
    values.iter().sum()
}
```

### 处理复杂类型

```rust
#[cxx::bridge]
mod ffi {
    struct Point {
        x: f64,
        y: f64,
    }

    unsafe extern "C++" {
        type Circle;
        fn get_center(&self) -> Point;
        fn get_radius(&self) -> f64;
        fn area(&self) -> f64;
    }
}
```

## 版本要求

| 依赖 | 版本要求 |
|-----|---------|
| Rust 编译器 | 1.70+ |
| C++ 编译器 | C++11 或更新 |

## 在 OpenHarmony 中的定位

cxx 库在 OpenHarmony 中扮演**基础设施**角色，主要服务于：

1. **Rust 生态桥接**：为 OH 中的 Rust crates 提供 C++ 互操作能力
2. **混合语言模块**：支持 Rust 模块调用系统级 C++ 组件
3. **构建工具链**：cxxbridge 工具支持非 Cargo 构建系统的集成

由于 OH 采用 Rust 作为系统开发语言之一，cxx 成为 Rust/C++ 混合编程的核心依赖。
