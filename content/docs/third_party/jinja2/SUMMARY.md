# 阅读路线建议

本 Wiki 提供多条阅读路线，根据您的角色和需求选择合适的路径。

---

## 路线一：快速了解 (5 分钟)

**适合**: 初次接触该库，需要快速了解基本信息

1. [README.md](README.md) - 阅读 "OpenHarmony 适配概述" 部分
2. [01_Overview.md](01_Overview.md) - 阅读 "该库在 OH 中的作用和定位"
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 阅读 "依赖图" 了解在系统中的位置

**关键要点**:
- 这是一个**零 Patch** 的第三方库
- 主要作为**构建工具**使用，非运行时依赖
- 28+ 个 Python 脚本依赖它进行代码生成

---

## 路线二：维护者指南 (15 分钟)

**适合**: 需要维护或升级该库的开发者

1. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 完整阅读项目评估报告
2. [03_Build_Integration.md](03_Build_Integration.md) - 理解 GN 集成方式
3. [06_Security.md](06_Security.md) - 了解安全风险
4. [02_Patches.md](02_Patches.md) - 确认无 Patch 需要维护

**关键要点**:
- 升级时**无需 Patch 迁移**
- 注意 `README.modification` 版本号未更新
- 检查上游版本间的 API 兼容性

---

## 路线三：使用者指南 (10 分钟)

**适合**: 需要在 Python 脚本中使用 jinja2 的开发者

1. [01_Overview.md](01_Overview.md) - 了解基本功能
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 阅读 "典型使用场景"
3. 参考已有代码:
   - `build/ohos/sdk/parse_sdk_description.py` - 简单模板使用
   - `test/testfwk/xdevice/plugins/devicetest/report/generation.py` - 复杂模板环境

**关键要点**:
- 使用 OH 特有的导入路径: `sys.path.insert(1, os.path.join(OHOS_ROOT, 'third_party'))`
- 标准 Jinja2 API，无 OH 特有扩展

---

## 路线四：全面审查 (30 分钟)

**适合**: 安全审计、架构评审、详细技术调研

按顺序阅读所有文档:

1. [README.md](README.md)
2. [_work/ASSESSMENT.md](_work/ASSESSMENT.md)
3. [01_Overview.md](01_Overview.md)
4. [02_Patches.md](02_Patches.md)
5. [03_Build_Integration.md](03_Build_Integration.md)
6. [04_Usage_in_OH.md](04_Usage_in_OH.md)
7. [05_API_Differences.md](05_API_Differences.md)
8. [06_Security.md](06_Security.md)

---

## 文档依赖关系

```
README.md (入口)
    ├── _work/ASSESSMENT.md (评估报告，可选)
    ├── 01_Overview.md (基础信息)
    ├── 02_Patches.md (Patch 分析)
    ├── 03_Build_Integration.md (构建集成)
    ├── 04_Usage_in_OH.md (使用场景)
    ├── 05_API_Differences.md (API 差异)
    └── 06_Security.md (安全分析)
```

---

## 附录：文档状态

| 文档 | 状态 | 备注 |
|------|------|------|
| README.md | ✅ 完成 | 主入口 |
| SUMMARY.md | ✅ 完成 | 本文件 |
| _work/ASSESSMENT.md | ✅ 完成 | 详细评估 |
| _work/NOTES.md | ✅ 完成 | 过程记录 |
| _work/PLAN.md | ✅ 完成 | 任务进度 |
| 01_Overview.md | ✅ 完成 | 库简介 |
| 02_Patches.md | ✅ 完成 | Patch 分析 |
| 03_Build_Integration.md | ✅ 完成 | 构建集成 |
| 04_Usage_in_OH.md | ✅ 完成 | 使用场景 |
| 05_API_Differences.md | ✅ 完成 | API 差异 |
| 06_Security.md | ✅ 完成 | 安全分析 |
