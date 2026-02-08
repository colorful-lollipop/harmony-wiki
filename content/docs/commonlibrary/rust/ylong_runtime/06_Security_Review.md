# 安全风险评审

## 评审范围

本评审覆盖 `ylong_runtime` 项目源码（不含测试），基于以下代码：

| 模块 | 源码位置 | 评审状态 |
|------|----------|----------|
| ylong_runtime | `ylong_runtime/src/` | ✅ 已评审 |
| ylong_io | `ylong_io/src/` | ✅ 已评审 |
| ylong_ffrt | `ylong_ffrt/src/` | ✅ 已评审 |
| ylong_signal | `ylong_signal/src/` | ✅ 已评审 |
| ylong_runtime_macros | `ylong_runtime_macros/src/` | ✅ 已评审 |

**评审时间**: 2026-02-06

## 攻击面分析

### 输入源分类

| 输入类型 | 来源 | 处理模块 |
|----------|------|----------|
| 网络数据 | TCP/UDP socket | `ylong_io/src/sys/unix/tcp/stream.rs` |
| 文件系统 | 文件读写 | `ylong_runtime/src/fs/` |
| 进程参数 | Command::arg() | `ylong_runtime/src/process/command.rs` |
| 信号 | Unix signals | `ylong_runtime/src/signal/` |
| 用户配置 | feature flags | `BUILD.gn` |

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                     不可信区域                               │
│  - 网络输入 (TCP/UDP 数据)                                   │
│  - 用户提供的文件路径                                        │
│  - 命令行参数                                               │
│  - 环境变量                                                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                     信任边界 (ylong_runtime)                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           输入验证 & 过滤                             │   │
│  │  - 路径规范化                                         │   │
│  │  - 参数校验                                          │   │
│  │  - 类型检查                                          │   │
│  └─────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                     敏感操作                                  │
│  - 文件系统操作 (读写/删除)                                  │
│  - 进程创建                                                 │
│  - socket 连接                                              │
│  - 用户/组 ID 设置                                          │
└─────────────────────────────────────────────────────────────┘
```

## 已识别风险

### 🔴 高风险

#### 1. 路径遍历漏洞

**证据**: `ylong_runtime/src/fs/open_options.rs`

**描述**: 文件路径未做规范化处理，可能导致路径遍历攻击。

**代码位置**: `ylong_runtime/src/fs/open_options.rs`

**触发条件**:
```rust
// 用户可能传入恶意路径
File::open("../../../etc/passwd").await
```

**影响**:
- 越权读取敏感文件
- 越权修改系统文件

**修复建议**:
```rust
// 应该使用 std::fs::canonicalize 或 Path::canonicalize 进行路径规范化
let canonical_path = path.as_ref().canonicalize()?;
if !canonical_path.starts_with(allowed_dir) {
    return Err(io::Error::new(io::ErrorKind::PermissionDenied, "path outside allowed directory"));
}
```

**当前状态**: ⚠️ 未修复

---

#### 2. 命令注入风险

**证据**: `ylong_runtime/src/process/command.rs:414`

**描述**: `Command::arg()` 未对参数进行转义，可能导致命令注入。

**代码位置**: `ylong_runtime/src/process/command.rs:414`

**触发条件**:
```rust
let output = Command::new("cat")
    .arg(file_name)  // file_name = "; rm -rf /"
    .output()
    .await;
```

**影响**:
- 执行任意命令
- 系统权限泄露

**修复建议**:
```rust
// 应该验证参数不包含特殊字符
fn validate_arg(arg: &str) -> Result<(), io::Error> {
    if arg.bytes().any(|b| b == b'\0' || b == b'\n' || b == b';') {
        return Err(io::Error::new(io::ErrorKind::InvalidInput, "invalid argument"));
    }
    Ok(())
}
```

**当前状态**: ⚠️ 未修复

---

### 🟡 中风险

#### 3. 进程 uid/gid 设置权限问题

**证据**: `ylong_runtime/src/process/command.rs:414`, `ylong_runtime/src/process/command.rs:525`

**描述**: `uid()` 和 `gid()` 方法缺少权限检查，普通用户可能设置任意用户 ID。

**代码位置**: 
- `ylong_runtime/src/process/command.rs:414` → `pub fn uid(&mut self, id: u32)`
- `ylong_runtime/src/process/command.rs:525` → `pub fn uid(&mut self, id: u32)`

**触发条件**:
```rust
// 非特权用户尝试设置 root uid
Command::new("some_program")
    .uid(0)  // 设置 root 权限
    .output()
    .await;
