# PyYAML OpenHarmony Patch 详细分析

## 概述

PyYAML 在 OpenHarmony 中没有使用传统的 `.patch` 文件格式，所有的适配修改都通过 Git 提交历史记录管理。本文档详细分析 OpenHarmony 对 PyYAML 的所有定制化修改。

## Patch 统计概览

| 类别 | 数量 | 说明 |
|------|------|------|
| **配置类 Patch** | 10 | bundle.json、OAT.xml、README.OpenSource 等 |
| **版本升级 Patch** | 4 | 5.4.1 → 6.0 → 6.0.1 → 6.0.2 |
| **元数据修复 Patch** | 3 | 部件名、版本号等修正 |
| **代码同步 Patch** | 1 | openEuler 替换为官方版本 |
| **总计** | **18** | 不含 Merge 提交 |

## Patch 清单表

| 提交 ID | 提交时间 | 类型 | 描述 | 关联的 OH 需求 |
|---------|----------|------|------|----------------|
| `a7d5773` | 2022-01 | 配置 | 添加 README 和基础配置 | 组件初始化 |
| `7e579f0` | 2022-01 | 配置 | 添加 OAT.xml、README.OpenSource | 开源合规 |
| `f3cee10` | 2022-01 | Bugfix | README 拼写错误修正 | 文档完善 |
| `a5fe34d` | 2022-01 | 配置 | OAT.xml 详细配置 | 开源合规检查 |
| `a80f304` | 2022-05 | 配置 | 添加 bundle.json 部件配置 | 组件声明 |
| `2ecaf6c` | 2022-06 | 版本升级 | 5.4.1 → 6.0 | 同步上游 |
| `99e0319` | 2024-12 | 版本升级 | 6.0 → 6.0.1 | 同步上游 |
| `4d38797` | 2025-04 | 版本升级 | 开源火车版本回退 | 兼容性调整 |
| `d806d6d` | 2024-09 | Bugfix | 部件名改为小写 pyyaml | 命名规范 |
| `2dfd1da` | 2024-09 | Bugfix | 部件名改回 PyYAML | 命名规范 |
| `626dc65` | 2025-05 | 元数据 | 修改开源属性为 openEuler | 来源说明 |
| `7940d84` | 2025-05 | Bugfix | 修复版本描述错误 (6.0 → 3.1) | 元数据修正 |
| `68d6b83` | 2025-06 | 代码同步 | 替换 openEuler 为原生 PyYAML | 官方版本 |
| `abf1b62` | 2025-06 | Merge | 使用 PyYAML 替换 OpenEular::PyYAML | 合并请求 |

---

## 详细 Patch 分析

### Patch 1: 组件初始化配置

**提交**: `a7d5773`, `7e579f0`
**修改文件**: `README.md`, `OAT.xml`, `README.OpenSource`

**原始问题**: PyYAML 需要在 OpenHarmony 中声明为独立的第三方组件

**修改内容**:
```xml
<!-- OAT.xml -->
<oatconfig>
  <licensefile></licensefile>
  <policylist>
    <policy name="projectPolicy" desc="">
      <policyitem type="license" name="MIT" path="LICENSE" desc="兼容license"/>
      ...
    </policy>
  </policylist>
  <filefilterlist>
    <filefilter name="defaultPolicyFilter" desc="Filters for compatibility">
      <filteritem type="filepath" name="tests/.*" desc="no license header"/>
      <filteritem type="filepath" name="lib/yaml/.*" desc="no license header"/>
      ...
    </filefilter>
  </filefilterlist>
</oatconfig>
```

```json
// README.OpenSource
[
  {
    "Name": "PyYAML",
    "License": "MIT License",
    "License File": "LICENSE",
    "Version Number": "3.1",
    "Owner": "xuyong59@huawei.com",
    "Upstream URL": "https://pypi.org/project/PyYAML",
    "Description": "A YAML parser and emitter for Python"
  }
]
```

**OH 需求**: 满足 OpenHarmony 开源合规性要求（OAT 检查），声明组件的基本信息和许可证。

**关键代码变更**:
- 添加 `OAT.xml` 配置文件，设置 MIT License 兼容性检查
- 添加 `README.OpenSource` 声明文件，记录上游来源
- 配置文件过滤规则，跳过测试文件、库文件的许可证头检查

**升级建议**: 无需修改，这是 OH 通用的合规配置模板。

---

### Patch 2: 部件化配置

**提交**: `a80f304`
**修改文件**: `bundle.json`

**原始问题**: PyYAML 需要在 OpenHarmony 构建系统中声明为独立组件

**修改内容**:
```json
{
  "name": "@ohos/PyYAML",
  "description": "A YAML parser and emitter for Python.",
  "version": "5.4.1",
  "license": "MIT",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/PyYAML"
  },
  "component": {
    "name": "PyYAML",
    "subsystem": "thirdparty",
    "syscap": [],
    "features": [],
    "adapted_system_type": [],
    "rom": "",
    "ram": "",
    "deps": {
      "components": [],
      "third_party": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [],
      "test": []
    }
  }
}
```

