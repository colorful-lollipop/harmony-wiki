# 阅读路线建议

根据你的角色和目的，选择以下阅读路线：

## 路线一：快速了解（5分钟）

适合人群：初次接触 RE2 的开发者

**阅读顺序**:
1. [README.md](./README.md) - 库概览和关键信息
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解谁在用这个库
3. 结论：RE2 是一个零 Patch 的标准库，主要用于 gRPC 的 URI 匹配

## 路线二：维护/升级（15分钟）

适合人群：需要维护或升级 RE2 的开发者

**阅读顺序**:
1. [README.md](./README.md) - 库概览
2. [02_Patches.md](./02_Patches.md) - 确认无功能性 Patch（重要！）
3. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 配置细节
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 检查依赖模块，规划测试

**关键要点**:
- RE2 无功能性 Patch，升级时只需关注 BUILD.gn 配置
- 需同步验证 gRPC 功能
- 注意 libre2.map 符号版本管理

## 路线三：深度分析（30分钟）

适合人群：需要全面理解 RE2 在 OH 中集成方式的架构师

**阅读顺序**:
1. [README.md](./README.md) - 库概览
2. [01_Overview.md](./01_Overview.md) - 技术背景
3. [02_Patches.md](./02_Patches.md) - Patch 策略分析
4. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统深度解析
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系全图

## 路线四：故障排查

适合人群：遇到 RE2 相关问题的开发者

**按问题类型选择**:

| 问题类型 | 查看文档 |
|----------|----------|
| 编译失败 | [03_Build_Integration.md](./03_Build_Integration.md) |
| 符号未找到 | [03_Build_Integration.md#符号导出](./03_Build_Integration.md#符号导出) |
| 正则匹配行为异常 | [01_Overview.md](./01_Overview.md) 查看 RE2 与 PCRE 差异 |
| 性能问题 | [01_Overview.md](./01_Overview.md) 查看 RE2 线性时间保证 |

## 文档清单

| 文档 | 阅读时间 | 适合场景 |
|------|----------|----------|
| [README.md](./README.md) | 3 min | 所有场景 |
| [01_Overview.md](./01_Overview.md) | 10 min | 技术背景了解 |
| [02_Patches.md](./02_Patches.md) | 5 min | Patch 策略、升级准备 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 10 min | 构建问题、编译配置 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 10 min | 依赖分析、影响评估 |

## 重要提示

> **RE2 是一个特殊的零 Patch 库**
> 
> 与大多数 OH 第三方库不同，RE2 未应用任何功能性 Patch。这意味着：
> 1. 升级时可以更直接地跟随上游
> 2. 不需要维护 Patch 兼容性
> 3. 行为和 Bug 与上游完全一致
> 
> 详见 [02_Patches.md](./02_Patches.md)

---

**文档导航**: [README.md](./README.md) | [01_Overview.md](./01_Overview.md) | [02_Patches.md](./02_Patches.md) | [03_Build_Integration.md](./03_Build_Integration.md) | [04_Usage_in_OH.md](./04_Usage_in_OH.md)
