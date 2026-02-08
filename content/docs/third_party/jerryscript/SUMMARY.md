# JerryScript 文档阅读路线

本文档提供 JerryScript OpenHarmony 集成文档的阅读指南，帮助您快速找到需要的信息。

---

## 读者类型

### 1. 初学者 / 快速了解

**目标**: 了解 JerryScript 是什么，在 OH 中的作用

**阅读顺序**:
1. **[README.md](README.md)** - 概述（5分钟）
2. **[01_Overview.md](01_Overview.md)** - 原始库简介（10分钟）
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 简要浏览使用场景（10分钟）

**预计时间**: 25 分钟

---

### 2. 构建工程师 / 编译系统开发者

**目标**: 了解 JerryScript 的构建配置和 OH 适配

**阅读顺序**:
1. **[README.md](README.md)** - 概述（5分钟）
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建适配详解（30分钟）
3. **[02_Patches.md](02_Patches.md)** - Patch 相关（可选，15分钟）
4. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 评估总结（10分钟）

**预计时间**: 60 分钟

**重点关注**:
- BUILD.gn 配置
- engine.gn 变量定义
- Lite/标准模式差异
- IAR 工具链适配

---

### 3. 应用开发者 / 使用 JerryScript 的开发者

**目标**: 了解如何在 OH 中使用 JerryScript

**阅读顺序**:
1. **[README.md](README.md)** - 概述（5分钟）
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - OH 使用场景（30分钟）
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建配置章节（15分钟）
4. **[01_Overview.md](01_Overview.md)** - API 和特性（15分钟）

**预计时间**: 65 分钟

**重点关注**:
- 依赖关系
- API 使用示例
- 内存配置
- 常见问题

---

### 4. 维护者 / 升级 JerryScript

**目标**: 了解 OH 的定制化内容，规划升级策略

**阅读顺序**:
1. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 完整评估（20分钟）
2. **[02_Patches.md](02_Patches.md)** - Patch 详细分析（30分钟）
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建系统差异（20分钟）
4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系（15分钟）
5. **[01_Overview.md](01_Overview.md)** - 上游信息（10分钟）

**预计时间**: 95 分钟

**重点关注**:
- 所有 Patch 和代码修改
- 可向上游贡献的内容
- 升级风险评估
- 回归测试重点

---

### 5. 架构师 / 技术决策者

**目标**: 理解 JerryScript 在 OH 架构中的位置

**阅读顺序**:
1. **[README.md](README.md)** - 概述（5分钟）
2. **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 评估总结（10分钟）
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系图（10分钟）
4. **[01_Overview.md](01_Overview.md)** - 技术对比（10分钟）

**预计时间**: 35 分钟

**重点关注**:
- 依赖关系图
- 性能特点
- 与其他 JS 引擎的对比
- OH 适配概述

---

## 文档内容概览

### README.md
- 库基本信息
- 文档导航
- OH 适配概述
- 快速开始指南

### 01_Overview.md
- 原始库详细介绍
- 技术架构
- 性能特点
- 与其他 JS 引擎对比
- OH 中的定位

### 02_Patches.md
- Patch 文件清单
- 直接代码修改分析
- 内存管理优化
- GC 控制增强
- 字符串优化
- 升级建议

### 03_Build_Integration.md
- 构建系统概述
- BUILD.gn 详细配置
- engine.gn 变量定义
- 子模块构建
- OH 特定配置
- 构建示例
- 常见问题

### 04_Usage_in_OH.md
- 依赖关系图
- 核心使用场景
  - ace_engine_lite
  - bundle_framework_lite
  - netmanager_base
  - hilog_lite
- API 使用示例
- 性能优化建议

### _work/ASSESSMENT.md
- 基础信息收集
- Patch 分析
- OH 使用情况分析
- 特殊适配识别
- 评估总结

---

## 按主题查找

### 我想了解...

#### ...JerryScript 是什么？
→ **[01_Overview.md](01_Overview.md)** - 第 2 节"核心功能概述"

