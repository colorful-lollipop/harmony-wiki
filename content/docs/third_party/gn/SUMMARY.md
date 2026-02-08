# 阅读路线建议

本文档针对不同读者提供不同的阅读路线：

---

## 🚀 快速入门（5 分钟）

适合：只想快速了解 GN 在 OH 中的作用

1. **[README.md](./README.md)** - 库概览和 OH 适配概述
2. **[02_Patches.md](./02_Patches.md)** - 重点阅读 Patch 影响评估部分

---

## 📚 完整理解（30 分钟）

适合：需要全面了解 GN 在 OH 中的集成

按顺序阅读：

1. **[01_Overview.md](./01_Overview.md)** - 了解 GN 基本功能和 OH 中的作用
2. **[02_Patches.md](./02_Patches.md)** - 深入理解 OH 对 GN 的修改
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 理解构建集成方式
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 了解依赖关系和使用场景

---

## 🔧 开发者指南

适合：OH 组件开发者，需要编写 BUILD.gn

必读：
- **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建系统集成
- **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 典型使用场景和示例

参考：
- **[05_API_Differences.md](./05_API_Differences.md)** - 确认 GN 语法兼容性

---

## 🛡️ 安全评估

适合：安全审计人员

1. **[06_Security.md](./06_Security.md)** - 完整安全风险分析
2. **[02_Patches.md](./02_Patches.md)** - Patch 安全影响评估

---

## 🔍 维护者指南

适合：负责升级 GN 版本的维护者

必读：
1. **[02_Patches.md](./02_Patches.md)** - Patch 详细分析和升级建议
2. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建系统集成细节
3. **[06_Security.md](./06_Security.md)** - 安全升级策略

参考：
- **`_work/ASSESSMENT.md`** - 项目评估基础信息

---

## 📖 文档地图

```
wiki/
├── README.md                    # 入口文档
├── SUMMARY.md                   # 本文档
│
├── 01_Overview.md              # 原始库简介
│   ├── 库基本信息
│   ├── 原始功能描述
│   └── 在 OH 中的作用
│
├── 02_Patches.md               # ★ 核心文档
│   ├── Patch 清单
│   ├── Patch 详细分析
│   ├── 修改目的推断
│   └── 升级建议
│
├── 03_Build_Integration.md     # OH 构建适配
│   ├── 上游构建方式
│   ├── OH 构建集成
│   └── Patch 对构建的影响
│
├── 04_Usage_in_OH.md           # 依赖关系与使用
│   ├── 使用方式概述
│   ├── 依赖关系图
│   └── 典型使用场景
│
├── 05_API_Differences.md       # API/接口差异
│   └── Patch 行为变更
│
├── 06_Security.md              # 安全风险分析
│   ├── CVE 分析
│   ├── Patch 安全影响
│   └── 升级策略
│
└── _work/
    ├── ASSESSMENT.md           # 项目评估报告
    ├── NOTES.md                # 分析过程记录
    └── PLAN.md                 # 任务进度
```

---

## 💡 关键要点速览

### Patch 核心变更
- **文件**: `src/gn/filesystem_utils.cc`
- **变更**: 移除 BUILD_DIR 占位符，使用实际路径
- **影响**: 仅路径字符串表示，不影响功能

### 构建集成
- **方式**: Prebuilt 二进制
- **位置**: `prebuilts/build-tools/linux-x64/bin/gn`
- **配置**: 通过 `//build/ohos.gni` 统一导入

### 安全状态
- **CVE**: 历史 CVE 极少
- **风险**: 低风险
- **供应链**: Prebuilt 需来源可信

---

## 反馈与更新

如发现文档问题或需要更新，请：
1. 联系维护者: huhao51@huawei.com
2. 提交 PR 更新 wiki 目录下的文档

文档最后更新: 2025-02-08
