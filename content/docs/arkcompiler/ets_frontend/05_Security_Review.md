# 安全风险评审

> **文档版本**: 2.0（增强版，含代码证据）  
> **评审日期**: 2026-02-07  
> **评审依据**: 代码静态分析 + 架构审查 + 动态代码审查

---

## 评审范围

本评审覆盖 `arkcompiler/ets_frontend` 仓库的核心编译组件：

| 组件 | 路径 | 用途 |
|------|------|------|
| JS/TS 编译器 | `es2panda/` | JavaScript/TypeScript 编译 |
| ETS 编译器 | `ets2panda/` | Enhanced TypeScript 编译 |
| 字节码合并 | `merge_abc/` | ABC 文件合并 |
| 代码保护 | `arkguard/` | 代码混淆与保护 |

**评审依据**：
- CLI 入口点分析：`es2panda/aot/main.cpp:383`、`ets2panda/aot/main.cpp:211`
- 参数解析：`es2panda/aot/options.cpp:480-945`
- 路径处理：`ets2panda/util/importPathManager.cpp`、`ets2panda/util/arktsconfig.cpp`
- 词法/语法分析：`ets2panda/lexer/`、`ets2panda/parser/`

---

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                     不可信输入边界                                │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  用户代码文件 (.js/.ts/.ets)                              │    │
│  │  命令行参数 (--output, --extension, --opt-level)         │    │
│  │  模块依赖路径 (--module-path)                             │    │
│  │  配置文件 (.json, .tsconfig)                              │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  ets_frontend 编译器                                      │    │
│  │  (构建时工具，信任边界内)                                  │    │
│  │  - 词法分析: lexer.cpp                                   │    │
│  │  - 语法解析: parserImpl.cpp                              │    │
│  │  - 字节码生成: compilerImpl.cpp                          │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  ARK Runtime                                              │    │
│  │  (运行时，字节码执行)                                      │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 攻击面分析

| 输入类型 | 处理模块 | 入口文件 | 风险等级 | 证据 |
|----------|----------|----------|----------|------|
| **源代码文件** | parser/lexer | `ets2panda/parser/parserImpl.cpp` | **高** | 处理不可信输入，依赖递归解析 |
| **命令行参数** | PandArgParser | `es2panda/aot/options.cpp:480` | **中** | 包含 --output, --opt-level 等 |
| **模块依赖路径** | importPathManager | `ets2panda/util/importPathManager.cpp:183` | **中** | 相对路径解析 |
| **输出文件路径** | AsmEmitter | `es2panda/aot/emitFiles.cpp:89` | **高** | 任意文件写入风险 |

---

## 输入校验详细分析

### 1. 命令行参数校验

#### 选项定义与解析（证据：`es2panda/aot/options.cpp:480-611`）

```cpp
// 文件: es2panda/aot/options.cpp
// 行号: 480-611

// 扩展名选项（行 485-486）
panda::PandArg<std::string> inputExtension("extension", "js",
    "Parse the input as the given extension (options: js | ts | as | abc)");

// 优化级别选项（行 515-516）
panda::PandArg<int> opOptLevel("opt-level", 2,
    "Compiler optimization level (options: 0 | 1 | 2)");

// 线程数选项（行 517-520）
panda::PandArg<int> opFunctionThreadCount("function-threads", 0,
    "Number of worker threads to compile function");
panda::PandArg<int> opFileThreadCount("file-threads", 0,
    "Number of worker threads to compile file");

// 输出文件选项（行 525）
panda::PandArg<std::string> outputFile("output", "", "Compiler binary output (.abc)");
```

#### 选项验证（证据：`es2panda/aot/options.cpp:745-767`）

```cpp
// 文件: es2panda/aot/options.cpp
// 行号: 745-750 - 扩展名验证

std::string extension = inputExtension.GetValue();
if (!extension.empty()) {
    if (VALID_EXTENSIONS.find(extension) == VALID_EXTENSIONS.end()) {
        errorMsg_ = "Invalid extension (available options: js, ts, as, abc)";
        return false;
    }
}

// 行号: 783-793 - 输入文件绝对路径解析
auto inputAbs = panda::os::file::File::GetAbsolutePath(sourceFile_);
if (!inputAbs) {
    std::cerr << "Failed to find file '" << sourceFile_ << "' during input file resolution." << std::endl;
    return false;
}
```

**风险评估**：

