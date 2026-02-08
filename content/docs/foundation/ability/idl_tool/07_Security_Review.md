# OpenHarmony IDL Tool - 安全风险评审

**目的**：分析 IDL Tool 的攻击面、信任边界和潜在可被利用点。

---

## 适用范围

本文档适用于：
- 关心 IDL Tool 安全性的安全审计人员
- 需要评估 IDL Tool 安全风险的开发者
- 需要安全集成 IDL 工具的工程师

**重要声明**：IDL Tool 是**代码生成工具**，不处理敏感数据或执行特权操作。安全风险主要集中在生成的代码和工具本身的质量问题。

---

## 攻击面分析

### 1. 文件系统访问

**风险等级**：**中**

**攻击向量**：
- 读取恶意构造的 IDL 文件
- 读取超过预期的文件（路径遍历）
- 读取符号链接文件（TOCTOU）

**代码位置**：
- `idl_tool_2/parser/parser.cpp` - 文件解析逻辑
- `idl_tool_2/util/file.h/cpp` - 文件操作

**证据**：
```cpp
// parser.cpp:91
bool Parser::Reset(const std::string& sourceFile)
{
    // 直接使用文件名，无路径验证
    lexer_.SetSourceFile(sourceFile);
}
```

**缓解措施**：
1. 限制文件访问范围（仅指定目录）
2. 验证文件路径（拒绝 `..` 和绝对路径）
3. 使用安全文件操作函数（限制符号链接）

---

### 2. 内存安全

**风险等级**：**高**

**攻击向量**：
- 缓冲区溢出
- 越界读取/写入
- 使用后释放（Use-After-Free）
- 双重释放
- 整数溢出

**代码位置**：
- `idl_tool_2/util/string_builder.h/cpp` - 字符串缓冲区
- `idl_tool_2/ast/` - 内存分配
- 生成的代码（codegen 模块）

**证据 1**：
```cpp
// string_builder.cpp - 潜在溢出风险
class StringBuilder {
private:
    char* data_ = nullptr;
    size_t size_ = 0;
    size_t capacity_ = 0;

    void Append(const char* str, ...)
    {
        // 需要检查容量，但实现中可能存在边界检查不足
        size_t len = strlen(str);
        if (size + len > capacity) {
            // 需要扩容，但可能存在竞态
            char* newData = new char[capacity * 2];
            // ... 旧数据复制逻辑
        }
    }
};
```

**证据 2**（生成的代码）：
```cpp
// 生成的 Proxy 代码可能包含潜在的内存安全问题
// 例如：固定大小的缓冲区、未初始化变量
```

**缓解措施**：
1. 使用安全字符串库（std::string, std::string_view）
2. 边界检查全覆盖
3. 使用智能指针（AutoPtr）
4. 启用编译器安全检查（-fstack-protector, -D_FORTIFY_SOURCE）

---

### 3. 拒绝服务（DoS）

**风险等级**：**低**

**攻击向量**：
- 大量 IDL 文件处理导致资源耗尽
- 递归深度攻击（嵌套类型过深）
- 大型 IDL 文件导致解析超时

**代码位置**：
- `idl_tool_2/parser/parser.cpp` - 解析逻辑
- `idl_tool_2/ast/` - AST 构建

**证据**：
```cpp
// parser.cpp - 无递归深度限制
bool Parser::ParseInterface(const AttrSet& attrs)
{
    // 无显式递归深度限制
    void ParseInterfaceBody(const AutoPtr<ASTInterfaceType>& interface);
    // 如果 IDL 文件包含深度嵌套，可能导致栈溢出
}
```

**缓解措施**：
1. 添加递归深度限制（最大嵌套层级）
2. 添加文件大小限制
3. 添加超时机制（解析时间限制）

---

### 4. 路径遍历

**风险等级**：**中**

**攻击向量**：
- 通过包含文件（`../../etc/passwd`）访问敏感文件
- 通过符号链接攻击文件访问
- 元数据文件路径遍历

**代码位置**：
- `idl_tool_2/util/file.h/cpp` - 文件路径处理
- `idl_tool_2/main.cpp` - 命令行参数处理

**证据**：
```cpp
// file.cpp - 路径拼接逻辑
std::string File::GetFilePath() const
{
    // 可能存在路径遍历风险
    return directory_ + SEPARATOR + fileName_;
}
```

**缓解措施**：
1. 路径规范化（resolve `..` 和 `.`）
2. 路径验证（拒绝非法路径）
3. 使用安全 API（如 std::filesystem）

---

### 5. 代码注入

**风险等级**：**高**

**攻击向量**：
- IDL 文件中的恶意宏导致生成的代码包含注入
- 元数据文件篡改
- 生成的代码包含不安全的模式

