# 文档导航

## 新人阅读顺序

建议按以下顺序阅读，快速理解项目：

1. **[README](README.md)** - 快速入门
2. **[01_Overview](01_Overview.md)** - 项目定位与核心能力
3. **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构
4. **[03_Architecture](03_Architecture.md)** - 架构说明（含调用链）
5. **[04_NAPI](04_NAPI.md)** - 对外 API（JS/ETS）
6. **[06_GN_Targets](06_GN_Targets.md)** - 构建目标
7. **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物
8. **[08_Security_Review](08_Security_Review.md)** - 安全评审

## 完整目录

| 文件 | 标题 | 摘要 |
|------|------|------|
| README.md | 首页 | 覆盖范围、更新方式 |
| SUMMARY.md | 本导航 | 全站导航与阅读顺序 |
| [01_Overview](01_Overview.md) | 项目概览 | 定位、能力、运行环境 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构 | 模块职责、依赖方向 |
| [03_Architecture](03_Architecture.md) | 架构说明 | 组件图、数据流、时序图 |
| [04_NAPI](04_NAPI.md) | 对外 N-API | JS/ETS API 清单、调用链 |
| [05_Inner_API](05_Inner_API.md) | 内部 API | 模块接口、稳定性标注 |
| [06_GN_Targets](06_GN_Targets.md) | GN 目标 | targets 清单、依赖、产物 |
| [07_Build_Artifacts](07_Build_Artifacts.md) | 编译产物 | .so/.hap 文件、加载关系 |
| [08_Security_Review](08_Security_Review.md) | 安全风险 | 攻击面、信任边界、风险点 |
| [09_FAQ](09_FAQ.md) | 常见问题 | 构建、运行、调试问题 |

## 附录

| 文件 | 标题 | 摘要 |
|------|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 | 入口→核心逻辑调用图 |
