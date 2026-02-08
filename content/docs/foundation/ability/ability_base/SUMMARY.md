# ability_base Wiki 导航

本页面提供 ability_base Wiki 的完整导航和阅读路线。

---

## 快速导航

### 📚 按主题浏览

| 主题 | 文档 | 说明 |
|------|------|------|
| 🏠 **项目概览** | [index.md](index.md) | 项目定位、边界、核心能力、运行环境 |
| 📁 **目录结构** | [01_Directory_Structure.md](01_Directory_Structure.md) | 目录组织、模块职责、文件分类 |
| 🏗️ **架构设计** | [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型、时序图 |
| 🔌 **Native C++ API** | [03_Native_CPP_API.md](03_Native_CPP_API.md) | Want、Configuration、URI、Base 类型接口 |
| 🔧 **C NDK API** | [04_C_NDK_API.md](04_C_NDK_API.md) | OH_AbilityBase_* C 函数接口 |
| ⚙️ **GN 构建** | [05_GN_Build.md](05_GN_Build.md) | Targets、依赖、编译产物、配置 |
| 🔒 **安全评审** | [06_Security_Review.md](06_Security_Review.md) | 攻击面、信任边界、风险评估 |
| 📊 **调用链** | [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用路径（入口→核心） |
| 🎛️ **配置标志** | [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏、feature flags |

---

## 🚀 新人阅读路线

### 路线 1：快速入门（约 1 小时）
**目标**：理解 ability_base 是什么、如何使用

1. **[index.md](index.md)** - 了解项目定位（15 分钟）
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 熟悉代码组织（20 分钟）
3. **[03_Native_CPP_API.md](03_Native_CPP_API.md)** - 学习核心 Want API（25 分钟）

### 路线 2：架构理解（约 2 小时）
**目标**：深入理解内部设计和实现细节

1. **[index.md](index.md)** - 项目定位（15 分钟）
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 代码组织（15 分钟）
3. **[02_Architecture.md](02_Architecture.md)** - 架构和数据流（30 分钟）
4. **[03_Native_CPP_API.md](03_Native_CPP_API.md)** - Native API（45 分钟）
5. **[05_GN_Build.md](05_GN_Build.md)** - 构建系统（15 分钟）

### 路线 3：完整学习（约 3.5 小时）
**目标**：全面掌握所有内容

**按顺序阅读所有文档**，从 [index.md](index.md) 到 [06_Security_Review.md](06_Security_Review.md)，最后查看附录。

---

## 📖 按角色导航

### 👨‍💻 应用开发者
**关注**：如何使用 ability_base API 启动 Ability

**推荐阅读**：
1. [index.md](index.md) - 了解 Want 的作用
2. [03_Native_CPP_API.md](03_Native_CPP_API.md) - 学习 Want API
3. [06_Security_Review.md](06_Security_Review.md) - 了解安全注意事项

### 👨‍🔧 系统开发者
**关注**：如何扩展或维护 ability_base

**推荐阅读**：
1. [index.md](index.md) - 整体定位
2. [01_Directory_Structure.md](01_Directory_Structure.md) - 代码组织
3. [02_Architecture.md](02_Architecture.md) - 内部架构
4. [05_GN_Build.md](05_GN_Build.md) - 构建系统
5. [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链
6. [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置选项

### 🔒 安全审计员
**关注**：安全风险和攻击面

**推荐阅读**：
1. [index.md](index.md) - 理解组件作用
2. [02_Architecture.md](02_Architecture.md) - 数据流和信任边界
3. [06_Security_Review.md](06_Security_Review.md) - 详细安全评审

### 🧪 测试工程师
**关注**：API 行为和边界条件

**推荐阅读**：
1. [03_Native_CPP_API.md](03_Native_CPP_API.md) - API 规范
2. [04_C_NDK_API.md](04_C_NDK_API.md) - C API 规范
3. [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用路径

---

## 🗂️ 按模块导航

### Want 模块
**核心模块**：组件启动参数

📄 **相关文档**：
- [index.md](index.md) - Want 模块职责
- [01_Directory_Structure.md](01_Directory_Structure.md) - Want 目录结构
- [02_Architecture.md](02_Architecture.md) - Want 数据流
- [03_Native_CPP_API.md](03_Native_CPP_API.md) - Want API 详细说明
- [06_Security_Review.md](06_Security_Review.md) - Want 安全考虑

### Configuration 模块
**核心模块**：系统环境参数

📄 **相关文档**：
- [index.md](index.md) - Configuration 模块职责
- [01_Directory_Structure.md](01_Directory_Structure.md) - Configuration 目录结构
- [02_Architecture.md](02_Architecture.md) - Configuration 线程模型
- [03_Native_CPP_API.md](03_Native_CPP_API.md) - Configuration API

### URI 模块
**核心模块**：统一资源标识符

📄 **相关文档**：
- [index.md](index.md) - URI 模块职责
- [01_Directory_Structure.md](01_Directory_Structure.md) - URI 目录结构
- [02_Architecture.md](02_Architecture.md) - URI 解析流程
- [03_Native_CPP_API.md](03_Native_CPP_API.md) - Uri 类 API

### Base 模块
**核心模块**：基础数据类型

📄 **相关文档**：
- [index.md](index.md) - Base 模块职责
- [01_Directory_Structure.md](01_Directory_Structure.md) - Base 目录结构
- [03_Native_CPP_API.md](03_Native_CPP_API.md) - 类型包装器 API

### 扩展模块
- **ViewData**：自动填充视图数据
- **SessionInfo**：UI 会话信息
- **ExtractorTool**：ZIP 资源提取

📄 **相关文档**：
- [01_Directory_Structure.md](01_Directory_Structure.md) - 扩展模块结构
- [03_Native_CPP_API.md](03_Native_CPP_API.md) - 扩展模块 API

---

## 📊 文档关系图

```
README.md (本文档)
    │
    ├─→ index.md (项目概览)
    │         │
    │         ├─→ 01_Directory_Structure.md (目录结构)
    │         │         │
    │         │         ├─→ 02_Architecture.md (架构)
    │         │         │         │
    │         │         │         ├─→ 03_Native_CPP_API.md (Native API)
    │         │         │         │         │
    │         │         │         │         └─→ 04_C_NDK_API.md (C NDK)
    │         │         │         │
    │         │         │         └─→ 05_GN_Build.md (构建)
    │         │         │
    │         │         └─→ 06_Security_Review.md (安全)
    │         │
    │         └─→ appendix/ (附录)
    │                   │
    │                   ├─→ Callgraphs.md (调用链)
    │                   └─→ Config_Flags.md (配置)
    │
    └─→ _work/ (工作区)
              │
              ├─→ NOTES.md (笔记)
              └─→ PLAN.md (计划)
```

---

## 🔍 搜索指南

### 按关键词查找

| 想找什么 | 去哪里 |
|----------|--------|
| Want API | [03_Native_CPP_API.md](03_Native_CPP_API.md) |
| C API | [04_C_NDK_API.md](04_C_NDK_API.md) |
| GN 构建 | [05_GN_Build.md](05_GN_Build.md) |
| 安全问题 | [06_Security_Review.md](06_Security_Review.md) |
| 类关系 | [02_Architecture.md](02_Architecture.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| Feature Flags | [appendix/Config_Flags.md](appendix/Config_Flags.md) |
| 调用链 | [appendix/Callgraphs.md](appendix/Callgraphs.md) |

### 常见问题快速链接

❓ **Want 如何传递参数？**
→ [03_Native_CPP_API.md](03_Native_CPP_API.md#want-参数操作)

❓ **Configuration 如何获取语言设置？**
→ [03_Native_CPP_API.md](03_Native_CPP_API.md#configuration-配置访问)

❓ **URI 如何解析？**
→ [03_Native_CPP_API.md](03_Native_CPP_API.md#uri-解析与访问)

❓ **如何编译 ability_base？**
→ [05_GN_Build.md](05_GN_Build.md#构建命令)

❓ **有哪些安全风险？**
→ [06_Security_Review.md](06_Security_Review.md#风险清单)

❓ **依赖哪些系统组件？**
→ [index.md](index.md#依赖关系)

---

## 📋 文档元数据

| 属性 | 值 |
|------|------|
| **文档版本** | v1.0 |
| **生成时间** | 2026-02-06 11:30 |
| **组件版本** | 3.1 |
| **系统能力** | SystemCapability.Ability.AbilityBase |
| **总页数** | 11 |
| **总字数** | ~50,000 |
| **代码证据** | 100+ 文件引用 |

---

## 🔄 更新历史

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| v1.0 | 2026-02-06 | 初始版本，完整文档 |

---

## 📞 获取帮助

如需进一步了解，请：
- 阅读相关文档页面
- 查看 [README.md](README.md) 了解文档覆盖范围
- 查阅 OpenHarmony 官方文档
- 在相关仓库提交 Issue

---

**返回首页**: [index.md](index.md)
