# OH 构建适配

本文档说明 cangjie_tools 在 OpenHarmony 中的构建系统适配。

---

## 重要发现：不使用 GN 构建系统

cangjie_tools **不使用 OpenHarmony 主构建系统的 GN/Ninja**，而是采用 **Python 构建脚本 + CMake** 的混合构建架构。

---

## 构建文件结构

### 顶层结构

```
cangjie_tools/
├── bundle.json                    # OpenHarmony 组件定义
├── README.md                      # 项目文档
├── cjpm/                         # 项目管理工具（Cangjie）
│   ├── build/build.py            # Python 构建脚本
│   └── src/                      # 源代码
├── cjfmt/                        # 代码格式化工具（C++）
│   ├── build/build.py            # Python 构建包装器
│   ├── CMakeLists.txt            # CMake 主配置
│   └── src/                      # 源代码
├── cjlint/                       # 代码检查工具（C++）
│   ├── build/build.py
│   ├── CMakeLists.txt
│   └── src/
├── cjcov/                        # 代码覆盖率工具（Cangjie）
│   ├── build/build.py
│   └── src/
├── cangjie-language-server/      # 语言服务器（C++）
│   ├── build/build.py
│   ├── CMakeLists.txt
│   └── src/
├── cjtrace-recover/              # 异常栈恢复工具（Cangjie+C++）
│   ├── build/build.py
│   ├── CMakeLists.txt
│   └── src/
├── hyperlangExtension/           # 跨语言互操作工具（Cangjie）
│   ├── build/build.py
│   └── src/
└── third_party/                  # 第三方依赖
    └── demangler/                # demangler 库
        └── CMakeLists.txt
```

---

## Python 构建脚本清单

| 文件路径 | 工具 | 编程语言 | 说明 |
|---------|------|---------|------|
| `cjpm/build/build.py` | cjpm | Python | 项目管理工具构建脚本 |
| `cjfmt/build/build.py` | cjfmt | Python | 代码格式化工具构建包装器 |
| `cjlint/build/build.py` | cjlint | Python | 代码检查工具构建包装器 |
| `cjcov/build/build.py` | cjcov | Python | 代码覆盖率工具构建脚本 |
| `cangjie-language-server/build/build.py` | lsp | Python | 语言服务器构建脚本 |
| `cjtrace-recover/build/build.py` | cjtrace-recover | Python | 异常栈恢复工具构建脚本 |
| `hyperlangExtension/build/build.py` | hle | Python | 跨语言互操作工具构建脚本 |

---

## CMakeLists.txt 文件清单

| 文件路径 | 工具 | 编程语言 | 说明 |
|---------|------|---------|------|
| `cjfmt/CMakeLists.txt` | cjfmt | C++17 | 代码格式化工具主 CMake 配置 |
| `cjlint/CMakeLists.txt` | cjlint | C++17 | 代码检查工具主 CMake 配置 |
| `cangjie-language-server/CMakeLists.txt` | lsp | C++17 | 语言服务器主 CMake 配置 |
| `cjtrace-recover/CMakeLists.txt` | cjtrace-recover | C++17 + Cangjie | 异常栈恢复工具主 CMake 配置 |
| `third_party/demangler/CMakeLists.txt` | demangler | C++ | demangler 库 CMake 配置 |

---

## OpenHarmony 组件配置

### bundle.json

**文件路径**：`./bundle.json`

**关键配置**：
```json
{
    "name": "@ohos/cangjie_tools",
    "description": "The Cangjie language provides a rich set of command line tools...",
    "version": "6.1",
    "license": "Apache-2.0 with Runtime Library Exceptions",
    "publishAs": "code-segment",
    "segment": {
        "destPath": "third_party/cangjie_tools"
    },
    "component": {
        "name": "cangjie_tools",
        "subsystem": "thirdparty",
        "syscap": [],
        "features": [],
        "adapted_system_type": ["standard"],
        "deps": {
            "components": [
                "flatbuffers",
                "json",
                "sqlite"
            ]
        }
    }
}
```

**说明**：
- **组件名称**：`@ohos/cangjie_tools`
- **子系统**：`thirdparty`
- **适用系统类型**：`standard`
- **依赖的 OH 组件**：flatbuffers, json, sqlite

---

## 各构建脚本关键配置

### 1. cjpm/build/build.py（Cangjie 项目管理工具）

**构建类型**：纯 Cangjie 语言编译（使用 cjc 编译器）

**关键环境变量**：
- `CANGJIE_HOME`：Cangjie SDK 路径
- `CANGJIE_STDX_PATH`：Cangjie 标准库路径

**支持平台**：
- native（Linux, macOS, Windows）
- windows-x86_64（交叉编译到 Windows）

**编译选项**：
```python
--trimpath={CURRENT_DIR}  # 去除路径信息
-g                      # Debug 模式
-O2                     # 优化级别（Release）
```

