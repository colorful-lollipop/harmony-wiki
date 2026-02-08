# 06 - 安全风险评估 (Security Review)

## 1. 评估范围与方法

### 1.1 评估范围

本文档对 OpenHarmony IDL Tool 进行安全风险评估，重点关注：
- 输入验证缺陷
- 内存安全问题
- 文件操作安全
- 代码生成安全

### 1.2 评估方法

- **代码审计**: 静态分析关键模块源代码
- **证据定位**: 提供具体的文件路径和行号
- **风险评级**: 高/中/低，基于可利用性和影响

---

## 2. 输入验证缺陷

### R1: 路径遍历风险（中危）

**位置**: `idl_tool_2/util/file.cpp:307-331`

**证据**:
```cpp
std::string File::CanonicalPath(const std::string &path)
{
    // ...
    for (const auto &pathItem : pathItems) {
        if (pathItem == "..") {
            if (!pathStack.empty()) {
                pathStack.pop_back();  // 只处理..，不验证根边界
            }
        }
    }
}
```

**触发路径**:
```
idl -gen-cpp -d "../../../etc/" -c test.idl
→ Options::Parse() 解析 -d
→ File::Open() 
→ File::CanonicalPath() 处理路径
→ 可能跳出预期目录
```

**影响**: 如果运行环境受限，可能导致文件写入预期之外的目录

**修复建议**:
```cpp
// 添加根目录边界检查
std::string File::CanonicalPath(const std::string &path, const std::string &baseDir) {
    // 规范化后验证是否在 baseDir 下
    if (!IsSubPath(canonicalPath, baseDir)) {
        return "";  // 拒绝越界路径
    }
}
```

---

### R2: 命令行参数长度验证不足（低危）

**位置**: `idl_tool_2/util/options.cpp:64-296`

**证据**:
```cpp
// 直接复制字符串，无长度限制
rootPaths_[key] = value.substr(index + 1);
```

**风险**: 超长参数可能导致内存分配过大

**修复建议**: 添加最大长度限制

---

## 3. 内存安全问题

### R3: malloc 返回值未充分验证（中危）

**位置**: `idl_tool_2/metadata/metadata_reader.cpp:56`

**证据**:
```cpp
void* data = malloc(header.size_);
// 有检查但size_来自文件
if (data == nullptr) {
    return nullptr;
}
```

**风险**:
- `header.size_` 来自元数据文件头部（行41）
- 虽然检查了 `header.size_ > UINT16_MAX`，但仍可分配64KB
- 多次分配可能耗尽内存

**触发路径**:
```
恶意.metadata文件 → MetadataReader::Read() 
→ 读取header.size_ = 65535
→ malloc(65535)  // 大量小分配
```

**修复建议**:
```cpp
// 添加总内存限制
static size_t totalAllocated = 0;
const size_t MAX_TOTAL_MEMORY = 10 * 1024 * 1024;  // 10MB

if (header.size_ > UINT16_MAX || 
    totalAllocated + header.size_ > MAX_TOTAL_MEMORY) {
    return nullptr;
}
```

---

### R4: StringBuilder 缓冲区增长无上限（低危）

**位置**: `idl_tool_2/util/string_builder.cpp:130-167`

**证据**:
```cpp
void StringBuilder::Grow(size_t needSize)
{
    size_t newSize = capacity_ * 2;
    while (newSize - size_ < needSize) {
        newSize <<= 1;  // 指数增长
    }
    // 无上界检查
    char *newBuffer = reinterpret_cast<char *>(calloc(newSize, 1));
}
```

**风险**:
- 持续追加字符串可能导致无限增长
- 极端输入下可能耗尽内存

**修复建议**:
```cpp
const size_t MAX_CAPACITY = 100 * 1024 * 1024;  // 100MB

if (newSize > MAX_CAPACITY) {
    // 报错或截断
    return;
}
```

---

### R5: 整数溢出风险（低危）

**位置**: `idl_tool_2/lexer/lexer.cpp:324-348`

**证据**:
```cpp
AutoPtr<ASTExpr> Lexer::ReadNum()
{
    uint64_t value = 0;
    while (IsDecDigit(PeekChar())) {
        value = value * 10 + (GetChar() - '0');  // 可能溢出
    }
}
```

**风险**: 超长的数字字面量可能导致整数溢出

**影响**: 溢出值被用于后续计算，可能产生意外行为

**修复建议**: 添加溢出检查

---

## 4. 文件操作安全

### R6: TOCTOU 竞争条件（中危）

**位置**: `idl_tool_2/util/file.cpp:392-408`

**证据**:
```cpp
bool File::CheckValid(const std::string &path)
{
    if (access(path.c_str(), F_OK) == 0) {      // 检查1
        if (stat(path.c_str(), &buf) == 0) {    // 检查2
            // 攻击窗口：文件可能在此时被替换
            return S_ISREG(buf.st_mode);
        }
    }
    return false;
}

// 之后单独调用 fopen
fd_ = fopen(realPath.c_str(), "r");             // 使用
```

**风险**: 检查和使用分离，存在竞争窗口

**影响**: 可能打开非预期的文件（如符号链接指向的敏感文件）

**修复建议**: 使用原子操作
```cpp
// 使用 open() + O_NOFOLLOW 避免符号链接
int fd = open(path.c_str(), O_RDONLY | O_NOFOLLOW);
if (fd < 0) return false;
// 然后使用 fdopen
```

---

### R7: 目录权限过度宽松（低危）

**位置**: `idl_tool_2/util/file.cpp:245`

**证据**:
```cpp
int result = mkdir(dirPath.c_str(), 
    S_IRWXU | S_IRWXG | S_IRWXO);  // 0777
```

