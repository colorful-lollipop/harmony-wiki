# OpenHarmony 方舟工具链组件 Wiki

本文档为 OpenHarmony `arkcompiler/toolchain` 仓库的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 接口、构建方式和安全风险。

## 文档覆盖范围

本 Wiki 涵盖方舟工具链的核心模块，包括调试调优协议实现、WebSocket 通信层、调试协议对接层以及平台抽象层。文档重点关注非测试代码的业务逻辑分析，所有结论均可追溯到源码证据。

当前版本文档基于代码仓库的以下版本生成：`待补充（执行 git log 获取）`

## 文档结构导航

详见 [SUMMARY.md](./SUMMARY.md)，该文件提供了完整的站点导航和推荐阅读顺序。

## 文档更新方式

当代码仓库发生以下变更时，建议同步更新本 Wiki：新增或删除核心模块、修改 N-API 导出接口、调整 GN 构建配置、变更安全敏感代码。更新时应在对应文档中注明版本和变更说明。

## 术语表

| 术语 | 说明 |
|------|------|
| 调试调优协议 | DevEco Studio 与运行时之间通信的协议规范 |
| 域（Domain） | 调试调优协议的功能划分，包括 Debugger、Profiler、HeapProfiler、Runtime |
| N-API | Node.js Native API，用于 JavaScript 与原生代码交互 |
| WebSocket | 全双工通信协议，用于调试工具连接 |
| GN | Generate Ninja，构建系统配置语言 |

## 相关资源链接

- OpenHarmony 官方文档：https://developer.harmonyos.com
- DevEco Studio 调试指南：https://developer.harmonyos.com/cn/docs/documentation/doc-guides/ide_debug_device-0000001053822404
- 方舟运行时仓库：arkcompiler_ets_runtime
- 方舟编译器核心仓库：arkcompiler_runtime_core

---

*本文档由 Wiki 生成 Agent 自动生成，最后更新时间：2026年2月*
