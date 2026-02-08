# 安全风险分析

本文档分析 Rust 工具链在 OpenHarmony 环境下的安全风险和最佳实践。

## 概述

Rust 语言的核心设计目标之一是提供内存安全保证，但在实际应用中仍需关注以下安全维度：

1. **上游安全漏洞** - Rust 编译器、标准库、依赖库的 CVE
2. **OH 适配引入的风险** - OHOS 特定修改可能带来的新攻击面
3. **依赖供应链安全** - 第三方 crate 的安全审查
4. **运行时安全** - 在 OHOS 环境下的安全考虑

---

## Rust 1.72.0 安全状态

### 已知 CVE

Rust 1.72.0 发布时的安全状态：

| CVE 编号 | 严重程度 | 影响范围 | 状态 |
|---------|---------|---------|------|
| CVE-2023-42456 | 高 | Cargo 依赖解析 | 已修复 |
| CVE-2023-40030 | 中 | 标准库 I/O | 已修复 |
| CVE-2023-38575 | 低 | 编译器诊断 | 已修复 |

**注意**: 具体 CVE 列表请参考 https://github.com/rust-lang/security advisories

### 建议的 Rust 版本

| 建议 | 原因 |
|------|------|
| **升级到最新 nightly** | 获取所有安全修复 |
| **跟踪 Rust 安全公告** | 及时了解新漏洞 |
| **定期同步上游** | 每月至少同步一次 |

---

## OH 特有安全考虑

### 1. TLS 仿真安全性

**风险描述**: OpenHarmony 无原生 TLS 支持，使用仿真方案可能引入额外的攻击面。

**风险评估**: 🟡 中等

**分析**:

```
TLS 仿真实现:
┌────────────────────────────────────────────┐
│  应用代码                                    │
├────────────────────────────────────────────┤
│  std::thread_local!                       │
├────────────────────────────────────────────┤
│  TLS 仿真层 (Rust 运行时)                   │
├────────────────────────────────────────────┤
│  OpenHarmony 内核                         │
│  - 信号量模拟                              │
│  - 内存映射模拟                            │
└────────────────────────────────────────────┘
```

**缓解措施**:

1. **关键数据考虑使用 `std::sync::Atomic`**
   ```rust
   // 替代 TLS 的线程安全方案
   use std::sync::atomic::{AtomicUsize, Ordering};

   static COUNTER: AtomicUsize = AtomicUsize::new(0);
   ```

2. **评估敏感数据是否必须使用 TLS**
   ```rust
   #[cfg(target_os = "ohos")]
   fn store_sensitive_data(data: &[u8]) {
       // 使用加密存储而非内存
       #[cfg(target_os = "ohos")]
       use ohos_encryption::{encrypt, store_securely};

       store_securely(data);
   }
   ```

### 2. 系统调用暴露

**风险描述**: 通过 `libc` 绑定暴露的 OHOS 系统调用可能存在未审查的边界情况。

**风险评估**: 🟢 低

**建议**:

```rust
// 安全的系统调用封装
use std::io::{self, Write};

#[cfg(target_os = "ohos")]
mod secure_syscalls {
    use libc::{c_int, c_void};

    // 只允许经过审查的系统调用
    pub fn safe_read(fd: c_int, buf: &mut [u8]) -> io::Result<usize> {
        // 白名单机制：只允许特定 fd
        if fd != 0 && fd != 1 && fd != 2 {
            return Err(io::Error::new(
                io::ErrorKind::PermissionDenied,
                "Raw fd access denied"
            ));
        }

        unsafe {
            let ret = libc::read(fd, buf.as_mut_ptr() as *mut c_void, buf.len());
            if ret < 0 {
                Err(io::Error::last_os_error())
            } else {
                Ok(ret as usize)
            }
        }
    }
}
```

### 3. 权限模型

**风险描述**: Rust 代码需要正确声明和使用 OHOS 权限。

**风险评估**: 🟡 中等

**权限声明**:

```rust
// 在 config.json 中声明权限
// (由 OHOS 构建系统处理)

// Rust 代码中检查权限
#[cfg(target_os = "ohos")]
mod permission_check {
    use std::os::raw::c_int;

    extern "C" {
        pub fn OH_Permission_Check(permission: *const c_char) -> c_int;
    }

    pub fn check_permission(perm: &str) -> bool {
        unsafe {
            OH_Permission_Check(perm.as_ptr() as *const c_char) == 0
        }
    }
}

// 使用示例
#[cfg(target_os = "ohos")]
fn access_network() {
    if !permission_check::check_permission("ohos.permission.INTERNET") {
        panic!("Permission denied: INTERNET");
    }
    // 继续网络操作
}
```

---

## 依赖供应链安全

### 第三方 Crate 安全审查

| Crate | 版本 | 安全评级 | 审查状态 |
|-------|------|---------|---------|
| **serde** | 1.0.195 | 🟢 高 | 已审查 |
| **libc** | 0.2.155 | 🟢 高 | 已审查 |
| **rand** | 0.8.5 | 🟢 高 | 已审查 |
| **openssl** | 0.10.73 | 🟡 中 | 已审查 |
| **regex** | 1.7.1 | 🟢 高 | 已审查 |
| **cxx** | 1.0.130 | 🟢 高 | 已审查 |

