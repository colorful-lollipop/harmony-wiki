# Request 项目文档目录

## 新人阅读顺序（推荐）

1. **开始** [00_Overview.md](00_Overview.md) - 了解项目定位、核心能力、运行环境
2. **结构** [01_Directory_Structure.md](01_Directory_Structure.md) - 熟悉代码组织和模块职责
3. **架构** [02_Architecture.md](02_Architecture.md) - 理解架构设计、数据流、时序
4. **API** [03_NAPI_JS_API.md](03_NAPI_JS_API.md) - 学习对外的 JavaScript API
5. **内部** [04_Inner_API.md](04_Inner_API.md) - 了解内部 API 和模块依赖
6. **构建** [05_GN_Targets.md](05_GN_Targets.md) - 理解构建系统
7. **产物** [06_Build_Artifacts.md](06_Build_Artifacts.md) - 了解编译产物
8. **安全** [07_Security_Review.md](07_Security_Review.md) - 安全风险分析
9. **调试** [08_Troubleshooting.md](08_Troubleshooting.md) - 常见问题排查

---

## 完整文档列表

### 核心文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [README.md](README.md) | Wiki 概述 | 覆盖范围、更新方式 |
| [00_Overview.md](00_Overview.md) | 项目概览 | 定位、边界、核心能力、关键概念 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构 | 模块职责、依赖方向 |
| [02_Architecture.md](02_Architecture.md) | 架构说明 | 组件图、数据流、线程模型、时序图 |
| [03_NAPI_JS_API.md](03_NAPI_JS_API.md) | 对外 N-API | API 清单表、参数校验、错误码 |
| [04_Inner_API.md](04_Inner_API.md) | 内部 API | 模块接口、依赖关系、稳定性标注 |
| [05_GN_Targets.md](05_GN_Targets.md) | GN 目标 | targets 列表、类型、依赖、产物 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物 | .so/.a/.hap、安装路径、加载关系 |
| [07_Security_Review.md](07_Security_Review.md) | 安全风险 | 攻击面、信任边界、可被利用点 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见问题 | 构建问题、运行时问题、定位路径 |

### 附录文档

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 配置标志和宏 |

---

## 快速跳转

### 按主题

- **API 文档** → [03_NAPI_JS_API.md](03_NAPI_JS_API.md), [04_Inner_API.md](04_Inner_API.md)
- **架构文档** → [02_Architecture.md](02_Architecture.md), [01_Directory_Structure.md](01_Directory_Structure.md)
- **构建文档** → [05_GN_Targets.md](05_GN_Targets.md), [06_Build_Artifacts.md](06_Build_Artifacts.md)
- **安全文档** → [07_Security_Review.md](07_Security_Review.md)

### 按角色

- **新人入门** → 从 [00_Overview.md](00_Overview.md) 开始，按顺序阅读
- **应用开发者** → 重点关注 [03_NAPI_JS_API.md](03_NAPI_JS_API.md)
- **系统开发者** → 重点关注 [02_Architecture.md](02_Architecture.md), [04_Inner_API.md](04_Inner_API.md)
- **安全工程师** → 重点关注 [07_Security_Review.md](07_Security_Review.md)
- **构建工程师** → 重点关注 [05_GN_Targets.md](05_GN_Targets.md), [06_Build_Artifacts.md](06_Build_Artifacts.md)
