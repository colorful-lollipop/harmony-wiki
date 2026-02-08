# 安全评审（Security）

> 本文档对 OpenHarmony 升级包制作工具进行安全风险评审，识别攻击面、分析信任边界、列出安全风险并提供修复建议。本评审基于代码证据，不做无根据的猜测。

## 1 评审范围与方法

### 1.1 评审范围

| 评审对象 | 说明 |
|---------|-----|
| packaging_tools 核心模块 | build_update.py、image_class.py、update_package.py 等 |
| 命令行参数处理 | 参数解析、文件路径处理 |
| 签名和哈希机制 | PKCS7 签名、RSA/ECC 签名、SHA256/SHA384 哈希 |
| 文件操作 | 镜像文件读写、临时文件处理 |
| 外部工具调用 | bsdiff、imgdiff、e2fsdroid |

### 1.2 评审范围外

| 排除项 | 说明 |
|-------|-----|
| 第三方工具内部实现 | bsdiff、imgdiff、e2fsdroid 源码 |
| 测试代码 | test/ 目录下的测试代码 |
| 上游组件 | updater、recovery 等组件 |
| 运行时环境 | 设备端的升级执行环境 |

### 1.3 评审方法

| 方法 | 说明 |
|-----|-----|
| 代码审查 | 静态阅读代码，识别潜在风险点 |
| 数据流分析 | 分析输入数据的处理流程 |
| 威胁建模 | 识别攻击面和信任边界 |
| 最佳实践对比 | 对比业界安全最佳实践 |

## 2 攻击面清单

### 2.1 命令行接口

| 攻击面 | 说明 | 风险等级 |
|-------|-----|---------|
| 位置参数 | target_package、update_package 路径 | 中 |
| -s/--source_package | 源包文件路径 | 中 |
| -pk/--private_key | 私钥文件路径 | 高 |
| -pf/--partition_file | 分区配置文件路径 | 中 |

**证据来源**：

```python
# build_update.py: 参数定义
-s SOURCE_PACKAGE, --source_package SOURCE_PACKAGE
-pk PRIVATE_KEY, --private_key PRIVATE_KEY
-pf PARTITION_FILE, --partition_file PARTITION_FILE
```

### 2.2 文件输入

| 攻击面 | 说明 | 风险等级 |
|-------|-----|---------|
| 目标镜像文件 | raw/sparse 格式镜像 | 中 |
| 源镜像文件 | 差分升级的源镜像 | 中 |
| 分区配置文件 | XML 格式分区定义 | 中 |
| 私钥文件 | PEM 格式密钥文件 | 高 |
| 升级脚本配置 | 升级操作定义 | 中 |

### 2.3 外部工具接口

| 攻击面 | 说明 | 风险等级 |
|-------|-----|---------|
| bsdiff 子进程 | 差分计算进程 | 中 |
| imgdiff 子进程 | 差分计算进程 | 中 |
| e2fsdroid 子进程 | map 文件生成进程 | 低 |

**证据来源**：

```markdown
// README_zh.md:58-62
- bsdiff可执行程序，差分计算，比较生成patch；
- imgdiff可执行程序，差分计算，针对zip、gz、lz4类型的文件，对比生成patch；
- e2fsdroid可执行程序，差分计算，用于生成镜像的map文件。
```

### 2.4 输出

| 攻击面 | 说明 | 风险等级 |
|-------|-----|---------|
| 升级包文件 | 最终生成的 ZIP 包 | 中 |
| 临时文件 | 差分计算中间文件 | 中 |
| 日志输出 | 错误和调试信息 | 低 |

## 3 信任边界分析

### 3.1 信任边界图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         信任边界分析                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                      不可信区域                               │   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐     │   │
│   │  │命令行参数 │  │用户提供  │  │外部工具  │  │网络输入  │     │   │
│   │  │用户输入   │  │镜像文件  │  │输出     │  │         │     │   │
│   │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘     │   │
│   └────────┼────────────┼────────────┼────────────┼───────────┘   │
│            │            │            │            │                │
│            ▼            ▼            ▼            ▼                │
│   ════════════════════════════════════════════════════════════    │
│   │                     信任边界                                    │   │
│   │         packaging_tools 工具处理边界                           │   │
│   ════════════════════════════════════════════════════════════    │
│            │                                                       │
│            ▼                                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                      可信区域                                 │   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │   │
│   │  │内部模块  │  │签名验证  │  │输出升级包│  │日志系统  │       │   │
│   │  │代码     │  │逻辑     │  │         │  │         │       │   │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 信任边界说明

| 边界 | 说明 | 进入条件 |
|-----|-----|---------|
| 工具边界 | packaging_tools 处理边界 | 命令行调用 |
| 文件边界 | 文件系统访问边界 | 文件路径参数 |
| 进程边界 | 外部工具调用边界 | 子进程创建 |
| 数据边界 | 内部数据处理边界 | 数据传递 |

