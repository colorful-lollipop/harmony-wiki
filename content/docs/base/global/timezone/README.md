# OpenHarmony 时区数据管理模块 Wiki

## 文档说明

本文档为 OpenHarmony `base/global/timezone` 模块的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、构建流程和安全考量。

### 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 已覆盖 | 定位、能力、依赖 |
| 目录结构 | ✅ 已覆盖 | 模块职责划分 |
| 架构设计 | ✅ 已覆盖 | 组件关系、数据流 |
| 构建配置 | ✅ 已覆盖 | GN targets、编译产物 |
| 安全评审 | ✅ 已覆盖 | 风险分析、修复建议 |
| 使用指南 | ✅ 已覆盖 | 下载、编译、部署 |
| N-API 接口 | ❌ 不适用 | 本模块不提供 N-API |
| IPC/SA 调用 | ❌ 不适用 | 本模块不涉及进程通信 |

### 文档更新方式

当代码发生变更时，请同步更新相关 Wiki 章节：

1. **新增功能**: 在对应章节添加功能说明和代码证据
2. **修改构建**: 更新 `02_Build.md` 中的 target 信息
3. **安全修复**: 在 `03_Security.md` 中记录新发现和修复措施

### 生成信息

- **生成时间**: 2024-02-06
- **代码版本**: 基于 `base/global/timezone` HEAD
- **文档版本**: 1.0.0

### 相关链接

- [GitHub 仓库](https://gitee.com/openharmony/global_timezone)
- [OpenHarmony 文档](https://gitee.com/openharmony/docs)
- [IANA Time Zone Database](https://data.iana.org/time-zones/releases/)
