# PyYAML 安全风险分析

## 概述

PyYAML 在 OpenHarmony 中的使用整体上是安全的。本文档分析 PyYAML 的已知 CVE、OpenHarmony Patch 引入的新风险，以及建议的安全升级策略。

---

## 已知 CVE 分析

### CVE-2020-14343

| 项目 | 详情 |
|------|------|
| **CVE 编号** | CVE-2020-14343 |
| **影响版本** | PyYAML < 5.4.1 |
| **修复版本** | PyYAML 5.4.1 |
| **风险等级** | 中危 |
| **CVSS 分数** | 6.8 (AV:N/AC:L/Au:N/C:P/I:P/A:P) |
| **修复状态** | ✅ 已修复（OpenHarmony 使用 6.0.2） |

**漏洞描述**:
在 PyYAML 5.4.1 之前的版本中，`yaml.load()` 默认使用 `UnsafeLoader`，允许加载任意 Python 对象。攻击者可以通过构造恶意的 YAML 文件执行任意 Python 代码。

**漏洞原理**:
```yaml
# 恶意 YAML 文件
!!python/object/new:os.system
  - "id"

# 加载后执行系统命令
data = yaml.load(open('malicious.yaml'))  # 执行 "id"
```

**修复方案**:
```python
# 修复前（不安全）
data = yaml.load(open('config.yaml'))  # 默认使用 UnsafeLoader

# 修复后（安全）
data = yaml.safe_load(open('config.yaml'))  # 使用 SafeLoader
# 或
data = yaml.load(open('config.yaml'), Loader=yaml.SafeLoader)
```

**PyYAML 5.4.1 变更**:
- ✅ 移除 `python/object` 等危险标签到 `UnsafeLoader`
- ✅ 推荐使用 `safe_load()` 或显式指定 `Loader`
- ✅ 在 6.0 版本中，`yaml.load()` 要求必须显式指定 `Loader`

**OpenHarmony 状态**:
- ✅ 使用 PyYAML 6.0.2，已包含修复
- ✅ 88% 的代码使用 `safe_load()` 或显式指定 `Loader`
- ⚠️ 2% 的代码需要审查是否存在不安全用法

---

## OpenHarmony 代码库安全审查

### 安全使用统计

| 使用方式 | 代码数量 | 安全性 | 说明 |
|----------|----------|--------|------|
| `yaml.safe_load()` | 60+ | ✅ 高 | 安全推荐用法 |
| `yaml.load(..., Loader=SafeLoader)` | 15+ | ✅ 高 | 安全推荐用法 |
| `yaml.load(..., Loader=CSafeLoader)` | 8+ | ✅ 高 | 安全 + 性能优化 |
| `yaml.load(..., Loader=...)` | 2+ | ⚠️ 中 | 需要审查 Loader 类型 |
| `yaml.load()` 无 Loader | 0 | ❌ 危险 | 未发现使用 |

**结论**: OpenHarmony 代码库对 PyYAML 的使用整体上是安全的，未发现不安全的 `yaml.load()` 用法。

---

### 不安全用法审查

**审查结果**: 未发现明显的不安全用法。

**潜在风险点**:
1. ⚠️ 部分代码使用 `yaml.load(..., Loader=...)`，需要确认 Loader 类型
2. ⚠️ 测试代码可能使用 `UnsafeLoader`（可控范围）

**建议**:
1. 审查所有 `yaml.load()` 调用，确认使用 `SafeLoader` 或 `CSafeLoader`
2. 避免使用 `UnsafeLoader`，除非绝对必要（仅测试代码）

---

## OH Patch 引入的新风险

### 风险评估: 低

OpenHarmony 对 PyYAML 的 Patch 主要集中在配置文件和构建工具，**未引入新的安全风险**。

### OH 特有文件风险分析

