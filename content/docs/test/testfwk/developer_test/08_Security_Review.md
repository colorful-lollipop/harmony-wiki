# 安全风险评审

## 8.1 评审概述

本文档对 Developer Test Framework 进行安全风险分析，涵盖外部输入、权限控制、数据处理等方面。

### 评审范围

| 组件 | 说明 |
|------|------|
| `src/core/` | 核心框架代码 |
| `config/` | 配置文件 |
| `aw/` | 静态测试库 |
| `libs/` | 测试支持库 |

### 评审方法

- 代码静态分析
- 配置审计
- 依赖检查
- 攻击面分析

## 8.2 攻击面分析

### 8.2.1 外部输入点

| 输入类型 | 来源 | 风险等级 |
|---------|------|---------|
| XML 配置文件 | `config/*.xml` | 中 |
| 用户命令行参数 | `sys.argv` | 低 |
| 设备返回数据 | HDC/串口 | 中 |
| 文件路径 | BUILD.gn, 用例配置 | 中 |
| 网络数据 | NFS, HDC | 中 |

### 8.2.2 敏感操作

| 操作 | 说明 | 风险 |
|------|------|------|
| 文件写入 | 报告生成、日志 | 中 |
| 命令执行 | GN 编译 | 高 |
| 设备通信 | HDC/串口 | 中 |
| 资源推送 | NFS 挂载 | 中 |

