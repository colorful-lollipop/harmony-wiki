# Preferences 模块 Wiki 文档

## 文档概述

本文档是 OpenHarmony **DistributedDataMgr - Preferences** 模块的工程 Wiki，涵盖模块架构、API 参考、构建配置、安全评审等完整技术文档。

## 模块定位

**首选项（Preferences）** 是 OpenHarmony 分布式数据管理子系统中的轻量级键值对存储模块，主要用于：

- 存储应用配置和用户设置
- 提供快速的数据访问（内存缓存 + 文件持久化）
- 支持数据变化监听

## 文档范围

### 包含内容

| 文档 | 说明 |
|------|------|
| [概览](./00_Overview.md) | 模块定位、能力边界、运行环境 |
| [架构设计](./01_Architecture.md) | 组件图、数据流、线程模型 |
| [API 参考](./02_API_Reference.md) | N-API、NDK、Inner API 完整清单 |
| [构建配置](./03_Build.md) | GN Targets、编译产物、依赖关系 |
| [安全评审](./04_Security_Review.md) | 攻击面、风险分析、修复建议 |
| [故障排查](./05_Troubleshooting.md) | 常见问题、定位方法 |

### 不包含内容

- 测试代码相关文档（见 `test/` 目录）
- 分布式同步深度实现细节
- 第三方依赖内部实现

## 阅读路线

建议新人按以下顺序阅读：

```
1. 00_Overview.md      → 理解模块定位
2. 01_Architecture.md   → 掌握整体架构
3. 02_API_Reference.md → 学习如何使用 API
4. 03_Build.md         → 了解构建配置
```

## 代码证据说明

本文档所有关键结论均基于代码证据，包含：

- 文件路径（必要时刻注行号）
- 关键符号名（函数/类/宏）
- 代码片段或调用链描述

示例：
> 模块入口注册于 `frameworks/js/napi/preferences/src/entry_point.cpp:49`
> 
> ```cpp
> napi_module_register(&_module);
> ```

## 版本信息

| 项目 | 值 |
|------|-----|
| 模块版本 | 3.1.0（来自 bundle.json） |
| OpenHarmony 版本 | 标准版（standard） |
| 文档生成时间 | 2025-02-06 |
| 负责人生成 | Sisyphus AI Agent |

## 贡献指南

### 如何更新文档

1. 在对应文档中查找对应章节
2. 修改完成后运行文档检查工具（待补充）
3. 提交 PR 至 OpenHarmony 仓库

### 注意事项

- 禁止在文档中引用测试代码作为业务证据
- API 变更需同步更新 API 参考表
- 安全评审需标注代码证据路径

## 联系方式

- 仓库地址：https://gitee.com/openharmony/distributeddatamgr_preferences
- 组件名：preferences
- 子系统：distributeddatamgr

---

## 目录

### 核心文档

- [概览](./00_Overview.md)
- [架构设计](./01_Architecture.md)
- [API 参考](./02_API_Reference.md)
- [构建配置](./03_Build.md)
- [安全评审](./04_Security_Review.md)
- [故障排查](./05_Troubleshooting.md)

### 附录

- [调用链图谱](./appendix/Callgraphs.md)（待补充）
