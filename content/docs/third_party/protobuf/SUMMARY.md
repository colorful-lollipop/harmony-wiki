# 文档索引与阅读路线

## 文档结构

```
wiki/
├── README.md              # 入口文档，概览与导航
├── SUMMARY.md             # 本文档，索引与路线
├── 01_Overview.md        # 库概述
├── 02_Patches.md          # Patch 分析（核心）
├── 03_Build_Integration.md # 构建适配
├── 04_Usage_in_OH.md      # 依赖与使用
├── 05_API_Differences.md  # API 差异
├── 06_Security.md         # 安全分析
└── _work/
    ├── ASSESSMENT.md     # 评估记录
    └── NOTES.md          # 分析笔记
```

## 阅读路线建议

### 路线 A：快速了解（5 分钟）

1. **[README.md](./README.md)** - 2 分钟
   - 了解 protobuf 在 OH 中的定位
   - 快速查看适配状态

2. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 3 分钟
   - 查看主要依赖模块
   - 了解使用场景

### 路线 B：深入理解（15 分钟）

1. **[01_Overview.md](./01_Overview.md)** - 3 分钟
   - 原始库功能简介
   - OH 集成策略

2. **[02_Patches.md](./02_Patches.md)** - 5 分钟
   - 详细分析每个 Patch
   - 理解 OH 定制化需求

3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 4 分钟
   - 构建配置详解
   - 平台适配说明

4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 3 分钟
   - 依赖关系图
   - 使用场景示例

### 路线 C：完整参考（30 分钟）

完成路线 B 后，额外阅读：

5. **[05_API_Differences.md](./05_API_Differences.md)** - 5 分钟
   - API 变更记录
   - 兼容性说明

6. **[06_Security.md](./06_Security.md)** - 5 分钟
   - 安全漏洞历史
   - OH 修复状态

## 按角色推荐

### 开发者（引入 Protobuf）

阅读顺序：README → 04 → 03

### 系统集成（维护适配）

阅读顺序：README → 02 → 03 → 06

### 安全审查

阅读顺序：README → 06 → 02 → 05

## 关键术语

| 术语 | 说明 |
|------|------|
| **protobuf_lite** | 轻量级运行时，去除反射等特性 |
| **protobuf_full** | 完整运行时，支持全部特性 |
| **protoc** | Protocol Buffers 编译器 |
| **HILOG** | OpenHarmony 日志系统 |
| **PAC-RET** | 指针认证码（指针混淆安全特性） |

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2024-XX-XX | 1.0 | 初始文档创建 |
