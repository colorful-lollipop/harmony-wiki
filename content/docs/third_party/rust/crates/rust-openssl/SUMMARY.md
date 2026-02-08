# 阅读路线建议

本文档提供 rust-openssl 库在 OpenHarmony 中集成的全面指南。根据您的需求，选择适合的阅读路线。

## 场景一：快速了解

**目标**: 了解 rust-openssl 在 OH 中的基本情况和适配状态

**建议阅读顺序**:
1. **[README.md](/README.md)** - 2 分钟概览
2. **[04_Usage_in_OH.md](/04_Usage_in_OH.md#概述)** - 了解使用场景

**预计时间**: 5 分钟

## 场景二：深入理解构建适配

**目标**: 理解 rust-openssl 如何适配 OH 构建系统

**建议阅读顺序**:
1. **[README.md](/README.md)** - 基础了解
2. **[03_Build_Integration.md](/03_Build_Integration.md)** - 深入理解 BUILD.gn 配置
3. **[04_Usage_in_OH.md](/04_Usage_in_OH.md)** - 了解依赖关系

**预计时间**: 15 分钟

## 场景三：贡献或维护

**目标**: 参与 rust-openssl 的 OH 适配工作

**建议阅读顺序**:
1. **[README.md](/README.md)** - 概览
2. **[01_Overview.md](/01_Overview.md)** - 原始库功能
3. **[03_Build_Integration.md](/03_Build_Integration.md)** - 构建适配细节
4. **[06_Security.md](/06_Security.md)** - 安全注意事项
5. **[_work/ASSESSMENT.md](/_work/ASSESSMENT.md)** - 项目评估

**预计时间**: 30 分钟

## 场景四：问题排查

**目标**: 排查 rust-openssl 集成问题

**建议阅读顺序**:
1. **[03_Build_Integration.md](/03_Build_Integration.md#常见问题)** - 构建问题
2. **[04_Usage_in_OH.md](/04_Usage_in_OH.md#常见问题)** - 使用问题
3. **[06_Security.md](/06_Security.md)** - 安全相关问题

**预计时间**: 10 分钟

## 文档索引

### 核心文档

| 文档 | 描述 | 优先级 |
|-----|------|-------|
| **[README.md](/README.md)** | 项目概览和快速导航 | ⭐⭐⭐ |
| **[03_Build_Integration.md](/03_Build_Integration.md)** | BUILD.gn 适配详解 | ⭐⭐⭐ |
| **[04_Usage_in_OH.md](/04_Usage_in_OH.md)** | 依赖和使用场景 | ⭐⭐⭐ |

### 补充文档

| 文档 | 描述 | 优先级 |
|-----|------|-------|
| **[01_Overview.md](/01_Overview.md)** | 原始库简介 | ⭐⭐ |
| **[02_Patches.md](/02_Patches.md)** | Patch 分析 | ⭐ |
| **[05_API_Differences.md](/05_API_Differences.md)** | API 差异 | ⭐⭐ |
| **[06_Security.md](/06_Security.md)** | 安全分析 | ⭐⭐ |

### 工作文档

| 文档 | 描述 | 优先级 |
|-----|------|-------|
| **[_work/ASSESSMENT.md](/_work/ASSESSMENT.md)** | 完整评估报告 | ⭐⭐ |

## 关键信息速查

### 版本信息

- **rust-openssl**: 0.10.73
- **openssl-sys**: 0.9.109
- **OpenSSL 兼容性**: 1.1.1 / 3.0.x

### 依赖关系

- **直接依赖者**: hdc_rust
- **外部依赖**: openssl C 库, rust_libc
- **内部依赖**: openssl-macros, openssl-sys

### 适配状态

- **BUILD.gn**: ✅ 完整适配
- **Patch**: ❌ 无 Patch
- **openssl-errors**: ⚠️ 需额外适配

## 常见问题

**Q: rust-openssl 和系统的 openssl 是什么关系？**
A: rust-openssl 是 Rust FFI 绑定，调用系统 openssl 库的函数。OH 中的 openssl C 库是独立的依赖。

**Q: 为什么没有 Patch？**
A: rust-openssl 的设计是 FFI 绑定，不包含业务逻辑。通过 BUILD.gn 配置即可完成适配。

**Q: 如何升级版本？**
A: 直接替换上游代码，更新 BUILD.gn 中的版本号即可。

---

**阅读时间估算**:
- 快速了解: 5 分钟
- 深入理解: 15 分钟  
- 完整贡献: 30 分钟
