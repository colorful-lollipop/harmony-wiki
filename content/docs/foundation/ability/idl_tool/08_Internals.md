# 08 - 内部实现细节 (Internals)

## 1. AST 节点详解

### 1.1 AST 继承层次

```
ASTNode (抽象基类)
│   - lineNumber_: 源代码行号
│   - columnNumber_: 源代码列号
│
├── ASTType (类型基类)
│   │   - typeName_: 类型名称
│   │   - typeKind_: TypeKind枚举
│   │
│   ├── ASTVoidType (void)
│   │
│   ├── 基本类型 (ast/base/)
│   │   ├── ASTBooleanType
│   │   ├── ASTByteType / ASTUcharType
│   │   ├── ASTShortType / ASTUshortType
│   │   ├── ASTIntegerType / ASTUintType
│   │   ├── ASTLongType / ASTUlongType
│   │   ├── ASTFloatType / ASTDoubleType
│   │   ├── ASTCharType
│   │   ├── ASTStringType / ASTString16Type / ASTU16stringType
│   │   └── ASTCStringType
│   │
│   ├── 复合类型
│   │   ├── ASTArrayType (数组)
│   │   │   └── ASTListType (列表)
│   │   ├── ASTMapType (映射)
│   │   ├── ASTOrderedMapType (有序映射)
│   │   ├── ASTSetType (集合)
│   │   └── ASTPtrType (智能指针)
│   │
│   ├── 用户定义类型
│   │   ├── ASTInterfaceType (接口)
│   │   ├── ASTStructType (结构体)
│   │   ├── ASTEnumType (枚举)
│   │   ├── ASTUnionType (联合体)
│   │   └── ASTSequenceableType (序列化)
│   │
│   └── 特殊类型
│       ├── ASTRawDataType (原始数据)
│       ├── ASTFdType (文件描述符)
│       ├── ASTFdsanType (FDsan)
│       ├── ASTSmqType (共享内存队列)
│       ├── ASTNativeBufferType (本地缓冲区)
│       └── ASTPointerType (指针)
│
├── ASTMethod (方法)
│   - name_: 方法名
│   - returnType_: 返回类型
│   - parameters_: 参数列表
│   - cmdId_: 命令ID
│
├── ASTParameter (参数)
│   - name_: 参数名
│   - type_: 参数类型
│   - attr_: 属性(in/out/inout)
│
├── ASTNamespace (命名空间)
│   - name_: 命名空间名
│   - interfaces_: 接口列表
│
├── ASTAttr / ASTParamAttr (属性)
│   - 各种标志位(full, lite, mini, oneway等)
│
├── ASTExpr (表达式)
│   ├── ASTUnaryExpr (一元表达式)
│   ├── ASTBinaryExpr (二元表达式)
│   ├── ASTNumExpr (数字表达式)
│   └── ASTEnumExpr (枚举表达式)
│
└── AST (根容器)
    - namespaces_: 命名空间列表
```

### 1.2 TypeKind 枚举

**位置**: `idl_tool_2/ast/ast_type.h:37-77`

| 类别 | 枚举值 |
|------|--------|
| 基础类型 | VOID, BOOLEAN, BYTE, SHORT, INT, LONG, FLOAT, DOUBLE, CHAR, UCHAR, USHORT, UINT, ULONG |
| 字符串 | CSTRING, STRING, STRING16, U16STRING |
| 文件描述符 | FILEDESCRIPTOR, FILEDESCRIPTORSAN |
| 接口类型 | INTERFACE, SEQUENCEABLE, RAWDATA |
| 集合 | LIST, MAP, ORDEREDMAP, ARRAY, SET |
| 智能指针 | SHAREDPTR, UNIQUEPTR, SPTR, NULL_SHAREDPTR, NULL_UNIQUEPTR, NULL_SPTR |
| 用户定义 | ENUM, STRUCT, UNION |
| 特殊 | SMQ, ASHMEM, NATIVE_BUFFER, POINTER |

---

## 2. 词法分析实现

### 2.1 Token 结构

**位置**: `idl_tool_2/lexer/token.h:26`

```cpp
struct Token {
    TokenType type;      // Token类型
    std::string value;   // 文本值
    size_t line;         // 行号
    size_t column;       // 列号
};
```