**OH 需求**: 在 OpenHarmony 构建系统中注册 PyYAML 为可用的第三方库组件。

**关键代码变更**:
- 组件名: `@ohos/PyYAML`
- 子系统: `thirdparty`
- 适配系统类型: `[]` (初始为空，后续更新为 `["mini", "standard"]`)

**升级建议**: 升级版本时需同步更新 `version` 字段。

---

### Patch 3: 版本升级 (5.4.1 → 6.0)

**提交**: `2ecaf6c`
**修改文件**: `bundle.json`, `README.OpenSource`, `CHANGES`, `lib/yaml/*.py`

**原始问题**: 上游发布 PyYAML 6.0 版本，主要变更包括：
- 移除 Python 2.7 支持
- 要求显式指定 Loader 参数
- 修复多个安全漏洞

**修改内容**:
```json
// bundle.json
{
  "version": "6.0",  // 从 5.4.1 升级
  ...
}
```

```json
// README.OpenSource
{
  "Version Number": "6.0",  // 从 5.4.1 升级
  "Upstream URL": "https://pypi.org/project/PyYAML",
  ...
}
```

```python
# lib/yaml/__init__.py
__version__ = '6.0'  # 从 5.4.1 升级
```

**OH 需求**: 同步上游版本以获得安全修复和新功能。

**关键代码变更**:
- 版本号更新至 6.0
- 大量核心代码更新（Python 3 统一，移除 Python 2 兼容代码）
- 移除 Python 2.6、2.7、3.3 支持

**回归风险**:
- ❌ 不再支持 Python 2.x
- ⚠️ `yaml.load()` 必须显式指定 `Loader` 参数，否则报错
- ✅ OpenHarmony 主要使用 Python 3.x，风险可控

**升级建议**: 此为上游版本升级，需全面回归测试，特别关注 `yaml.load()` 的调用方式。

---

### Patch 4: 版本升级 (6.0 → 6.0.1)

**提交**: `99e0319`
**修改文件**: `bundle.json`, `README.OpenSource`, `lib/yaml/__init__.py`, `setup.py`, `pyproject.toml`

**原始问题**: 上游发布 PyYAML 6.0.1 版本，限制 Cython 版本 < 3.0

**修改内容**:
```toml
# pyproject.toml
[build-system]
requires = [
  "setuptools",
  "wheel",
  "Cython<3.0",  # 限制 Cython 版本
]
```

```json
// bundle.json
{
  "version": "6.0.1",
  ...
}
```

**OH 需求**: 同步上游的小版本更新，修复构建兼容性问题。

**关键代码变更**:
- 版本号更新至 6.0.1
- 限制 Cython 版本 < 3.0 以兼容旧版 Cython

**回归风险**: 低，主要是构建工具链的兼容性调整。

**升级建议**: 后续版本已移除 Cython 版本限制，可考虑升级到 6.0.2。

---

### Patch 5: 版本升级 (6.0.1 → 6.0.2)

**提交**: `68d6b83`, `abf1b62`
**修改文件**: `bundle.json`, `README.OpenSource`, `lib/yaml/__init__.py`, `setup.py`, `pyproject.toml`, `packaging/_pyyaml_pep517.py`, `packaging/build/libyaml.sh`, `tests/legacy_tests/conftest.py`

**原始问题**: 上游发布 PyYAML 6.0.2 版本，主要变更：
- 支持 Cython 3.x
- 支持 Python 3.13
- 从 openEuler 版本切换到官方版本

**修改内容**:
```json
// README.OpenSource
{
  "Name": "PyYAML",  // 从 "openEuler:PyYAML" 改回
  "Version Number": "6.0.2",
  "Upstream URL": "https://github.com/yaml/pyyaml",  // 从 openEuler 改回官方
  ...
}
```

```toml
# pyproject.toml
[build-system]
requires = [
  "setuptools",
  "wheel",
  "Cython; python_version < '3.13'",
  "Cython>=3.0; python_version >= '3.13'"  # Python 3.13 使用 Cython 3.x
]
backend-path = ["packaging"]
build-backend = "_pyyaml_pep517"  # OH 添加的构建后端
```

```python
# packaging/_pyyaml_pep517.py (OH 新增)
import inspect

def _bridge_build_meta():
    import functools
    import sys
    from setuptools import build_meta
    # 桥接 setuptools.build_meta 接口
    for attr_name in build_meta.__all__:
        attr_value = getattr(build_meta, attr_name)
        if callable(attr_value):
            setattr(self_module, attr_name, functools.partial(_expose_config_settings, attr_value))

# 支持通过 pyyaml_build_config 传递构建选项
class ActiveConfigSettings:
    _current = {}
    # ...
```

