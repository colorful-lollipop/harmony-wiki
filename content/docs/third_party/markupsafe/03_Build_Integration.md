# OH 构建适配

## 3.1 构建系统概述

### 3.1.1 构建方式

MarkupSafe 在 OpenHarmony 中采用**纯 Python 包**的方式集成，不涉及传统的 Native 构建流程。

| 构建特性 | 说明 |
|----------|------|
| **构建系统** | Python 包（无 GN 构建） |
| **Native 编译** | 可选（C 扩展按需编译） |
| **编译配置** | 无 |
| **运行时依赖** | Python 3.8+ |

### 3.1.2 无 BUILD.gn 的原因

MarkupSafe 不需要 BUILD.gn 文件的原因：

**Python 包的独立性**——作为一个标准的 Python 包，MarkupSafe 通过 Python 的包管理机制进行分发和安装，不需要也不应该使用 OpenHarmony 的 GN 构建系统。

**模块化设计**——在 OpenHarmony 中，MarkupSafe 被视为 Python 运行时环境的一部分，通过标准的 import 机制被 Jinja2 引用。

**可选 C 扩展**——`_speedups.c` 是可选的性能优化扩展，即使不编译也不会影响库的基本功能。Jinja2 在检测到 C 扩展不可用时会自动回退到纯 Python 实现。

## 3.2 文件结构

### 3.2.1 目录布局

```
third_party/markupsafe/
├── __init__.py              # 主模块入口
├── _native.py               # 纯 Python 实现（回退）
├── _speedups.c              # C 扩展源码（可选）
├── _speedups.pyi            # 类型提示
├── py.typed                 # PEP 561 标记
├── bundle.json              # OH 组件描述
├── README.OpenSource        # 开源声明
├── README.modification      # 修改说明
├── LICENSE.rst              # 许可证文件
├── NOTICE → LICENSE.rst     # 许可证软链接
└── CHANGES.rst              # 变更日志
```

### 3.2.2 核心文件说明

**bundle.json**——OpenHarmony 组件描述文件，定义了组件的基本信息和元数据。

```json
{
    "name": "@ohos/markupsafe",
    "version": "3.1",
    "publishAs": "code-segment",
    "segment": {
        "destPath": "third_party/markupsafe"
    },
    "component": {
        "name": "markupsafe",
        "subsystem": "thirdparty",
        "adapted_system_type": ["mini", "small", "standard"]
    }
}
```

## 3.3 构建配置

### 3.3.1 组件配置

| 配置项 | 值 |
|--------|-----|
| **子系统** | thirdparty |
| **组件名** | markupsafe |
| **适配类型** | mini、small、standard |
| **发布类型** | code-segment |

### 3.3.2 依赖配置

**无外部依赖**——MarkupSafe 在 bundle.json 中声明了空的依赖列表：

```json
"deps": {
    "components": [],
    "third_party": []
}
```

这表明该库完全自包含，不依赖 OpenHarmony 的其他组件。

## 3.4 Python 环境集成

### 3.4.1 导入方式

MarkupSafe 通过标准的 Python 导入机制被使用：

```python
# 完整导入
import markupsafe

# 按需导入
from markupsafe import Markup, escape, soft_str

# 别名导入
import markupsafe as ms
```

### 3.4.2 Jinja2 的使用模式

Jinja2 通过以下方式使用 MarkupSafe：

```python
# compiler.py
from markupsafe import escape

# runtime.py
from markupsafe import Markup, escape, soft_str

# sandbox.py
from markupsafe import EscapeFormatter
```

## 3.5 C 扩展处理

### 3.5.1 编译策略

`_speedups.c` 是可选的 C 扩展，用于加速 HTML 转义操作。其处理策略如下：

**可用时优先使用**——`__init__.py` 中的导入逻辑：

```python
try:
    from ._speedups import escape as escape
    from ._speedups import escape_silent as escape_silent
    from ._speedups import soft_str as soft_str
except ImportError:
    from ._native import escape as escape
    from ._native import escape_silent as escape_silent
    from ._native import soft_str as soft_str
```

**无影响运行**——即使 C 扩展编译失败，Python 回退实现也能提供完整功能，只是性能略低。

### 3.5.2 性能考量

| 场景 | 性能表现 |
|------|----------|
| C 扩展可用 | 高性能（推荐用于高负载场景） |
| 纯 Python 回退 | 性能降低约 50%（仍可接受） |

## 3.6 版本管理

### 版本同步要求

需要确保以下版本的同步：

| 配置位置 | 应有版本 | 当前状态 |
|----------|----------|----------|
| README.OpenSource | 2.1.5 | ✅ 正确 |
| __init__.__version__ | 2.1.5 | ✅ 正确 |
| bundle.json | 2.1.5 | ❌ 错误（显示 3.1） |

**修复建议**：更新 bundle.json 中的版本号为 2.1.5。

## 3.7 构建验证

### 验证步骤

1. 验证 Python 导入
   ```bash
   python3 -c "import markupsafe; print(markupsafe.__version__)"
   ```

2. 验证核心功能
   ```python
   from markupsafe import Markup, escape
   assert escape("<script>") == Markup('&lt;script&gt;')
   ```

3. 验证 Jinja2 集成
   ```python
   from jinja2 import Template
   template = Template("Hello {{ name }}")
   result = template.render(name="<script>alert(1)</script>")
   assert "<script>" not in result
   ```
