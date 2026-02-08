# API 差异分析

## 5.1 概述

rustix 在 OpenHarmony 中的 API 与上游版本 **完全一致**。由于采用 Linux 兼容模式，没有对 rustix 源代码进行任何修改，因此不存在 API 差异。

### 差异概览

| 维度 | 差异状态 | 说明 |
|------|---------|------|
| **新增 API** | 无 | OH 未添加任何新 API |
| **行为变更** | 无 | 所有 API 行为与上游一致 |
| **废弃 API** | 无 | 未禁用任何上游 API |
| **条件 API** | 无 | 无仅在 OH 条件编译的 API |

## 5.2 无 API 差异的原因

### 架构设计

rustix 的双后端架构设计使得平台适配更加简单：

```
┌─────────────────────────────────────────┐
│            rustix 公共 API               │
├─────────────────────────────────────────┤
│         libc 后端 / linux_raw 后端        │
├─────────────────────────────────────────┤
│         Linux / Android / OH            │  ← 统一接口
├─────────────────────────────────────────┤
│              libc / syscalls            │
└─────────────────────────────────────────┘
```

### OH 适配策略

OpenHarmony 通过以下方式适配，而非修改 API：

1. **伪装为 Linux**：通过 `CARGO_CFG_TARGET_OS=linux` 环境变量
2. **使用 libc 后端**：通过 `features = ["libc"]` 配置
3. **透明兼容**：OH 的 POSIX 兼容层使得系统调用行为一致

## 5.3 功能模块启用状态

### 上游 vs OH 功能对比

| 功能模块 | 上游默认 | OH 启用 | 差异说明 |
|---------|---------|---------|----------|
| `std` | ✓ | ✓ | 一致 |
| `io-lifetimes` | ✓ | ✓ | 一致 |
| `libc` | ✗ | ✓ | OH 强制启用 |
| `use-libc-auxv` | ✓ | ✓ | 一致 |
| `termios` | ✗ | ✓ | OH 启用 |
| `fs` | ✗ | ✗ | OH 未启用 |
| `net` | ✗ | ✗ | OH 未启用 |
| `process` | ✗ | ✗ | OH 未启用 |
| `thread` | ✗ | ✗ | OH 未启用 |
| `mm` | ✗ | ✗ | OH 未启用 |
| `time` | ✗ | ✗ | OH 未启用 |
| `rand` | ✗ | ✗ | OH 未启用 |
| `io_uring` | ✗ | ✗ | OH 未启用 |

### "差异"说明

虽然上表显示了一些"差异"，但这些并非 API 差异，而是 **功能启用状态** 的差异：

- **libc**：上游默认使用 linux_raw 后端（如果支持），OH 强制使用 libc 后端
- **termios**：上游需要显式启用，OH 构建中启用了此功能
- **其他功能**：上游需要显式启用，OH 当前未启用这些功能

这些不影响 API 兼容性，只是功能子集的选择。

## 5.4 条件编译分析

### 代码中的条件编译

rustix 源代码中使用条件编译来适配不同平台：

```rust
#[cfg(target_os = "linux")]
use rustix::fs::statfs;

#[cfg(target_os = "android")]
use rustix::fs::statfs;  // Android 也支持
```

### OH 中的条件编译

```rust
// 由于设置了 CARGO_CFG_TARGET_OS=linux
// 以下条件编译将匹配 Linux 分支

#[cfg(target_os = "linux")]
fn platform_specific() {
    // Linux 实现路径
}
```

### 关键发现

| 条件编译 | 在 OH 中的行为 |
|---------|---------------|
| `target_os = "linux"` | ✓ 匹配 |
| `target_os = "android"` | ✗ 不匹配 |
| `target_os = "ohos"` | ✗ 不存在 |
| `target_os = "openharmony"` | ✗ 不存在 |
| `unix` | ✓ 匹配（OH 是 Unix-like） |

## 5.5 使用注意事项

### 平台感知代码

如果代码中包含平台检测逻辑：

```rust
// 需要注意：OH 会被识别为 Linux
if cfg!(target_os = "linux") {
    // 会在 OH 上执行
}
```

### OpenHarmony 特定检测

如需检测 OpenHarmony 平台，需要使用其他方式：

```rust
// 注意：rustix 不支持 ohos 检测
// 需要通过其他机制（如环境变量）
#[cfg(feature = "ohos")]
fn ohos_specific() {
    // OH 特定代码
}
```

### 建议

1. **避免平台假设**：尽量使用 POSIX 通用接口
2. **延迟检测**：在运行时检测功能可用性，而非编译时
3. **功能检测**：使用 `std::os::unix::fs::PermissionsExt` 等特性检测

## 5.6 未来可能的 API 扩展

### 可考虑启用的功能

| 功能 | 建议优先级 | 说明 |
|------|----------|------|
| `fs` | 高 | 文件系统是常用功能 |
| `net` | 中 | 网络编程需求 |
| `process` | 中 | 进程管理需求 |
| `thread` | 低 | 大多数场景不需要 |
| `rand` | 低 | 随机数需求较少 |

### 添加 OH 特有 API

如果未来需要 OpenHarmony 特有的系统调用支持：

1. **上游优先**：尝试将 OH 特有 API 提交到上游
2. **条件编译**：使用 `#[cfg(target_os = "ohos")]`
3. **Feature 隔离**：通过独立的 feature 开关控制

---

## 5.7 总结

rustix 在 OpenHarmony 中 **不存在 API 差异**。所有上游 API 均可正常使用，OH 的适配完全在构建层面完成，不影响运行时 API 行为。
