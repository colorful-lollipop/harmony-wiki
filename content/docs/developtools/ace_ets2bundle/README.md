# Ace ETS2Bundle 工程 Wiki

## 文档概述

| 项目 | 说明 |
|------|------|
| **项目名称** | developtools_ace_ets2bundle |
| **版本** | 3.1 |
| **License** | Apache License 2.0 |
| **子系统** | developtools |
| **描述** | 提供声明式范式语法编译转换、语法验证、友好语法错误提示能力 |

## 覆盖范围

本 Wiki 文档涵盖以下模块的完整技术文档：

### 核心编译器模块
- **compiler/src/** - ETS 编译器核心源码（TypeScript）
- **compiler/components/** - 组件定义
- **compiler/form_components/** - 卡片组件
- **compiler/server/** - 编译器服务
- **compiler/codegen/** - 代码生成

### ArkUI 插件系统
- **arkui-plugins/ui-plugins/** - UI 插件核心
- **arkui-plugins/ui-syntax-plugins/** - 语法转换插件
- **arkui-plugins/interop-plugins/** - 互操作插件

### Koala 运行时包装器
- **koala-wrapper/native/** - Native C++ 实现
- **koala-wrapper/koalaui/** - Koala UI 互操作层

## 不包含范围

以下内容不在本 Wiki 覆盖范围内：

- **测试代码** - test/ tests/ unittest/ *_test.*
- **第三方依赖** - node_modules/
- **构建产物** - out/ build/
- **Git 元数据** - .git/ .gitee/

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航
├── 01_Overview.md              # 项目概览
├── 02_Architecture.md           # 架构说明
├── 03_Directory_Structure.md    # 目录结构
├── 04_Compiler_Core.md          # 编译器核心
├── 05_ArkUI_Plugins.md          # ArkUI 插件系统
├── 06_Koala_Wrapper.md          # Koala 包装器
├── 07_GN_Targets.md             # GN 构建目标
├── 08_Build_Artifacts.md        # 编译产物
├── 09_Security_Review.md        # 安全风险评审
├── 10_Troubleshooting.md        # 常见问题
└── appendix/
    ├── A_API_Reference.md      # API 参考
    ├── B_Callgraphs.md         # 关键调用链
    └── C_Config_Flags.md       # 配置开关
```

## 更新方式

当代码发生变更时，需同步更新相应文档：

1. **新增功能** - 在对应模块文档中添加说明
2. **修改 API** - 更新 API 清单表和调用链
3. **变更构建** - 更新 GN Targets 和产物文档
4. **安全修复** - 在安全评审文档中记录

## 文档维护

- **最后更新**: 2026-02-06
- **负责团队**: OpenHarmony developtools
- **问题反馈**: 提交 Issue 到 OpenHarmony 仓库

## 相关链接

- [OpenHarmony 主仓库](https://gitee.com/openharmony)
- [ArkUI 官方文档](https://developer.harmonyos.com)
- [ETS 编译器源码](../compiler/src/)
- [GN 构建配置](../BUILD.gn)