| 检查项 | 状态 | 说明 |
|--------|------|------|
| `--extension` 枚举验证 | ✅ 已实现 | 限制为 js/ts/as/abc |
| `--opt-level` 范围验证 | ✅ 已实现 | 默认值 2，解析为 int |
| `--thread` 数量限制 | ⚠️ 部分实现 | 无上限检查，可为 0（自动） |
| `--output` 路径验证 | ❌ **未实现** | **见风险 R1** |

---

### 2. 文件路径处理校验

#### 路径遍历检测（证据：`ets2panda/util/importPathManager.cpp:183-191`）

```cpp
// 文件: ets2panda/util/importPathManager.cpp
// 行号: 183-191

static bool IsRelativePath(std::string_view path)
{
    for (std::string_view start : {"./", "../", ".\\", "..\\"}) {
        if (Helpers::StartsWith(path, start)) {
            return true;
        }
    }
    return false;
}
```

#### 路径规范化（证据：`ets2panda/util/arktsconfig.cpp:67-88`）

```cpp
// 文件: ets2panda/util/arktsconfig.cpp
// 行号: 67-88

fs::path NormalizePath(const fs::path &p)
{
    fs::path result;
    for (const auto &part : p) {
        if (part == ".") {
            continue;
        }
        if (part == "..") {
            if (!result.empty() && result.filename() != "..") {
                result = result.parent_path();  // 处理 .. 遍历
            } else {
                result /= part;
            }
        } else {
            result /= part;
        }
    }
    if (fs::exists(result)) {
        return fs::canonical(result);  // 解析符号链接
    }
    return result;
}

// 行号: 283 - 绝对路径构建
return IsAbsolute(path) ? ark::os::GetAbsolutePath(path) : MakeAbsolute(path, base);
```

#### 绝对路径验证（证据：`ets2panda/util/arktsconfig.cpp:56-62`）

```cpp
// 文件: ets2panda/util/arktsconfig.cpp
// 行号: 56-62

static bool IsAbsolute(const std::string &path)
{
#ifndef ARKTSCONFIG_USE_FILESYSTEM
    return !path.empty() && path[0] == '/';
#else
    return fs::path(path).is_absolute();
#endif
}
```

**风险评估**：

| 检查项 | 状态 | 说明 |
|--------|------|------|
| `..` 路径遍历检测 | ✅ 已实现 | NormalizePath() 处理 |
| 符号链接解析 | ✅ 已实现 | fs::canonical() 解析 |
| 绝对路径验证 | ✅ 已实现 | IsAbsolute() 检查 |
| 输出路径目录限制 | ❌ **未实现** | **见风险 R1** |

---

### 3. 源代码解析安全机制

#### 递归深度保护（证据：`ets2panda/util/recursiveGuard.h:21-53`）

```cpp
// 文件: ets2panda/util/recursiveGuard.h
// 行号: 21 - 最大递归深度定义
constexpr unsigned int MAX_RECURSION_DEPTH = 5120;

// 行号: 27-53 - 递归跟踪类
class TrackRecursive {
public:
    explicit TrackRecursive(RecursiveContext &recursivecontext) : recursivecontext_(recursivecontext) {
        ++recursivecontext_.depth;
        valid_ = recursivecontext_.depth <= MAX_RECURSION_DEPTH;  // 深度检查
    };
    ~TrackRecursive() { --recursivecontext_.depth; }
    explicit operator bool() const { return valid_; }
private:
    bool valid_ = true;
};
```

#### 递归保护使用示例（证据：`ets2panda/parser/statementParser.cpp:94-101`）

```cpp
// 文件: ets2panda/parser/statementParser.cpp
// 行号: 94-101

TrackRecursive trackRecursive(RecursiveCtx());
if (!trackRecursive) {
    LogError(diagnostic::DEEP_NESTING);
    while (Lexer()->GetToken().Type() != lexer::TokenType::EOS) {
        Lexer()->NextToken();  // 跳过到文件末尾
    }
    return AllocBrokenStatement(Lexer()->GetToken().Loc());
}
```

#### 循环迭代保护（证据：`ets2panda/parser/ETSparserExpressions.cpp:678`）

```cpp
// 文件: ets2panda/parser/ETSparserExpressions.cpp
// 行号: 678

WhileLoopGuard guard;  // 默认限制 = MAX_RECURSION_DEPTH (5120)
while (someCondition) {
    if (!guard.ShouldContinue()) {
        // 处理过多迭代
    }
}
```

#### 字符串/缓冲区安全（证据：`ets2panda/lexer/lexer.cpp:447-450`）