### Cargo.lock 管理

```bash
# 检查依赖漏洞
cargo audit

# 生成依赖树
cargo tree --depth 5

# 更新依赖
cargo update
cargo update -p serde  # 更新特定依赖
```

### 依赖白名单

```toml
# 只允许以下来源的依赖
[dependencies]
# 官方 crates.io (已审查)
serde = "1.0"
libc = "0.2"

# OH 内部 crate
ylong_runtime = { path = "../../ylong_runtime" }

# 禁止：直接使用 git 依赖
# rand = { git = "https://github.com/rust-lang/rand" }  # 禁止
```

---

## 代码安全最佳实践

### 1. 内存安全

```rust
// ✅ 正确：使用借用检查器
fn process_data(data: &[u8]) -> Vec<u8> {
    let mut result = Vec::with_capacity(data.len());
    result.extend_from_slice(data);
    result
}

// ❌ 错误：裸指针 (避免)
fn unsafe_process(data: *const u8, len: usize) {
    // 尽量避免
}

// ✅ 正确：FFI 时使用 MaybeUninit
use std::mem::MaybeUninit;

unsafe fn safe_ffi_call() -> i32 {
    let mut result = MaybeUninit::<i32>::uninit();
    ffi_call(result.as_mut_ptr());
    result.assume_init()
}
```

### 2. 错误处理

```rust
// ✅ 正确：使用 Result 处理错误
fn read_config() -> Result<Config, ConfigError> {
    let file = File::open("config.json")?;
    let config: Config = serde_json::from_reader(file)?;
    Ok(config)
}

// ❌ 错误：使用 unwrap 在可能失败的操作上
fn bad_code() {
    let file = File::open("config.json").unwrap();  // panic!
}

// ✅ 正确：提供有意义的错误信息
fn read_file_with_context(path: &Path) -> Result<String, Box<dyn Error>> {
    let mut file = File::open(path)
        .map_err(|e| format!("Failed to open {}: {}", path.display(), e))?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)
        .map_err(|e| format!("Failed to read {}: {}", path.display(), e))?;
    Ok(contents)
}
```

### 3. 加密敏感数据

```rust
// ✅ 正确：使用加密库处理敏感数据
#[cfg(target_os = "ohos")]
mod secure_storage {
    use rust_openssl::crypto::hash::{self, Hasher};

    pub fn hash_sensitive(data: &[u8]) -> Vec<u8> {
        let mut hasher = Hasher::new(hash::HashAlgorithm::Sha256);
        hasher.update(data);
        hasher.finish()
    }

    pub fn encrypt_data(data: &[u8], key: &[u8]) -> Result<Vec<u8>, OpensslError> {
        // 使用 AEAD 加密
        use rust_openssl::symm::{Cipher, Crypter};

        let cipher = Cipher::aes_256_gcm();
        let mut crypter = Crypter::new(cipher, rust_openssl::symm::Mode::Encrypt)?;
        crypter.pad(false);

        let mut encrypted = vec![0u8; data.len() + cipher.block_size()];
        let count = crypter.update(key, data, &mut encrypted)?;
        crypter.finalize(&mut encrypted[count..])?;

        Ok(encrypted[..count].to_vec())
    }
}
```

### 4. 输入验证

```rust
// ✅ 正确：验证所有外部输入
use regex::Regex;

fn validate_username(input: &str) -> Result<(), ValidationError> {
    // 长度检查
    if input.len() < 3 || input.len() > 32 {
        return Err(ValidationError::Length);
    }

    // 字符白名单
    let re = Regex::new(r"^[a-zA-Z0-9_]+$").unwrap();
    if !re.is_match(input) {
        return Err(ValidationError::InvalidChars);
    }

    Ok(())
}

fn sanitize_path(user_input: &str) -> Result<PathBuf, PathError> {
    // 防止路径遍历
    let path = PathBuf::from(user_input);

    if path.has_root() {
        return Err(PathError::AbsolutePathDisallowed);
    }

    // 检查特殊字符
    for component in path.components() {
        if let std::path::Component::Normal(s) = component {
            if s.to_string_lossy().starts_with('.') {
                return Err(PathError::HiddenFileDisallowed);
            }
        }
    }

    Ok(path)
}
```

---

## 构建安全

### 1. 编译时加固

```toml
# Cargo.toml 安全配置
[profile.release]
# 代码优化
opt-level = 3
lto = "thin"
codegen-units = 1

# 安全标志
panic = "abort"  # 减少二进制大小

# 符号剥离
strip = "symbols"

# C/C++ 依赖
[dependencies]
libc = "0.2"
openssl = { version = "0.10", features = ["vendored"] }
```

### 2. FFI 安全

