# 安全风险分析

## 概述

rustc-hash 是一个轻量级的非加密哈希库。本节分析该库在 OpenHarmony 中的安全状态和潜在风险。

## 已知漏洞状态

### CVE 记录

| 漏洞编号 | 状态 | 说明 |
|----------|------|------|
| 无已知 CVE | ✅ 无 | 该库未发现公开的 CVE 漏洞记录 |

### 无漏洞的原因

1. **代码简洁**：库代码仅 149 行，逻辑简单，审计难度低
2. **无复杂逻辑**：仅实现标准的哈希算法，无网络交互或复杂状态管理
3. **使用场景受限**：该库专门设计用于无需考虑安全攻击的场景

## OH Patch 风险评估

### Patch 引入的风险

| 风险类型 | 状态 | 说明 |
|----------|------|------|
| 新增攻击面 | ❌ 无 | 该库未应用任何 OH 定制 Patch |
| 代码注入 | ❌ 无 | 无 Patch，无代码变更 |
| 权限提升 | ❌ 无 | 库本身不涉及权限管理 |
| 隐私泄露 | ❌ 无 | 库仅进行哈希计算 |

**结论**：由于该库未应用任何 OH 定制补丁，因此**不存在 Patch 引入的新攻击面**。

## 安全使用建议

### ✅ 安全使用场景

该库适用于以下**可信环境**：

1. **编译器内部**：rustc、rust-analyzer 等编译器工具
2. **构建时工具**：bindgen、代码生成工具等
3. **确定性构建**：需要可预测输出的构建系统
4. **内部数据结构**：应用内部使用的缓存和映射表
5. **性能敏感的可信数据**：键来源可信、无用户输入的场景

### ❌ 不安全使用场景

该库**不适用于**以下场景：

1. **Web 服务**：处理不可信用户输入的网络服务
2. **密码学应用**：需要加密级别安全性的场景
3. **用户数据哈希**：可能包含敏感信息的用户数据
4. **网络协议**：需要防止 DOS 攻击的网络协议
5. **访问控制**：与权限验证相关的哈希操作

### 风险场景示例

```rust
// ❌ 错误示例：使用 FxHashMap 处理用户输入
fn handle_user_request(user_id: String, data: &[u8]) {
    let mut cache: FxHashMap<String, Vec<u8>> = FxHashMap::default();
    cache.insert(user_id, data.to_vec()); // 不安全：用户输入作为键
}

// ✅ 正确示例：使用标准 HashMap 处理用户输入
use std::collections::HashMap;
fn handle_user_request_safe(user_id: String, data: &[u8]) {
    let mut cache: HashMap<String, Vec<u8>> = HashMap::default();
    cache.insert(user_id, data.to_vec()); // 安全：SipHash 提供 DOS 防护
}
```

## Rust 哈希安全最佳实践

### 哈希算法选择指南

| 场景 | 推荐算法 | 原因 |
|------|----------|------|
| 编译器内部 | FxHash (rustc-hash) | 速度优先，无需 DOS 防护 |
| 不可信输入 | SipHash (std HashMap) | 抗 DOS 攻击 |
| 加密用途 | sha2 / blake3 | 加密安全 |
| 高性能可信数据 | FxHash / ahash | 速度优化 |

### Clippy 辅助检查

可以使用 Clippy 避免意外混用不同类型的哈希映射：

```rust
// 在代码库根目录添加 clippy 配置，禁止特定类型
# .clippy.toml
[[disallowed-types]]
path = "std::collections::HashMap"
reason = "Use FxHashMap for performance, or specify allowed usage"
```

## 升级策略建议

### 安全更新策略

| 更新类型 | 策略 | 优先级 |
|----------|------|--------|
| 安全补丁 | 立即评估并升级 | 高 |
| 功能更新 | 评估新功能价值后升级 | 中 |
| 大版本升级 | 完整测试后升级 | 低 |

### 升级检查清单

在升级 rustc-hash 版本时，应检查：

1. [ ] **API 兼容性**：新版本是否有 API 变更
2. [ ] **安全公告**：新版本是否包含安全修复
3. [ ] **依赖者兼容**：bindgen 是否兼容新版本
4. [ ] **性能回归**：新版本的性能是否满足要求

## 相关文档

- [库概述](./01_Overview.md)
- [OH 使用情况](./04_Usage_in_OH.md)
- [上游安全建议](https://github.com/rust-lang/rustc-hash/security)
