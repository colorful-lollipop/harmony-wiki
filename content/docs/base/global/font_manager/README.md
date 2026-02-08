# Font Manager Wiki - 项目文档

## 📊 文档概述

本文档为 OpenHarmony `font_manager` 子系统的完整工程 Wiki，旨在帮助**新人学习者**和**安全研究员**快速理解项目架构、API 使用、构建配置和安全风险。

## 📈 文档统计

| 指标 | 值 | 说明 |
|------|-----|------|
| 总文档数 | 10 篇 | 核心文档 + 工作区文档 |
| 总行数 | 2321+ 行 | 代码证据 + 说明文字 |
| 代码证据数 | 50+ 处 | 文件路径 + 行号 + 函数名 |
| 安全风险点 | 10 个 | 已缓解 5 个 + 潜在 5 个 |
| 错误码覆盖 | 14 个 | 完整错误码表 |

## 📚 覆盖范围

### ✅ 已覆盖内容

| 模块 | 说明 | 证据来源 | 受众 |
|------|------|----------|------|
| 项目概览 | 功能定位、目录结构、系统能力 | `bundle.json`, `README.md` | 👶 新人 |
| 架构设计 | 分层架构、组件关系、数据流 | `README.md:1-15`, 代码分析 | 👶 新人 |
| N-API 对外接口 | 3 个 JS API 详解、参数校验、错误码 | `font_manager_addon.cpp` | 👶 新人 |
| Inner API | FontManagerKits、FontManagerClient 接口 | `font_manager_kits.h`, `font_manager_client.h` | 👶 新人 |
| GN 构建目标 | 全部 targets、依赖、输出 | `service/BUILD.gn`, `interfaces/js/kits/BUILD.gn` | 👷 开发者 |
| 编译产物 | .so 文件、安装路径、加载关系 | `sa_profile/66262.json` | 👷 开发者 |
| 安全风险 | 攻击面、信任边界、风险点（10个） | 代码审计 | 🛡️ 安全研究员 |
| 故障排查 | 常见问题、定位方法 | 代码分析 | 👷 开发者 |

### ❌ 未覆盖内容

| 模块 | 原因 |
|------|------|
| 测试代码 | 根据规范，测试相关内容不作为业务证据来源 |
| ANI 接口 | 代码存在但未深入分析 (interfaces/ani/) |

## 🎯 受众导向

### 👶 新人学习者

**你能获得什么**：

- ✅ **5分钟内理解项目定位**：清晰的项目概览和能力边界
- ✅ **15分钟内找到核心代码**：完整的目录结构和代码地图
- ✅ **30分钟内理解基本架构**：分层架构图和数据流图
- ✅ **快速上手 API**：3个 N-API 接口的完整使用示例
- ✅ **常见问题解答**：故障排查文档覆盖常见问题

**推荐阅读顺序**：
1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [架构说明](01_Architecture.md) - 理解整体架构和数据流
3. [N-API 参考](02_NAPI_Reference.md) - 掌握 API 使用方法
4. [内部 API](03_Inner_API.md) - 了解模块边界和职责
5. [构建目标](04_Build_Targets.md) - 学习编译和构建配置

**预期学习时间**：2 小时（完整阅读）

### 🛡️ 安全研究员

**你能获得什么**：

- ✅ **快速识别所有攻击面**：外部输入清单（N-API 参数、IPC 数据、文件输入）
- ✅ **定位所有信任边界**：5层信任边界和权限校验点
- ✅ **完整的风险评估**：10个安全风险点，每个都有代码证据和修复建议
- ✅ **漏洞利用路径**：详细的触发路径和影响评估
- ✅ **修复建议**：每个风险都有代码级别的修复建议

