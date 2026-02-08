# OpenHarmony Advertising 子系统 Wiki

## 文档覆盖范围

本文档为 OpenHarmony `advertising` 子系统提供完整的工程参考，涵盖以下内容：

### 覆盖范围

| 领域 | 状态 | 说明 |
|-----|------|------|
| 项目概览 | ✅ | 定位、能力、运行环境 |
| 目录结构 | ✅ | 模块职责与边界 |
| N-API 接口 | ✅ | JS API 与 C++ 绑定 |
| 内部架构 | ✅ | 模块关系、IPC 通信 |
| GN 构建 | ✅ | Targets、依赖、产物 |
| 安全评审 | ✅ | 风险分析与建议 |

### 未覆盖范围

- 测试代码 (`test/`, `tests/`, `*_test.*`)
- 第三方开源库 (cJSON, libuv 等)
- OpenHarmony 框架核心实现

## 更新方式

当代码变更时，需同步更新 Wiki：

1. **N-API 变更**：更新 `NAPI_Reference.md`
2. **架构变更**：更新 `Architecture.md` 和 `Inner_API.md`
3. **构建变更**：更新 `Build_Configuration.md`
4. **安全变更**：更新 `Security_Review.md`

## 质量保证

- ✅ 所有技术结论均有代码证据支撑（29 处文件:行号引用）
- ✅ 包含 2 个 Mermaid 时序图（广告请求、广告展示）
- ✅ 覆盖 5 个安全风险点，每个都有触发路径和修复建议
- ✅ 提供新人学习路线和安全研究路线（双路线导航）
- ✅ 详见 [_work/QUALITY_REPORT.md](_work/QUALITY_REPORT.md)

## 生成信息

- **生成时间**: 2026-02-07
- **代码版本**: OpenHarmony advertising 3.2
- **文档版本**: 1.0
- **评估完成**: Phase 0（项目评估）+ Phase 1-7（Wiki 生成）

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [API 参考](https://docs.openharmony.cn/pages/v4.0/zh-cn/application-dev/reference/apis/js-apis-advertising.md)