#### ...如何在 OH 中使用 JerryScript？
→ **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 第 2 节"核心使用场景"

#### ...如何配置 JerryScript？
→ **[03_Build_Integration.md](03_Build_Integration.md)** - 第 3 节"全局变量定义"

#### ...OH 对 JerryScript 做了哪些修改？
→ **[02_Patches.md](02_Patches.md)** - 全文

#### ...哪些模块依赖 JerryScript？
→ **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 第 1 节"依赖关系概览"

#### ...如何升级 JerryScript？
→ **[02_Patches.md](02_Patches.md)** - 第 5 节"升级建议"

#### ...JerryScript 的性能特点？
→ **[01_Overview.md](01_Overview.md)** - 第 6 节"性能特点"

#### ...构建时遇到问题？
→ **[03_Build_Integration.md](03_Build_Integration.md)** - 第 9 节"常见问题"

#### ...使用时遇到问题？
→ **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 第 6 节"常见问题"

#### ...安全相关的问题？
→ **[README.md](README.md)** - 第 10 节"安全说明"

---

## 深度学习路径

### 路径 1: 完整理解（2小时）
1. [README.md](README.md)
2. [01_Overview.md](01_Overview.md)
3. [02_Patches.md](02_Patches.md)
4. [03_Build_Integration.md](03_Build_Integration.md)
5. [04_Usage_in_OH.md](04_Usage_in_OH.md)
6. [_work/ASSESSMENT.md](_work/ASSESSMENT.md)

### 路径 2: 构建系统专家（1.5小时）
1. [README.md](README.md)
2. [03_Build_Integration.md](03_Build_Integration.md)（全文精读）
3. [02_Patches.md](02_Patches.md)（第 4 节"Patch 分类汇总"）
4. [_work/ASSESSMENT.md](_work/ASSESSMENT.md)

### 路径 3: 应用开发专家（1小时）
1. [README.md](README.md)
2. [04_Usage_in_OH.md](04_Usage_in_OH.md)（全文精读）
3. [03_Build_Integration.md](03_Build_Integration.md)（第 8 节"构建优化建议"）
4. [01_Overview.md](01_Overview.md)（第 5 节"技术架构"）

---

## 常见使用场景

### 场景 1: 我要开发使用 JS 的应用

**必读**:
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 第 2.1 节"ace_engine_lite - UI 引擎"

**选读**:
- [03_Build_Integration.md](03_Build_Integration.md) - 第 7 节"构建示例"

### 场景 2: 我要编译 JerryScript

**必读**:
- [03_Build_Integration.md](03_Build_Integration.md) - 全文

**选读**:
- [README.md](README.md) - 第 6 节"开发指南"

### 场景 3: 我要调试内存问题

**必读**:
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 第 5 节"性能优化建议"
- [02_Patches.md](02_Patches.md) - 第 2.1 节"内存管理优化"

**选读**:
- [03_Build_Integration.md](03_Build_Integration.md) - 第 8 节"构建优化建议"

### 场景 4: 我要升级 JerryScript 版本

**必读**:
- [02_Patches.md](02_Patches.md) - 第 5 节"升级建议"
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 第 0.5 节"评估总结"

**选读**:
- [03_Build_Integration.md](03_Build_Integration.md) - 第 10 节"升级构建系统"

---

## 补充资源

### 官方文档

- **JerryScript 官方文档**: https://jerryscript.net/docs/
- **OpenHarmony 文档**: https://docs.openharmony.cn/

### 相关技术

- **JSI (JavaScript Interface)**: ace_engine_lite 使用的 JS 绑定层
- **PAC (Proxy Auto-Configuration)**: 网络代理配置标准

### 联系方式

- **JerryScript 问题**: https://github.com/jerryscript-project/jerryscript/issues
- **OH 适配问题**: pengbiao1@huawei.com

---

## 反馈与贡献

如果您发现文档问题或有改进建议，欢迎反馈：
1. 提交 Issue 到相关代码仓库
2. 联系文档维护者

---

**祝您阅读愉快！**
