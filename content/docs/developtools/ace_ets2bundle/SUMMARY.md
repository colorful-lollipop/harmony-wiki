# Wiki 导航索引

本文档提供 Ace ETS2Bundle 工程 Wiki 的完整导航。

## 快速入口

| 章节 | 描述 | 推荐阅读 |
|------|------|----------|
| [README](README.md) | 文档概述与更新方式 | 所有人必读 |
| [01_Overview](01_Overview.md) | 项目定位与核心能力 | 新人入门 |
| [02_Architecture](02_Architecture.md) | 系统架构与组件图 | 开发者必读 |

## 完整阅读路径

### 新人入门路径

```
1. [README](README.md)          → 了解文档结构
2. [01_Overview](01_Overview.md) → 项目定位
3. [03_Directory_Structure](03_Directory_Structure.md) → 目录结构
4. [10_Troubleshooting](10_Troubleshooting.md) → 常见问题
```

### 开发者深入路径

```
1. [02_Architecture](02_Architecture.md) → 整体架构
2. [04_Compiler_Core](04_Compiler_Core.md) → 编译器核心
3. [05_ArkUI_Plugins](05_ArkUI_Plugins.md) → 插件系统
4. [06_Koala_Wrapper](06_Koala_Wrapper.md) → Native 模块
5. [07_GN_Targets](07_GN_Targets.md) → 构建配置
6. [08_Build_Artifacts](08_Build_Artifacts.md) → 产物说明
```

### 安全相关路径

```
1. [09_Security_Review](09_Security_Review.md) → 安全风险评审
   └── 攻击面清单 → 信任边界 → 风险点列表 → 修复建议
```

## 模块索引

### 编译器模块

| 模块 | 路径 | 描述 |
|------|------|------|
| 入口 | [compiler/main.js](../compiler/main.js) | 编译主入口 |
| 类型检查 | [compiler/src/ets_checker.ts](../compiler/src/ets_checker.ts) | ETS 类型检查器 |
| 字节码生成 | [compiler/src/gen_abc_plugin.ts](../compiler/src/gen_abc_plugin.ts) | ABC 插件 |
| 组件处理 | [compiler/src/process_component_*.ts](../compiler/src/) | 组件转换 |
| UI 语法 | [compiler/src/process_ui_syntax.ts](../compiler/src/process_ui_syntax.ts) | UI 语法处理 |

### ArkUI 插件模块

| 模块 | 路径 | 描述 |
|------|------|------|
| UI 插件 | [arkui-plugins/ui-plugins/](../arkui-plugins/ui-plugins/) | UI 转换核心 |
| 语法插件 | [arkui-plugins/ui-syntax-plugins/](../arkui-plugins/ui-syntax-plugins/) | 语法转换 |
| 互操作 | [arkui-plugins/interop-plugins/](../arkui-plugins/interop-plugins/) | JS/TS 互操作 |

### Koala 模块

| 模块 | 路径 | 描述 |
|------|------|------|
| Native | [koala-wrapper/native/](../koala-wrapper/native/) | C++ 实现 |
| N-API | [koala-wrapper/koalaui/interop/src/cpp/napi/](../koala-wrapper/koalaui/interop/src/cpp/napi/) | N-API 绑定 |
| 互操作 | [koala-wrapper/koalaui/interop/](../koala-wrapper/koalaui/interop/) | JS/Native 桥接 |

## 附录索引

| 文档 | 描述 |
|------|------|
| [A_API_Reference](appendix/A_API_Reference.md) | API 参考手册 |
| [B_Callgraphs](appendix/B_Callgraphs.md) | 关键调用链图 |
| [C_Config_Flags](appendix/C_Config_Flags.md) | 配置开关说明 |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本 |

## 贡献指南

如需贡献文档，请：

1. Fork 本仓库
2. 修改对应 wiki 文件
3. 提交 Pull Request
4. 等待 Code Review