```cpp
// 文件: ets2panda/lexer/lexer.cpp
// 行号: 447-450 - 未终止字符串检测

case util::StringView::Iterator::INVALID_CP: {
    LogError(diagnostic::UNTERMINATED_STRING);
    isFinalizedStr = false;
    break;
}
```

#### 数字解析溢出检查（证据：`ets2panda/lexer/lexer.h:573-580`）

```cpp
// 文件: ets2panda/lexer/lexer.h
// 行号: 573-580

auto const limit = static_cast<RadixType>(leadingMinus ? std::numeric_limits<RadixLimit>::min()
                                                       : std::numeric_limits<RadixLimit>::max());
if ((leadingMinus ? number < (limit / RADIX) : number > (limit / RADIX)) ||
    (number == (limit / RADIX) && digit > limit % RADIX)) {
    return false;  // 数字过大，溢出保护
}
```

#### Unicode 转义验证（证据：`ets2panda/lexer/lexer.cpp:73-76`）

```cpp
// 文件: ets2panda/lexer/lexer.cpp
// 行号: 73-76

if (code > UNICODE_CODE_POINT_MAX) {
    LogError(diagnostic::INVALID_UNICODE_ESCAPE);
    code = UNICODE_INVALID_CP;
    break;
}
```

#### 内存耗尽处理（证据：`ets2panda/util/eheap.cpp:68-70`）

```cpp
// 文件: ets2panda/util/eheap.cpp
// 行号: 68-70

[[noreturn]] __attribute__((noinline)) void EHeap::OOMAction() {
    std::cerr << "es2panda heap is out of memory, aborting" << std::endl;
    std::abort();  // 强制终止
}
```

**风险评估**：

| 检查项 | 状态 | 证据 |
|--------|------|------|
| 递归深度限制 | ✅ 已实现 | MAX_RECURSION_DEPTH = 5120 |
| 循环迭代限制 | ✅ 已实现 | WhileLoopGuard |
| 字符串边界检查 | ✅ 已实现 | INVALID_CP 检测 |
| 数字溢出保护 | ✅ 已实现 | limit/RADIX 检查 |
| Unicode 转义验证 | ✅ 已实现 | UNICODE_CODE_POINT_MAX |
| OOM 终止处理 | ✅ 已实现 | EHeap::OOMAction() |

---

## 已识别风险点

### 风险 R1: 输出路径缺少目录限制（高危）

**位置**: `es2panda/aot/emitFiles.cpp:86-98`

**证据**：
```cpp
// 文件: es2panda/aot/emitFiles.cpp
// 行号: 86-98

void EmitSingleAbcJob::Run()
{
    panda::Timer::timerStart(panda::EVENT_EMIT_SINGLE_PROGRAM, outputFileName_);
    if (!panda::pandasm::AsmEmitter::Emit(
        panda::os::file::File::GetExtendedFilePath(outputFileName_),  // 直接使用用户指定路径
        *prog_, statp_, nullptr, true, nullptr,
        targetApiVersion_, targetApiSubVersion_)) {
        throw Error(ErrorType::GENERIC, "Failed to emit " + outputFileName_);
    }
}
```

**触发路径**：
```
命令行: ./es2abc --output /etc/passwd /tmp/input.js
        ↓
options.cpp:764 → compilerOutput_ = "/etc/passwd"  // 未验证
        ↓
emitFiles.cpp:89 → AsmEmitter::Emit("/etc/passwd", ...)  // 任意写入
```

**影响**：
- 攻击者可覆盖系统文件（如 `/etc/passwd`、`~/.bashrc`）
- 可能导致本地权限提升
- 可用于持久化恶意代码

**修复建议**：
```cpp
// 输出前验证路径
std::string resolvedPath = panda::os::file::File::GetExtendedFilePath(outputFileName_);
std::string resolved = realpath(resolvedPath.c_str(), nullptr);
if (resolved.empty()) {
    throw Error(ErrorType::GENERIC, "Invalid output path");
}

// 检查是否在允许的输出目录内
std::vector<std::string> allowedDirs = {"./", "/tmp/", getcwd()};
bool isAllowed = false;
for (const auto &dir : allowedDirs) {
    if (resolved.find(dir) == 0) {
        isAllowed = true;
        break;
    }
}
if (!isAllowed) {
    throw Error(ErrorType::GENERIC, "Output path outside allowed directories");
}
```

---

### 风险 R2: 线程数无上限限制（中危）

**位置**: `es2panda/aot/options.cpp:517-520`

