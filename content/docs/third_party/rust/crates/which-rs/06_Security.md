# 安全风险分析

## 概述

which-rs 是一个功能单一的基础工具库，主要用于在系统 PATH 中查找可执行文件。本章节分析该库的安全风险、已知 CVE 以及使用建议。

## 已知 CVE

### 当前版本状态

| 版本 | CVE 数量 | 状态 |
|------|----------|------|
| 4.4.0 | 0 | ✅ 无已知安全漏洞 |

截至本文档编写时，**which-rs 4.4.0 版本无任何公开的 CVE 记录**。

### 历史安全记录

通过查询 RustSec 和 CVE 数据库：
- crates.io: which 包无安全公告
- RustSec Advisory Database: 无 which-rs 相关条目
- CVE Details: 无 CVE 记录

## 安全风险分析

### 风险类别 1: 命令注入 (低风险)

**风险描述**: 使用 which-rs 查找的可执行文件路径直接用于命令执行时，如果路径未正确处理，可能存在命令注入风险。

**代码示例（风险）**:
```rust
use which::which;
use std::process::Command;

// 风险：未验证用户输入
fn run_user_command(user_input: &str) {
    if let Ok(path) = which(user_input) {
        // 如果 user_input 包含特殊字符，可能有问题
        let _ = Command::new(path).output();
    }
}
```

**缓解措施**:
```rust
use which::which;
use std::process::Command;
use std::ffi::OsStr;

// 安全：使用 OsStr 处理，避免字符串解析
fn run_user_command_safe(user_input: &OsStr) {
    if let Ok(path) = which(user_input) {
        let _ = Command::new(path).output();
    }
}
```

**评估**: 该风险主要来自调用方的不当使用，而非 which-rs 本身。which-rs 正确处理了路径作为 OsStr/OsString，避免了字符串层面的注入。

### 风险类别 2: TOCTOU (Time-of-check to time-of-use) (中等风险)

**风险描述**: which-rs 查找文件和调用方实际使用文件之间存在时间窗口，文件可能被替换或修改。

**时序图**:
```
时间线:
T1: which-rs 检查文件存在性和可执行性
    ↓ [时间窗口]
T2: 攻击者替换/修改文件
    ↓
T3: 调用方使用文件路径
```

**代码示例（风险）**:
```rust
use which::which;
use std::process::Command;

fn run_tool(name: &str) {
    // T1: which 检查 /usr/bin/tool 存在且可执行
    let path = which(name).unwrap();
    
    // [攻击窗口] 攻击者可能替换 /usr/bin/tool
    
    // T2: 执行时可能运行了被替换的程序
    let _ = Command::new(path).output();
}
```

**缓解措施**:

1. **使用 CanonicalPath 获取规范化路径**:
```rust
use which::CanonicalPath;

fn run_tool_safe(name: &str) {
    // 获取规范路径（解析所有符号链接）
    let canonical = CanonicalPath::new(name).unwrap();
    let _ = Command::new(canonical).output();
}
```

2. **执行前再次验证**:
```rust
use std::fs;
use std::os::unix::fs::MetadataExt;

fn run_with_verification(path: &Path) {
    // 获取查找时的 inode
    let meta1 = fs::metadata(path).unwrap();
    let inode1 = meta1.ino();
    
    // 执行前再次验证
    let meta2 = fs::metadata(path).unwrap();
    if meta1.ino() != meta2.ino() || meta1.mtime() != meta2.mtime() {
        panic!("File changed between check and use!");
    }
    
    let _ = Command::new(path).output();
}
```

3. **最小权限原则**: 确保可执行文件所在目录的写权限受控

### 风险类别 3: PATH 环境变量篡改 (中等风险)

**风险描述**: which-rs 依赖 PATH 环境变量，恶意程序可能篡改 PATH 导致查找到错误的可执行文件。

**攻击场景**:
```bash
# 攻击者设置恶意 PATH
export PATH=/tmp/malicious:$PATH
# /tmp/malicious/ls 是恶意程序
```

**代码风险**:
```rust
use which::which;

fn run_ls() {
    // 可能查找到 /tmp/malicious/ls
    let path = which("ls").unwrap();
    // ...
}
```

**缓解措施**:

1. **使用自定义 PATH**:
```rust
use which::which_in;

fn run_ls_safe() {
    // 使用已知安全的 PATH
    let safe_path = "/usr/bin:/bin";
    let path = which_in("ls", Some(safe_path), ".").unwrap();
    // ...
}
```

2. **验证查找到的路径**:
```rust
use which::which;
use std::path::Path;

fn run_ls_verified() {
    let path = which("ls").unwrap();
    
    // 验证路径在预期目录
    if !path.starts_with("/usr/bin") && !path.starts_with("/bin") {
        panic!("Unexpected ls location: {:?}", path);
    }
    
    // ...
}
```

3. **使用绝对路径（如已知）**:
```rust
use std::path::PathBuf;
use std::process::Command;

fn run_ls_absolute() {
    // 直接使用已知安全的绝对路径
    let _ = Command::new("/usr/bin/ls").output();
}
```