**推荐阅读顺序**：
1. [攻击面分析](06_Security_Review.md#攻击面分析) - 识别所有外部输入和敏感操作
2. [信任边界](06_Security_Review.md#信任边界) - 理解权限校验点和安全域跨越
3. [安全风险清单](06_Security_Review.md#安全风险清单) - 审查10个风险点
4. [深入代码审计](06_Security_Review.md#潜在风险) - 分析TOCTOU、配置注入等潜在风险
5. [修复建议](06_Security_Review.md#安全加固建议) - 查看优先级和修复方案

**预期审计时间**：5 小时（深度审计）

**风险概览**：
- ✅ **已缓解风险**（5个）：权限校验、路径遍历防护、fd验证、数量限制、格式验证
- ⚠️ **潜在风险**（5个）：TOCTOU竞态条件（中危）、配置注入（低危）、SA卸载（低危）、临时文件（低危）、内存安全（已缓解）

## 🚀 快速跳转

### 新人快速通道

| 需求 | 跳转到 | 用时 |
|------|--------|------|
| 了解项目是什么 | [00_Overview.md](00_Overview.md) | 5 分钟 |
| 查看如何使用 API | [02_NAPI_Reference.md](02_NAPI_Reference.md) | 15 分钟 |
| 理解架构设计 | [01_Architecture.md](01_Architecture.md) | 30 分钟 |
| 查找错误码含义 | [02_NAPI_Reference.md#错误码](02_NAPI_Reference.md#错误码) | 2 分钟 |
| 构建和编译 | [04_Build_Targets.md](04_Build_Targets.md) | 15 分钟 |

### 安全研究快速通道

| 需求 | 跳转到 | 用时 |
|------|--------|------|
| 识别所有攻击面 | [06_Security_Review.md#攻击面分析](06_Security_Review.md#攻击面分析) | 15 分钟 |
| 查看信任边界 | [06_Security_Review.md#信任边界](06_Security_Review.md#信任边界) | 10 分钟 |
| 审计安全风险 | [06_Security_Review.md#安全风险清单](06_Security_Review.md#安全风险清单) | 60 分钟 |
| 查看修复建议 | [06_Security_Review.md#安全加固建议](06_Security_Review.md#安全加固建议) | 15 分钟 |

## 📖 详细目录

### 核心文档

| 章节 | 标题 | 受众 | 用时预估 |
|------|------|------|---------|
| [00](00_Overview.md) | 项目概览 | 👶 新人 | 15 分钟 |
| [01](01_Architecture.md) | 架构说明 | 👶 新人 | 30 分钟 |
| [02](02_NAPI_Reference.md) | N-API 参考 | 👶 新人 | 30 分钟 |
| [03](03_Inner_API.md) | 内部 API | 👶 新人 | 30 分钟 |
| [04](04_Build_Targets.md) | 构建目标 | 👷 开发者 | 20 分钟 |
| [05](05_Artifacts.md) | 编译产物 | 👷 开发者 | 15 分钟 |
| [06](06_Security_Review.md) | 安全评审 | 🛡️ 安全研究员 | 60 分钟 |
| [07](07_Troubleshooting.md) | 故障排查 | 👷 开发者 | 15 分钟 |

### 工作区文档

| 文件 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果（Phase 0） |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据汇总 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

## 📝 文档更新

### 更新方式

1. **代码修改时**: 如果修改了 N-API、架构或安全逻辑，同步更新对应 Wiki 章节
2. **Bugfix 时**: 如果发现文档错误，及时修正
3. **版本发布时**: 检查文档与代码的一致性

### 更新责任矩阵

| 修改模块 | 需要更新的文档 | 责任人 |
|---------|---------------|--------|
| `interfaces/js/kits/` | `02_NAPI_Reference.md` | N-API 开发者 |
| `interfaces/ani/` | (待补充) | ANI 开发者 |
| `service/client/` | `03_Inner_API.md`, `04_Build_Targets.md` | 客户端开发者 |
| `service/server/` | `03_Inner_API.md`, `06_Security_Review.md` | 服务端开发者 |
| `frameworks/fontmgr/` | `01_Architecture.md`, `06_Security_Review.md` | 框架开发者 |
| `sa_profile/` | `05_Artifacts.md` | SA 开发者 |
| `common/` | `00_Overview.md`, `06_Security_Review.md` | 通用开发者 |

### 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| 2.0 | 2025-02-07 | 完善双路线导航，增加受众导向说明 |
| 1.0 | 2025-02-06 | 初始版本，完成核心文档 |

## 🔗 外部链接

### 官方文档

| 链接 | 说明 |
|------|------|
| [OpenHarmony 官方文档](https://gitee.com/openharmony/docs) | 系统级文档和开发指南 |
| [font_manager 仓库](https://gitee.com/openharmony/global_font_manager) | 源代码仓库 |
| [SystemAbility 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/standard/systemability/sysa-development.md) | SA 开发最佳实践 |
| [N-API 开发指南](https://gitee.com/openharmony/napi) | N-API 接口规范 |
| [IPC 通信指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/standard/ipc/ipc-overview.md) | Binder IPC 开发 |

### 系统能力

| 系统能力 | 说明 | 权限 |
|---------|------|------|
| `SystemCapability.Global.FontManager` | 字体管理系统能力 | - |
| `ohos.permission.UPDATE_FONT` | 更新字体权限 | system_core 级别 |

## 💬 反馈与贡献

如有文档问题或建议，请：

1. **提交 Issue**：到 [font_manager 仓库](https://gitee.com/openharmony/global_font_manager/issues)
2. **提交 PR**：直接修改文档，描述改进点
3. **联系维护者**：在 Issues 中 @Font Manager 团队

**文档维护者**：Font Manager 团队
**最后更新**：2025-02-07