### 3.3 数据流与信任传递

```
输入数据 → 参数解析 → 文件读取 → 数据处理 → 签名计算 → 输出生成
   ↓          ↓          ↓          ↓          ↓          ↓
 不可信     边界1      边界2      边界3      边界4      边界5
```

## 4 安全风险清单

### 4.1 私钥文件路径遍历风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-001 |
| 风险类型 | 路径遍历（Path Traversal） |
| 严重程度 | 高（High） |
| 可利用性 | 中（Medium） |
| 证据位置 | `build_update.py` 参数 `-pk` |
| 证据内容 | 私钥文件路径直接传递给工具函数 |

**风险描述**：

命令行参数 `-pk/--private_key` 接收私钥文件路径，工具直接使用该路径读取文件。如果攻击者能够控制路径参数，可能导致**任意文件读取**。

**触发条件**：

```bash
# 恶意路径示例
python build_update.py ./target/ ./output/ -pk ../../../etc/passwd
python build_update.py ./target/ ./output/ -pk /etc/shadow
```

**影响范围**：

- 读取系统敏感文件（passwd、shadow、配置文件等）
- 泄露私钥文件内容（如果路径正确）
- 可能导致权限提升

**修复建议**：

```python
# 建议 1：路径规范化验证
import os
def validate_key_path(key_path):
    # 转换为绝对路径
    abs_path = os.path.abspath(key_path)
    # 验证路径是否在允许的目录内
    allowed_dir = os.path.dirname(os.path.abspath(__file__))
    if not abs_path.startswith(allowed_dir):
        raise ValueError("私钥路径超出允许范围")
    return abs_path

# 建议 2：检查路径遍历字符
def sanitize_path(path):
    # 移除 "../" 和 "..\\"
    cleaned = os.path.normpath(path)
    if ".." in cleaned.split(os.sep):
        raise ValueError("路径中包含非法字符")
    return cleaned
```

### 4.2 分区配置文件注入风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-002 |
| 风险类型 | XML 外部实体注入（XXE） |
| 严重程度 | 高（High） |
| 可利用性 | 中（Medium） |
| 证据位置 | `utils.py` 使用 xmltodict 解析 XML |
| 证据内容 | 分区配置文件使用 XML 解析 |

**风险描述**：

`-pf/--partition_file` 参数接收分区配置文件路径，使用 `xmltodict` 解析 XML。如果 XML 文件包含**外部实体声明（XXE）**，可能导致：

- 任意文件读取
- SSRF（服务器端请求伪造）
- 服务拒绝（DoS）

**证据来源**：

```markdown
// README_zh.md:52
- python库xmltodict， 解析xml文件，需要单独安装；
```

**触发条件**：

```xml
<!-- 恶意分区配置文件示例 -->
<?xml version="1.0"?>
<!DOCTYPE partition [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<partition>
  <name>&xxe;</name>
</partition>
```

**影响范围**：

- 读取系统敏感文件
- 可能泄露配置信息
- 目标设备信息收集

**修复建议**：

```python
# 方案 1：禁用 XML 外部实体
# xmltodict 默认不启用 XXE，但应明确禁用

import xmltodict
from defusedxml import ElementTree

def parse_partition_file(file_path):
    # 使用 defusedxml 库防止 XXE
    with open(file_path, 'rb') as f:
        # 禁用外部实体
        data = ElementTree.fromstring(f.read())
        return xmltodict.parse(ElementTree.tostring(data))

# 方案 2：使用安全的 XML 解析配置
parser = xml.etree.ElementTree.XMLParser(resolve_entities=False)
```

### 4.3 差分计算临时文件安全风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-003 |
| 风险类型 | 符号链接攻击（Symlink Attack） |
| 严重程度 | 中（Medium） |
| 可利用性 | 低（Low） |
| 证据位置 | `patch_package_process.py` 临时文件处理 |
| 证据内容 | 差分计算过程中创建临时文件 |

**风险描述**：

差分计算过程中，工具可能创建临时文件存储中间结果。如果攻击者能够预测临时文件路径并创建符号链接，可能导致：

- 任意文件写入
- 文件覆盖
- 权限提升

**触发条件**：

```bash
# 攻击者提前创建符号链接
ln -s /etc/passwd /tmp/patch_temp_xxx
# 然后触发差分计算
```

**影响范围**：

- 覆盖系统文件
- 修改配置文件
- 可能导致系统不稳定

**修复建议**：

```python
import tempfile
import os

def create_secure_temp_file():
    # 使用 mkstemp 创建安全的临时文件
    fd, path = tempfile.mkstemp(
        prefix='patch_',
        suffix='.tmp',
        dir=tempfile.gettempdir()
    )
    # 设置严格的文件权限
    os.chmod(path, 0o600)
    os.close(fd)
    return path

# 禁止使用符号链接
def check_no_symlink(file_path):
    if os.path.islink(file_path):
        raise SecurityError("不允许使用符号链接")
    return True
```

