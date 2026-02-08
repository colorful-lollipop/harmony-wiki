# SELinux Wiki - OpenHarmony 适配文档

## 目录

1. [库概览](01_Overview.md) - SELinux 简介及在 OH 中的作用
2. [Patch 详细分析](02_Patches.md) - OH 特有修改和适配分析
3. [OH 构建适配](03_Build_Integration.md) - BUILD.gn 配置详解
4. [依赖关系与使用](04_Usage_in_OH.md) - OH 中的依赖者和使用场景
5. [API/接口差异](05_API_Differences.md) - OH 特有 API 和行为变更
6. [安全风险分析](06_Security.md) - CVE 和安全建议

## 快速导航

### 如果您是...

**系统开发者** → 推荐阅读 [01_Overview.md](01_Overview.md) → [03_Build_Integration.md](03_Build_Integration.md) → [04_Usage_in_OH.md](04_Usage_in_OH.md)

**安全工程师** → 推荐阅读 [02_Patches.md](02_Patches.md) → [06_Security.md](06_Security.md)

**应用开发者** → 推荐阅读 [01_Overview.md](01_Overview.md) → [04_Usage_in_OH.md](04_Usage_in_OH.md)

**升级维护者** → 推荐阅读 [02_Patches.md](02_Patches.md) → [03_Build_Integration.md](03_Build_Integration.md)

## 重要提示

- 本文档专注于 **OpenHarmony 对 SELinux 的定制和适配**
- SELinux 原始库文档请参考上游: https://github.com/SELinuxProject/selinux
- 本库没有传统 Patch 文件，OH 特有修改通过条件编译和新增模块实现

## 关键 OH 定制点

| 定制点 | 说明 | 影响范围 |
|-------|------|---------|
| `OHOS_FC_INIT` 宏 | 支持多文件 file_contexts | 标签管理 |
| `app_allow_config` | 应用白名单配置 | restorecon 行为 |
| `BUILD.gn` | OH 构建系统集成 | 编译和链接 |

## 相关文档

- [ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估报告
- [NOTES.md](_work/NOTES.md) - 分析过程记录
- [PLAN.md](_work/PLAN.md) - 任务进度