### 2.2 Lexer 状态机

```mermaid
graph LR
    A[Start] --> B{PeekChar}
    B -->|空格/换行| C[SkipWhitespace]
    B -->|字母/_| D[ReadId]
    B -->|数字| E[ReadNum]
    B -->|"| F[ReadString]
    B -->|/| G[ReadComment]
    B -->|符号| H[ReadSymbol]
    
    C --> B
    D --> I[LookupKeyword]
    E --> J[Binary/Oct/Hex/Dec]
    F --> K[Token]
    G --> L[Line/Block]
    H --> K
    I --> K
    J --> K
    L --> K
```

### 2.3 关键字表

**位置**: `idl_tool_2/lexer/lexer.cpp:23-85`

```cpp
Lexer::StrTokenTypeMap Lexer::keyWords_ = {
    {"void",        TokenType::VOID},
    {"boolean",     TokenType::BOOLEAN},
    {"byte",        TokenType::BYTE},
    {"short",       TokenType::SHORT},
    {"int",         TokenType::INT},
    {"long",        TokenType::LONG},
    {"float",       TokenType::FLOAT},
    {"double",      TokenType::DOUBLE},
    {"String",      TokenType::STRING},
    {"interface",   TokenType::INTERFACE},
    {"enum",        TokenType::ENUM},
    {"struct",      TokenType::STRUCT},
    {"union",       TokenType::UNION},
    // ... 60+ 关键字
};
```

---

## 3. 语法分析实现

### 3.1 递归下降解析

**核心方法**: `Parser::ParseFile()`

```cpp
void Parser::ParseFile() {
    ParseLicense();         // 解析许可证
    ParsePackage();         // 解析包声明
    ParseInterfaceToken();  // 解析接口标记
    ParseImports();         // 解析导入
    ParseTypeDecls();       // 解析类型定义
}
```

### 3.2 方法解析流程

```cpp
AutoPtr<ASTMethod> Parser::ParseMethod() {
    // 1. 解析属性 [oneway, full, lite, ...]
    AutoPtr<ASTAttr> attr = ParseMethodAttr();
    
    // 2. 解析返回类型
    AutoPtr<ASTType> returnType = ParseType();
    
    // 3. 解析方法名
    std::string name = ParseIdentifier();
    
    // 4. 解析参数列表
    ParseMethodParamList(method);
    
    // 5. 解析分号
    Expect(TokenType::SEMICOLON);
}
```

### 3.3 类型解析

**类型优先级** (从高到低):
1. 基本类型 (int, String, ...)
2. 泛型类型 (List<T>, Map<K,V>, ...)
3. 数组类型 (Type[])
4. 智能指针 (shared_ptr<T>, ...)
5. 用户定义类型 (Interface, Struct, ...)

---

## 4. 代码生成实现

### 4.1 生成器注册

**注册表模式**:
```cpp
// code_generator.h
class CodegenBuilder {
public:
    static CodegenBuilder& GetInstance();
    void Register(InterfaceType type, CodeGenerator* generator);
    bool Generate(const StrAstMap& asts);
};

// 自动注册
static struct HDIGeneratorRegister {
    HDIGeneratorRegister() {
        CodegenBuilder::GetInstance().Register(
            InterfaceType::HDI, new HDICodeGenerator());
    }
} g_hdiGeneratorRegister;
```

### 4.2 HDI 生成策略

**策略矩阵**:
```cpp
// hdi_code_generator.cpp:36-92
void HDICodeGenerator::InitGeneratePolicies() {
    // MINI + LOW + C
    generatePolicies_[SystemLevel::MINI][GenMode::LOW][Language::C] = 
        &HDICodeGenerator::GenLowCCode;
    
    // LITE + PASSTHROUGH + C/CPP
    generatePolicies_[SystemLevel::LITE][GenMode::PASSTHROUGH][Language::C] = 
        &HDICodeGenerator::GenPassthroughCCode;
    
    // FULL + IPC + CPP
    generatePolicies_[SystemLevel::FULL][GenMode::IPC][Language::CPP] = 
        &HDICodeGenerator::GenFullCppCode;
    
    // ... 其他组合
}
```

### 4.3 发射器工作流

