# 阅读路线建议

本文档提供针对不同角色的 elfutils OpenHarmony 文档阅读路径。

---

## 按角色推荐

### 新手入门
**目标**: 快速了解 elfutils 在 OH 中的定位和用途

1. [README.md](./README.md) - 文档导航和概览
2. [01_Overview.md](./01_Overview.md) - 库的基础信息
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解谁在使用 elfutils

**预计时间**: 30 分钟

### 构建工程师
**目标**: 了解如何构建和修改 elfutils

1. [README.md](./README.md) - 文档导航
2. [03_Build_Integration.md](./03_Build_Integration.md) - GN 构建系统详解
3. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整技术评估
4. [02_Patches.md](./02_Patches.md) - 了解代码修改

**预计时间**: 1 小时

### 维护者
**目标**: 维护和升级 elfutils

1. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整评估信息
2. [02_Patches.md](./02_Patches.md) - 源代码修改清单
3. [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系
5. [06_Security.md](./06_Security.md) - 安全考虑

**预计时间**: 2 小时

### 安全审计人员
**目标**: 了解安全风险和合规性

1. [README.md](./README.md) - 基础信息
2. [06_Security.md](./06_Security.md) - 安全分析
3. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 许可证信息

**预计时间**: 45 分钟

---

## 按主题深入

### 构建系统
- [03_Build_Integration.md](./03_Build_Integration.md) - GN 构建配置
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 第 0.4 节 - OH 特有文件

### 代码修改
- [02_Patches.md](./02_Patches.md) - Patch 详细分析
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 第 0.2 节 - 源代码修改清单

### 依赖关系
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖图和使用场景
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 第 0.3 节 - OH 使用情况

### 新功能
- [01_Overview.md](./01_Overview.md) - libdwfl_stacktrace 介绍
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 第 0.5 节 - 实验性功能

---

## 文档结构说明

```
wiki/
├── README.md                 # 文档导航和概览
├── SUMMARY.md               # 本文件 - 阅读路线
├── 01_Overview.md          # 库概览
├── 02_Patches.md          # Patch 分析
├── 03_Build_Integration.md # 构建适配
├── 04_Usage_in_OH.md     # 使用关系
├── 05_API_Differences.md  # API 差异
├── 06_Security.md        # 安全分析
└── _work/
    ├── ASSESSMENT.md      # Phase 0 评估报告
    ├── NOTES.md          # 分析过程记录
    └── PLAN.md          # 任务进度
```

---

## 常见问题快速索引

### Q: elfutils 为什么在 OH 中？
**A**: [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 作为 libabigail 的依赖

### Q: OH 版本与上游有什么不同？
**A**: [02_Patches.md](./02_Patches.md) - 代码修改，[03_Build_Integration.md](./03_Build_Integration.md) - 构建差异

### Q: 如何添加新架构支持？
**A**: [03_Build_Integration.md](./03_Build_Integration.md) - GN 配置说明

### Q: 有什么安全风险？
**A**: [06_Security.md](./06_Security.md) - 安全风险分析

---

## 反馈与更新

如发现文档问题或有补充建议，请：
1. 查阅 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 中的待确认项
2. 参考 [README.md](./README.md) 的上游链接获取最新信息
3. 更新相关文档并记录变更
