# API/接口差异

## 5.1 概述

libloading 在 OpenHarmony 中的 API 与上游版本**完全一致**，未进行任何修改或扩展。

这意味着：
- ✅ 上游 API 100% 兼容
- ✅ 无新增 OH 特有 API
- ✅ 无行为变更
- ✅ 无禁用功能
- ✅ 可直接使用上游文档和示例

## 5.2 核心 API 列表

### Library 结构体

| 方法 | 返回类型 | 说明 | OH 状态 |
|------|----------|------|---------|
| `new(path: &Path)` | `Result<Self, Error>` | 加载动态库 | ✅ 一致 |
| `get<T>(name: &[u8])` | `Result<Symbol<T>, Error>` | 获取符号地址 | ✅ 一致 |
| `into_raw(self)` | `*mut libc::c_void` | 获取原始句柄 | ✅ 一致 |
| `from_raw(handle: *mut libc::c_void)` | `Result<Self, Error>` | 从原始句柄创建 | ✅ 一致 |

### Symbol 结构体

| 方法 | 返回类型 | 说明 | OH 状态 |
|------|----------|------|---------|
| `into_raw(self)` | `*mut T` | 获取原始指针 | ✅ 一致 |
| `as_ref(&self)` | `Option<&T>` | 获取引用 | ✅ 一致 |
| `as_ref_mut(&mut self)` | `Option<&mut T>` | 获取可变引用 | ✅ 一致 |

### Error 类型

| 错误变体 | 说明 | OH 状态 |
|----------|------|---------|
| `InvalidParameter` | 无效参数 | ✅ 一致 |
| `LibraryLoadFailed` | 库加载失败 | ✅ 一致 |
| `LibraryAlreadyLoaded` | 库已加载 | ✅ 一致 |
| `SymbolLoadFailed` | 符号加载失败 | ✅ 一致 |
| `SymbolNotFound` | 符号不存在 | ✅ 一致 |
| `Os` | 操作系统错误 | ✅ 一致 |

## 5.3 平台特定 API

### Unix 平台

```rust
// src/unix.rs 中的公开 API（适用于 OpenHarmony）

pub struct Library {
    handle: *mut libc::c_void,
}

impl Library {
    /// 获取 dlopen 返回的原始句柄
    pub fn handle(&self) -> *mut libc::c_void {
        self.handle
    }
    
    /// 获取最后一次 dlerror 错误信息
    pub fn last_error(&self) -> Option<String> {
        unsafe {
            let error = libc::dlerror();
            if error.is_null() {
                None
            } else {
                Some(std::ffi::CStr::from_ptr(error).to_string_lossy().into_owned())
            }
        }
    }
}
```

| Unix API | 平台支持 | OH 状态 |
|----------|----------|---------|
| dlopen | ✅ | ✅ 一致 |
| dlsym | ✅ | ✅ 一致 |
| dlclose | ✅ | ✅ 一致 |
| dlerror | ✅ | ✅ 一致 |
| RTLD_NOW | ✅ | ✅ 一致 |
| RTLD_LAZY | ✅ | ✅ 一致 |
| RTLD_GLOBAL | ✅ | ✅ 一致 |
| RTLD_LOCAL | ✅ | ✅ 一致 |

### Windows 平台

| Windows API | OH 状态 |
|-------------|---------|
| LoadLibraryA/W | N/A（OH 基于 Linux） |
| GetProcAddress | N/A |
| FreeLibrary | N/A |

## 5.4 使用示例

### 基本用法（与上游完全相同）

```rust
use libloading::{Library, Symbol};

// 加载动态库
let library = Library::new("./module.so")?;

// 获取函数符号
type MyFunc = unsafe extern "C" fn(i32) -> i32;
let func: Symbol<MyFunc> = library.get(b"my_function")?;

// 调用函数
unsafe {
    let result = func(42);
}
```

### 错误处理（与上游完全相同）

```rust
use libloading::{Error, Library};

fn load_and_call() -> Result<i32, Error> {
    let lib = Library::new("./module.so")?;
    
    type ComputeFunc = unsafe extern "C" fn(i32, i32) -> i32;
    let compute: Symbol<ComputeFunc> = lib.get(b"compute")?;
    
    unsafe {
        Ok(compute(10, 20))
    }
}
```

## 5.5 API 兼容性保证

### 版本兼容性

| libloading 版本 | Rust 版本 | OH 兼容性 |
|-----------------|-----------|-----------|
| 0.7.4 | 1.40.0+ | ✅ 完全兼容 |
| 0.8.x | 1.56.0+ | ⚠️ 需升级 |
| 0.9.x | 1.56.0+ | ⚠️ 需升级 |

### 升级时的 API 变更

如上游发布新版本，可能的 API 变更：

| 变更类型 | 影响 | 升级注意事项 |
|----------|------|-------------|
| 新增方法 | 低 | 向后兼容 |
| 新增错误变体 | 低 | 向后兼容 |
| 方法签名变更 | 高 | 需要代码修改 |
| 移除方法 | 高 | 需要代码重写 |
| 语义变更 | 高 | 需要重新测试 |

## 5.6 扩展建议

### 如果需要 OH 特定功能

libloading 当前不提供以下 OH 特有功能，如有需要可考虑扩展：

| 潜在扩展 | 说明 | 实现复杂度 |
|----------|------|------------|
| OH 资源加载 | 加载 OH 资源文件格式 | 中 |
| 权限检查 | 动态库的权限验证 | 低 |
| 符号缓存 | OH 特有的符号缓存机制 | 中 |
| 远程加载 | 跨设备的动态库加载 | 高 |

### 扩展实现示例

```rust
// 如果需要添加 OH 资源加载功能，可以创建包装类型

pub struct OhResourceLibrary {
    library: Library,
    resource_path: String,
}

impl OhResourceLibrary {
    /// 从 OH 资源路径加载
    pub fn from_oh_resource(resource_id: &str) -> Result<Self, Error> {
        // OH 特有的资源解析逻辑
        let path = format!("/resources/{}", resource_id);
        let library = Library::new(&path)?;
        Ok(Self { library, resource_path: path })
    }
}
```

## 5.7 总结

| 维度 | 状态 | 说明 |
|------|------|------|
| API 一致性 | ✅ 100% | 与上游完全一致 |
| 新增 API | ❌ 无 | 未添加 OH 特有 API |
| 行为变更 | ❌ 无 | 无任何行为修改 |
| 功能禁用 | ❌ 无 | 所有功能可用 |
| 文档兼容性 | ✅ 完全兼容 | 可直接使用上游文档 |

**结论**：libloading 在 OpenHarmony 中提供了与上游完全一致的 API，可以直接参考上游文档和示例进行开发。