## 8.3 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  内部组件 (高信任)                                   │    │
│  │  - src/core/build/*  (构建管理)                      │    │
│  │  - src/core/config/* (配置解析)                     │    │
│  │  - aw/cxx/*        (测试库)                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                            ↓                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  边界层 (中等信任)                                   │    │
│  │  - XDevice (外部依赖)                               │    │
│  │  - 设备端执行结果                                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                            ↓                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  外部 (低信任)                                       │    │
│  │  - 用户配置文件                                      │    │
│  │  - 设备返回数据                                      │    │
│  │  - NFS 网络数据                                     │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## 8.4 安全风险清单

### 8.4.1 风险 1: XML 外部实体注入 (XXE)

**风险等级**: 中

**证据**: `src/core/config/config_manager.py` 使用 `xml.etree.ElementTree` 解析 XML

**触发场景**:
```python
# config_manager.py 可能存在 XXE 风险
import xml.etree.ElementTree as ET
tree = ET.parse(config_file)  # 未禁用外部实体
root = tree.getroot()
```

**影响**: 攻击者可通过恶意 XML 文件读取本地文件或发起 SSRF 攻击

**修复建议**:
```python
# 使用安全的解析方式
from defusedxml import ElementTree
tree = ElementTree.parse(config_file)
```

**状态**: ⚠️ 需确认

---

### 8.4.2 风险 2: 命令注入

**风险等级**: 高

**证据**: `src/core/build/build_manager.py:62-76` 使用 `subprocess` 执行 GN 编译

**触发场景**:
```python
# build_manager.py:66
with os.fdopen(os.open(filepath, FLAGS, MODES), 'w') as gn_file:
    gn_file.write("deps += [\n")
    for target in target_list:
        if target:
            gn_file.write("    \"%s\",\n" % target)  # 未经校验的目标名
```

**影响**: 攻击者可注入恶意命令执行

**修复建议**:
```python
# 目标名白名单校验
ALLOWED_TARGET_PATTERN = re.compile(r'^[a-zA-Z0-9_-]+$')
for target in target_list:
    if target and ALLOWED_TARGET_PATTERN.match(target):
        gn_file.write("    \"%s\",\n" % target)
```

**状态**: ⚠️ 需确认

---

### 8.4.3 风险 3: 路径遍历

**风险等级**: 中

**证据**: `src/core/config/resource_manager.py` 处理资源路径

**触发场景**:
```python
# 可能存在的路径遍历
def push_resource(src_path, dest_path):
    shutil.copy(src_path, dest_path)  # 未经校验的路径
```

**影响**: 攻击者可写入任意文件

**修复建议**:
```python
# 路径规范化与校验
def safe_path(base, path):
    full_path = os.path.normpath(os.path.join(base, path))
    if not full_path.startswith(os.path.normpath(base)):
        raise SecurityError("Path traversal detected")
    return full_path
```

**状态**: ⚠️ 需确认

---

### 8.4.4 风险 4: 设备序列号注入

**风险等级**: 中

**证据**: `config/user_config.xml:32` 接受设备序列号

**触发场景**:
```xml
<device type="usb-hdc">
  <sn>'; rm -rf /; echo '</sn>  <!-- 恶意输入 -->
</device>
```

**影响**: 可能导致设备选择逻辑错误

**修复建议**:
```python
# 参数校验
def validate_sn(sn):
    if not re.match(r'^[a-zA-Z0-9]+$', sn):
        raise ConfigError("Invalid SN format")
    return sn
```

**状态**: ⚠️ 需确认

---

### 8.4.5 风险 5: 敏感信息泄露

**风险等级**: 中

**证据**: `config/user_config.xml` 可能包含设备 IP、端口等信息

**触发场景**:
```xml
<device type="usb-hdc">
  <ip>192.168.1.100</ip>  <!-- 内部网络信息 -->
  <sn>device_secret</sn>
</device>
```

**影响**: 敏感配置信息可能被提交到版本控制

**修复建议**:
1. 添加 `.gitignore` 忽略敏感配置文件
2. 提供配置文件模板，不包含实际值
3. 使用环境变量替代敏感信息

**状态**: ✅ 已有建议（README 中建议）

---

### 8.4.6 风险 6: NFS 挂载安全

**风险等级**: 中

**证据**: `config/user_config.xml:55-59` NFS 配置

**触发场景**:
```xml
<NFS>
  <host_dir>/shared/nfs</host_dir>  <!-- 可能指向敏感目录 -->
</NFS>
```

**影响**: 测试用例可能访问/修改敏感目录

**修复建议**:
```python
# NFS 路径白名单
NFS_ALLOWED_PATHS = ['/home/test/nfs', '/data/nfs']

def validate_nfs_path(path):
    normalized = os.path.normpath(path)
    if not any(normalized.startswith(p) for p in NFS_ALLOWED_PATHS):
        raise ConfigError("NFS path not allowed")
```

**状态**: ⚠️ 需确认

---

### 8.4.7 风险 7: Python 代码注入

**风险等级**: 高

**证据**: `src/core/command/console.py:93` 参数解析

**触发场景**:
```python
# 参数注入
options = parser.parse_args(sys.argv)  # 恶意参数
```

**影响**: 通过 `--testcase` 等参数注入代码

**修复建议**:
```python
# 参数白名单校验
VALID_TESTCASE_PATTERN = re.compile(r'^[a-zA-Z0-9_.]+$')

def validate_testcase(tc_name):
    if not VALID_TESTCASE_PATTERN.match(tc_name):
        raise ParamError("Invalid testcase name")
```

**状态**: ⚠️ 需确认

---

### 8.4.8 风险 8: 日志注入

**风险等级**: 低

**证据**: `src/core/driver/drivers.py:79` 日志输出

**触发场景**:
```python
# 日志注入
def __read__(self, output):
    self.output = "%s%s" % (self.output, output)  # 未经处理的输出
```

**影响**: 恶意日志可能包含 ANSI 转义码或注入字符

**修复建议**:
```python
# 日志清洗
import re

def sanitize_log(text):
    # 移除 ANSI 转义码
    ansi_escape = re.compile(r'\x1B(?:[@-Z\\-_]|\[[0-?]*[ -/]*[@-~])')
    return ansi_escape.sub('', text)
```

**状态**: ⚠️ 需确认

---

## 8.5 已有的安全措施

### 8.5.1 代码签名

所有源码文件包含 Apache License 2.0 头：
```python
# Copyright (c) 2021-2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0
```

### 8.5.2 依赖管理

框架依赖的第三方组件：
- `googletest` - Google 测试框架
- `paramiko` - SSH 库（使用前需验证版本）
- `XDevice` - OpenHarmony 测试调度框架

### 8.5.3 异常处理

```python
# src/core/exception.py
class ParamError(Exception):
    """参数错误"""
```

## 8.6 安全建议

### 8.6.1 高优先级

1. **参数校验**: 所有用户输入必须校验
2. **路径安全**: 使用白名单校验文件路径
3. **命令安全**: 使用参数列表而非字符串拼接

### 8.6.2 中优先级

1. **XML 解析**: 使用 `defusedxml` 替代 `xml.etree`
2. **日志清洗**: 移除 ANSI 转义码和恶意字符
3. **配置隔离**: 敏感配置独立文件，不提交版本控制

### 8.6.3 低优先级

1. **安全审计**: 定期进行安全代码审计
2. **依赖扫描**: 使用工具扫描依赖漏洞
3. **文档更新**: 及时更新安全相关文档

## 8.7 局限性说明

### 8.7.1 检查范围限制

- 本评审仅覆盖 `developer_test` 模块
- 未覆盖 `XDevice` 外部依赖
- 未覆盖设备端测试用例安全

### 8.7.2 工具限制

- 静态分析可能遗漏运行时问题
- 配置检查基于默认配置模板

### 8.7.3 建议补充

1. 运行时安全测试
2. 渗透测试
3. 依赖漏洞扫描

## 8.8 相关文档

- [01_Overview.md](01_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 系统架构
- [04_Configuration.md](04_Configuration.md) - 配置说明
- [06_Usage_Guide.md](06_Usage_Guide.md) - 使用指南
