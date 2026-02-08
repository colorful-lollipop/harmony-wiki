# API/接口差异

## 5.1 概述

### 结论：API 无差异

MarkupSafe 在 OpenHarmony 中使用的 API 与上游版本**完全一致**，未进行任何修改或扩展。

OpenHarmony 对 MarkupSafe 的集成方式是**原样引用**，没有新增任何 OH 特有的 API，没有修改任何现有函数的行为，也没有禁用任何功能。

## 5.2 API 对比表

| API | 上游状态 | OH 状态 | 差异 |
|-----|----------|---------|------|
| `Markup` | ✅ 可用 | ✅ 可用 | 无 |
| `escape()` | ✅ 可用 | ✅ 可用 | 无 |
| `soft_str()` | ✅ 可用 | ✅ 可用 | 无 |
| `escape_silent()` | ✅ 可用 | ✅ 可用 | 无 |
| `EscapeFormatter` | ✅ 可用 | ✅ 可用 | 无 |
| `__html__` 协议 | ✅ 支持 | ✅ 支持 | 无 |

## 5.3 核心 API 清单

### 5.3.1 Markup 类

```python
# 上游定义
class Markup(str):
    """安全 HTML/XML 字符串"""
    def __new__(cls, base="", encoding=None, errors="strict")
    def __html__(self) -> "Markup"
    def escape(cls, s: t.Any) -> "Markup"
    def unescape(self) -> str
    def striptags(self) -> str
    # ... 字符串方法的转义版本
```

**OH 状态**：完全一致，未修改

### 5.3.2 escape 函数

```python
# 上游定义
def escape(s: t.Any) -> Markup:
    """将对象转换为转义的 Markup"""
```

**OH 状态**：完全一致，未修改

### 5.3.3 soft_str 函数

```python
# 上游定义
def soft_str(s: t.Any) -> str:
    """在保持 Markup 语义的前提下转换为字符串"""
```

**OH 状态**：完全一致，未修改

### 5.3.4 EscapeFormatter 类

```python
# 上游定义
class EscapeFormatter(string.Formatter):
    """支持自动转义的字符串格式化器"""
```

**OH 状态**：完全一致，未修改

## 5.4 特殊行为说明

### 5.4.1 无 OH 特有行为

以下情况在 MarkupSafe 中**不存在**：

**无平台条件分支**——代码中没有 `#ifdef OHOS` 或类似的平台判断宏，所有代码路径在所有平台上表现一致。

**无 OH 配置选项**——没有 OH 特有的环境变量、配置项或初始化参数。

**无扩展 API**——OpenHarmony 没有为 MarkupSafe 添加任何新 API。

### 5.4.2 回退机制

MarkupSafe 的回退机制在 OH 中正常工作：

```python
try:
    from ._speedups import escape
except ImportError:
    from ._native import escape
```

当 C 扩展不可用时，自动回退到纯 Python 实现。

## 5.5 类型注解

### 5.5.1 类型支持

MarkupSafe 提供了完整的类型注解：

| 类型文件 | 用途 |
|----------|------|
| `_speedups.pyi` | C 扩展的类型提示 |
| `py.typed` | PEP 561 标记 |

**OH 状态**：完全一致，未修改

### 5.5.2 类型使用

```python
# 类型检查支持
from markupsafe import Markup

def render_html(content: str) -> Markup:
    return escape(content)
```

## 5.6 兼容性说明

### 6.6.1 向后兼容

MarkupSafe 在 API 设计上保持向后兼容：

- 新版本不会移除或修改已有 API
- 新增 API 采用新的命名空间
- 废弃的 API 会保留至少一个大版本

### 5.6.2 跨版本兼容

| OH 版本 | MarkupSafe 版本 | 兼容性 |
|---------|-----------------|--------|
| 4.0+ | 2.1.5 | ✅ 兼容 |
| 未来版本 | 升级版本 | ✅ 需验证 |

## 5.7 使用建议

### 5.7.1 标准 API 使用

建议始终使用标准的 MarkupSafe API：

```python
# 推荐：使用标准 API
from markupsafe import Markup, escape

# 不推荐：使用内部 API
from markupsafe._native import _escape_impl
```

### 5.7.2 最佳实践

| 场景 | 推荐 API |
|------|----------|
| 转义用户输入 | `escape()` |
| 标记安全 HTML | `Markup()` |
| 格式化字符串 | `Markup.format()` |
| 沙箱格式化 | `EscapeFormatter` |

## 5.8 未来变更预测

### 可能的 API 演进

根据上游项目的历史，MarkupSafe 未来可能的 API 变更：

| 变更类型 | 可能性 | 说明 |
|----------|--------|------|
| 新增函数 | 低 | 功能已稳定 |
| 新增类 | 低 | 核心类已完善 |
| 废弃 API | 中 | 可能废弃过时的 API |
| 类型改进 | 中 | 类型注解持续优化 |

OpenHarmony 在升级 MarkupSafe 版本时，应关注上游的变更日志。
