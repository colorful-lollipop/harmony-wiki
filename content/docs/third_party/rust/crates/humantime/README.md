# OpenHarmony humantime Wiki

## 库概览

**humantime** 是一个 Rust 库，提供人性化的**时间格式化和解析**功能。在 OpenHarmony 中，该库主要用于**日志系统的时间戳格式化**。

### 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | Human Time |
| **上游版本** | 2.1.0 |
| **OH 组件** | @ohos/rust_humantime |
| **许可证** | Apache-2.0 / MIT |
| **上游地址** | https://github.com/tailhook/humantime |

### 在 OpenHarmony 中的定位

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 日志系统                          │
├─────────────────────────────────────────────────────────────────┤
│  应用日志  │  系统日志  │  HDC 调试日志  │  其他组件日志          │
├─────────────────────────────────────────────────────────────────┤
│                         env_logger                               │
│                    (使用 humantime 格式化)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   humantime     │
                    │ RFC3339 时间戳   │
                    └─────────────────┘
```

## 快速导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异（无差异） |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 关键特性

### OH 适配要点

- **Patch 数量**: `0` (零 Patch 库)
- **定制程度**: 极低（仅 BUILD.gn 构建适配）
- **维护难度**: 低（可直接跟随上游升级）

### 主要使用场景

1. **日志时间戳**: `format_rfc3339_millis(SystemTime::now())`
2. **日志文件名**: HDC 调试工具生成带时间戳的日志文件
3. **时间格式化**: env_logger 包装提供多精度时间戳支持

## 文档状态

| 文档 | 状态 | 说明 |
|------|------|------|
| ASSESSMENT.md | ✅ 完成 | Phase 0 评估报告 |
| 01_Overview.md | ✅ 完成 | 库概览 |
| 02_Patches.md | ✅ 完成 | Patch 分析（零 Patch） |
| 03_Build_Integration.md | ✅ 完成 | 构建适配 |
| 04_Usage_in_OH.md | ✅ 完成 | 依赖分析 |
| 05_API_Differences.md | ✅ 完成 | API 差异 |
| 06_Security.md | ✅ 完成 | 安全分析 |

---

**最后更新**: 2024-XX-XX
