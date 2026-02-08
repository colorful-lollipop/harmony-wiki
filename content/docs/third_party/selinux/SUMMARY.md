# 阅读路线建议

## 阅读路线一：快速了解（15 分钟）

适合需要快速了解 SELinux 在 OH 中的作用和定制内容的读者。

1. [README.md](README.md) - 文档导航
2. [01_Overview.md](01_Overview.md) - 库概览
   - 原始库简介
   - OH 中的作用
3. [02_Patches.md](02_Patches.md) - Patch 分析（重点阅读 Patch 清单表）

## 阅读路线二：系统开发者（30 分钟）

适合需要在 OH 系统中集成或使用 SELinux 的开发者。

1. [01_Overview.md](01_Overview.md) - 了解整体情况
2. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 详解
   - 编译选项
   - 依赖关系
   - 安装配置
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用场景
   - 依赖关系图
   - 主要使用模块
4. [05_API_Differences.md](05_API_Differences.md) - API 差异（如需要）

## 阅读路线三：安全/维护工程师（45 分钟）

适合负责 SELinux 安全审计或版本升级的工程师。

1. [01_Overview.md](01_Overview.md) - 基础了解
2. [02_Patches.md](02_Patches.md) - 详细 Patch 分析
   - OHOS_FC_INIT 宏详解
   - app_allow_config 模块
   - 升级建议
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置
4. [06_Security.md](06_Security.md) - 安全风险分析
5. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系（重点看安全相关模块）

## 阅读路线四：版本升级（60 分钟）

适合进行 SELinux 上游版本升级的技术人员。

1. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 评估报告
   - 基础信息对比
   - 修改点清单
2. [02_Patches.md](02_Patches.md) - Patch 详细分析
   - 每个 Patch 的升级建议
   - 回归风险
3. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 调整
4. [06_Security.md](06_Security.md) - CVE 检查

---

## 文档结构说明

```
wiki/
├── README.md              # 文档导航
├── SUMMARY.md             # 本文件：阅读路线
├── 01_Overview.md         # 库概览
├── 02_Patches.md          # Patch 分析（核心文档）
├── 03_Build_Integration.md # 构建适配
├── 04_Usage_in_OH.md      # OH 使用场景
├── 05_API_Differences.md  # API 差异（可选）
├── 06_Security.md         # 安全风险分析
└── _work/                 # 工作目录
    ├── ASSESSMENT.md      # 项目评估
    ├── NOTES.md           # 分析过程
    └── PLAN.md            # 任务进度
```

## 术语速查

| 术语 | 说明 |
|-----|------|
| SELinux | Security-Enhanced Linux，强制访问控制安全机制 |
| libselinux | SELinux 用户空间核心库 |
| libsepol | SELinux 策略编译库 |
| restorecon | 恢复文件安全上下文 |
| file_contexts | 文件安全上下文配置文件 |
| CIL | Common Intermediate Language，SELinux 策略中间语言 |
| TE | Type Enforcement，SELinux 核心安全策略 |
| OHOS | OpenHarmony Operating System |