```python
# tests/legacy_tests/conftest.py (OH 新增)
# pytest 自定义收集器，将旧测试框架适配到 pytest
class PyYAMLCollector(pytest.Collector):
    def collect(self):
        # 将旧测试用例转换为 pytest items
        # ...
```

**OH 需求**:
1. 支持最新 Python 版本 (3.13)
2. 支持最新 Cython 版本 (3.x)
3. 支持配置化构建（通过 `pyyaml_build_config` 参数）
4. 将测试框架从 unittest 适配到 pytest

**关键代码变更**:
- 版本号更新至 6.0.2
- 添加 PEP 517 构建后端 (`_pyyaml_pep517.py`)
- 支持条件 Cython 依赖（Python 3.13 使用 Cython>=3.0）
- 改进 LibYAML 构建脚本
- 添加 pytest 测试适配器

**回归风险**: 低，主要是构建工具和测试框架的改进。

**升级建议**:
- ✅ 支持最新的 Python 3.13 和 Cython 3.x
- ✅ PEP 517 构建后端符合现代 Python 打包标准
- ✅ pytest 适配便于持续集成
- ⚠️ 新增的 `_pyyaml_pep517.py` 需要维护

---

### Patch 6: 部件名修正

**提交**: `d806d6d`, `2dfd1da`
**修改文件**: `bundle.json`

**原始问题**: 部件命名不统一，需要遵循 OpenHarmony 的命名规范

**修改内容**:
```json
{
  "component": {
    "name": "PyYAML"  // 从 "pyyaml" 改回 "PyYAML"
  }
}
```

**OH 需求**: 统一 OpenHarmony 组件命名规范（建议首字母大写）。

**回归风险**: 无，仅影响构建系统的组件名显示。

**升级建议**: 保持当前的 "PyYAML" 命名，符合 OH 规范。

---

## OH 特定宏使用

**搜索结果**: 未发现任何 OpenHarmony 特定的宏定义。

检查内容:
- `#ifdef OHOS`
- `#ifdef OPENHARMONY`
- `#ifdef __OHOS__`

**说明**: PyYAML 是纯 Python 库，没有使用 C 预处理器宏进行平台适配。所有 OH 适配都通过配置文件和构建工具实现。

---

## Patch 影响范围分析

### 修改文件分类

```
配置类文件 (OH 特有):
  ├── bundle.json                    # OH 部件配置
  ├── README.OpenSource              # 开源属性声明
  └── OAT.xml                       # 开源合规检查配置

版本相关文件:
  ├── lib/yaml/__init__.py           # 版本号定义
  ├── setup.py                       # Python 包配置
  ├── pyproject.toml                 # PEP 517 项目配置
  └── CHANGES                        # 上游变更日志

OH 特有文件:
  ├── packaging/_pyyaml_pep517.py   # PEP 517 构建后端
  ├── packaging/build/libyaml.sh      # LibYAML 构建脚本
  └── tests/legacy_tests/conftest.py  # pytest 适配器

核心代码文件 (上游同步):
  └── lib/yaml/*.py                 # 核心库文件（与上游保持一致）
```

### 关键修改点总结

| 修改点 | 类型 | OH 价值 | 回归风险 |
|--------|------|---------|---------|
| bundle.json | 配置 | 组件声明 | 低 |
| OAT.xml | 配置 | 开源合规 | 低 |
| README.OpenSource | 配置 | 来源追踪 | 低 |
| 版本号 | 同步 | 安全更新 | 中 |
| _pyyaml_pep517.py | OH 特有 | 构建灵活性 | 低 |
| pytest 适配 | OH 特有 | 测试现代化 | 低 |

---

## 升级建议

### 推送向上游的 Patch

以下 OH 特有的改进可以考虑推向上游：

1. **PEP 517 构建后端** (`packaging/_pyyaml_pep517.py`)
   - 价值: 支持配置化构建，便于交叉编译
   - 可行性: 高，符合 PEP 517 标准
   - 建议: 向上游提交 PR

2. **pytest 适配器** (`tests/legacy_tests/conftest.py`)
   - 价值: 现代化测试框架
   - 可行性: 高，符合测试框架趋势
   - 建议: 向上游提交 PR

### OH 特有的 Patch

以下 Patch 不适合推向上游，需要 OH 维护：

1. **bundle.json** - OpenHarmony 特有的组件声明格式
2. **OAT.xml** - OpenHarmony 特有的开源合规配置
3. **README.OpenSource** - OpenHarmony 特有的开源属性声明

### 版本升级建议

1. **定期同步上游**: 关注 PyYAML 的安全公告和版本更新
2. **全面回归测试**: 每次升级后运行完整的测试套件
3. **验证 C 扩展**: 确保 LibYAML C 扩展在各平台正常编译
4. **更新版本号**: 同步更新 bundle.json、README.OpenSource、__init__.py 中的版本号

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - PyYAML 原始库简介
- [03_Build_Integration.md](./03_Build_Integration.md) - OpenHarmony 构建适配详细分析
- [06_Security.md](./06_Security.md) - 安全风险分析
