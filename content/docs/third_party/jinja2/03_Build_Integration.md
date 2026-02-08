# OH 构建适配

## 概述

Jinja2 是纯 Python 库，**没有 BUILD.gn 文件**。它通过 `jinja2.gni` 文件与 OpenHarmony 的 GN 构建系统集成。

---

## jinja2.gni 详解

### 文件位置

```
third_party/jinja2/jinja2.gni
```

### 文件内容

```gn
jinja2_sources = [
  "//third_party/jinja2/__init__.py",
  "//third_party/jinja2/_compat.py",
  "//third_party/jinja2/_identifier.py",
  "//third_party/jinja2/asyncfilters.py",
  "//third_party/jinja2/asyncsupport.py",
  "//third_party/jinja2/bccache.py",
  "//third_party/jinja2/compiler.py",
  "//third_party/jinja2/constants.py",
  "//third_party/jinja2/debug.py",
  "//third_party/jinja2/defaults.py",
  "//third_party/jinja2/environment.py",
  "//third_party/jinja2/exceptions.py",
  "//third_party/jinja2/ext.py",
  "//third_party/jinja2/filters.py",
  "//third_party/jinja2/idtracking.py",
  "//third_party/jinja2/lexer.py",
  "//third_party/jinja2/loaders.py",
  "//third_party/jinja2/meta.py",
  "//third_party/jinja2/nativetypes.py",
  "//third_party/jinja2/nodes.py",
  "//third_party/jinja2/optimizer.py",
  "//third_party/jinja2/parser.py",
  "//third_party/jinja2/runtime.py",
  "//third_party/jinja2/sandbox.py",
  "//third_party/jinja2/tests.py",
  "//third_party/jinja2/utils.py",
  "//third_party/jinja2/visitor.py",
]
```

### 用途说明

`jinja2.gni` 定义了 jinja2 库的所有 Python 源文件路径列表。其他 GN 目标可以：

1. **导入该文件** 获取源码列表
2. **将 jinja2 作为数据依赖** 打包到 SDK 或其他输出中
3. **复制 jinja2 文件** 到其他位置

### 使用示例

```gn
# 其他模块的 BUILD.gn
import("//third_party/jinja2/jinja2.gni")

# 将 jinja2 作为数据源复制
action("copy_jinja2") {
  script = "//build/scripts/copy_files.py"
  
  # 使用 jinja2_sources 作为输入
  sources = jinja2_sources
  
  outputs = [
    "$target_out_dir/jinja2/{{source_file_part}}",
  ]
  
  args = [
    "--input-dir",
    rebase_path("//third_party/jinja2"),
    "--output-dir",
    rebase_path(target_out_dir),
  ]
}
```

---

## 为什么没有 BUILD.gn？

### Python 库的特殊性

Python 库与 C/C++ 库在构建系统中有本质区别：

| 特性 | C/C++ 库 | Python 库 |
|------|----------|-----------|
| 编译 | 需要编译为二进制 | 解释执行，无需编译 |
| 构建产物 | .so, .a, .o 文件 | 无 (源码即产物) |
| 依赖声明 | `deps` 指向编译目标 | 运行时 `sys.path` 配置 |
| BUILD.gn | 必须 (定义编译规则) | 可选 (通常不需要) |

### OH 中的 Python 库处理

在 OpenHarmony 中，Python 库通常：

1. **直接作为源码存在** - 不进行编译
2. **通过 sys.path 动态导入** - 不链接
3. **作为数据文件打包** - 需要时复制

因此，不需要 BUILD.gn 来定义编译规则。

---

## 与上游构建系统的差异

### 上游构建方式

Jinja2 上游项目使用 Python 生态的标准工具：

```bash
# 上游安装方式
pip install jinja2

# 或从源码安装
python setup.py install
```

**文件结构**:
```
jinja2/                    # Python 包目录
    __init__.py
    environment.py
    ...
setup.py                   # 安装脚本
setup.cfg
pyproject.toml
```

### OpenHarmony 构建方式

OpenHarmony 不使用 pip 或 setup.py：

```
third_party/jinja2/        # 源码目录
    __init__.py
    environment.py
    ...
    jinja2.gni             # GN 源文件列表 (OH 特有)
    bundle.json            # OH 组件描述 (OH 特有)
    OAT.xml                # 许可证扫描配置 (OH 特有)
    README.OpenSource      # 开源声明 (OH 特有)
```

### 关键差异

| 项目 | 上游 | OpenHarmony |
|------|------|-------------|
| 构建工具 | setuptools/pip | GN (仅用于文件管理) |
| 安装方式 | pip install | 源码直接使用 |
| 依赖管理 | requirements.txt | 手动 sys.path 配置 |
| 元数据 | setup.py | bundle.json |

---

## OH 特有的导入模式

### 标准 Python 导入

```python
# 标准方式 (使用系统安装的 jinja2)
from jinja2 import Template
```

### OpenHarmony 导入方式

所有使用 jinja2 的 OH Python 脚本都采用以下模式：

