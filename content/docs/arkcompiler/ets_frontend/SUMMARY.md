# 文档导航

## 首页与概览

- [README](README.md) - 文档说明、更新方式、生成信息
- [00_Overview](00_Overview.md) - 项目定位、核心能力、架构概览

## 项目结构

- [01_Directory_Structure](01_Directory_Structure.md) - 目录结构与模块职责

## 使用指南

- [02_CLI_Reference](02_CLI_Reference.md) - es2abc 命令行接口参考
- [编译产物与运行时](06_Build_Outputs.md) - 编译产物、安装路径、加载关系

## 开发指南

- [03_NAPI_Bindings](03_NAPI_Bindings.md) - N-API 绑定与 Native Interop
- [04_Build_System](04_Build_System.md) - GN 构建系统与 Targets
- [架构详解](07_Architecture.md) - 编译流程、数据流、关键时序

## 安全与审计

- [05_Security_Review](05_Security_Review.md) - 安全风险评审

## 附录

- [附录：术语表](appendix/Glossary.md) - 术语定义
- [附录：FAQ](appendix/FAQ.md) - 常见问题与解决方案
- [附录：API 清单](appendix/API_Reference.md) - 完整 API 列表

---

## 新人阅读推荐

### 场景 1：首次接触项目
阅读路径：README → 00_Overview → 01_Directory_Structure → 02_CLI_Reference

### 场景 2：进行开发工作
阅读路径：README → 04_Build_System → 相关模块文档 → 07_Architecture

### 场景 3：安全审计
阅读路径：README → 05_Security_Review → 相关安全章节

### 场景 4：集成与部署
阅读路径：README → 02_CLI_Reference → 06_Build_Outputs → 04_Build_System

---

## 文档版本

| 版本 | 更新日期 | 更新内容 |
|------|----------|----------|
| 1.2 | 2026-02-07 | 新增 FAQ 文档，增强安全评审（代码证据） |
| 1.1 | 2026-02-06 | 补充 N-API 绑定文档，完善攻击面分析 |
| 1.0 | 2026-02-05 | 初始版本 |