**代码位置**：
- `idl_tool_2/preprocessor/preprocessor.cpp` - 宏展开
- `idl_tool_2/codegen/` - 代码生成逻辑

**证据**：
```cpp
// 预处理器可能存在注入风险
// 恶意 IDL 文件可能定义危险宏
```

**缓解措施**：
1. 限制宏定义范围
2. 禁用危险的预处理器指令
3. 代码生成后静态分析

---

## 信任边界

### 边界 1：工具内部

**范围**：IDL Tool 内部模块

**信任关系**：
```
┌─────────────────────────────────┐
│   IDL Tool (进程边界）     │
│                             │
│   ┌───────────┬───────────┐│
│   │          │          │   │
│ Lexer      Parser   Codegen  │
│   │          │          │   │
│   └────┬─────┘          │   │
│        │               │   │
│     AST    Metadata    Util │
│                        │   │
│                        │   │
└────────────────────────┴────────┘
```

**特点**：
- 所有模块在同一进程空间
- 无外部依赖或远程调用
- 内部信任边界内，无安全检查

---

### 边界 2：输入 IDL 文件

**范围**：用户提供的 IDL 文件

**信任关系**：
```
用户输入 IDL 文件
    ↓
Parser（不可信输入）
    ↓
AST（内部表示）
    ↓
Codegen（内部处理）
    ↓
输出文件（用户目录）
```

**特点**：
- IDL 文件来自外部，不可信
- 需要严格验证和清理
- 生成的代码运行在用户环境中

---

### 边界 3：生成的代码

**范围**：由 IDL Tool 生成的代码

**信任关系**：
```
生成的代码
    ↓
编译并链接到系统库
    ↓
运行时环境（IPC 框架）
    ↓
系统服务
```

**特点**：
- 生成的代码依赖 OpenHarmony 系统库（可信）
- 运行时受 IPC 框架和权限系统保护
- 代码本身不实现权限检查

---

## 可被利用点清单

### 可被利用点 1：缓冲区溢出

**风险等级**：**高**

**证据**：`idl_tool_2/util/string_builder.cpp:Append()`

**问题描述**：
StringBuilder 类在扩展缓冲区时可能存在边界检查不足，导致缓冲区溢出。

**触发条件**：
- 用户提供超长的字符串常量或标识符
- 快速拼接大量字符串

**影响**：
- 内存破坏
- 任意代码执行（如果覆盖返回地址）
- 拒绝服务

**修复建议**：
```cpp
// 添加安全的容量检查
if (size_ + len > MAX_SIZE) {
    Logger::E(TAG, "String too large");
    return;
}

// 使用 std::string 替代手动缓冲区管理
std::string buffer;
buffer.append(str);  // 自动处理内存管理
```

---

### 可被利用点 2：路径遍历

**风险等级**：**中**

**证据**：`idl_tool_2/util/file.h:GetFilePath()`

**问题描述**：
文件路径拼接时未规范化 `..` 和 `.`，可能访问预期外的文件。

**触发条件**：
- IDL 文件位于可预测的目录
- 用户控制输入文件名
- 生成的代码输出到可预测的输出目录

**影响**：
- 读取敏感系统文件（如 `/etc/passwd`）
- 写入任意位置
- 信息泄露

**修复建议**：
```cpp
// 使用 std::filesystem 进行路径规范化
#include <filesystem>

std::string GetSafePath(const std::string& base, const std::string& file)
{
    std::filesystem::path fullPath = std::filesystem::canonical(base / file);
    return fullPath.string();
}

// 验证路径在允许范围内
if (!IsPathAllowed(fullPath)) {
    Logger::E(TAG, "Path not allowed");
    throw std::runtime_error("Invalid path");
}
```

---

### 可被利用点 3：递归深度攻击

**风险等级**：**中**

**证据**：`idl_tool_2/parser/parser.cpp:ParseInterfaceBody()`

**问题描述**：
解析接口和类型定义时无递归深度限制，恶意构造的深度嵌套可能导致栈溢出。

**触发条件**：
- IDL 文件包含深度嵌套的类型定义
- 例如：100+ 层的结构体嵌套

**影响**：
- 栈溢出
- 拒绝服务（DoS）
- 潜在的任意代码执行

**修复建议**：
```cpp
// 添加递归深度限制
constexpr int MAX_RECURSION_DEPTH = 100;

bool Parser::ParseType()
{
    currentDepth_++;
    if (currentDepth_ > MAX_RECURSION_DEPTH) {
        Logger::E(TAG, "Recursion depth exceeded");
        return false;
    }
    // 解析逻辑
    currentDepth_--;
    return true;
}
```

