# HiTrace Wiki

## 文档说明

本 Wiki 是 HiTrace（调用链追踪框架）的技术文档，旨在帮助开发者快速理解项目架构、使用方法、API 参考及安全注意事项。

### 文档覆盖范围

| 模块 | 说明 |
|------|------|
| [项目概览](./index.md) | 项目定位、核心能力、关键概念 |
| [架构说明](./Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [N-API 参考](./NAPI_Reference.md) | JS API 清单、参数、返回值、绑定位置 |
| [Native API 参考](./Native_API_Reference.md) | C/C++ API 接口说明 |
| [GN 构建系统](./Build_System.md) | Targets 列表、依赖、编译产物 |
| [安全风险评审](./Security_Review.md) | 攻击面分析、风险点、修复建议 |

### 文档维护

- **最后更新**: 2024-02-06
- **更新方式**: 随代码变更手动更新
- **证据来源**: 所有关键结论均可追溯到代码证据（路径+符号）

### 贡献指南

1. 修改代码后请同步更新相关文档
2. 新增 API 需要在对应参考文档中添加条目
3. 安全问题请在 Security_Review.md 中更新

### 相关链接

- [OpenHarmony HiTrace 源码](https://gitee.com/openharmony/base_hiviewdfx_hitrace)
- [API 文档](https://gitee.com/openharmony/docs)
