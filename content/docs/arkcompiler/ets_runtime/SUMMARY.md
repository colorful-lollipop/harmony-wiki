# 文档导航

本文档为 ArkCompiler ETS Runtime 的完整工程 Wiki，建议按以下顺序阅读。

## 新人阅读路线

1. **[项目概览](01_Project_Overview.md)** → 快速了解项目定位和核心能力
2. **[目录结构](02_Directory_Structure.md)** → 熟悉模块职责划分
3. **[架构说明](03_Architecture.md)** → 理解组件关系和数据流
4. **[N-API 参考](04_NAPI_Reference.md)** → 查询对外接口
5. **[内部 API](05_Inner_API.md)** → 了解模块间接口
6. **[构建系统](06_Build_System.md)** → 掌握编译产物和依赖
7. **[安全评审](07_Security_Review.md)** → 识别安全风险

## 详细文档列表

### 核心文档

| 文档 | 说明 | 阅读优先级 |
|------|------|-----------|
| [项目概览](01_Project_Overview.md) | 项目定位、边界、运行环境 | 高 |
| [目录结构](02_Directory_Structure.md) | 模块职责说明 | 高 |
| [架构说明](03_Architecture.md) | 组件图、时序图、数据流 | 高 |
| [N-API 参考](04_NAPI_Reference.md) | 对外 C++ API 接口 | 高 |
| [内部 API](05_Inner_API.md) | 模块间 Inner API | 中 |
| [构建系统](06_Build_System.md) | GN Targets 与产物 | 中 |
| [安全评审](07_Security_Review.md) | 安全风险分析 | 中 |

### 附录

| 文档 | 说明 |
|------|------|
| [附录：关键调用链](appendix/Callgraphs.md) | 入口→核心逻辑调用链 |
| [附录：配置开关](appendix/Config_Flags.md) | 关键宏与 Feature Flags |
| [常见问题与定位](08_Troubleshooting.md) | 构建/运行/调试问题 |

### 其他

| 文档 | 说明 |
|------|------|
| [项目首页](index.md) | 项目概览与快速开始 |

## 快速跳转

### 按功能查找

- **运行时相关** → [架构说明](03_Architecture.md)
- **API 接口** → [N-API 参考](04_NAPI_Reference.md)
- **编译构建** → [构建系统](06_Build_System.md)
- **安全问题** → [安全评审](07_Security_Review.md)

### 按模块查找

- **N-API 模块** → `ecmascript/napi/`
- **JS API 模块** → `ecmascript/js_api/`
- **容器模块** → `ecmascript/containers/`
- **系统集成** → `ecmascript/ohos/`
