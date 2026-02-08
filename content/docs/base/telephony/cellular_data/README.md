# Cellular Data 模块 Wiki

## 概述

本文档是 OpenHarmony Telephony 子系统 **Cellular Data（蜂窝数据）** 模块的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、编译构建及安全考量。

## 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 模块定位与边界 | ✅ | 核心职责、依赖关系 |
| 目录结构 | ✅ | 按职责划分的目录组织 |
| N-API 接口 | ✅ | JS/TS API 完整清单 |
| 内部架构 | ✅ | 服务层、状态机、IPC |
| GN 构建配置 | ✅ | Targets、依赖、产物 |
| 安全风险评审 | ⚠️ | 基础评估，详细待补充 |
| 故障排查 | ⏳ | 待补充 |

## 文档导航

请参考 [SUMMARY.md](./SUMMARY.md) 获取完整目录和阅读路线。

## 更新方式

本 Wiki 由代码自动生成，建议：

1. **代码变更时同步更新**：修改 API 或架构后，同步更新对应文档
2. **定期审查**：每月检查文档与代码一致性
3. **贡献指南**：直接在 `wiki/` 目录提交 PR

## 相关链接

- **代码仓库**: https://gitee.com/openharmony/telephony_cellular_data
- **OpenHarmony Telephony**: https://gitee.com/openharmony/telephony
- **系统能力文档**: https://gitee.com/openharmony/docs/blob/master/zh-cn/system-dev-guides/

---

*最后更新: 2026-02-06*