```

**影响**:
- 权限提升
- 绕过安全检查

**修复建议**:
```rust
// 应该检查调用者是否有权限设置指定 uid
fn check_uid_permission(uid: u32) -> Result<(), io::Error> {
    let current_uid = std::os::unix::fs::MetadataExt::uid(std::fs::metadata(".")?);
    // 非 root 用户不能设置其他用户的 uid
    if current_uid != 0 && uid != current_uid {
        return Err(io::Error::new(io::ErrorKind::PermissionDenied, "permission denied"));
    }
    Ok(())
}
```

**当前状态**: ⚠️ 未修复

---

#### 4. Socket 所有权设置无验证

**证据**: `ylong_io/src/sys/unix/tcp/stream.rs:42`

**描述**: `connect_with_owner()` 未验证调用者是否有权限设置指定 uid/gid。

**代码位置**: `ylong_io/src/sys/unix/tcp/stream.rs:42`

**触发条件**:
```rust
TcpStream::connect_with_owner("127.0.0.1:8080", 0, 0).await;
```

**影响**:
- 权限伪装
- 网络权限绕过

**修复建议**:
```rust
// 应该验证 uid/gid 有效性
if uid != 0 && uid != getuid() {
    return Err(io::Error::new(io::ErrorKind::PermissionDenied, "cannot set other user's socket owner"));
}
```

**当前状态**: ⚠️ 未修复

---

### 🟢 低风险

#### 5. 定时器资源耗尽 (DoS)

**证据**: `ylong_runtime/src/time/wheel.rs`

**描述**: 无限创建定时器可能导致内存耗尽。

**代码位置**: `ylong_runtime/src/time/wheel.rs`

**触发条件**:
```rust
loop {
    ylong_runtime::time::sleep(Duration::from_millis(1)).await;
}
```

**影响**:
- 内存耗尽
- 服务拒绝

**缓解因素**:
- Linux 系统通常有进程内存限制
- 建议在应用层实现速率限制

**修复建议**: 应用层应实现定时器速率限制

---

#### 6. 任务队列阻塞 (DoS)

**证据**: `ylong_runtime/src/task/mod.rs`

**描述**: 无限创建任务可能导致队列阻塞。

**触发条件**:
```rust
loop {
    ylong_runtime::spawn(async { /* 空任务 */ });
}
```

**缓解因素**:
- 运行时通常有任务队列上限
- 建议在应用层实现背压

---

#### 7. 信号处理竞争条件

**证据**: `ylong_runtime/src/signal/unix/driver.rs`

**描述**: 信号处理可能在任意时刻中断，存在竞态条件。

**代码位置**: `ylong_runtime/src/signal/unix/driver.rs`

**影响**:
- 状态不一致
- 资源泄露

**修复建议**: 信号处理应尽量简单，避免在信号处理程序中执行复杂操作

---

## 安全最佳实践建议

### 开发者指南

#### 1. 文件操作

```rust
// ✅ 正确做法：路径白名单
async fn read_safe_file(base_dir: &Path, filename: &str) -> io::Result<Vec<u8>> {
    let canonical_base = base_dir.canonicalize()?;
    let file_path = base_dir.join(filename);
    let canonical_file = file_path.canonicalize()?;
    
    if !canonical_file.starts_with(&canonical_base) {
        return Err(io::Error::new(io::ErrorKind::PermissionDenied, "path outside allowed directory"));
    }
    
    File::open(&file_path).await?.read_to_end().await
}

// ❌ 错误做法：直接使用用户输入
File::open(user_input).await  // 可能导致路径遍历
```

#### 2. 进程创建

```rust
// ✅ 正确做法：参数白名单
let allowed_commands = ["cat", "ls", "grep"];
if !allowed_commands.contains(&cmd.get_program()) {
    return Err(io::Error::new(io::ErrorKind::InvalidInput, "command not allowed"));
}

// ✅ 正确做法：参数验证
for arg in cmd.get_args() {
    if arg.contains(|c: char| c.is_control() || c == ';' || c == '|') {
        return Err(io::Error::new(io::ErrorKind::InvalidInput, "invalid argument"));
    }
}
```

#### 3. 网络操作

```rust
// ✅ 正确做法：地址白名单
let allowed_hosts = ["127.0.0.1", "10.0.0.1"];
let addr: SocketAddr = parse_addr(target)?;
if !allowed_hosts.contains(&addr.ip().to_string()) {
    return Err(io::Error::new(io::ErrorKind::PermissionDenied, "host not allowed"));
}
```

---

## 风险汇总表

| ID | 风险类型 | 严重程度 | 状态 | 建议修复 |
|----|----------|----------|------|----------|
| 1 | 路径遍历 | 🔴 高 | 未修复 | 路径规范化 + 白名单 |
| 2 | 命令注入 | 🔴 高 | 未修复 | 参数验证/转义 |
| 3 | uid 设置无检查 | 🟡 中 | 未修复 | 权限检查 |
| 4 | Socket 所有者无验证 | 🟡 中 | 未修复 | uid/gid 验证 |
| 5 | 定时器 DoS | 🟢 低 | 缓解 | 应用层限流 |
| 6 | 任务队列 DoS | 🟢 低 | 缓解 | 应用层背压 |
| 7 | 信号竞态 | 🟢 低 | 已知 | 简化处理 |

## 结论

`ylong_runtime` 项目整体安全性**需改进**。主要风险集中在：

1. **输入验证不足** - 路径、命令参数缺少严格校验
2. **权限检查缺失** - uid/gid 设置缺少权限验证

建议优先修复 **路径遍历** 和 **命令注入** 漏洞。
