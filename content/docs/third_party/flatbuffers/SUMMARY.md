# 阅读指南

本文档旨在帮助开发者快速理解 FlatBuffers 在 OpenHarmony 系统中的集成方式、Patch 详情以及使用场景。

## 你属于哪类读者？

### 我想了解 FlatBuffers 是什么

建议阅读顺序：
1. **[概述](01_Overview.md)** - 了解 FlatBuffers 基础概念和 OH 定位
2. **[构建适配](03_Build_Integration.md)** - 了解如何在 OH 中使用

### 我需要分析或修改 Patch

建议阅读顺序：
1. **[概述](01_Overview.md)** - 快速了解背景
2. **[Patch 详解](02_Patches.md)** - 详细分析每个 Patch 的修改内容和目的

### 我想了解 OH 如何使用 FlatBuffers

建议阅读顺序：
1. **[OH 使用场景](04_Usage_in_OH.md)** - 查看依赖关系和使用方式
2. **[Patch 详解](02_Patches.md)** - 了解相关适配

### 我需要维护或升级 FlatBuffers

建议完整阅读所有文档：
1. **[概述](01_Overview.md)** - 了解整体架构
2. **[Patch 详解](02_Patches.md)** - 记录所有 OH 修改
3. **[构建适配](03_Build_Integration.md)** - 了解构建配置
4. **[OH 使用场景](04_Usage_in_OH.md)** - 评估升级影响

## 文档结构

```
wiki/
├── README.md              # 快速入口和导航
├── SUMMARY.md             # 阅读指南（本文档）
├── _work/
│   ├── ASSESSMENT.md     # 原始评估数据
│   ├── NOTES.md          # 分析过程记录
│   └── PLAN.md           # 任务进度
├── 01_Overview.md        # 库概述和 OH 定位
├── 02_Patches.md         # Patch 详细分析
├── 03_Build_Integration.md # 构建系统适配
└── 04_Usage_in_OH.md     # OH 使用场景和依赖关系
```

## 关键信息速查

### Patch 清单速查

| Patch | 修改文件数 | 主要修改 |
|-------|-----------|---------|
| build_grpc_with_cxx14.patch | 1 | C++14 标准指定 |
| boringssl.patch | 2 | 链接修复、变量修复 |

### 主要依赖模块

| 模块 | 重要性 | 使用场景 |
|-----|-------|---------|
| MindSpore Lite | 高 | AI 模型加载 |
| NNRT | 高 | 模型数据序列化 |

### 与上游差异速查

| 差异项 | 状态 |
|-------|------|
| 构建系统 | CMake/Bazel → GN |
| 仓颉语言支持 | OH 新增 |
| gRPC 集成 | 包含修复 Patch |

## 常见问题

### Q: FlatBuffers 和 Protobuf 有什么区别？

FlatBuffers 强调零拷贝访问，数据可在序列化后直接读取；Protobuf 需要先解析为内存对象。对于 AI 模型加载等场景，FlatBuffers 的性能优势明显。

### Q: 为什么 OH 使用 FlatBuffers？

主要原因是性能。AI 模型文件通常较大，FlatBuffers 的零拷贝特性可以显著减少模型加载时间和内存占用。

### Q: Patch 可以向上游提交吗？

大部分 Patch 属于 OH 特有适配（如仓颉语言支持、GN 构建），但 `boringssl.patch` 中的修复具有通用性，可尝试向上游提交。

### Q: 升级 FlatBuffers 版本需要注意什么？

1. 确认 Patch 是否仍需要
2. 检查仓颉语言绑定的兼容性
3. 验证 MindSpore Lite 和 NNRT 的兼容性

## 反馈与贡献

如发现文档错误或需要补充内容，请提交 Issue 或 Pull Request 到 OpenHarmony 第三方库维护团队。
