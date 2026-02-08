# KV Store Wiki 文档

## 概述

本 Wiki 是 OpenHarmony `distributeddatamgr/kv_store` 仓库的工程文档，旨在帮助开发者快速理解项目架构、API 使用、编译构建和安全机制。

## 覆盖范围

### 包含内容

- **项目定位与架构**：模块职责、目录结构、组件图
- **对外 API**：N-API（JS API）接口清单、使用示例
- **内部架构**：Inner API、模块依赖、线程模型
- **编译构建**：GN Targets、编译产物、依赖关系
- **安全风险**：攻击面分析、信任边界、修复建议
- **故障排查**：常见问题、调试方法

### 不包含内容

- 测试代码分析（`test/`, `*_test.*`, `fuzztest/` 等）
- 第三方依赖内部实现（SQLite, OpenSSL 等）
- 完整代码注释或行级解释

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── 01_Overview.md        # 项目概述
├── 02_Architecture.md    # 架构设计
├── 03_CodeMap.md         # 代码地图
├── 04_NAPI_Reference.md  # N-API 接口参考
├── 05_Inner_API.md       # Inner API 参考
├── 06_Build_GN.md        # GN 构建指南
├── 07_AttackSurface.md   # 攻击面分析
├── 08_Security_Review.md # 安全风险评审
├── 09_Troubleshooting.md # 故障排查
└── appendix/
    ├── Callgraphs.md     # 关键调用链
    └── Config_Flags.md   # 编译配置开关
```

## 阅读建议

**新人入门路线**：

1. `01_Overview.md` - 了解项目定位和核心能力
2. `03_CodeMap.md` - 掌握代码结构，快速定位文件
3. `04_NAPI_Reference.md` - 学习 API 使用方法
4. `06_Build_GN.md` - 理解构建配置

**安全研究路线**：

1. `07_AttackSurface.md` - 识别所有外部入口和攻击面
2. `08_Security_Review.md` - 深入分析安全风险和修复建议
3. `02_Architecture.md` - 理解信任边界
4. `03_CodeMap.md` - 定位安全相关代码

**深入开发顺序**：

1. `02_Architecture.md` - 理解模块划分
2. `03_CodeMap.md` - 代码文件导航
3. `05_Inner_API.md` - 了解内部接口
4. `08_Security_Review.md` - 关注安全要点

## 更新方式

当代码发生以下变更时，需同步更新 Wiki：

| 变更类型 | 需更新文档 |
|---------|-----------|
| 新增/删除 JS API | `04_NAPI_Reference.md` |
| 新增/删除代码文件 | `03_CodeMap.md` |
| 修改模块依赖 | `02_Architecture.md`, `05_Inner_API.md` |
| 新增/删除 GN Target | `06_Build_GN.md` |
| 修改外部暴露接口 | `07_AttackSurface.md` |
| 修改安全机制 | `08_Security_Review.md` |

**更新步骤**：

1. 修改对应 `.md` 文件
2. 验证链接（`SUMMARY.md` 中的锚点）
3. 提交变更（随代码 PR 一起）

## 文档版本

- **生成时间**：2024-02-06
- **对应版本**：OpenHarmony 5.0+
- **最后更新**：2024-02-06

## 贡献指南

欢迎完善和修正 Wiki 内容：

1. 遵循文档结构
2. 关键结论需提供代码证据（文件路径 + 符号名）
3. 保持术语一致性
4. 测试验证链接有效性