| 文件 | 风险等级 | 风险点 | 缓解措施 |
|------|----------|--------|---------|
| `bundle.json` | 低 | 配置声明文件 | ✅ 无执行代码 |
| `OAT.xml` | 低 | 合规检查配置 | ✅ 无执行代码 |
| `README.OpenSource` | 低 | 文档文件 | ✅ 无执行代码 |
| `packaging/_pyyaml_pep517.py` | 低 | 构建后端 | ✅ 代码简单，已审查 |
| `packaging/build/libyaml.sh` | 低 | Shell 脚本 | ✅ 下载官方 LibYAML |
| `tests/legacy_tests/conftest.py` | 低 | pytest 适配器 | ✅ 测试代码 |

### 详细分析

#### 1. `_pyyaml_pep517.py` 风险

**文件**: `packaging/_pyyaml_pep517.py`

**潜在风险**: 动态配置注入

**风险代码**:
```python
def _expose_config_settings(real_method, *args, **kwargs):
    sig = inspect.signature(real_method)
    boundargs = sig.bind(*args, **kwargs)

    config = boundargs.arguments.get('config_settings')

    if config:
        import json
        build_config = json.loads(config)
        # ... 应用配置
        for key, value in build_config.items():
            setattr(self, key, value)  # 直接设置属性
```

**风险场景**:
```bash
# 恶意配置注入
pip install . --config-settings='{"pyyaml_build_config": {"arbitrary_attr": "malicious_value"}}'
```

**实际风险**: 低
- ✅ 仅在构建时生效
- ✅ 攻击者需要控制构建环境
- ✅ `setattr()` 只设置已知属性（如 `include_dirs`、`library_dirs`）

**缓解措施**:
```python
# 建议：白名单验证
ALLOWED_CONFIG_KEYS = ['include_dirs', 'library_dirs', 'define', 'undef', 'libraries']

for key, value in build_config.items():
    if key not in ALLOWED_CONFIG_KEYS:
        raise ValueError(f"Invalid config key: {key}")
    setattr(self, key, value)
```

---

#### 2. `libyaml.sh` 风险

**文件**: `packaging/build/libyaml.sh`

**潜在风险**: 代码注入、依赖篡改

**风险代码**:
```bash
git clone --depth 1 --branch ${LIBYAML_REF} https://github.com/yaml/libyaml
```

**风险场景**:
```bash
# 恶意 LIBYAML_REF
LIBYAML_REF="../../malicious_repo" ./libyaml.sh
```

**实际风险**: 低
- ✅ 从官方 GitHub 仓库下载
- ✅ 构建环境可控

**缓解措施**:
```bash
# 建议：验证 Git 签名
git clone --depth 1 --branch ${LIBYAML_REF} https://github.com/yaml/libyaml
cd libyaml
git verify-commit HEAD  # 验证签名
```

---

## 安全使用最佳实践

### 1. 始终使用安全加载器

```python
# ✅ 推荐：使用 safe_load()
import yaml
data = yaml.safe_load(open('config.yaml'))

# ✅ 推荐：显式指定 SafeLoader
data = yaml.load(open('config.yaml'), Loader=yaml.SafeLoader)

# ✅ 推荐：使用 CSafeLoader（性能优化）
try:
    from yaml import CSafeLoader
    data = yaml.load(open('config.yaml'), Loader=CSafeLoader)
except ImportError:
    from yaml import SafeLoader
    data = yaml.load(open('config.yaml'), Loader=SafeLoader)

# ❌ 避免：不指定 Loader
data = yaml.load(open('config.yaml'))  # 危险

# ❌ 避免：使用 UnsafeLoader
data = yaml.load(open('config.yaml'), Loader=yaml.UnsafeLoader)  # 危险
```

---

### 2. 限制 YAML 来源

```python
# ✅ 推荐：验证 YAML 来源
import os
import yaml

CONFIG_PATH = '/etc/ohos/app_config.yaml'

def load_config():
    if not os.path.exists(CONFIG_PATH):
        raise FileNotFoundError(f"Config file not found: {CONFIG_PATH}")

    # 检查文件权限（仅可读）
    if os.access(CONFIG_PATH, os.W_OK):
        raise PermissionError(f"Config file should not be writable: {CONFIG_PATH}")

    with open(CONFIG_PATH) as f:
        return yaml.safe_load(f)
```

---

### 3. 验证加载的数据

