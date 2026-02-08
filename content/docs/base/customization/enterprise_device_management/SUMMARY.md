# Enterprise Device Management Wiki - 导航

## 概览
- [00_Overview.md](00_Overview.md) - 项目概览和快速入门

## 核心文档
### 定位与边界
- [01_Project_Positioning.md](01_Project_Positioning.md) - 项目定位、边界、核心能力、运行环境、关键概念

### 代码组织
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构、模块职责、依赖关系

### 架构设计
- [03_Architecture.md](03_Architecture.md) - 组件图、数据流、线程模型、关键时序

## API文档
### 对外API（N-API）
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - JS API清单、参数、错误码、权限、同步异步模式

### 内部API
- [05_Internal_API.md](05_Internal_API.md) - 模块接口、依赖方向、稳定性、可替换点

## 构建与产物
### GN构建系统
- [06_GN_Targets.md](06_GN_Targets.md) - targets列表、类型、依赖、产物、开关

### 编译产物
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 产物清单、安装路径、运行时加载关系

## 安全与运维
### 安全风险评审
- [08_Security_Review.md](08_Security_Review.md) - 攻击面、信任边界、可被利用点、修复建议

### 常见问题
- [09_Troubleshooting.md](09_Troubleshooting.md) - 构建问题、运行问题、调试路径

## 附录
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 关键宏和feature flags

---

## 新人推荐阅读路径

### 第一天：了解项目
1. [00_Overview.md](00_Overview.md) - 30分钟
2. [01_Project_Positioning.md](01_Project_Positioning.md) - 30分钟
3. [02_Directory_Structure.md](02_Directory_Structure.md) - 1小时

### 第二天：深入架构
1. [03_Architecture.md](03_Architecture.md) - 2小时
2. [04_External_API_NAPI.md](04_External_API_NAPI.md) - 2小时
3. [05_Internal_API.md](05_Internal_API.md) - 2小时

### 第三天：构建和安全
1. [06_GN_Targets.md](06_GN_Targets.md) - 1小时
2. [07_Build_Artifacts.md](07_Build_Artifacts.md) - 30分钟
3. [08_Security_Review.md](08_Security_Review.md) - 2小时

---

## 快速导航

| 我想了解... | 推荐文档 |
|-----------|----------|
| EDM是什么，做什么用 | [00_Overview.md](00_Overview.md) |
| 代码在哪，怎么组织 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 如何使用JS API | [04_External_API_NAPI.md](04_External_API_NAPI.md) |
| 插件机制怎么工作 | [03_Architecture.md](03_Architecture.md) + [05_Internal_API.md](05_Internal_API.md) |
| 如何编译和构建 | [06_GN_Targets.md](06_GN_Targets.md) + [07_Build_Artifacts.md](07_Build_Artifacts.md) |
| 安全注意事项 | [08_Security_Review.md](08_Security_Review.md) |
| 调试和排错 | [09_Troubleshooting.md](09_Troubleshooting.md) |
| 完整的调用链 | [appendix/Callgraphs.md](appendix/Callgraphs.md) |
| 编译开关和宏 | [appendix/Config_Flags.md](appendix/Config_Flags.md) |

---

## 文档约定

- 所有路径相对于仓库根目录：`/base/customization/enterprise_device_management/`
- 文件引用格式：`path/to/file:line`
- 类型定义格式：`Class`（类）、`function()`（函数）、`CONSTANT`（常量）
- 方向指示：↓ 调用、→ 数据流、⇔ 相互依赖