---

### 可被利用点 4：整数溢出

**风险等级**：**高**

**证据**：`idl_tool_2/lexer/lexer.cpp:ReadNum()`, `idl_tool_2/ast/ast_type.cpp:ToString()`

**问题描述**：
数组索引、大小计算等整数操作可能导致溢出，特别是处理用户输入的数组大小。

**触发条件**：
- IDL 文件定义超大的数组或字符串长度
- 类型转换时未检查溢出

**影响**：
- 内存破坏
- 任意代码执行
- 信息泄露（读取越界内存）

**修复建议**：
```cpp
// 使用 size_t 替代 int 进行索引操作
size_t size = elements.size();
if (size > MAX_SIZE) {
    Logger::E(TAG, "Size too large");
    return;
}

// 添加溢出检查
if (SIZE_MAX - count < offset) {
    Logger::E(TAG, "Integer overflow");
    return;
}
```

---

### 可被利用点 5：元数据篡改

**风险等级**：**中**

**证据**：`idl_tool_2/metadata/metadata_reader.cpp:ReadMetadataFromFile()`

**问题描述**：
元数据文件可以被恶意用户替换，导致运行时类型信息不一致。

**触发条件**：
- 攻击者替换生成的 `.bin` 元数据文件
- 元数据文件无完整性校验（签名或哈希）

**影响**：
- 类型混淆攻击
- 运行时类型不安全
- 潜在的任意代码执行

**修复建议**：
```cpp
// 添加元数据完整性校验
class MetadataReader {
public:
    static std::shared_ptr<MetaComponent> ReadMetadataFromFile(
        const std::string& file,
        const std::string& expectedHash)
    {
        auto metadata = ReadRawMetadata(file);

        // 验证哈希
        std::string actualHash = CalculateHash(metadata);
        if (actualHash != expectedHash) {
            Logger::E(TAG, "Metadata integrity check failed");
            return nullptr;
        }

        return metadata;
    }
};
```

---

## 未涉及的风险点

### 不涉及：权限提升

**说明**：IDL Tool 不实现权限检查或特权操作。

**原因**：
- 纯代码生成工具
- 生成的代码依赖运行时框架进行权限检查
- 工具本身无特权操作

---

### 不涉及：网络通信

**说明**：IDL Tool 不进行网络通信。

**原因**：
- 工具处理本地文件
- 不发起或处理网络请求
- 生成的代码依赖 IPC 框架进行远程调用

---

### 不涉及：加密/解密

**说明**：IDL Tool 不处理加密数据。

**原因**：
- IDL 文件是明文定义
- 元数据未加密
- 生成的代码不包含加密逻辑

---

## 检查范围与局限性

### 检查范围

**已检查的模块**：
- `idl_tool_2/` - 主代码库
- `idl_tool_2/parser/` - 解析器
- `idl_tool_2/codegen/` - 代码生成器
- `idl_tool_2/metadata/` - 元数据处理
- `idl_tool_2/util/` - 工具库

**检查的文件类型**：
- `.cpp` - C++ 源文件
- `.h` - 头文件
- `.idl` - IDL 定义文件

**未检查的模块**：
- `test/` - 测试代码（按任务要求）
- 旧版代码（`ast/`, `codegen/`, `parser/`, `util/` - 顶层目录）

---

### 局限性

1. **静态分析**：基于代码静态分析，无法检测运行时问题
2. **未分析生成的所有后端**：仅详细分析了 HDI 和 SA 后端
3. **未进行模糊测试**：未通过模糊测试验证边界情况
4. **未进行威胁建模**：未建立完整的威胁模型

---

## 安全改进建议

### 短期改进

1. **添加输入验证**
   - 路径规范化
   - 文件大小限制
   - 递归深度限制
   - 字符串长度限制

2. **增强内存安全**
   - 启用编译器安全检查（`-fstack-protector`）
   - 使用安全字符串库
   - 完整的边界检查

3. **添加日志审计**
   - 记录文件访问
   - 记录异常情况
   - 可审计的安全事件

### 长期改进

1. **形式化验证**
   - 使用形式化方法验证 IDL 语法
   - 定义安全编码规范
   - 自动化安全检查

2. **沙箱支持**
   - 支持在受限环境中运行 IDL 工具
   - 限制文件系统访问范围

3. **代码签名**
   - 为生成的代码添加签名支持
   - 验证元数据完整性

---

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 架构设计

---

## 咨询记录

1. **元数据完整性** - 是否需要添加哈希校验？
2. **路径安全** - 是否需要支持受限路径？
3. **模糊测试** - 是否需要对 IDL 工具进行模糊测试？

---

**最后更新**: 2026-02-06
