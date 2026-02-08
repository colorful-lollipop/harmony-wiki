# Jinja2 原始库简介

## 基本信息

| 属性 | 内容 |
|------|------|
| **库名称** | Jinja2 |
| **版本** | 3.1.6 |
| **许可证** | BSD 3-clause License |
| **上游地址** | https://github.com/pallets/jinja |
| **编程语言** | Python |
| **维护组织** | Pallets Projects |

## 原始功能

Jinja2 是一个**快速、富有表现力、可扩展的模板引擎**。其核心功能包括：

### 核心特性

1. **模板继承和包含**
   - 支持模板继承 (`{% extends %}`)
   - 支持模板包含 (`{% include %}`)

2. **宏定义**
   - 在模板中定义和导入宏 (`{% macro %}`)

3. **自动转义**
   - HTML 模板自动转义，防止 XSS 攻击

4. **沙箱环境**
   - 可选的沙箱环境，可安全渲染不受信任的模板

5. **异步支持**
   - 支持 AsyncIO 生成模板

6. **国际化**
   - 通过 Babel 支持 i18n

7. **性能优化**
   - 模板即时编译为优化的 Python 代码
   - 支持字节码缓存

8. **调试支持**
   - 异常指向模板中的正确行号

9. **可扩展性**
   - 自定义过滤器、测试、函数、语法

### 基本用法示例

```jinja
{% extends "base.html" %}
{% block title %}Members{% endblock %}
{% block content %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}
```

### Python API 示例

```python
from jinja2 import Template, Environment, FileSystemLoader

# 简单模板
template = Template("Hello, {{ name }}!")
result = template.render(name="World")

# 从文件加载
env = Environment(loader=FileSystemLoader('templates'))
template = env.get_template('page.html')
result = template.render(title="Home", items=[1, 2, 3])
```

---

## Jinja2 在 OpenHarmony 中的作用和定位

### 功能定位

在 OpenHarmony 中，Jinja2 的定位是：**构建系统的代码生成引擎**。

```
┌─────────────────────────────────────────────────────────┐
│                    OpenHarmony 构建系统                    │
├─────────────────────────────────────────────────────────┤
│  GN/Ninja 构建流程                                       │
│       ↓                                                 │
│  Python 构建脚本 ←── 使用 Jinja2 生成代码/配置            │
│       ↓                                                 │
│  生成 .gn, .gni, .json, 测试代码等                       │
│       ↓                                                 │
│  最终编译产物                                            │
└─────────────────────────────────────────────────────────┘
```

### 主要使用场景

#### 1. SDK 描述解析与生成

**位置**: `build/ohos/sdk/`

```python
from jinja2 import Template

# 生成 SDK 模块配置
template = Template("""
sdk_modules = [
  {% for module in modules %}
  {
    name = "{{ module.name }}"
    type = "{{ module.type }}"
  },
  {% endfor %}
]
""")
```

**用途**: 解析 SDK 描述文件，生成 GN 构建配置。

#### 2. 构建目标列表生成

**位置**: `build/hb/util/loader/`

```python
from jinja2 import Template

PARTS_LIST_GNI_TEMPLATE = """
parts_list = [
  {% for part in parts %}
  "{{ part }}",
  {% endfor %}
]
"""
```

**用途**: 生成部件列表、内部 kit 列表等 GN 变量定义文件。

#### 3. 测试代码生成

**位置**: `arkcompiler/runtime_core/static_core/tests/`

```python
from jinja2 import Environment, FileSystemLoader

# 批量生成测试用例
env = Environment(loader=FileSystemLoader(template_dir))
template = env.get_template('test_case_template.ets')
for test_spec in test_specs:
    code = template.render(**test_spec)
    write_file(f"{test_spec.name}.ets", code)
```

**用途**: 基于 YAML/JSON 测试规范批量生成 TypeScript/JavaScript 测试代码。

#### 4. 测试报告生成

**位置**: `test/testfwk/xdevice/`

```python
from jinja2 import Environment, FileSystemLoader

# 生成 HTML 测试报告
env = Environment(loader=FileSystemLoader(report_templates))
template = env.get_template('report_template.html')
html_report = template.render(
    test_results=results,
    summary=summary,
    timestamp=now()
)
```

**用途**: 生成美观的 HTML 格式测试报告。

#### 5. 第三方库代码生成

**位置**: `third_party/mbedtls/`, `third_party/grpc/`

```python
import jinja2

# 生成驱动包装代码
template = jinja2.Template(driver_wrapper_template)
code = template.render(drivers=driver_list)
```

**用途**: 根据配置文件生成重复性代码。

### 使用特点

| 特点 | 说明 |
|------|------|
| **构建时工具** | 仅在构建过程中使用，不参与运行时 |
| **无状态** | 不保存状态，每次构建重新生成 |
| **确定性** | 相同输入产生相同输出，保证构建可复现 |
| **纯 Python** | 跨平台，无需编译 |

### 与 OH 子系统的关系

```
third_party/jinja2
    ├── build/ohos/sdk/ ←────── 构建子系统
    ├── build/hb/ ←──────────── build.sh (hb) 工具
    ├── build/dfx/ ←─────────── DFX 子系统
    ├── test/testfwk/ ←──────── 测试框架
    ├── arkcompiler/ ←───────── ArkCompiler
    └── third_party/*/scripts/ ← 其他第三方库脚本
```

### 替代方案分析

| 方案 | 优缺点 | 是否适用 |
|------|--------|----------|
| **Jinja2** (当前) | 功能丰富、文档完善、广泛使用 | ✅ 推荐 |
| Python string.Template | 功能简单，不支持复杂逻辑 | ❌ 不满足需求 |
| Mustache | 逻辑-less，功能受限 | ❌ 不满足需求 |
| Mako | 功能类似，但社区较小 | ⚠️ 可用但没必要切换 |

**结论**: Jinja2 是 OpenHarmony 构建系统的合适选择，无需替换。

---

## 版本历史 (节选)

### Version 3.1.6 (2025-01-10)
当前 OpenHarmony 使用的版本。

- 修复编译器错误检查父模板中空块的问题
- xmlattr 过滤器不再允许带空格的键
- 改进 {% trans %} 块嵌套错误的提示信息

### Version 3.1.0 (2022-03-24)
主要版本更新：

- 移除 Python 3.6 支持
- 删除已弃用的代码
- 宏支持原生类型
- 添加 `items` 过滤器

### Version 3.0.0 (2021-05-11)
重大更新版本。

---

## 参考资料

- [Jinja2 官方文档](https://jinja.palletsprojects.com/)
- [Jinja2 GitHub 仓库](https://github.com/pallets/jinja)
- [OpenHarmony 构建系统文档](../_work/ASSESSMENT.md)
