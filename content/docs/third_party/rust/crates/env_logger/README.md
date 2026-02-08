# env_logger - OpenHarmony 集成文档

> Rust 日志系统在 OpenHarmony 中的集成与使用指南

## 文档导航

### 快速开始

- **[阅读路线建议](SUMMARY.md)** - 根据你的角色选择适合的阅读路径
- **[基础概览](01_Overview.md)** - 库的简介和在 OH 中的定位

### 核心文档

- **[构建集成](03_Build_Integration.md)** - BUILD.gn 配置与编译选项
- **[使用场景](04_Usage_in_OH.md)** - 谁在使用 env_logger、如何使用

### 参考文档

- **[Patch 分析](02_Patches.md)** - OH 的修改与适配（本库无 Patch）
- **[API 差异](05_API_Differences.md)** - OH 版本与上游的差异（本库无差异）
- **[安全分析](06_Security.md)** - 已知漏洞与升级建议

### 工作文件

- **[项目评估](./_work/ASSESSMENT.md)** - 详细的技术评估报告
- **[分析笔记](./_work/NOTES.md)** - 分析过程中的笔记
- **[工作计划](./_work/PLAN.md)** - 任务进度跟踪

---

## 快速了解

### env_logger 是什么？

env_logger 是 Rust 生态中最常用的日志实现之一，提供：
- ✅ 通过环境变量（`RUST_LOG`）配置日志级别
- ✅ 灵活的日志格式化和过滤
- ✅ 彩色输出支持
- ✅ 正则表达式日志过滤

### 在 OpenHarmony 中

**集成模式**：原汁原味集成
- ❌ 无源代码修改
- ✅ 完全同步上游 v0.10.2 版本
- ✅ 仅添加 BUILD.gn 构建配置

**使用场景**：
- HDC（OpenHarmony Device Connector）- 设备调试工具
- bindgen-cli - Rust FFI 绑定生成工具

### 关键特性

```bash
# 通过环境变量控制日志级别
export RUST_LOG=info          # 全局 info 级别
export RUST_LOG=hdc=debug    # hdc 模块 debug 级别
export RUST_LOG=off           # 关闭所有日志

# 在代码中初始化
env_logger::init();
```

---

## 贡献者

本文档由 OpenHarmony 第三方库 Wiki 生成 Agent 创建。

## 许可证

本文档遵循 Apache License 2.0 / MIT License（与 env_logger 一致）。