**风险**: 创建的目录对所有人可读写执行

**影响**: 在多用户环境下可能导致信息泄露

**修复建议**:
```cpp
mkdir(dirPath.c_str(), S_IRWXU | S_IRGRP | S_IXGRP);  // 0750
```

---

### R8: 文件截断风险（中危）

**位置**: `idl_tool_2/util/file.cpp:49`

**证据**:
```cpp
fd_ = fopen(path.c_str(), "w+");  // w+ 会截断现有文件
```

**风险**: 如果路径指向现有文件，会被清空

**影响**: 可能意外破坏重要文件

**修复建议**: 添加存在性检查或要求确认

---

## 5. 代码生成安全

### R9: 生成代码中的 strcpy_s 风险（中危）

**位置**: `idl_tool_2/codegen/HDI/type/hdi_array_type_emitter.cpp:473`

**证据**:
```cpp
// 生成的代码
sb.AppendFormat(
    "if (strcpy_s(%s[i], strlen(%s) + 1, %s) != EOK)",
    name.c_str(), element.c_str(), element.c_str());
```

**风险**:
- `strlen` 和 `strcpy_s` 之间，字符串可能被修改
- 如果 IDL 定义的参数名包含特殊字符，可能破坏代码结构

**触发路径**:
```idl
interface Test {
    void Method([in] String "; malicious code /* param);
};
```

**修复建议**: 对生成的标识符进行验证和转义

---

### R10: 生成代码中的 strdup 未检查返回值（低危）

**位置**: `idl_tool_2/codegen/HDI/type/hdi_array_type_emitter.cpp:479`

**证据**:
```cpp
"%s[i] = strdup(%s);\n"  // 未检查返回值
```

**风险**: 内存分配失败时返回 NULL，后续使用可能导致崩溃

**修复建议**:
```cpp
"%s[i] = strdup(%s);\n"
"if (%s[i] == NULL) { return ERR_NO_MEMORY; }\n"
```

---

### R11: 标识符注入风险（中危）

**位置**: 多处代码生成器

**证据**:
```cpp
// sa_cpp_interface_code_emitter.cpp
sb.AppendFormat("class %s : public IRemoteStub<%s>",
    interfaceName.c_str(), interfaceName.c_str());
```

**风险**: 如果 IDL 中的标识符包含特殊字符，可能破坏生成的代码

**示例**:
```idl
interface ITest {
    void Method{};();  // 特殊字符
};
```

**修复建议**: 在解析阶段验证标识符格式
```cpp
bool IsValidIdentifier(const std::string& name) {
    // 只允许 [a-zA-Z_][a-zA-Z0-9_]*
}
```

---

## 6. 日志与信息泄露

### R12: 日志函数格式字符串风险（低危）

**位置**: `idl_tool_2/util/logger.cpp:59-71`

**证据**:
```cpp
void Logger::Log(Level level, const char* tag, 
                 const char* fmt, ...)
{
    va_list args;
    va_start(args, fmt);
    vprintf(fmt, args);  // 直接使用用户格式
    va_end(args);
}
```

**风险**: 如果日志内容来自用户输入，可能包含格式说明符

**影响**: 信息泄露或崩溃

**修复建议**: 使用固定格式或验证格式字符串

---

## 7. 风险汇总

| 风险ID | 类别 | 等级 | 位置 | 状态 |
|--------|------|------|------|------|
| R1 | 输入验证 | 中 | file.cpp:307 | 待修复 |
| R2 | 输入验证 | 低 | options.cpp:64 | 可接受 |
| R3 | 内存安全 | 中 | metadata_reader.cpp:56 | 待修复 |
| R4 | 内存安全 | 低 | string_builder.cpp:130 | 可接受 |
| R5 | 内存安全 | 低 | lexer.cpp:324 | 可接受 |
| R6 | 文件操作 | 中 | file.cpp:392 | 建议修复 |
| R7 | 文件操作 | 低 | file.cpp:245 | 建议修复 |
| R8 | 文件操作 | 中 | file.cpp:49 | 建议修复 |
| R9 | 代码生成 | 中 | hdi_array_type_emitter.cpp:473 | 待修复 |
| R10 | 代码生成 | 低 | hdi_array_type_emitter.cpp:479 | 建议修复 |
| R11 | 代码生成 | 中 | 多处 | 待修复 |
| R12 | 信息泄露 | 低 | logger.cpp:59 | 可接受 |

---

## 8. 修复优先级建议

### 立即修复（P0）
- **R9, R11**: 代码生成安全问题 - 影响生成的代码质量

### 建议修复（P1）
- **R1**: 路径遍历防护
- **R3**: 内存分配限制
- **R6**: TOCTOU 防护
- **R8**: 文件截断防护

### 可选修复（P2）
- **R7**: 目录权限
- **R10**: strdup 检查

### 可接受风险（P3）
- **R2, R4, R5, R12**: 低影响或需要极端条件

---

## 9. 安全开发建议

### 9.1 对工具开发者

1. **输入验证**: 所有外部输入都需要验证长度和格式
2. **边界检查**: 数组、缓冲区操作都要检查边界
3. **资源限制**: 设置内存、文件大小的上限
4. **安全函数**: 优先使用带_s后缀的安全函数
5. **原子操作**: 文件检查和操作使用原子方式

### 9.2 对IDL文件编写者

1. 使用有意义的标识符名称
2. 避免过深的嵌套结构
3. 合理控制文件大小
4. 遵循 IDL 语法规范

---

**上一步**: [05_AttackSurface.md](05_AttackSurface.md)
**下一步**: [07_Build.md](07_Build.md)
