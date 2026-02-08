# 阅读路线建议

## 选择你的阅读路径

### 路径 A：快速概览（5 分钟）

适合：需要快速了解 spirv-headers 在 OH 中定位的开发者

| 顺序 | 文档 | 预计时间 | 核心内容 |
|------|------|----------|----------|
| 1 | [README.md](README.md) | 2 分钟 | 库概览和文档导航 |
| 2 | [01_Overview.md](01_Overview.md) | 3 分钟 | 原始功能和 OH 定位 |

**输出**：了解 spirv-headers 是什么，以及它在 OH 图形栈中的位置。

---

### 路径 B：深入集成（15 分钟）

适合：需要理解 spirv-headers 构建适配和使用的开发者

| 顺序 | 文档 | 预计时间 | 核心内容 |
|------|------|----------|----------|
| 1 | [README.md](README.md) | 2 分钟 | 库概览和文档导航 |
| 2 | [01_Overview.md](01_Overview.md) | 3 分钟 | 原始功能和 OH 定位 |
| 3 | [03_Build_Integration.md](03_Build_Integration.md) | 5 分钟 | BUILD.gn 适配详解 |
| 4 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 5 分钟 | 依赖关系和使用场景 |

**输出**：理解如何构建和使用 spirv-headers，以及它与其他组件的关系。

---

### 路径 C：完整文档（30 分钟）

适合：需要全面了解 spirv-headers 在 OH 中所有细节的开发者

| 顺序 | 文档 | 预计时间 | 核心内容 |
|------|------|----------|----------|
| 1 | [README.md](README.md) | 2 分钟 | 库概览和文档导航 |
| 2 | [01_Overview.md](01_Overview.md) | 3 分钟 | 原始功能和 OH 定位 |
| 3 | [02_Patches.md](02_Patches.md) | 3 分钟 | Patch 分析（本库无 Patch） |
| 4 | [03_Build_Integration.md](03_Build_Integration.md) | 5 分钟 | BUILD.gn 适配详解 |
| 5 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 5 分钟 | 依赖关系和使用场景 |
| 6 | [05_API_Differences.md](05_API_Differences.md) | 3 分钟 | API 差异（本库无差异） |
| 7 | [06_Security.md](06_Security.md) | 5 分钟 | 安全风险分析 |
| 8 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 4 分钟 | 详细项目评估 |

**输出**：全面掌握 spirv-headers 在 OH 中的集成细节、安全考量和维护策略。

---

## 主题索引

### 构建相关

| 主题 | 文档 | 章节 |
|------|------|------|
| BUILD.gn 结构 | [03_Build_Integration.md](03_Build_Integration.md) | 全部 |
| 与上游差异 | [03_Build_Integration.md](03_Build_Integration.md) | 与上游构建系统的差异 |
| 版本选择 | [03_Build_Integration.md](03_Build_Integration.md) | 特殊处理 |
| 头文件引用 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 使用方式 |

### 依赖相关

| 主题 | 文档 | 章节 |
|------|------|------|
| 直接依赖者 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 直接依赖者 |
| 依赖图 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖图 |
| 版本兼容性 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖版本兼容性 |

### 适配相关

| 主题 | 文档 | 章节 |
|------|------|------|
| Patch 分析 | [02_Patches.md](02_Patches.md) | 全部 |
| API 差异 | [05_API_Differences.md](05_API_Differences.md) | 全部 |
| 安全考量 | [06_Security.md](06_Security.md) | 全部 |

### 维护相关

| 主题 | 文档 | 章节 |
|------|------|------|
| 升级策略 | [02_Patches.md](02_Patches.md) | 维护策略建议 |
| 安全更新 | [06_Security.md](06_Security.md) | 安全建议 |
| 应急响应 | [06_Security.md](06_Security.md) | 应急响应 |

## 常见问题快速定位

| 问题 | 答案位置 |
|------|----------|
| spirv-headers 是什么？ | [01_Overview.md](01_Overview.md) |
| 为什么没有 Patch？ | [02_Patches.md](02_Patches.md) |
| 如何构建？ | [03_Build_Integration.md](03_Build_Integration.md) |
| 谁在使用？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| 有安全风险吗？ | [06_Security.md](06_Security.md) |
| 如何升级？ | [02_Patches.md](02_Patches.md) 或 [06_Security.md](06_Security.md) |

## 文档更新日志

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-07 | 1.0 | 初始文档版本 |

## 贡献指南

如需更新本文档：

1. 修改对应的 `.md` 文件
2. 更新 [PLAN.md](_work/PLAN.md) 中的任务状态
3. 运行文档验证（如果有）
4. 提交更改

## 相关文档链接

- [spirv-tools Wiki](../spirv-tools/wiki/README.md)
- [vk-gl-cts Wiki](../vk-gl-cts/wiki/README.md)
- [OpenHarmony 图形架构](../README.md)
- [Khronos SPIR-V Registry](https://www.khronos.org/registry/spir-v/)