**安全链接选项（Linux）**：
```python
--link-options="-z noexecstack -z relro -z now -s"
```

**安全链接选项（macOS）**：
```python
--link-options="-rpath {rpath}"
```

**安全链接选项（Windows）**：
```python
--link-options=--no-insert-timestamp  # 可重现构建
```

**静态库生成**：
```
libcjpm.toml.a
libcjpm.util.a
libcjpm.config.a
libcjpm.implement.a
libcjpm.command.a
```

**依赖的 stdx 库**：
```
stdx.logger, stdx.log
stdx.encoding.json.stream
stdx.serialization.serialization
stdx.encoding.json, stdx.encoding.url
```

---

### 2. cjfmt/build/build.py + CMakeLists.txt（代码格式化工具）

**构建类型**：CMake + Ninja/MinGW，C++ 项目

**支持目标平台**：
```python
choices=['native', 'windows-x86_64', 'ohos-x86_64', 'ohos-aarch64']
```

**OpenHarmony 交叉编译配置**：

| 目标 | C 编译器 | C++ 编译器 |
|------|----------|-----------|
| ohos-x86_64 | `x86_64-unknown-linux-ohos-clang` | `x86_64-unknown-linux-ohos-clang++` |
| ohos-aarch64 | `aarch64-unknown-linux-ohos-clang` | `aarch64-unknown-linux-ohos-clang++` |

**CMake 关键配置**（`cjfmt/CMakeLists.txt`）：

- C++ 标准：C++17
- 版本号：`1.1.0-alpha.69`
- 安全编译选项（Release）：
  ```cmake
  -D_FORTIFY_SOURCE=2 -O2
  -fstack-protector-all -ftrapv -fPIE
  ```
- 链接器选项（Linux）：
  ```cmake
  -s -Wl,-z,relro,-z,now,-z,noexecstack
  ```
- 预定义宏：
  ```cmake
  DCJFMT_VERSION
  DCANGJIE_CODEGEN_CJNATIVE_BACKEND
  DNDEBUG（非 Debug 模式）
  ```
- 代码覆盖率选项：`CJFMT_CODE_COVERAGE`

---

### 3. cjlint/build/build.py + CMakeLists.txt（代码检查工具）

**构建类型**：CMake + Ninja/MinGW，C++ 项目

**支持目标平台**：
```python
choices=['native', 'windows-x86_64']
```

**第三方依赖**：自动下载 `json-v3.11.3`（从 OpenHarmony 仓库）
```python
JSON_GIT = "https://gitcode.com/openharmony/third_party_json.git"
cmd = ["git", "clone", "-b", "OpenHarmony-v6.0-Release", "--depth=1", JSON_GIT, "json-v3.11.3"]
```

**CMake 关键配置**（`cjlint/CMakeLists.txt`）：

- 版本号：`1.1.0-alpha.69`
- 预定义宏：
  ```cmake
  DCANGJIE_AST2CHIR
  DCANGJIE_CODEGEN_CJNATIVE_BACKEND
  DCJLINT_VERSION
  DCJC_VERSION（自动检测）
  DCANGJIE_WIN_NATIVE_DEV（Windows 原生开发）
  D__ARM__（ARM 架构）
  ```
- 安全编译选项与 cjfmt 类似

---

### 4. cjcov/build/build.py（代码覆盖率工具）

**构建类型**：纯 Cangjie 语言编译

**关键环境变量**：
- `CANGJIE_HOME`：Cangjie SDK 路径
- `CANGJIE_STDX_PATH`：Cangjie 标准库路径

**支持平台**：
- native（Linux, macOS, Windows）
- windows-x86_64（交叉编译到 Windows）

**静态库生成**：
```
libcjcov.util.a
libcjcov.core.a
```

**安全链接选项（Linux）**：
```python
--link-options="-z noexecstack -z relro -z now -s"
```

---

### 5. cangjie-language-server/build/build.py + CMakeLists.txt（语言服务器）

**构建类型**：CMake，C++ 项目

**第三方依赖**（从 OpenHarmony 仓库下载）：

| 依赖 | 用途 | OpenHarmony 仓库 |
|------|------|----------------|
| json-v3.11.3 | JSON 解析 | `third_party_json` |
| flatbuffers | 序列化/反序列化 | `third_party_flatbuffers` |
| sqlite3 | 索引存储 | `third_party_sqlite` |

**下载分支**：`OpenHarmony-v6.0-Release`

**CMake 关键配置**：

- C++ 标准：C++17
- 预定义宏：
  ```cmake
  DCANGJIE_CODEGEN_CJNATIVE_BACKEND
  DNOT_RELEASE_VERSION_CODE
  DDEBUG（Debug 模式）
  DTEST_FLAG（启用测试）
  DMACRO_DYNAMIC
  ```