### 风险类别 4: 符号链接攻击 (中等风险)

**风险描述**: 攻击者可能创建指向特权文件的符号链接，诱使程序执行非预期操作。

**攻击场景**:
```
/tmp/malicious/
└── tool -> /etc/shadow  # 符号链接指向敏感文件
```

**缓解措施**:

1. **使用 CanonicalPath**:
```rust
use which::CanonicalPath;

fn run_tool_canonical(name: &str) {
    // CanonicalPath 解析所有符号链接
    let path = CanonicalPath::new(name).unwrap();
    println!("Actual path: {:?}", path);
    // 可以看到真实的 /etc/shadow 路径
}
```

2. **检查文件类型**:
```rust
use std::fs;
use std::path::Path;

fn is_safe_path(path: &Path) -> bool {
    let metadata = fs::symlink_metadata(path).unwrap();
    
    // 拒绝符号链接
    if metadata.file_type().is_symlink() {
        return false;
    }
    
    true
}
```

## 安全使用建议

### 推荐实践

#### 1. 敏感场景使用 CanonicalPath

```rust
use which::CanonicalPath;

// 安全：获取规范化路径，解析所有符号链接
let path = CanonicalPath::new("tool").expect("Tool not found");
```

#### 2. 验证路径白名单

```rust
use which::which;
use std::path::Path;

const ALLOWED_PREFIXES: &[&str] = &[
    "/usr/bin",
    "/bin",
    "/usr/local/bin",
];

fn which_verified(name: &str) -> Option<PathBuf> {
    let path = which(name).ok()?;
    
    let allowed = ALLOWED_PREFIXES.iter()
        .any(|prefix| path.starts_with(prefix));
    
    if allowed {
        Some(path)
    } else {
        None
    }
}
```

#### 3. 使用安全的自定义 PATH

```rust
use which::which_in;

// 定义安全的 PATH 环境
const SAFE_PATH: &str = "/usr/bin:/bin";

fn find_tool(name: &str) -> Option<PathBuf> {
    which_in(name, Some(SAFE_PATH), ".").ok()
}
```

#### 4. 执行前再次检查

```rust
use std::fs;
use std::os::unix::fs::PermissionsExt;

fn safe_execute(path: &Path) -> std::io::Result<Output> {
    // 再次验证文件存在且可执行
    let metadata = fs::metadata(path)?;
    if !metadata.is_file() {
        return Err(std::io::Error::new(
            std::io::ErrorKind::InvalidInput,
            "Not a regular file"
        ));
    }
    
    let permissions = metadata.permissions();
    if permissions.mode() & 0o111 == 0 {
        return Err(std::io::Error::new(
            std::io::ErrorKind::PermissionDenied,
            "File not executable"
        ));
    }
    
    Command::new(path).output()
}
```

### 避免的做法

#### ❌ 不要直接使用用户输入作为程序名

```rust
// 危险！
let user_input = std::env::args().nth(1).unwrap();
let path = which(user_input).unwrap();  // 可能被注入
```

#### ❌ 不要在查找和使用之间执行耗时操作

```rust
let path = which("tool").unwrap();
// 长时间操作，增大 TOCTOU 窗口
std::thread::sleep(Duration::from_secs(10));
Command::new(path).output();  // 文件可能已被替换
```

#### ❌ 不要忽略查找失败

```rust
// 危险：unwrap 可能导致 panic
let path = which("tool").unwrap();

// 更好：显式处理错误
match which("tool") {
    Ok(path) => { /* ... */ },
    Err(e) => {
        eprintln!("Tool not found: {}", e);
        return;
    }
}
```

## 安全升级策略

### 版本升级检查清单

升级 which-rs 版本时，应检查：

- [ ] 新版本是否有安全公告
- [ ] API 变更是否影响现有安全假设
- [ ] 依赖树是否有新增的安全风险

### 监控渠道

| 渠道 | URL | 用途 |
|------|-----|------|
| RustSec | https://rustsec.org/ | Rust 安全公告 |
| crates.io | https://crates.io/crates/which | 包信息 |
| GitHub Security | https://github.com/harryfei/which-rs/security | 上游安全公告 |

## 总结

| 风险类型 | 等级 | 说明 |
|----------|------|------|
| 命令注入 | 低 | 库本身设计安全，主要依赖调用方正确使用 |
| TOCTOU | 中 | 查找和使用之间存在时间窗口 |
| PATH 篡改 | 中 | 依赖环境变量，可能被恶意修改 |
| 符号链接 | 中 | 可能指向非预期目标 |

**整体评估**: which-rs 是一个**安全性较高**的基础库。主要风险来自调用方的使用方式，而非库本身。遵循本文档的安全建议，可以有效降低风险。

**关键建议**:
1. 敏感场景使用 `CanonicalPath`
2. 验证查找结果路径在白名单内
3. 执行前再次验证文件状态
4. 控制 PATH 环境变量