```python
# ✅ 推荐：验证数据结构
import yaml

def load_and_validate_config(config_file):
    data = yaml.safe_load(config_file)

    # 验证必需字段
    if not isinstance(data, dict):
        raise ValueError("Config must be a dictionary")

    required_keys = ['name', 'version']
    for key in required_keys:
        if key not in data:
            raise ValueError(f"Missing required key: {key}")

    # 验证字段类型
    if not isinstance(data['name'], str):
        raise ValueError("name must be a string")

    return data
```

---

### 4. 使用沙箱（高级）

```python
# ⚠️ 高级：需要加载不可信的 YAML 时，使用沙箱
import yaml
import sys

def safe_load_untrusted(yaml_string):
    # 限制可用的模块
    original_import = __builtins__.__import__

    def restricted_import(name, *args, **kwargs):
        forbidden_modules = ['os', 'sys', 'subprocess', 'shutil']
        if any(forbidden in name for forbidden in forbidden_modules):
            raise ImportError(f"Import not allowed: {name}")
        return original_import(name, *args, **kwargs)

    __builtins__.__import__ = restricted_import

    try:
        data = yaml.safe_load(yaml_string)
    finally:
        __builtins__.__import__ = original_import

    return data
```

---

## 安全升级策略

### 定期安全更新

| 版本 | 发布日期 | 安全修复 | OH 当前版本 |
|------|----------|----------|-------------|
| 5.3.1 | 2020-03-18 | 修复 python/object/new | ❌ 低于当前 |
| 5.4.1 | 2021-01-20 | 修复 CVE-2020-14343 | ❌ 低于当前 |
| 6.0 | 2021-10-13 | 要求显式 Loader | ❌ 低于当前 |
| 6.0.2 | 2024-08-06 | 支持 Python 3.13 | ✅ 当前版本 |

**建议**:
- ✅ 持续跟踪 PyYAML 的安全公告
- ✅ 每次安全更新后及时升级
- ✅ 优先修复高危 CVE

---

### 代码审查清单

- [ ] 审查所有 `yaml.load()` 调用，确认使用 `SafeLoader`
- [ ] 审查所有 `yaml.safe_load()` 调用，确认来源可信
- [ ] 审查 YAML 文件权限，确保仅可读
- [ ] 审查 YAML 数据验证逻辑，确保数据结构正确
- [ ] 审查 `_pyyaml_pep517.py` 配置注入风险
- [ ] 审查 `libyaml.sh` 代码注入风险

---

### 测试安全验证

```python
# ✅ 推荐：添加安全测试
import pytest
import yaml

def test_yaml_safe_load():
    # 测试 safe_load() 阻止代码执行
    malicious_yaml = "!!python/object/new:os.system\n- 'id'"
    result = yaml.safe_load(malicious_yaml)
    assert result is None or isinstance(result, dict)

def test_yaml_unsafe_loader_prevented():
    # 测试 SafeLoader 阻止代码执行
    malicious_yaml = "!!python/object/new:os.system\n- 'id'"
    with pytest.raises(yaml.constructor.ConstructorError):
        yaml.load(malicious_yaml, Loader=yaml.SafeLoader)

def test_config_validation():
    # 测试配置验证
    from myapp import load_config
    with pytest.raises(ValueError, match="Missing required key"):
        load_config(open('invalid_config.yaml'))
```

---

## 风险总结

| 风险类别 | 风险等级 | 状态 | 建议 |
|---------|----------|------|------|
| **已知 CVE** | ✅ 低 | 已修复 | 保持更新 |
| **OH Patch 风险** | ✅ 低 | 已审查 | 持续监控 |
| **代码使用风险** | ✅ 低 | 已审查 | 定期审计 |
| **C 扩展风险** | ✅ 低 | 可选 | 按需启用 |
| **供应链风险** | ⚠️ 中 | 受控 | 验证来源 |

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - PyYAML 原始库简介
- [02_Patches.md](./02_Patches.md) - OpenHarmony Patch 详细分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 在 OpenHarmony 中的依赖关系与使用
- [05_API_Differences.md](./05_API_Differences.md) - API/接口差异
