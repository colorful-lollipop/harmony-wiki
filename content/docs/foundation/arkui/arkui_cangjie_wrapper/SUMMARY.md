# 文档导航 (SUMMARY)

本文档为 OpenHarmony **arkui_cangjie_wrapper** 仓库的工程 Wiki，提供新人可快速完整理解项目的多篇 Markdown 文档。

---

## 📖 新人阅读路线

建议阅读顺序：

1. **[00_Overview.md](./00_Overview.md)** - 项目定位、核心能力、运行环境
2. **[01_Directory_Structure.md](./01_Directory_Structure.md)** - 目录结构与模块职责
3. **[02_Architecture.md](./02_Architecture.md)** - 系统架构、组件图、数据流
4. **[03_N-API.md](./03_N-API.md)** - 对外 N-API/Cangjie API 清单
5. **[04_Inner_API.md](./04_Inner_API.md)** - 内部模块接口
6. **[05_GN_Build.md](./05_GN_Build.md)** - GN Targets 与编译产物
7. **[06_Security.md](./06_Security.md)** - 安全风险评审
8. **[07_Troubleshooting.md](./07_Troubleshooting.md)** - 常见问题与调试

---

## 📚 完整文档列表

### 入门与概览

| 文档 | 说明 |
|------|------|
| [README.md](./README.md) | Wiki 覆盖范围、更新方式 |
| [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |

### 架构与设计

| 文档 | 说明 |
|------|------|
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责（不含测试） |
| [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |

### API 参考

| 文档 | 说明 |
|------|------|
| [03_N-API.md](./03_N-API.md) | N-API（JS/Cangjie API 面）、导出符号、权限/参数/错误码 |
| [04_Inner_API.md](./04_Inner_API.md) | 模块接口、依赖方向、稳定性、可替换点 |

### 构建与部署

| 文档 | 说明 |
|------|------|
| [05_GN_Build.md](./05_GN_Build.md) | Targets 列表、依赖、产物、开关 |

### 安全与运维

| 文档 | 说明 |
|------|------|
| [06_Security.md](./06_Security.md) | 攻击面、信任边界、风险分析 |
| [07_Troubleshooting.md](./07_Troubleshooting.md) | 构建/运行/调试问题与定位路径 |

---

## 🔗 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑）|
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键宏/feature flags |

---

## 📝 变更日志

| 日期 | 变更内容 | 贡献者 |
|------|----------|--------|
| 2025-02-06 | 初始 Wiki 生成 | Wiki Agent |