- 支持 Windows 交叉编译（`CROSS_WINDOWS`）

---

### 6. cjtrace-recover/build/build.py + CMakeLists.txt（异常栈恢复工具）

**构建类型**：CMake + Ninja，Cangjie + C++ 混合项目

**CMake 关键配置**：

- C++ 标准：C++17
- 交叉编译支持：通过 toolchain 文件
- 自定义 CMake 模块：`CangjieTarget`
- 外部项目依赖：`cangjie-demangler`
- 安全编译选项：
  ```cmake
  -fstack-protector-all -fPIE -D_FORTIFY_SOURCE=2 -ftrapv
  ```
- 平台特定链接选项：
  ```cmake
  Linux: -z noexecstack -z relro -z now -s
  macOS: -lc++ -lc++abi
  Windows: -lc++ -lunwind
  ```

---

### 7. hyperlangExtension/build/build.py（跨语言互操作工具）

**构建类型**：纯 Cangjie 语言编译

**关键环境变量**：
- `CANGJIE_HOME`：Cangjie SDK 路径
- `CANGJIE_STDX_PATH`：Cangjie 标准库路径

**静态库生成**：
```
libhle.dtsparser.a
libhle.tool.a
libhle.entry.a
```

**生成的代码包含 OpenHarmony 特定导入**：
```cangjie
import ohos.ark_interop.*
import ohos.ark_interop_helper.*
import ohos.base.*
```

---

## OpenHarmony 特定配置

### 1. 目标平台支持

| 工具 | 支持的目标平台 |
|------|--------------|
| cjpm | native, windows-x86_64 |
| cjfmt | native, windows-x86_64, **ohos-x86_64, ohos-aarch64** |
| cjlint | native, windows-x86_64 |
| cjcov | native, windows-x86_64 |
| lsp | native, windows-x86_64 |
| cjtrace-recover | native（支持 toolchain 交叉编译） |
| hle | native, windows-x86_64 |

### 2. OpenHarmony 特定编译器

在 `cjfmt/build/build.py` 中定义了 OHOS 专用编译器：
```python
# ohos-x86_64
'-DCMAKE_C_COMPILER=' + 'x86_64-unknown-linux-ohos-clang'
'-DCMAKE_CXX_COMPILER=' + 'x86_64-unknown-linux-ohos-clang++'

# ohos-aarch64
'-DCMAKE_C_COMPILER=' + 'aarch64-unknown-linux-ohos-clang'
'-DCMAKE_CXX_COMPILER=' + 'aarch64-unknown-linux-ohos-clang++'
```

### 3. OpenHarmony 第三方依赖

所有第三方依赖均从 OpenHarmony 官方仓库下载，使用 `OpenHarmony-v6.0-Release` 分支：

| 依赖 | OpenHarmony 仓库地址 |
|------|---------------------|
| json | `https://gitcode.com/openharmony/third_party_json.git` |
| flatbuffers | `https://gitcode.com/openharmony/third_party_flatbuffers.git` |
| sqlite | `https://gitcode.com/openharmony/third_party_sqlite.git` |

---

## 安全编译选项（全平台通用）

### 编译选项

```bash
-fstack-protector-all  # 栈保护
-ftrapv                # 整数溢出检测
-fPIE                  # 位置无关可执行文件
-D_FORTIFY_SOURCE=2     # 缓冲区溢出保护
```

### 链接选项（Linux）

```bash
-s                        # 去除符号表
-Wl,-z,relro            # 只读重定位
-Wl,-z,now              # 立即绑定
-Wl,-z,noexecstack      # 不可执行栈
-Wl,-Bsymbolic           # 符号本地绑定
-rdynamic                # 导出动态符号
-Wl,--no-undefined     # 未定义符号报错
```

### 链接选项（Windows）

```bash
--no-insert-timestamp   # 可重现构建
```

---

## 与上游构建系统的差异

该仓库是 OpenHarmony 第三方库，**没有明显的上游构建系统**。其构建系统特点：

1. **独立于 OpenHarmony 主构建系统**：不使用 GN/Ninja，而是自定义的 Python + CMake
2. **自包含构建**：每个工具子目录包含独立的构建脚本
3. **OpenHarmony 集成点**：
   - `bundle.json` 定义 OH 组件信息
   - 支持 `ohos-x86_64` 和 `ohos-aarch64` 目标平台
   - 依赖 OH 官方第三方库（json, flatbuffers, sqlite）

---

## 构建依赖关系

### bundle.json 依赖

```json
"deps": {
    "components": [
        "flatbuffers",
        "json",
        "sqlite"
    ]
}
```

### 各工具依赖关系