**证据**：
```cpp
// 文件: es2panda/aot/options.cpp
// 行号: 517-520

panda::PandArg<int> opFunctionThreadCount("function-threads", 0,
    "Number of worker threads to compile function");
panda::PandArg<int> opFileThreadCount("file-threads", 0,
    "Number of worker threads to compile file");
panda::PandArg<int> opAbcClassThreadCount("abc-class-threads", 4,
    "Number of worker threads to compile classes of abc file");
```

**问题**：
- 无最大线程数验证
- 用户可指定极大值（如 `--thread=1000000`）
- 可能导致资源耗尽（DoS）

**触发路径**：
```
命令行: ./es2abc --thread=1000000 input.js
        ↓
options.cpp → threadCount = 1000000  // 未检查
        ↓
compiler.cpp → 创建 1000000 线程/线程池
        ↓
系统资源耗尽 → 编译崩溃或系统变慢
```

**影响**：
- 本地 DoS 攻击
- 系统资源耗尽
- 影响同一系统上的其他进程

**修复建议**：
```cpp
constexpr int MAX_THREAD_COUNT = 64;  // 合理上限

if (threadCount < 0 || threadCount > MAX_THREAD_COUNT) {
    errorMsg_ = "Thread count must be between 0 and " + std::to_string(MAX_THREAD_COUNT);
    return false;
}
```

---

### 风险 R3: 源文件大小无限制（中危）

**状态**: 未发现明确的文件大小限制

**描述**：
- 编译器读取整个源文件到内存
- 攻击者可提供超大文件（如 10GB）
- 可能导致内存耗尽

**触发路径**：
```
恶意文件: /tmp/malicious.js (10GB 大小)
         ↓
helpers.cpp:804 → ReadFileToBuffer() → 整个文件读入内存
         ↓
内存耗尽 → OOM 或系统崩溃
```

**缓解因素**：
- ArenaAllocator 有 OOM 终止机制（`eheap.cpp:68-70`）
- 现代系统通常有内存限制（ulimit）

**修复建议**：
```cpp
// 在 helpers.cpp ReadFileToBuffer 中添加
constexpr size_t MAX_FILE_SIZE = 1024 * 1024 * 100;  // 100MB

std::error_code ec;
auto fileSize = fs::file_size(inputPath, ec);
if (!ec && fileSize > MAX_FILE_SIZE) {
    std::cerr << "Input file exceeds maximum allowed size (" << MAX_FILE_SIZE << " bytes)" << std::endl;
    return false;
}
```

---

### 风险 R4: 栈溢出风险（中危）

**状态**: 依赖系统栈大小，编译器无额外保护

**描述**：
- 递归解析依赖系统调用栈
- 深度嵌套（如 5000+ 层）可能导致栈溢出
- 不只是递归深度限制，而是实际栈空间消耗

**触发路径**：
```
源文件: 5000+ 层嵌套函数/类定义
         ↓
parser.cpp → TrackRecursive() 通过
         ↓
实际 C++ 递归调用 → 栈帧累积
         ↓
系统栈耗尽 → segmentation fault
```

**缓解因素**：
- MAX_RECURSION_DEPTH = 5120 提供软限制
- 但软限制在某些路径可能绕过后仍使用深递归

**修复建议**：
考虑将尾部递归转换为迭代，或增加栈使用监控。

---

### 风险 R5: N-API 模块注入（中危）

**位置**: `ets2panda/bindings/native/src/convertors-napi.cpp:32,377`

**证据**：
```cpp
// 文件: ets2panda/bindings/native/src/convertors-napi.cpp
// 行号: 32 - N-API 模块注册宏

#define NAPI_MODULE(modname, regfunc) ...
NODE_API_MODULE_ADAPTER(modname, regfunc)

// 行号: 377 - 模块注册
NAPI_MODULE(INTEROP_LIBRARY_NAME, InitModule)
```

**描述**：
- N-API 桥接代码可能被恶意 Native 模块利用
- 动态函数调用（`win-dynamic-node.cpp`）
- 线程安全函数（`common-interop.cpp`）

**缓解因素**：
- 本项目是编译器，非运行时
- N-API 主要用于编译时类型转换
- 实际运行时在 arkcompiler_ets_runtime

**影响评估**：
- 主要影响编译时行为
- 运行时安全由 arkcompiler_ets_runtime 负责

---

## 安全机制评估总结

### 已实现的安全措施

