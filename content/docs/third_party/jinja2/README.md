# Jinja2 - OpenHarmony 第三方库 Wiki

## 简介

本文档为 OpenHarmony `third_party/jinja2` 目录的 Wiki，重点记录 Jinja2 在 OpenHarmony 中的集成方式、使用场景和适配信息。

**文档版本**: 1.0  
**最后更新**: 2025-02-07  
**对应库版本**: 3.1.6

---

## Jinja2 是什么？

Jinja2 是一个现代化的、功能强大的 Python 模板引擎，广泛应用于 Web 开发、代码生成、配置文件生成等场景。

在 OpenHarmony 中，Jinja2 主要用作**构建系统的代码生成工具**，而非运行时组件。

---

## OpenHarmony 适配概述

### 关键事实

| 项目 | 详情 |
|------|------|
| **Patch 数量** | 0 (无 Patch) |
| **OH 特有修改** | 无 |
| **BUILD.gn** | 无 (使用 jinja2.gni) |
| **主要用途** | 构建时代码生成 |
| **直接依赖者** | 28+ 个 Python 脚本 |

### 为什么不需要 Patch？

1. **纯 Python 实现**: 不依赖平台特定代码
2. **标准功能使用**: OpenHarmony 仅使用核心模板功能
3. **构建时工具**: 非运行时依赖，不涉及系统级适配

### OH 特有的构建集成

```gn
# third_party/jinja2/jinja2.gni
jinja2_sources = [
  "//third_party/jinja2/__init__.py",
  "//third_party/jinja2/compiler.py",
  // ... 共 28 个 Python 文件
]
```

---

## 文档导航

### 核心文档

| 文档 | 内容 |
|------|------|
| [01_Overview.md](01_Overview.md) | 原始库简介、OH 中的作用和定位 |
| [02_Patches.md](02_Patches.md) | Patch 分析 (本文库无 Patch) |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 适配、jinja2.gni 说明 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系、使用场景、依赖图 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异 (无差异) |
| [06_Security.md](06_Security.md) | 安全风险分析 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度 |

---

## 快速参考

### 库信息

```yaml
名称: Jinja2
版本: 3.1.6
许可证: BSD 3-clause
上游: https://github.com/pallets/jinja
OH 组件: @ohos/jinja2
OH 子系统: thirdparty
```

### 在 OpenHarmony 中使用

```python
import sys
import os

# OH 特有的导入路径设置
sys.path.insert(1, os.path.join(OHOS_ROOT, 'third_party'))
from jinja2 import Template, Environment

# 使用模板
template = Template("Hello, {{ name }}!")
result = template.render(name="OpenHarmony")
```

### 主要使用场景

1. **SDK 生成** - `build/ohos/sdk/*.py`
2. **目标列表生成** - `build/hb/util/loader/*.py`
3. **测试代码生成** - `arkcompiler/**/tests/**/*.py`
4. **报告生成** - `test/testfwk/xdevice/**/*.py`

---

## 维护信息

### 升级注意事项

- ✅ **可直接升级上游版本**: 无 Patch 需要迁移
- ⚠️ **检查 API 兼容性**: 3.x 版本间可能存在 API 变化
- ⚠️ **更新 README.modification**: 当前版本信息未同步更新

### 已知问题

1. `README.modification` 中的版本号 (2.11.1) 与实际版本 (3.1.6) 不一致
2. 无其他已知问题

---

## 贡献与反馈

如有问题或建议，请联系：
- 负责人: anguanglin@huawei.com
- 相关子系统: thirdparty