```python
import sys
import os

# 方法 1: 显式路径构造
sys.path.insert(1, os.path.join(os.path.abspath(
    os.path.join(os.path.dirname(__file__), '..', '..', '..')), 'third_party'))
from jinja2 import Template  # noqa: E402

# 方法 2: 使用预定义变量
from resources.global_var import CURRENT_OHOS_ROOT
sys.path.insert(1, os.path.join(CURRENT_OHOS_ROOT, 'third_party'))
from jinja2 import Template
```

### 为什么需要这种模式？

1. **隔离性** - 确保使用 OH 自带的 jinja2，而非系统版本
2. **可移植性** - 不依赖系统 Python 包
3. **版本控制** - 使用特定版本的 jinja2
4. **确定性** - 构建环境一致性

---

## 特殊配置

### 编译选项

Jinja2 作为纯 Python 库，**没有编译选项**：
- 无 `defines`
- 无 `cflags`
- 无 `configs`

### 特性禁用

OpenHarmony 未禁用任何 Jinja2 特性：
- 完整功能可用
- 包括异步支持 (`asyncfilters.py`, `asyncsupport.py`)
- 包括沙箱环境 (`sandbox.py`)

### 依赖关系

Jinja2 在运行时依赖：
- Python 3.7+ (OpenHarmony 使用 Python 3.8+)
- MarkupSafe (可选，用于 HTML 转义)

**注意**: OpenHarmony 当前未明确管理 MarkupSafe 依赖，使用时需注意。

---

## 构建集成示例

### 示例 1: SDK 生成脚本

**文件**: `build/ohos/sdk/parse_sdk_description.py`

```python
#!/usr/bin/env python
import sys
import os

# OH 特有的 jinja2 导入路径
sys.path.insert(1, os.path.join(os.path.abspath(
    os.path.join(os.path.dirname(__file__), '..', '..', '..')), 'third_party'))
from jinja2 import Template  # noqa: E402

# 使用模板生成 GN 配置
template_str = """
{% for target in targets %}
{{ target.name }} = {{ target.value }}
{% endfor %}
"""

template = Template(template_str)
output = template.render(targets=target_list)
```

### 示例 2: 测试报告生成

**文件**: `test/testfwk/xdevice/plugins/devicetest/report/generation.py`

```python
from jinja2 import Environment, FileSystemLoader

# 创建模板环境
env = Environment(
    loader=FileSystemLoader(template_dir),
    autoescape=True  # 启用 HTML 自动转义
)

template = env.get_template('report_template.html')
html = template.render(results=test_results)
```

---

## 构建时依赖声明

虽然 jinja2 没有 BUILD.gn，但使用它的 Python 脚本需要确保 jinja2 文件存在。

### 隐式依赖

```gn
# 使用 jinja2 的 Python 脚本应声明数据依赖
action("generate_sdk_config") {
  script = "//build/ohos/sdk/parse_sdk_description.py"
  
  # 隐式依赖: 脚本会读取 third_party/jinja2
  inputs = [
    "//third_party/jinja2/__init__.py",  # 至少声明主文件
  ]
  
  # ... 其他配置
}
```

### 最佳实践

建议在 BUILD.gn 中添加注释说明 jinja2 依赖：

```gn
# 注意: 此脚本使用 third_party/jinja2 进行模板渲染
# jinja2 文件路径通过 sys.path 在运行时动态加载
```

---

## 升级指南

### 升级步骤

由于无 BUILD.gn 和 Patch，升级非常简单：

1. **下载新版本**
   ```bash
   wget https://github.com/pallets/jinja/archive/refs/tags/3.2.x.tar.gz
   ```

2. **替换源文件**
   ```bash
   # 备份旧版本
   mv third_party/jinja2 third_party/jinja2.old
   
   # 解压新版本
   tar xzf jinja-3.2.x.tar.gz
   mv jinja-3.2.x/src/jinja2 third_party/jinja2
   
   # 恢复 OH 配置文件
   cp third_party/jinja2.old/*.gni third_party/jinja2/
   cp third_party/jinja2.old/bundle.json third_party/jinja2/
   cp third_party/jinja2.old/OAT.xml third_party/jinja2/
   cp third_party/jinja2.old/README.OpenSource third_party/jinja2/
   ```

3. **更新 jinja2.gni**
   - 检查新版本是否有新增/删除的 .py 文件
   - 相应更新 `jinja2_sources` 列表

4. **验证**
   - 运行使用 jinja2 的构建脚本
   - 检查是否有 API 不兼容问题

5. **更新文档**
   - 更新 `README.OpenSource` 版本号
   - 更新 `bundle.json` 版本号
   - 更新本 Wiki

---

## 总结

| 项目 | 状态 |
|------|------|
| BUILD.gn | ❌ 无 (Python 库不需要) |
| jinja2.gni | ✅ 有 (定义源文件列表) |
| 编译配置 | ❌ 无 (纯 Python) |
| 特殊适配 | ❌ 无 (标准使用) |
| 升级难度 | ⭐ 低 (直接替换) |