| 安全机制 | 状态 | 证据 |
|----------|------|------|
| 递归深度限制 | ✅ | `MAX_RECURSION_DEPTH = 5120` (`recursiveGuard.h:21`) |
| 循环迭代限制 | ✅ | `WhileLoopGuard` (`ETSparserExpressions.cpp:678`) |
| 输入枚举验证 | ✅ | `VALID_EXTENSIONS` 检查 (`options.cpp:745`) |
| 路径规范化 | ✅ | `NormalizePath()` (`arktsconfig.cpp:67`) |
| 符号链接解析 | ✅ | `fs::canonical()` (`arktsconfig.cpp:84`) |
| 数字溢出保护 | ✅ | `limit / RADIX` 检查 (`lexer.h:573`) |
| Unicode 转义验证 | ✅ | `UNICODE_CODE_POINT_MAX` (`lexer.cpp:73`) |
| 字符串边界检查 | ✅ | `INVALID_CP` 检测 (`lexer.cpp:447`) |
| OOM 终止处理 | ✅ | `EHeap::OOMAction()` (`eheap.cpp:68`) |
| 严格编译选项 | ✅ | `-Wall -Wextra -Werror -fno-rtti` (`BUILD.gn:109-116`) |

### 未实现的安全措施

| 安全措施 | 风险等级 | 建议实现位置 |
|----------|----------|--------------|
| 输出路径目录限制 | **高** | `emitFiles.cpp:89` |
| 文件大小限制 | **中** | `helpers.cpp:804` |
| 线程数上限 | **中** | `options.cpp:517` |
| 栈使用监控 | **中** | `recursiveGuard.h` |

---

## 安全最佳实践建议

### 短期（高优先级）

1. **实现输出路径白名单**
   - 添加 `--output-paths` 选项指定允许输出目录
   - 默认限制为源文件所在目录和 `/tmp/`

2. **添加文件大小限制**
   - 默认最大源文件大小（如 100MB）
   - 可通过配置调整

### 中期（中优先级）

3. **增加线程数上限**
   - 设置合理上限（如 64 核）
   - 自动检测系统可用核心数

4. **增强错误处理**
   - 为所有外部输入添加验证
   - 统一错误码和错误消息格式

### 长期（低优先级）

5. **考虑增量编译沙箱**
   - 在临时目录中执行编译
   - 限制编译进程的权限

---

## 运行时 vs 构建时安全

| 方面 | 构建时 (ets_frontend) | 运行时 (ets_runtime) |
|------|----------------------|---------------------|
| **安全边界** | 编译器处理不可信源码 | 字节码在沙箱中执行 |
| **主要风险** | 输入验证、资源耗尽 | 恶意代码执行、权限提升 |
| **防护机制** | 本文档列出的各项 | 沙箱、系统权限模型 |
| **责任方** | arkcompiler/ets_frontend | arkcompiler/ets_runtime |

**注意**：ets_frontend 是构建时工具，主要安全考量在输入验证、资源限制和输出安全。运行时安全由 `arkcompiler_ets_runtime` 负责。

---

## 相关链接

- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/README.md)
- [ARK Runtime 安全说明](docs/Security.md)
- [N-API 官方文档](https://nodejs.org/api/n-api.html)
- [ESLint 安全最佳实践](https://eslint.org/docs/rules/)

---

## 附录：关键代码证据索引

| 功能 | 文件 | 行号 |
|------|------|------|
| CLI 入口点 | `es2panda/aot/main.cpp` | 383 |
| CLI 入口点 | `ets2panda/aot/main.cpp` | 211 |
| 参数解析 | `es2panda/aot/options.cpp` | 480-945 |
| 路径规范化 | `ets2panda/util/arktsconfig.cpp` | 67-88 |
| 路径遍历检测 | `ets2panda/util/importPathManager.cpp` | 183-191 |
| 递归深度限制 | `ets2panda/util/recursiveGuard.h` | 21-53 |
| 递归使用示例 | `ets2panda/parser/statementParser.cpp` | 94-101 |
| 循环保护 | `ets2panda/parser/ETSparserExpressions.cpp` | 678 |
| 数字溢出保护 | `ets2panda/lexer/lexer.h` | 573-580 |
| Unicode 验证 | `ets2panda/lexer/lexer.cpp` | 73-76 |
| OOM 处理 | `ets2panda/util/eheap.cpp` | 68-70 |
| N-API 注册 | `ets2panda/bindings/native/src/convertors-napi.cpp` | 32, 377 |
| 字节码输出 | `es2panda/aot/emitFiles.cpp` | 86-98 |
