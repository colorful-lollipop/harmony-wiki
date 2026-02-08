# SUMMARY - 阅读路线建议

## 文档阅读路线图

### 路线 A：快速了解（10分钟）

适合想快速了解本项目基本情况的读者。

1. **[README.md](./README.md)** - 库概览和导航
   - 了解基本信息和功能定位
   - 查看文档导航

2. **[01_Overview.md](./01_Overview.md)** - 原始库简介
   - 了解仓颉运行时的核心功能
   - 了解在 OH 中的作用

3. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 依赖关系
   - 了解谁在使用这个库
   - 查看依赖关系图

### 路线 B：深度技术（30分钟）

适合需要深入理解集成细节的开发者。

1. **[01_Overview.md](./01_Overview.md)** - 架构理解
   - 运行时架构
   - 标准库架构

2. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建系统
   - BUILD.gn 结构详解
   - OH 构建配置
   - CMake 工具链适配

3. **[02_Patches.md](./02_Patches.md)** - 适配分析
   - 理解预编译模式
   - 源码级 OH 适配分析

4. **[05_API_Differences.md](./05_API_Differences.md)** - API 限制
   - OH 不支持的 API 列表
   - 平台差异原因

### 路线 C：安全审计（20分钟）

适合安全工程师进行风险评估。

1. **[06_Security.md](./06_Security.md)** - 安全风险分析
   - 依赖库的 CVE 状态
   - 安全升级建议

2. **[02_Patches.md](./02_Patches.md)** - Patch 分析
   - 了解修改点（虽然本项目无 Patch，但有源码适配）

3. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 使用场景
   - 了解攻击面

### 路线 D：维护升级（40分钟）

适合负责版本升级的维护者。

1. **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 完整评估
   - 查看所有证据清单
   - 了解项目特点

2. **[02_Patches.md](./02_Patches.md)** - Patch/适配清单
   - 了解需要维护的修改点

3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 构建配置
   - 了解版本相关配置

4. **[06_Security.md](./06_Security.md)** - 安全更新
   - 了解 CVE 跟踪

---

## 文档依赖关系

```
README.md (起点)
    ├── 01_Overview.md (基础)
    │       └── 03_Build_Integration.md (深入)
    │       └── 04_Usage_in_OH.md (关联)
    │
    ├── 02_Patches.md (技术细节)
    │       └── 05_API_Differences.md (延伸)
    │
    └── 06_Security.md (独立)
```

---

## 按角色推荐

| 角色 | 推荐路线 | 关键文档 |
|-----|---------|---------|
| 架构师 | A → B | 01、03、04 |
| 系统开发者 | B | 03、02、05 |
| 应用开发者 | A → 05 | 01、05 |
| 安全工程师 | C | 06、04、_work/ASSESSMENT |
| 维护者 | D | 全部 |
| 新人入门 | A | README、01 |

---

## 关键信息速查

### 快速获取信息

| 我想知道... | 查看文档 |
|------------|---------|
| 这是什么库？ | [01_Overview.md](./01_Overview.md) |
| 有哪些 Patch？ | [02_Patches.md](./02_Patches.md) |
| 怎么构建的？ | [03_Build_Integration.md](./03_Build_Integration.md) |
| 谁在用它？ | [04_Usage_in_OH.md](./04_Usage_in_OH.md) |
| API 有什么限制？ | [05_API_Differences.md](./05_API_Differences.md) |
| 有安全风险吗？ | [06_Security.md](./06_Security.md) |
| 详细评估报告？ | [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) |

---

*按您的角色选择合适路线开始阅读*
