# API/接口差异说明

## 概述

Python 3.11.4 在 OpenHarmony 中的主要差异体现在**模块可用性**和**平台特定功能**上。大部分标准 API 保持一致，但部分模块被禁用。

## 禁用模块 API

以下模块在 OHOS 构建中被禁用，相关 API 不可用:

### 1. `_uuid` 模块

**禁用 API:**
```python
import uuid  # 可用，但底层 _uuid 加速被禁用

# 以下仍然可用 (纯 Python 实现)
uuid.uuid1()
uuid.uuid4()
uuid.uuid5()

# 性能影响: UUID 生成可能较慢
```

**替代方案**: 使用标准库 `uuid` 模块，它提供纯 Python 回退实现。

### 2. `_socket` 模块

**禁用 API:**
```python
import socket  # 底层 _socket 模块被禁用

# 受影响功能:
socket.socket()  # 可能无法使用
socket.getaddrinfo()  # 可能无法使用
```

**影响**: 网络编程能力受限

**替代方案**: 使用 OHOS 原生网络 API (C++ ArkTS 绑定)

### 3. `zlib` 模块

**禁用 API:**
```python
import zlib  # 完全禁用

zlib.compress(data)  # 不可用
zlib.decompress(data)  # 不可用
```

**替代方案**:
- 使用外部命令行工具 (`gzip`, `pigz`)
- 使用其他压缩库 (如 `lzma` 如果可用)

### 4. `_ctypes` 模块

**禁用 API:**
```python
import ctypes  # 底层 _ctypes 被禁用

ctypes.CDLL()  # 不可用
ctypes.c_int()  # 不可用
```

**影响**: 无法调用 C 动态库

**替代方案**: 使用 Python C API 编写原生扩展模块

### 5. `binascii` 模块

**禁用 API:**
```python
import binascii  # 完全禁用

binascii.hexlify(data)  # 不可用
binascii.unhexlify(data)  # 不可用
binascii.a2b_base64()  # 不可用
```

**替代方案**:
```python
# hexlify/unhexlify
hex_str = data.hex()
bytes_obj = bytes.fromhex(hex_str)

# base64
import base64
base64.b64encode(data)
base64.b64decode(data)
```

## MinGW Patch 新增 API

### 1. 路径分隔符处理函数

**Include/pylifecycle.h 新增:**

```c
// 获取替代路径分隔符
PyAPI_FUNC(wchar_t) Py_GetAltSepW(const wchar_t *path);

// 获取主路径分隔符  
PyAPI_FUNC(wchar_t) Py_GetSepW(const wchar_t *path);
PyAPI_FUNC(char) Py_GetSepA(const char *path);

// 规范化路径分隔符
PyAPI_FUNC(void) Py_NormalizeSepsW(wchar_t *path);
PyAPI_FUNC(void) Py_NormalizeSepsA(char *path);

// 扩展的路径规范化
PyAPI_FUNC(void) Py_NormalizeSepsPathcchW(wchar_t *path);
```

**用途**: 在 MinGW/MSYS2 环境中正确处理路径分隔符

### 2. Cygwin/MSYS2 PTY 检测

**Include/iscygpty.h 新增:**

```c
#ifdef _WIN32
int is_cygpty(int fd);
int is_cygpty_used(void);
#else
#define is_cygpty(fd)       0
#define is_cygpty_used()    0
#endif
```

**用途**: 检测是否在 MSYS2/Cygwin 伪终端中运行

## 行为变更

### 1. sysconfig 路径方案

**变更**: MinGW 环境使用 Unix 风格路径

```python
import sysconfig

# MinGW 环境下
sysconfig.get_path('purelib')
# 上游: C:\Python311\Lib\site-packages
# OH/MinGW: /mingw64/lib/python3.11/site-packages
```

### 2. distutils 编译器检测

**变更**: 改进 MinGW 编译器检测

```python
from distutils import cygwinccompiler

# 自动检测并使用现代 MinGW-w64 工具链
# 支持 Clang/LLVM MinGW
```

### 3. 站点包路径

**变更**: MinGW 环境使用 POSIX 风格站点包路径

```python
import site

# MinGW 环境下
site.getsitepackages()
# 返回: ['/mingw64/lib/python3.11/site-packages']
```

## 平台检测差异

### sys.platform

| 环境 | sys.platform |
|-----|-------------|
| 标准 Windows | 'win32' |
| MinGW | 'win32' (但 'GCC' in sys.version) |
| MinGW MSYS2 | 'win32' (MSYSTEM 环境变量设置) |
| OHOS 构建主机 | 'linux' 或 'darwin' |

### 检测代码示例

```python
import sys
import os

# 检测 MinGW 构建
is_mingw = 'GCC' in sys.version and sys.platform == 'win32'

# 检测 MSYS2 环境
is_msys2 = os.environ.get('MSYSTEM', '') != ''

# 检测 POSIX 风格构建 (MinGW)
is_posix_build = os.name == 'posix' or (os.name == 'nt' and 'GCC' in sys.version)
```

## 兼容性建议

### 1. 编写兼容代码

```python
import sys
import os

# 检查模块可用性
try:
    import zlib
    HAS_ZLIB = True
except ImportError:
    HAS_ZLIB = False

# 替代实现
def compress_data(data):
    if HAS_ZLIB:
        return zlib.compress(data)
    else:
        # 使用其他方式或抛出异常
        raise NotImplementedError("zlib not available")
```

### 2. 路径处理

```python
import os
import sys

# 使用 pathlib 处理跨平台路径
from pathlib import Path

path = Path('/some/path') / 'file.txt'
# 自动适配平台路径分隔符

# 避免硬编码分隔符
# 不推荐: path = '/some/path/file.txt'
# 推荐: path = os.path.join('some', 'path', 'file.txt')
```

### 3. 网络编程

由于 `_socket` 可能被禁用，建议:

```python
# 检查网络支持
try:
    import socket
    HAS_SOCKET = True
except ImportError:
    HAS_SOCKET = False

# 或使用高级 HTTP 库 (如果可用)
try:
    import urllib.request
    HAS_URLLIB = True
except ImportError:
    HAS_URLLIB = False
```

## 文档说明

### 注意事项

1. **模块禁用是构建时决定的**: 无法在运行时启用被禁用的模块
2. **纯 Python 替代方案**: 某些模块有纯 Python 实现作为回退
3. **平台差异**: MinGW 环境引入的行为差异主要针对 Windows 构建

### 与上游兼容性

- 标准库 API 保持兼容
- C API 保持兼容 (新增函数除外)
- 字节码格式兼容
- 模块格式 (.pyd/.so) 与平台相关