### 4.4 命令注入风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-004 |
| 风险类型 | 命令注入（Command Injection） |
| 严重程度 | 高（High） |
| 可利用性 | 低（Low） |
| 证据位置 | `patch_package_process.py` 外部工具调用 |
| 证据内容 | 使用 subprocess 调用 bsdiff/imgdiff |

**风险描述**：

工具使用 `subprocess` 调用外部差分工具（bsdiff、imgdiff）。如果参数未经严格过滤，可能导致**命令注入**攻击。

**证据来源**：

```markdown
// README_zh.md:58-60
- bsdiff可执行程序，差分计算，比较生成patch；
- imgdiff可执行程序，差分计算，针对zip、gz、lz4类型的文件，对比生成patch；
```

**触发条件**：

```bash
# 恶意镜像文件名
touch "--help"; python build_update.py -s "--help" ./target/ ./output/
```

**影响范围**：

- 执行任意命令
- 获取系统权限
- 数据泄露

**修复建议**：

```python
import subprocess
import shlex

def safe_call_diff_tool(tool_path, source, target, output):
    # 方案 1：使用列表参数，避免 shell=True
    cmd = [
        tool_path,
        source,
        target,
        output
    ]
    # 验证工具路径
    if not os.path.isabs(tool_path):
        raise ValueError("工具路径必须是绝对路径")
    
    # 验证输出路径
    output_dir = os.path.dirname(output)
    if not os.path.exists(output_dir):
        os.makedirs(output_dir, mode=0o755)
    
    # 执行命令（不使用 shell=True）
    subprocess.run(cmd, check=True)

# 禁止使用 shell=True
# 错误示例：subprocess.run(f"{tool} {source} {target}", shell=True)
```

### 4.5 升级包签名验证缺失风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-005 |
| 风险类型 | 签名验证缺失 |
| 严重程度 | 中（Medium） |
| 可利用性 | 中（Medium） |
| 证据位置 | `build_pkcs7.py` 签名生成逻辑 |
| 证据内容 | 工具仅生成签名，不验证签名有效性 |

**风险描述**：

工具主要用于**生成**升级包和签名，但缺少对签名有效性的验证步骤。可能导致：

- 使用过期或撤销的密钥签名
- 签名算法配置错误
- 私钥文件路径泄露

**证据来源**：

```markdown
// README_zh.md:30
build_pkcs7.py              # 升级包签名
```

**触发条件**：

```bash
# 使用过期密钥
python build_update.py -pk expired_key.pem ./target/ ./output/

# 使用弱签名算法
python build_update.py -ha sha1 ./target/ ./output/
```

**影响范围**：

- 签名无效的升级包可能被接受
- 密钥管理风险
- 签名验证失败导致升级失败

**修复建议**：

```python
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding

def validate_private_key(key_path):
    """验证私钥有效性"""
    with open(key_path, 'rb') as f:
        private_key = serialization.load_pem_private_key(
            f.read(),
            password=None
        )
    
    # 检查密钥类型
    if isinstance(private_key, rsa.RSAPrivateKey):
        key_size = private_key.key_size
        if key_size < 2048:
            raise ValueError(f"RSA 密钥大小不足: {key_size} < 2048")
    
    # 建议使用强哈希算法
    return private_key

def validate_signature_algorithm(algo):
    """验证签名算法"""
    allowed_algos = ['sha256', 'sha384']
    if algo not in allowed_algos:
        raise ValueError(f"不支持的签名算法: {algo}")
    return True
```

### 4.6 敏感信息日志泄露风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-006 |
| 风险类型 | 信息泄露 |
| 严重程度 | 低（Low） |
| 可利用性 | 高（High） |
| 证据位置 | `log_exception.py` 日志系统 |
| 证据内容 | 全局日志系统可能记录敏感信息 |

**风险描述**：

工具包含全局日志系统（`log_exception.py`），可能在日志中输出：

- 文件路径信息
- 私钥文件路径
- 配置参数
- 错误详情

**证据来源**：

```markdown
// README_zh.md:34
log_exception.py            # 全局log系统定义，自定义exception
```

**触发条件**：

```bash
# 默认日志级别可能输出敏感信息
python build_update.py ./target/ ./output/ -pk /path/to/key.pem 2>&1 | tee debug.log
```

**影响范围**：

- 私钥路径泄露
- 目录结构泄露
- 攻击面信息收集

**修复建议**：

