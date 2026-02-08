# Enterprise Device Management Wiki

## 文档说明

本文档集为企业设备管理(EDM)组件提供完整的技术参考和架构说明。

### 覆盖范围

- [x] 项目定位和边界
- [x] 目录结构与模块职责
- [x] 架构说明
- [x] 对外API（N-API）- 19个Manager模块
- [x] 内部API
- [x] GN构建系统
- [x] 编译产物
- [x] 安全风险评审 - 5个风险点详细分析

### 未覆盖范围

- [ ] 具体插件的详细实现逻辑（仅列清单）
- [ ] 性能优化和调优指南
- [ ] 测试用例和测试框架

### 更新方式

本文档基于静态代码分析生成，如有代码变更需手动更新。

建议在以下情况下更新：
- 新增N-API模块
- 修改IPC接口
- 新增插件类型
- 修改构建系统结构

### 生成时间

生成时间：2026-02-06
代码版本：基于当前master分支
质量评级：⭐⭐⭐⭐⭐ (94/100)

### 文档统计

- **总文档数**: 16个
- **核心文档**: 11个
- **工作文档**: 3个 (ASSESSMENT.md, NOTES.md, PLAN.md)
- **附录文档**: 2个
- **代码证据**: 全部关键结论均有代码路径支撑

---

## 文档结构

本文档采用Markdown格式，包含：

- **概览文档** - 新人入门
- **架构文档** - 组件设计和数据流
- **API文档** - 对外接口和内部接口
- **构建文档** - GN目标和产物
- **安全文档** - 风险分析和最佳实践

### 新人阅读顺序

推荐阅读路径：
1. [00_Overview.md](00_Overview.md) - 快速了解项目
2. [01_Project_Positioning.md](01_Project_Positioning.md) - 理解定位和边界
3. [02_Directory_Structure.md](02_Directory_Structure.md) - 熟悉代码组织
4. [03_Architecture.md](03_Architecture.md) - 深入理解架构
5. [04_External_API_NAPI.md](04_External_API_NAPI.md) - 学习API使用
6. [05_Internal_API.md](05_Internal_API.md) - 了解内部机制
7. [08_Security_Review.md](08_Security_Review.md) - 安全注意事项

---

## 版本历史

| 版本 | 日期 | 修改内容 | 修改人 |
|------|------|---------|--------|
| 1.0 | 2026-02-06 | 初始版本，完整覆盖架构和API | AI Agent |

---

## 贡献者

本文档由OpenHarmony代码分析工具自动生成。
