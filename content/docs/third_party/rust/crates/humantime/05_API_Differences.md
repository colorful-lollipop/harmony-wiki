# API 差异

## 概述

**humantime 在 OpenHarmony 中未进行任何 API 修改**。

该库在 OH 中的 API 与上游原始库完全一致，没有新增、修改或废弃的 API。

## API 对比表

### 公共 API 清单

| API | 上游状态 | OH 状态 | 差异 |
|-----|----------|---------|------|
| `parse_duration()` | ✅ 可用 | ✅ 可用 | 无 |
| `DurationError` | ✅ 可用 | ✅ 可用 | 无 |
| `format_duration()` | ✅ 可用 | ✅ 可用 | 无 |
| `FormattedDuration` | ✅ 可用 | ✅ 可用 | 无 |
| `Duration` (wrapper) | ✅ 可用 | ✅ 可用 | 无 |
| `Timestamp` (wrapper) | ✅ 可用 | ✅ 可用 | 无 |
| `parse_rfc3339()` | ✅ 可用 | ✅ 可用 | 无 |
| `parse_rfc3339_weak()` | ✅ 可用 | ✅ 可用 | 无 |
| `TimestampError` | ✅ 可用 | ✅ 可用 | 无 |
| `format_rfc3339()` | ✅ 可用 | ✅ 可用 | 无 |
| `format_rfc3339_millis()` | ✅ 可用 | ✅ 可用 | 无 |
| `format_rfc3339_micros()` | ✅ 可用 | ✅ 可用 | 无 |
| `format_rfc3339_nanos()` | ✅ 可用 | ✅ 可用 | 无 |
| `format_rfc3339_seconds()` | ✅ 可用 | ✅ 可用 | 无 |
| `Rfc3339Timestamp` | ✅ 可用 | ✅ 可用 | 无 |

**结论**: 所有 15 个公共 API 在 OH 中均可用且与上游一致。

## API 使用现状

### 实际使用统计

| API | 使用状态 | 使用组件 |
|-----|----------|----------|
| `format_rfc3339_millis()` | ✅ 使用 | hdc_rust, env_logger |
| `format_rfc3339_seconds()` | ✅ 使用 | env_logger |
| `format_rfc3339_micros()` | ✅ 使用 | env_logger |
| `format_rfc3339_nanos()` | ✅ 使用 | env_logger |
| `parse_duration()` | ❌ 未使用 | - |
| `format_duration()` | ❌ 未使用 | - |
| `parse_rfc3339()` | ❌ 未使用 | - |
| 其他 API | ❌ 未使用 | - |

### 使用特点

1. **时间戳格式化为主**: OH 主要使用 `format_rfc3339_*` 系列函数
2. **持续时间功能未使用**: `parse_duration` 和 `format_duration` 暂未被调用
3. **包装类型未使用**: `Duration` 和 `Timestamp` wrapper 类型暂未被直接实例化

## 与上游的行为一致性

### 格式输出一致性

```rust
// 上游和 OH 的输出完全一致
let ts = humantime::format_rfc3339_millis(SystemTime::now());
// 输出: "2024-01-15T10:30:45.123Z"
```

### 错误处理一致性

```rust
// Error 类型完全一致
use humantime::{parse_duration, DurationError};

fn parse_timeout(s: &str) -> Result<Duration, DurationError> {
    parse_duration(s)  // 返回的 Error 类型与上游一致
}
```

## 潜在差异场景

虽然当前无差异，但以下场景可能引入差异：

| 场景 | 可能性 | 影响 | 建议 |
|------|--------|------|------|
| 添加 OH 特定时间格式 | 低 | 需要新增 API | 不建议，RFC3339 已是标准 |
| 修改时间精度默认值 | 极低 | 行为变更 | 不建议 |
| 扩展 Error 类型 | 低 | API 变更 | 需谨慎评估 |

## 升级兼容性

由于 API 无差异，升级上游版本时：

### 向后兼容升级

如果上游保持向后兼容（patch 或 minor 版本升级）：
- ✅ 无需修改 OH 代码
- ✅ 直接替换代码即可
- ✅ 重新编译后直接使用

### 破坏性升级

如果上游发布重大版本（如 v3.0）并包含 API 变更：
- ⚠️ 需要检查 API 变更影响
- ⚠️ 更新依赖者的调用代码
- ⚠️ 进行全面测试

**当前状态**: v2.1.0 是稳定版本，API 已固化，短期内无破坏性变更风险。

## 使用建议

### 对于开发者

1. **参考上游文档**: 可直接参考 https://docs.rs/humantime 使用
2. **标准 API 调用**: 无需考虑 OH 特定差异
3. **依赖管理**: 通过 GN 的 `deps` 引用即可

### 示例代码

```rust
// 在 OH 中使用与上游完全一致
use std::time::SystemTime;
use humantime::format_rfc3339_millis;

fn log_with_timestamp(msg: &str) {
    let ts = format_rfc3339_millis(SystemTime::now());
    println!("[{}] {}", ts, msg);
}
```

## 结论

humantime 是 OH 中 **API 完全一致** 的第三方库：

- ✅ 无 API 新增
- ✅ 无 API 修改  
- ✅ 无 API 废弃
- ✅ 行为与上游一致
- ✅ 升级路径简单

**对于开发者而言，可以将其视为与上游完全等同的库使用，无需学习 OH 特定的 API 差异。**