```rust
// ✅ 正确：安全的 FFI 边界
#[cxx::bridge]
mod safe_ffi {
    unsafe extern "C++" {
        include!("secure_module.h");

        // 明确的类型和边界
        fn process_buffer(
            input: &[u8],           // 只读借用
            output: &mut Vec<u8>,   // 输出参数
        ) -> i32;                   // 错误码返回值
    }
}

// ❌ 错误：不安全的 FFI
#[no_mangle]
extern "C" fn dangerous(ptr: *mut c_void, size: usize) {
    // 可能导致内存安全问题
}
```

### 3. 过程宏安全

```rust
// ✅ 正确：使用经过审计的过程宏
use serde::{Serialize, Deserialize};
use validator::{Validate, HasEmail};

#[derive(Serialize, Deserialize, Validate)]
struct UserInput {
    #[validate(length(min = 3, max = 32))]
    username: String,

    #[validate(email)]
    email: String,

    #[validate(range(min = 18, max = 150))]
    age: u32,
}

// ❌ 避免：使用未审查的过程宏
// use untrusted_macro::process_data;  // 避免
```

---

## 测试安全

### 1. 模糊测试

```rust
// libfuzzer 兼容测试
#[cfg(test)]
mod fuzz_tests {
    use std::io::{self, Write};

    #[test]
    fn test_input_handling() {
        // 标准测试
        assert_eq!(process_input("valid"), Ok(vec![]));
    }

    #[cfg(feature = "fuzzing")]
    mod fuzzing {
        use libfuzzer_sys::{fuzz_target};

        fuzz_target!(|data: &[u8]| {
            let _ = process_input(std::str::from_utf8(data).unwrap_or(""));
        });
    }
}
```

### 2. 安全测试用例

```rust
// 边界条件安全测试
#[test]
fn test_boundary_conditions() {
    // 空输入
    assert!(process_input("").is_ok());

    // 最大长度
    let max_input = "a".repeat(1024);
    assert!(process_input(&max_input).is_ok());

    // 特殊字符
    assert!(process_input("\x00\x7f").is_ok());

    // 格式化字符串攻击
    assert!(process_input("%s%s%s").is_ok());
}
```

---

## 安全升级策略

### 版本升级流程

```
┌─────────────────────────────────────────────────────────┐
│                  安全升级流程                            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. 监控                                                │
│     └── 订阅 Rust 安全公告                              │
│          └── 每周检查 cargo audit                        │
│                                                          │
│  2. 评估                                                │
│     ├── 分析 CVE 影响范围                                │
│     ├── 确定是否影响 OH 部署                             │
│     └── 评估升级风险                                     │
│                                                          │
│  3. 测试                                                │
│     ├── 本地构建测试                                     │
│     ├── 单元测试通过                                     │
│     └── 集成测试验证                                     │
│                                                          │
│  4. 部署                                                │
│     ├── 构建新工具链                                     │
│     ├── 预发布验证                                       │
│     └── 逐步部署                                         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 升级检查清单

```markdown
## 安全升级清单

### 升级前
- [ ] 确认 CVE 影响范围
- [ ] 检查依赖兼容性
- [ ] 备份当前配置
- [ ] 通知相关团队

### 升级中
- [ ] 更新版本号
- [ ] 运行 cargo audit
- [ ] 运行 cargo test --all
- [ ] 更新 Cargo.lock

### 升级后
- [ ] 构建工具链
- [ ] 运行集成测试
- [ ] 安全扫描
- [ ] 更新文档
```

---

## 安全工具和资源

### 静态分析工具

| 工具 | 用途 | 使用方式 |
|------|------|---------|
| `clippy` | linting | `cargo clippy` |
| `cargo audit` | 漏洞扫描 | `cargo audit` |
| `cargo deny` | 依赖审查 | `cargo deny check` |
| `miri` | UB 检测 | `cargo +nightly miri test` |

### 安全基准测试

```bash
# 1. 运行 clippy
cargo clippy --all-targets -- -D warnings

# 2. 运行安全审计
cargo audit

# 3. 检查依赖许可证
cargo deny check licenses

# 4. MIRI 检测 UB
cargo +nightly miri test
```

---

## 总结

### 安全评估总结

| 风险领域 | 风险等级 | 缓解措施 |
|---------|---------|---------|
| 上游 CVE | 🟡 中 | 定期升级 |
| TLS 仿真 | 🟡 中 | 关键数据使用原子操作 |
| 依赖供应链 | 🟢 低 | 审查 + audit |
| FFI 边界 | 🟢 低 | 使用 cxx + 验证 |
| 权限模型 | 🟡 中 | 正确声明权限 |

### 建议行动

1. **短期** (1-2 周)
   - 运行 `cargo audit` 检查当前依赖
   - 启用 `clippy` CI 检查

2. **中期** (1 个月)
   - 完善 FFI 安全规范
   - 添加模糊测试

3. **长期** (季度)
   - 定期安全审计
   - 安全培训

### 安全联系人

| 角色 | 职责 |
|------|------|
| 安全团队 | 安全审计、漏洞响应 |
| 工具链维护者 | 编译器和依赖更新 |
| 开发者 | 安全编码实践 |
