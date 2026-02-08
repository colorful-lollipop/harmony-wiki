# PyYAML 原始库简介

## 概述

PyYAML 是一个功能完整的 YAML 处理框架，专为 Python 设计。它提供了完整的 YAML 1.1 解析器、Unicode 支持、pickle 支持、强大的扩展 API 和友好的错误消息。

## 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | PyYAML |
| **当前版本** | 6.0.2 |
| **许可证** | MIT License |
| **上游地址** | https://github.com/yaml/pyyaml |
| **Python 版本** | 3.8 - 3.13 |
| **主要作者** | Kirill Simonov <xi@resolvent.net> |

## 核心功能

### 1. YAML 解析与生成

PyYAML 支持完整的 YAML 1.1 规范，包括：

- **数据类型**: 标量、序列、映射、自定义类型
- **标签系统**: 标准 YAML 标签和 Python 特定标签
- **Unicode 支持**: 完整的 Unicode 字符处理
- **多文档流**: 支持在单个流中包含多个 YAML 文档

### 2. 加载器 (Loaders)

PyYAML 提供了多种加载器，以平衡安全性和功能性：

| 加载器 | 用途 | 安全性 |
|--------|------|--------|
| `SafeLoader` | 加载标准 YAML 标签 | ✅ 安全（推荐） |
| `FullLoader` | 加载完整的 YAML 1.1 标签 | ⚠️ 较安全 |
| `UnsafeLoader` | 加载任意 Python 对象 | ❌ 危险 |

**示例**:
```python
import yaml

# 推荐使用 SafeLoader
with open('config.yaml', 'r') as f:
    data = yaml.safe_load(f)  # 等同于 yaml.load(f, Loader=yaml.SafeLoader)

# 如果需要加载任意 Python 对象
data = yaml.load(f, Loader=yaml.UnsafeLoader)  # 不推荐，存在安全风险
```

### 3. 转储器 (Dumpers)

PyYAML 支持将 Python 对象序列化为 YAML：

| 转储器 | 用途 |
|--------|------|
| `SafeDumper` | 转储标准 YAML 标签 |
| `Dumper` | 转储完整的 YAML 1.1 标签 |

**示例**:
```python
import yaml

data = {
    'name': 'OpenHarmony',
    'version': '1.0',
    'features': ['mini', 'standard']
}

# 转储为 YAML
yaml.dump(data, open('output.yaml', 'w'), default_flow_style=False)
```

### 4. LibYAML C 扩展

PyYAML 可选使用 LibYAML C 库以提升性能：

- **Cython 编译**: 通过 Cython 将 Python 代码编译为 C 扩展
- **性能提升**: 解析和生成速度比纯 Python 实现快 5-10 倍
- **透明集成**: 通过 `yaml.CLoader` 和 `yaml.CDumper` 使用

**示例**:
```python
import yaml

# 使用 C 扩展（如果已编译）
data = yaml.load(stream, Loader=yaml.CLoader)
yaml.dump(data, Dumper=yaml.CDumper)
```

## 项目结构

```
PyYAML/
├── lib/yaml/           # Python 纯实现
│   ├── __init__.py    # 公共 API
│   ├── parser.py      # YAML 解析器
│   ├── scanner.py     # 词法扫描器
│   ├── emitter.py     # 输出生成器
│   ├── composer.py    # 事件组合器
│   ├── constructor.py # 数据构造器
│   ├── representer.py # 数据表示器
│   ├── resolver.py    # 类型解析器
│   ├── loader.py      # 加载器
│   ├── dumper.py      # 转储器
│   ├── nodes.py       # 节点类型定义
│   ├── events.py      # 事件类型定义
│   ├── tokens.py      # 词法标记定义
│   ├── error.py       # 错误处理
│   └── cyaml.py       # LibYAML C 扩展绑定
├── yaml/              # Cython 源文件
│   ├── _yaml.pyx      # LibYAML 绑定源码
│   ├── _yaml.pxd      # Cython 声明
│   └── _yaml.h        # C 头文件
└── tests/             # 测试套件
```

## 在 OpenHarmony 中的作用

### 定位

PyYAML 在 OpenHarmony 中作为 **第三方库组件** 提供，归属于 `thirdparty` 子系统。它是一个**纯 Python 库**，不涉及 C/C++ 代码的直接集成。

### 使用场景

虽然 PyYAML 在 OpenHarmony 中的具体使用场景需要进一步确认（见 `04_Usage_in_OH.md`），但基于其功能特性，可能的应用场景包括：

1. **配置文件解析**
   - 应用配置（如 `config.yaml`）
   - 系统配置文件
   - 构建配置（部分场景）

2. **数据序列化**
   - Python 对象持久化
   - 数据交换格式
   - 测试数据描述

3. **测试支持**
   - 测试用例配置
   - 测试数据定义

### 集成方式

PyYAML 在 OpenHarmony 中采用 **轻量级适配** 方式：

- **无源代码修改**: 保持与上游版本完全一致
- **配置文件适配**: 通过 `bundle.json`、`OAT.xml` 等声明组件信息
- **构建工具增强**: 添加 PEP 517 构建后端支持配置化构建

详见 `03_Build_Integration.md`。

## 版本历史

### 当前版本 (6.0.2)

- 发布日期: 2024-08-06
- 主要更新:
  - 支持 Cython 3.x
  - 支持 Python 3.13
  - 修复了若干兼容性问题

### 主要版本变更

| 版本 | 发布日期 | 主要变更 |
|------|---------|----------|
| 6.0 | 2021-10-13 | 移除 Python 2.7 支持，要求显式指定 Loader |
| 5.4.1 | 2021-01-20 | 修复 CVE-2020-14343，移除任意 Python 标签到 UnsafeLoader |
| 5.3.1 | 2020-03-18 | 防止 python/object/new 构造器的任意代码执行 |

完整的版本变更记录见 `CHANGES` 文件。

## 相关文档

- [02_Patches.md](./02_Patches.md) - OpenHarmony Patch 详细分析
- [03_Build_Integration.md](./03_Build_Integration.md) - OpenHarmony 构建适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 在 OpenHarmony 中的依赖关系与使用
- [06_Security.md](./06_Security.md) - 安全风险分析

## 参考资料

- [PyYAML 官方文档](https://pyyaml.org/wiki/PyYAMLDocumentation)
- [PyYAML GitHub 仓库](https://github.com/yaml/pyyaml)
- [YAML 1.1 规范](https://yaml.org/spec/1.1/)
