# libloading 原始库简介

## 1.1 基本信息

| 属性 | 值 |
|------|-----|
| 库名称 | libloading |
| 当前版本 | 0.7.4 |
| 上游地址 | https://github.com/nagisa/rust_libloading |
| 许可证 | MIT / ISC |
| 首次发布 | 2016 年 |
| Rust 版本要求 | 1.40.0+ |
| Rust Edition | 2015 |

## 1.2 核心功能

libloading 是一个 Rust 库，为 Rust 程序提供**动态链接库（Dynamic Library）加载**能力。其主要功能包括：

### 动态库加载

- **`Library::new()`** - 加载动态库文件
- **`Library::symbol()`** - 获取函数或变量符号地址
- **`Library::close()`** - 卸载动态库

### 符号解析

- 安全地将 C 字符串转换为 Rust 字符串
- 支持函数指针获取
- 支持变量地址获取

### 平台抽象

- Unix/Linux: 使用 `dlopen`, `dlsym`, `dlclose`
- macOS: 使用 `dlopen`, `dlsym`, `dlclose`
- Windows: 使用 `LoadLibrary`, `GetProcAddress`, `FreeLibrary`

## 1.3 设计特点

### 内存安全增强

libloading 的核心价值在于将不安全的原生 C API 封装为安全的 Rust 接口：

```rust
// 原生 C API（不安全）
let handle = dlopen(path.as_ptr(), flags);
let symbol = dlsym(handle, name.as_ptr());

// libloading 封装（安全）
let library = Library::new(path)?;
let func: Symbol<unsafe extern "C" fn()> = library.get(name)?;
```

### 零依赖设计

libloading 除平台特定的系统库外，仅依赖 `cfg-if` 一个 Rust crate，实现了极简的依赖管理。

### 跨平台一致性

通过统一的 Rust 接口抽象不同平台的动态库加载 API，开发者可以使用相同的代码在不同平台上工作。

## 1.4 典型使用场景

### Plugin 系统

```rust
// 加载 Plugin 动态库
let plugin = Library::new("./plugins/my_plugin.so")?;

// 获取 Plugin 初始化函数
let init_func: Symbol<fn() -> i32> = plugin.get("plugin_init")?;

// 调用初始化
let result = init_func();
```

### FFI 互操作

```rust
// 加载 C 库
let crypto = Library::new("libcrypto.so")?;

// 获取加密函数
let encrypt: Symbol<fn(*const u8, usize, *mut u8) -> i32> = 
    crypto.get("MD5_Update")?;
```

### 运行时扩展

```rust
// 根据配置动态加载模块
let module_path = config.get_module_path();
let module = Library::new(module_path)?;

// 获取模块接口
let create_interface: Symbol<fn() -> Box<dyn ModuleInterface>> = 
    module.get("create_module")?;
```

## 1.5 与同类库的对比

| 库名称 | 特点 | 适用场景 |
|--------|------|----------|
| libloading | 简单、安全、跨平台 | 通用动态库加载 |
| winapi | Windows API 绑定 | Windows 特定功能 |
| nix | Unix 系统调用 | Unix 系统编程 |
| libc | C 标准库绑定 | 底层系统交互 |

libloading 在功能简洁性和安全性之间取得了良好的平衡，适合大多数动态库加载需求。

## 1.6 在 OpenHarmony 中的定位

在 OpenHarmony 生态中，libloading 作为 Rust 基础设施组件，提供以下能力：

1. **Rust 动态扩展**：支持 Rust 程序运行时加载原生代码
2. **Native 模块集成**：方便 Rust 与 C/C++ 模块的互操作
3. **Plugin 机制基础**：为可能的 Plugin 系统提供底层支持

由于 OpenHarmony 基于 Linux 内核，libloading 的 Unix 分支代码可以直接工作，无需进行 OH 特定的修改。