```
cangjie_tools
├── cjpm (Cangjie)
│   └── stdx (logger, log, encoding.json, ...)
├── cjfmt (C++17)
│   └── Cangjie SDK
├── cjlint (C++17)
│   └── JSON for Modern C++
├── cjcov (Cangjie)
│   └── stdx
├── lsp (C++17)
│   ├── flatbuffers
│   ├── JSON for Modern C++
│   └── SQLite
├── cjtrace-recover (C++17 + Cangjie)
│   ├── demangler
│   └── Cangjie SDK
└── hle (Cangjie)
    └── stdx
```

---

## 构建流程

### Cangjie 项目（cjpm, cjcov, hle）

1. 检查环境变量（`CANGJIE_HOME`, `CANGJIE_STDX_PATH`）
2. 使用 `cjc` 编译器编译源文件
3. 生成静态库或可执行文件
4. 链接 stdx 库
5. 输出到 `bin/` 和 `dist/` 目录

### C++ 项目（cjfmt, cjlint, lsp, cjtrace-recover）

1. Python 脚本调用 CMake 生成构建系统
2. 下载第三方依赖（如需）
3. 使用 Ninja/MinGW 编译
4. 生成可执行文件
5. 输出到 `build/build/bin/` 或 `dist/` 目录

---

## 集成到 OpenHarmony 主构建系统

### 当前状态

**未集成**：cangjie_tools 当前独立于 OpenHarmony 主构建系统（GN/Ninja）。

### 集成方案（如需）

如需将 cangjie_tools 集成到 OH 主构建系统，需要：

1. **创建 BUILD.gn 文件**：
   - 定义 `ohos_executable` 或 `ohos_shared_library` 模板
   - 指定源文件、依赖、编译选项

2. **调用现有构建系统**：
   - 通过 GN 的 `action` 模板调用 Python 构建脚本
   - 或使用 GN 的 `group` 模块包装现有构建产物

3. **处理第三方依赖**：
   - 将 flatbuffers, json, sqlite 依赖转换为 GN 的 `external_deps`

4. **示例 BUILD.gn 结构**：
   ```gn
   import("//build/ohos.gni")

   ohos_executable("cjpm") {
     sources = [...]
     deps = [":cjpm_static_libs"]
     external_deps = [ "cangjie_stdx:cangjie_stdx" ]
     defines = [ "OHOS_TARGET" ]
   }

   group("cangjie_tools") {
     deps = [
       ":cjpm",
       ":cjfmt",
       ":lsp",
       ...
     ]
   }
   ```

---

## 常见问题

### Q1: 为什么不使用 GN 构建系统？

**A**：cangjie_tools 是华为自研的 Cangjie 语言工具链，有自己的构建系统（Python + CMake）。不使用 GN 的原因可能包括：

1. Cangjie 编译器（cjc）有自己的编译流程
2. 第三方依赖管理需要特定逻辑
3. 构建脚本需要下载和管理 OH 特定版本的三方库

### Q2: 如何构建 OHOS 目标平台？

**A**：以 cjfmt 为例：

```bash
# 构建 OHOS x86_64 目标
python3 cjfmt/build/build.py build -t release --target ohos-x86_64

# 构建 OHOS aarch64 目标
python3 cjfmt/build/build.py build -t release --target ohos-aarch64 --target-sysroot /path/to/ohos/sysroot
```

### Q3: 如何验证构建配置？

**A**：验证方法：

1. **检查编译选项**：
   ```bash
   grep -r "CMAKE_C_COMPILER\|CMAKE_CXX_COMPILER" cjfmt/build/
   ```

2. **验证宏定义**：
   ```bash
   grep -r "add_definitions\|-D" cjfmt/CMakeLists.txt
   ```

3. **编译并检查产物**：
   ```bash
   python3 cjfmt/build/build.py build -t release
   file dist/bin/cjfmt
   ```

---

## 总结

### 构建系统特点

1. **独立构建系统**：不使用 GN，采用 Python + CMake 混合架构
2. **多语言支持**：Cangjie 工具使用 cjc 编译；C++ 工具使用 CMake
3. **OH 目标支持**：cjfmt 支持 `ohos-x86_64` 和 `ohos-aarch64`
4. **第三方依赖管理**：自动从 OH 官方仓库下载特定版本的三方库
5. **安全编译**：全平台应用安全编译选项

### 关键差异

| 方面 | OpenHarmony 主构建系统 | cangjie_tools |
|------|----------------------|---------------|
| 构建工具 | GN + Ninja | Python + CMake |
| 配置文件 | BUILD.gn | build.py + CMakeLists.txt |
| 组件定义 | bundle.json | bundle.json |
| OH 集成 | 深度集成 | 独立构建 |

---

## 参考文档

- [README.md](./README.md) - 库概览和 OH 适配概述
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估结果
- [01_Overview.md](./01_Overview.md) - 原始库简介
- [../README.md](../README.md) - 项目主文档
- [../third_party/README.md](../third_party/README.md) - 第三方依赖说明