```cpp
bool CodeEmitter::OutPut(const AutoPtr<AST>& ast, 
                         const std::string& targetDir,
                         GenMode mode) {
    // 1. 解析目录
    if (!ResolveDirectory(targetDir)) {
        return false;
    }
    
    // 2. 发射代码
    EmitCode();
    
    return true;
}
```

---

## 5. 元数据系统

### 5.1 元数据结构

```
MetaComponent (根)
├── MetaNamespace[] (命名空间数组)
├── MetaSequenceable[] (序列化类型)
├── MetaRawData[] (原始数据)
├── MetaInterface[] (接口数组)
│   └── MetaMethod[] (方法数组)
│       └── MetaParameter[] (参数数组)
├── MetaType[] (类型数组)
└── StringPool (字符串池)
```

### 5.2 元数据构建流程

```cpp
// metadata_builder.cpp
bool MetadataBuilder::Build() {
    // Pass 1: 计算大小
    CalculateSize();
    
    // Pass 2: 分配内存
    void* metadata = calloc(size_, 1);
    
    // Pass 3: 写入数据
    WriteMetadata(metadata);
    
    // Pass 4: 序列化（指针转偏移）
    MetadataSerializer::Serialize(metadata);
    
    return true;
}
```

### 5.3 元数据文件格式

```
[Header - 16 bytes]
  - magic: 0x1DF02ED1
  - version: uint32
  - size: uint32
  
[Namespaces - offset + count]
[Sequenceables - offset + count]
[RawDatas - offset + count]
[Interfaces - offset + count]
  - 每个接口包含方法索引数组
[Types - offset + count]
[String Pool - 紧凑存储]
```

---

## 6. 类型系统

### 6.1 TypeMode

**位置**: `idl_tool_2/ast/ast_type.h:81-86`

```cpp
enum class TypeMode {
    NO_MODE,        // 无模式
    PARAM_IN,       // 输入参数
    PARAM_OUT,      // 输出参数
    PARAM_INOUT,    // 输入输出参数
    LOCAL_VAR,      // 局部变量
};
```

### 6.2 类型检查

**接口类型检查** (`intf_type_check.cpp`):
- 验证接口继承关系
- 验证方法签名唯一性
- 验证参数类型合法性

---

## 7. 内存管理

### 7.1 引用计数

**基类**: `LightRefCountBase`

```cpp
class LightRefCountBase {
public:
    void IncRefCount();
    void DecRefCount();
    int GetRefCount() const;
    
private:
    std::atomic_int refCount_;
};
```

### 7.2 智能指针

**AutoPtr**:
```cpp
template<typename T>
class AutoPtr {
public:
    AutoPtr(T* ptr = nullptr) : ptr_(ptr) {
        if (ptr_ != nullptr) {
            ptr_->IncRefCount();
        }
    }
    
    ~AutoPtr() {
        if (ptr_ != nullptr) {
            ptr_->DecRefCount();
            if (ptr_->GetRefCount() == 0) {
                delete ptr_;
            }
        }
    }
    
    // ... 其他方法
};
```

---

## 8. 工具类

### 8.1 StringBuilder

**用途**: 高效构建字符串

```cpp
class StringBuilder {
public:
    void Append(const std::string& str);
    void AppendFormat(const char* fmt, ...);
    std::string ToString() const;
    
private:
    char* buffer_;
    size_t size_;
    size_t capacity_;
};
```

### 8.2 StringPool

**用途**: 字符串去重，节省内存

```cpp
class StringPool {
public:
    const char* Add(const std::string& str);
    
private:
    std::unordered_set<std::string> pool_;
};
```

---

## 9. 关键证据

| 组件 | 文件 | 关键行号 |
|------|------|---------|
| AST基类 | `ast/ast_node.h` | 25-60 |
| TypeKind | `ast/ast_type.h` | 37-77 |
| Token | `lexer/token.h` | 23-123 |
| Lexer | `lexer/lexer.h` | 30-80 |
| Parser | `parser/parser.h` | 27-60 |
| 生成器 | `codegen/code_generator.h` | 32-55 |
| 发射器 | `codegen/code_emitter.h` | 79-151 |
| 元数据 | `metadata/meta_component.h` | 1-50 |

---

**上一步**: [07_Build.md](07_Build.md)
**返回**: [README.md](README.md)