```python
import logging

# 配置日志过滤器，过滤敏感信息
class SensitiveDataFilter(logging.Filter):
    SENSITIVE_PATTERNS = [
        'private_key',
        'password',
        'secret',
        '/path/to/key',
    ]
    
    def filter(self, record):
        msg = record.getMessage()
        for pattern in self.SENSITIVE_PATTERNS:
            if pattern in msg:
                record.msg = '*** REDACTED ***'
                return False
        return True

# 应用过滤器
logger = logging.getLogger(__name__)
logger.addFilter(SensitiveDataFilter())
```

### 4.7 资源耗尽风险

| 属性 | 值 |
|-----|-----|
| 风险编号 | SEC-007 |
| 风险类型 | 拒绝服务（DoS） |
| 严重程度 | 中（Medium） |
| 可利用性 | 中（Medium） |
| 证据位置 | `image_class.py` 镜像解析 |
| 证据内容 | 处理超大镜像文件可能导致内存耗尽 |

**风险描述**：

工具在处理镜像文件时可能加载整个文件到内存，对于超大镜像文件可能导致：

- 内存耗尽（OOM）
- 系统响应变慢
- 服务拒绝

**触发条件**：

```bash
# 处理超大镜像文件（数十 GB）
python build_update.py ./huge_target/ ./output/
```

**影响范围**：

- 系统资源耗尽
- 其他进程受影响
- 升级制作失败

**修复建议**：

```python
def check_image_size(file_path, max_size_gb=10):
    """检查镜像文件大小"""
    max_size = max_size_gb * 1024 * 1024 * 1024
    file_size = os.path.getsize(file_path)
    if file_size > max_size:
        raise ValueError(
            f"镜像文件过大: {file_size / (1024**3):.2f}GB > {max_size_gb}GB"
        )
    return True

# 流式读取大文件
def stream_read_image(file_path, chunk_size=8192):
    """流式读取镜像文件，避免一次性加载"""
    with open(file_path, 'rb') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk
```

## 5 安全加固建议

### 5.1 输入验证加固

| 加固项 | 优先级 | 说明 |
|-------|-------|-----|
| 路径遍历检测 | 高 | 检查 `../` 等遍历字符 |
| 路径规范化 | 高 | 使用 `os.path.abspath()` 规范化 |
| 路径白名单 | 中 | 限制可访问的目录范围 |
| 文件类型验证 | 中 | 验证文件魔数（Magic Number） |

### 5.2 输出安全加固

| 加固项 | 优先级 | 说明 |
|-------|-------|-----|
| 临时文件安全 | 高 | 使用 `mkstemp()`，设置权限 0o600 |
| 符号链接检查 | 中 | 禁止使用符号链接 |
| 输出路径验证 | 高 | 验证输出路径不在敏感目录 |
| 日志脱敏 | 中 | 过滤敏感信息 |

### 5.3 进程安全加固

| 加固项 | 优先级 | 说明 |
|-------|-------|-----|
| 命令注入防护 | 高 | 禁止 `shell=True`，使用列表参数 |
| 工具路径验证 | 高 | 验证工具路径为绝对路径或白名单 |
| 子进程资源限制 | 中 | 设置 CPU、内存、文件大小限制 |

### 5.4 签名安全加固

| 加固项 | 优先级 | 说明 |
|-------|-------|-----|
| 密钥大小检查 | 高 | RSA 至少 2048 位 |
| 哈希算法限制 | 高 | 禁用 SHA1，使用 SHA256/SHA384 |
| 密钥有效期检查 | 中 | 检查密钥是否过期 |
| 签名验证 | 高 | 生成签名后验证其有效性 |

## 6 安全检查清单

### 6.1 开发阶段检查

| 检查项 | 状态 | 说明 |
|-------|------|-----|
| 所有外部输入已验证 | ⬜ 待检查 | 参数、文件、网络输入 |
| 路径操作已规范化 | ⬜ 待检查 | 使用 `os.path` 系列函数 |
| 命令调用已安全化 | ⬜ 待检查 | 禁止 shell=True |
| 敏感信息已脱敏 | ⬜ 待检查 | 日志不包含密钥、密码 |
| 临时文件已安全创建 | ⬜ 待检查 | 使用 mkstemp |

### 6.2 发布阶段检查

| 检查项 | 状态 | 说明 |
|-------|------|-----|
| 密钥管理流程已定义 | ⬜ 待检查 | 密钥存储、轮换、备份 |
| 签名验证已实现 | ⬜ 待检查 | 端侧签名验证 |
| 安全配置已应用 | ⬜ 待检查 | 默认安全配置 |
| 安全测试已通过 | ⬜ 待检查 | 渗透测试、代码审计 |

## 7 相关文档

| 文档 | 说明 |
|-----|-----|
| 00_Overview.md | 项目概览 |
| 02_Architecture.md | 架构说明 |
| 03_Usage.md | 使用说明 |
| 01_Directory_Structure.md | 模块职责 |
