# MarkupSafe

## 库概述

MarkupSafe 是一个用于 HTML 和 XML 字符转义的 Python 库。在 OpenHarmony 生态系统中，它是 Jinja2 模板引擎的核心依赖组件，负责确保模板渲染过程中的安全性，防止 XSS（跨站脚本）等注入攻击。

| 属性 | 值 |
|------|-----|
| **版本** | 2.1.5 |
| **许可证** | BSD 3-Clause License |
| **上游地址** | https://github.com/pallets/markupsafe |
| **OH 组件名** | @ohos/markupsafe |

## OpenHarmony 适配特点

### 零 Patch 策略

MarkupSafe 在 OpenHarmony 中**未应用任何 Patch**。这并非疏漏，而是因为该库具备以下特性，使其能够无缝运行于 OpenHarmony 环境：

1. **纯 Python 实现**：不涉及操作系统底层的 Native 代码
2. **通用转义逻辑**：HTML/XML 字符转义是跨平台通用需求
3. **标准接口导出**：仅通过 Python 模块系统提供 Markup、escape、soft_str 等标准接口
4. **无平台依赖**：核心转义算法不依赖任何特定操作系统 API

### 文件裁剪优化

根据 README.modification 的说明，OpenHarmony 对 MarkupSafe 进行了**文件级别的裁剪优化**：

- 仅保留核心的 markup 相关文件
- 包含 LICENSE 和 AUTHORS 许可证文件
- 创建 NOTICE 软链接指向 LICENSE
- 移除了上游源码中非必要的文件和目录

这种轻量化处理既满足了许可证合规要求，又减少了存储空间占用。

## 核心功能

MarkupSafe 的核心功能是将包含特殊 HTML 字符的字符串转换为安全版本。以下是主要功能说明：

### Markup 类

```python
from markupsafe import Markup

# 创建安全的 Markup 对象
safe_text = Markup("<strong>Hello World</strong>")

# 字符串操作会自动转义参数
formatted = Markup("<em>{name}</em>").format(name='`<script>`alert(1)</script>')
# 结果：<em>&lt;script&gt;alert(1)&lt;/script&gt;</em>
```

### escape 函数

```python
from markupsafe import escape

# 将任意对象转换为安全的 Markup 对象
escaped = escape("`<script>`alert(document.cookie);</script>")
# 结果：Markup('&lt;script&gt;alert(document.cookie);&lt;/script&gt;')
```

### soft_str 函数

用于在保持转义语义的前提下，将对象转换为字符串。

## 在 OpenHarmony 中的角色

```
OpenHarmony 模板系统
        │
        ▼
    ┌───────┐
    │ Jinja2│  ←── 核心模板引擎
    └───────┘
        │
        ▼
┌───────────────────┐
│   MarkupSafe      │  ←── HTML/XML 转义层
└───────────────────┘
        │
        ▼
   安全的页面渲染输出
```

MarkupSafe 在 OpenHarmony 模板渲染栈中承担**安全基础设施**的角色，确保所有用户可控的模板变量都经过适当的转义处理。

## 文档导航

### 快速开始

- **概述与背景**：01_Overview.md
- **为什么不需要 Patch**：02_Patches.md
- **构建集成方式**：03_Build_Integration.md

### 深入了解

- **在 OH 中的使用**：04_Usage_in_OH.md
- **API 接口说明**：05_API_Differences.md
- **安全风险分析**：06_Security.md

### 技术参考

- **评估报告**：_work/ASSESSMENT.md
- **分析笔记**：_work/NOTES.md
- **任务进度**：_work/PLAN.md

## 相关资源

- [上游官方文档](https://markupsafe.palletsprojects.com/)
- [上游 GitHub 仓库](https://github.com/pallets/markupsafe)
- [Jinja2 文档](https://jinja.palletsprojects.com/)
