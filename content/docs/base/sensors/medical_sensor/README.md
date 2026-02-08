# Medical_Sensor Wiki

> 本 Wiki 为 medical_sensor 仓库生成，提供完整的工程文档。

## 生成信息
- **生成时间**: 2026-02-07
- **更新时间**: 2026-02-07
- **仓库路径**: `/base/sensors/medical_sensor`
- **组件名称**: `@ohos/medical_sensor`
- **组件版本**: 3.1
- **系统能力**: SystemCapability.Sensors.Medical_sensor
- **许可证**: Apache License 2.0

## 覆盖范围

### 已覆盖内容
- ✅ 项目概览和定位
- ✅ 目录结构和模块职责
- ✅ 架构说明（组件图、数据流、线程模型、关键时序）
- ✅ N-API 接口文档（JS API）
- ✅ 攻击面分析（外部输入、敏感操作、攻击路径）
- ✅ 内部 API 文档（模块接口、依赖方向）
- ✅ GN Targets 梳理
- ✅ 编译产物说明
- ✅ 安全风险评审（5+ 可利用点详细分析）

### 未覆盖内容
- ⚠️ 性能分析和优化建议
- ⚠️ 详细的单元测试和集成测试说明
- ⚠️ 硬件适配指南（厂商定制）

## 如何更新文档

1. **定期更新**: 当代码结构发生变化时，应同步更新相关 Wiki 页面
2. **证据追溯**: 所有结论必须有代码证据（文件路径+行号）
3. **测试引用**: 不得引用测试代码作为业务证据
4. **术语统一**: 确保整个 Wiki 中术语使用一致

## 快速导航

### 🎯 按角色导航

**新人学习路线**：[阅读路线图](SUMMARY.md#-新人学习路线推荐-3-小时)
- [项目概览](00_Overview.md) → [目录结构](01_Directory_Structure.md) → [架构说明](02_Architecture.md) → [N-API 参考](03_N-API_Reference.md)

**安全研究路线**：[阅读路线图](SUMMARY.md#-安全研究路线推荐-4-小时)
- [攻击面分析](04_AttackSurface.md) → [安全风险评估](08_Security_Review.md) → [架构说明](02_Architecture.md)

### 📑 完整文档索引

| 编号 | 文档 | 说明 |
|------|------|------|
| 00 | [项目概览](00_Overview.md) | 项目定位、核心能力、运行环境 |
| 01 | [目录结构](01_Directory_Structure.md) | 模块职责说明 |
| 02 | [架构说明](02_Architecture.md) | 组件图、数据流、时序图 |
| 03 | [N-API 参考](03_N-API_Reference.md) | JS API 完整文档 |
| 04 | [攻击面分析](04_AttackSurface.md) | 外部输入、敏感操作、攻击路径 |
| 05 | [内部 API](05_Internal_API.md) | 模块接口和依赖关系 |
| 06 | [GN Targets](06_GN_Targets.md) | 构建目标和依赖图 |
| 07 | [编译产物](07_Build_Artifacts.md) | 输出文件和安装路径 |
| 08 | [安全风险评估](08_Security_Review.md) | 漏洞分析、修复建议 |
| 09 | [常见问题](09_Troubleshooting.md) | 构建、运行、调试问题 |

### 📁 工作文档
- `_work/ASSESSMENT.md` - 项目评估报告
- `_work/NOTES.md` - 代码证据汇总
- `_work/PLAN.md` - 任务进度追踪

---

## 文档质量标准

每篇文档遵循以下标准：
- ✅ **目的**: 清晰说明文档目标和适用范围
- ✅ **关键结论**: 基于**代码证据**（路径+符号+行号）
- ✅ **相关链接**: 指向相关 Wiki 页面
- ✅ **中文为主**: 除非另有说明
- ✅ **无测试引用**: 不使用测试代码作为业务逻辑证据
