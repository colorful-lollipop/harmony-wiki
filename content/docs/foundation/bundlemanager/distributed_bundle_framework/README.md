# 分布式包管理服务 (DBMS) Wiki

**生成时间**: 2026-02-06
**覆盖范围**: 分布式包管理服务 (DBMS) 的完整技术文档

---

## 文档说明

本 Wiki 包含以下内容：

| 文档 | 描述 | 适用人员 |
|--------|------|----------|
| [README](./README.md) | Wiki 使用指南和更新方式 | 所有人 |
| [SUMMARY](./SUMMARY.md) | 完整导航和阅读顺序 | 新人、开发者 |
| [01_Overview](./01_Overview.md) | 项目定位、核心能力、运行环境 | 所有人 |
| [02_Directory_Structure](./02_Directory_Structure.md) | 目录结构和模块职责 | 开发者 |
| [03_Architecture](./03_Architecture.md) | 架构说明、组件图、数据流 | 架构师、开发者 |
| [04_JS_API](./04_JS_API.md) | 对外 N-API 清单和调用链 | 应用开发者 |
| [05_Inner_API](./05_Inner_API.md) | 内部 API 和接口稳定性 | 内部开发者 |
| [06_GN_Build](./06_GN_Build.md) | GN targets 和编译产物 | 构建工程师 |
| [07_Security_Analysis](./07_Security_Analysis.md) | 安全风险评审和攻击面 | 安全工程师 |

### 可选附录

| 文档 | 描述 |
|--------|------|
| [appendix/Callgraphs](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags](./appendix/Config_Flags.md) | 关键宏和 feature flags |

---

## 未覆盖范围

以下内容未在本次 Wiki 中覆盖：

- 测试相关代码实现细节（但会引用测试用例作为验证依据）
- 性能优化建议（仅描述现有实现）
- 未来 roadmap 计划

---

## 更新指南

### 如何更新文档

1. **代码变更后**：更新对应文档中的证据引用（文件路径、行号）
2. **API 变更后**：更新 `04_JS_API.md` 中的 API 清单
3. **构建系统变更后**：更新 `06_GN_Build.md` 中的 targets 列表
4. **安全评估**：代码变更后，重新评估 `07_Security_Analysis.md` 中的风险点

### 证据追踪规范

- 所有关键结论必须包含代码证据
- 证据格式：`文件路径:行号` 或 `文件路径`（符号名）
- 无法确认的内容需标注 `TODO(需确认)` 并说明缺少的证据

---

## 新人阅读顺序

推荐按以下顺序阅读 Wiki 文档：

1. **快速入门**（30分钟）：
   - [README](./README.md) - 了解 Wiki 结构
   - [01_Overview](./01_Overview.md) - 理解项目定位

2. **深入理解**（2小时）：
   - [02_Directory_Structure](./02_Directory_Structure.md) - 熟悉代码组织
   - [03_Architecture](./03_Architecture.md) - 理解架构设计
   - [04_JS_API](./04_JS_API.md) - 学习对外 API

3. **专业参考**（按需）：
   - [05_Inner_API](./05_Inner_API.md) - 内部接口开发
   - [06_GN_Build](./06_GN_Build.md) - 构建系统
   - [07_Security_Analysis](./07_Security_Analysis.md) - 安全评审
   - [附录文档](./appendix/) - 详细调用链和配置

---

## 版本历史

| 版本 | 日期 | 变更说明 | 作者 |
|------|------|----------|------|
| 1.0 | 2026-02-06 | 初始版本，完整覆盖 DBMS 各个方面 | OpenHarmony Wiki Agent |

---

## 联系与反馈

如有问题或建议，请联系：
- 代码仓库：[gitee.com/openharmony/bundlemanager_distributed_bundle_framework](https://gitee.com/openharmony/bundlemanager_distributed_bundle_framework)
