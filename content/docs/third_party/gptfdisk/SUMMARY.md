# 阅读路线建议 (SUMMARY)

> 根据您的角色和目的选择阅读路径

---

## 🎯 按角色推荐

### 我是系统开发者 (Storage/Partition)

**目标**: 理解 sgdisk 在 OH 中的工作原理和调用方式

**阅读顺序**:
1. [01_Overview.md](./01_Overview.md) - 5分钟了解库概况
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 理解在 storage_daemon 中的使用
3. [02_Patches.md](./02_Patches.md) - 重点理解 `--ohos-dump` 实现
4. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建配置

**重点关注**:
- `disk_info.cpp` 中 sgdisk 的调用方式
- `--ohos-dump` 输出格式解析
- ForkExec 执行的命令序列

---

### 我是构建工程师 (Build/GN)

**目标**: 维护 BUILD.gn 或升级上游版本

**阅读顺序**:
1. [03_Build_Integration.md](./03_Build_Integration.md) - GN 配置详解
2. [02_Patches.md](./02_Patches.md) - 源码修改点
3. [01_Overview.md](./01_Overview.md) - 了解库背景

**重点关注**:
- `external_deps` 的依赖声明
- `cflags_cc` 的编译选项
- 与上游 Makefile 的差异对比

---

### 我是安全工程师 (Security)

**目标**: 评估安全风险和攻击面

**阅读顺序**:
1. [06_Security.md](./06_Security.md) - 安全风险分析
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 理解调用上下文
3. [02_Patches.md](./02_Patches.md) - 检查 OH 修改引入的风险

**重点关注**:
- sgdisk 以 root 权限执行的风险
- 输入验证 (设备路径、分区参数)
- 缓冲区溢出风险 (C++ 代码)

---

### 我是新接触此库的开发者

**目标**: 快速了解 gptfdisk 在 OH 中的作用

**阅读顺序**:
1. [README.md](./README.md) - 本页概览
2. [01_Overview.md](./01_Overview.md) - 库简介
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 在 OH 中的使用

**时间投入**: 约 15-20 分钟

---

## 📚 按主题推荐

### 主题: Patch 与适配

**相关文档**:
- [02_Patches.md](./02_Patches.md) - Patch 详细分析
- [05_API_Differences.md](./05_API_Differences.md) - 接口差异
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 评估报告

---

### 主题: 构建与集成

**相关文档**:
- [03_Build_Integration.md](./03_Build_Integration.md) - GN 构建
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系
- [01_Overview.md#oh-适配概述](./01_Overview.md)

---

### 主题: 安全审计

**相关文档**:
- [06_Security.md](./06_Security.md) - 安全分析
- [02_Patches.md](./02_Patches.md) - 检查修改引入的攻击面
- [04_Usage_in_OH.md#使用场景](./04_Usage_in_OH.md)

---

## 🔍 快速查找

| 我想了解... | 查看文档 |
|------------|---------|
| 这是什么库? | [01_Overview.md](./01_Overview.md) |
| OH 修改了什么? | [02_Patches.md](./02_Patches.md) |
| 如何编译? | [03_Build_Integration.md](./03_Build_Integration.md) |
| 谁在用它? | [04_Usage_in_OH.md](./04_Usage_in_OH.md) |
| 安全吗? | [06_Security.md](./06_Security.md) |
| 上游版本? | [README.md#库概览](./README.md) |

---

## 📋 文档结构

```
wiki/
├── README.md              ← 入口文档 (您在这里)
├── SUMMARY.md             ← 本文件
├── 01_Overview.md         ← 库概览
├── 02_Patches.md          ← Patch 分析 (核心)
├── 03_Build_Integration.md ← 构建配置
├── 04_Usage_in_OH.md      ← 依赖与使用
├── 05_API_Differences.md  ← API 差异
├── 06_Security.md         ← 安全风险
└── _work/                 ← 工程文档
    ├── ASSESSMENT.md      ← 评估报告
    ├── NOTES.md           ← 分析笔记
    └── PLAN.md            ← 任务计划
```

---

## ⚡ 5分钟速览

如果只有 5 分钟，阅读以下内容:

1. [README.md#库概览](./README.md#库概览) - 基本信息
2. [README.md#oh-特有功能](./README.md#oh-特有功能) - 关键修改
3. [02_Patches.md#ohos_dump-函数分析](./02_Patches.md#ohos_dump-函数分析) - 核心代码

---

## 📝 更新记录

| 日期 | 更新内容 |
|------|---------|
| 2026-02-07 | 初始版本创建 |
