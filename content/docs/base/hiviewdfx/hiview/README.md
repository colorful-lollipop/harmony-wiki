# Hiview 工程 Wiki

## 文档说明

本文档为 **OpenHarmony Hiview** 组件的工程 Wiki，旨在帮助开发者快速理解项目架构、接口规范、构建方式和安全机制。

### 覆盖范围

| 分类 | 内容 |
|------|------|
| 项目概览 | 定位、边界、核心能力、运行环境 |
| 架构说明 | 组件图、数据流、线程模型、关键时序 |
| N-API 接口 | JavaScript/ArkTS API、参数、返回值、错误码 |
| 内部 API | 模块接口、依赖方向、稳定性标注 |
| 构建系统 | GN Targets、编译产物、特性开关 |
| 安全评审 | 攻击面、信任边界、风险点与修复建议 |

### 未覆盖范围

- 测试相关代码（`test/`, `*_test.*`）
- Fuzzer 测试（`fuzztest/`, `*_fuzzer.*`）
- 示例插件（`test/plugins/examples/`）
- 具体业务逻辑实现细节

### 更新说明

本文档基于代码仓库 `base/hiviewdfx/hiview` 的以下版本生成：

- **源码版本**: 参见 `bundle.json` 中的版本号
- **生成时间**: 2026-02-06
- **更新方式**: 手动更新，需随代码变更同步修改

如需更新本文档，请：

1. 扫描新增/修改的 N-API 接口
2. 更新对应的 API 清单表
3. 检查构建文件的 target 变更
4. 审查新增代码的安全风险

### 快速链接

- [README.md](README.md) - 项目自述
- [SUMMARY.md](SUMMARY.md) - 全文导航
- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Architecture.md](01_Architecture.md) - 架构说明
- [02_NAPI_Reference.md](02_NAPI_Reference.md) - N-API 接口
- [03_Inner_API.md](03_Inner_API.md) - 内部 API
- [04_Build_System.md](04_Build_System.md) - 构建系统
- [05_Artifacts.md](05_Artifacts.md) - 编译产物
- [06_Security_Review.md](06_Security_Review.md) - 安全评审
- [07_Troubleshooting.md](07_Troubleshooting.md) - 常见问题
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
