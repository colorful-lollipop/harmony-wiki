# OpenHarmony IDL Tool - 内部 API

**目的**：详细说明各内部模块的接口定义、依赖方向、稳定性和可替换点。

---

## 适用范围

本文档适用于：
- 理解 IDL Tool 内部模块设计的开发者
- 需要扩展或优化代码生成的工程师
- 希望调试或维护代码库的工程师

---

## AST 模块接口

### ASTNode 基类

**位置**: `idl_tool_2/ast/ast_node.h`

**接口**:
```cpp
class ASTNode : public LightRefCountBase {
public:
    virtual ~ASTNode() override = default;
    virtual std::string ToString() const = 0;
    virtual std::string Dump(const std::string& prefix) const = 0;
};
```

**关键方法**:
- `ToString()` - 获取字符串表示
- `Dump()` - 调试输出（带缩进）
- 继承 `LightRefCountBase` - 引用计数管理

**稳定性**: **稳定** - AST 节点是核心数据结构，不应轻易修改

---

### AST 类型系统

**位置**: `idl_tool_2/ast/ast_type.h`

**ASTType 基类**:
```cpp
class ASTType : public ASTNode {
public:
    enum class TypeKind {
        // 37 种类型：VOID, BOOLEAN, BYTE, ..., INTERFACE, STRUCT, ENUM, UNION, LIST, MAP, ...
    };

    enum class TypeMode { POD, SHARED, UNIQUE, INTERFACE };

    virtual TypeKind GetTypeKind() const = 0;
    virtual std::string GetSignature() const = 0;

    // 类型检查方法
    virtual bool IsVoidType() const = 0;
    virtual bool IsIntegerType() const = 0;
    virtual bool IsStringType() const = 0;
    virtual bool IsInterfaceType() const = 0;
    virtual bool IsPod() const = 0;
};
```

**关键接口**:
- `GetTypeKind()` - 获取类型种类
- `GetSignature()` - 获取类型签名（用于代码生成）
- `IsXXXType()` - 类型检查方法
- `IsPod()` - 是否为 Plain Old Data

**稳定性**: **稳定** - 类型系统是 IDL Tool 的核心，修改需要同步所有发射器

---

### AST 接口类型

**位置**: `idl_tool_2/ast/ast_interface_type.h`

**ASTInterfaceType 接口**:
```cpp
class ASTInterfaceType : public ASTType {
public:
    std::string GetName() const;
    std::string GetFullName() const;
    size_t GetMethodNumber() const;
    AutoPtr<ASTMethod> GetMethod(size_t index);

    void AddMethod(const AutoPtr<ASTMethod>& method);
    void AddExtendsInterface(const AutoPtr<ASTInterfaceType>& interface);

    // 属性检查
    bool IsOneWay() const;
    bool IsCallback() const;
    bool IsSerializable() const;
    bool IsCacheable() const;
};
```

**稳定性**: **稳定** - 接口定义是 IDL 的核心结构

---

### AST 方法类型

**位置**: `idl_tool_2/ast/ast_method.h`

**ASTMethod 接口**:
```cpp
class ASTMethod : public ASTNode {
public:
    std::string GetName() const;
    AutoPtr<ASTType> GetReturnType() const;
    size_t GetParameterNumber() const;
    AutoPtr<ASTParameter> GetParameter(size_t index);

    void AddParameter(const AutoPtr<ASTParameter>& parameter);

    // 命令 ID（用于 IPC）
    int GetCmdId() const;

    // 属性
    bool IsOneWay() const;
    bool IsFull() const;
    bool IsLite() const;
};
```

**稳定性**: **稳定** - 方法是接口的核心组成部分

---

## Parser 模块接口

### Lexer 类

**位置**: `idl_tool_2/lexer/lexer.h`

**Lexer 接口**:
```cpp
class Lexer {
public:
    bool Reset(const std::string& filePath);
    Token GetToken();
    Token PeekToken();

private:
    Token ReadId();
    Token ReadNum();
    Token ReadComment();
    // ... 其他读取方法
};
```

**关键方法**:
- `Reset()` - 初始化 lexer，加载文件
- `GetToken()` - 获取下一个 token
- `PeekToken()` - 预览下一个 token（不消耗）

**稳定性**: **稳定** - Lexer 是解析流程的基础

---

### Parser 类

**位置**: `idl_tool_2/parser/parser.h`

**Parser 接口**:
```cpp
class Parser {
public:
    bool Parse(const std::vector<FileDetail>& fileDetails);

    inline const StrAstMap& GetAllAst() const {
        return allAsts_;
    }

private:
    // 解析方法
    bool ParseInterface(const AttrSet& attrs = {});
    bool ParseTypeDecls();
    AutoPtr<ASTType> ParseType();
    AutoPtr<ASTMethod> ParseMethod();
    AutoPtr<ASTParameter> ParseParam();

    // 类型检查
    void CheckInterfaceAttr(const AutoPtr<ASTInterfaceType>& interface);
    void CheckMethodAttr(const AutoPtr<ASTInterfaceType>& interface, const AutoPtr<ASTMethod>& method);
};
```

**稳定性**: **稳定** - Parser 是 AST 构建的核心

---

## Codegen 模块接口

### CodeEmitter 基类

**位置**: `idl_tool_2/codegen/code_emitter.h`

**CodeEmitter 接口**:
```cpp
class CodeEmitter : public LightRefCountBase {
public:
    virtual bool OutPut(const AutoPtr<AST>& ast, const std::string& targetDirectory, GenMode mode) = 0;

protected:
    virtual void EmitCode() = 0;

    // 代码生成辅助方法
    void EmitLicense(StringBuilder& sb);
    std::string EmitMethodCmdID(const AutoPtr<ASTMethod>& method);
    std::string PackageToFilePath(const std::string& packageName) const;
    std::string InterfaceToFilePath(const std::string& interfaceName) const;
};
```

**关键接口**:
- `OutPut()` - 主入口，调用 `EmitCode()` 并写入文件
- `EmitCode()` - 实际代码生成逻辑（由子类实现）

**稳定性**: **稳定** - 所有后端都继承此基类

---

### CodeGenerator 类

**位置**: `idl_tool_2/codegen/code_generator.h`

**CodeGenerator 接口**:
```cpp
class CodeGenerator : public LightRefCountBase {
public:
    virtual bool DoGenerate(const StrAstMap& allAst);
};

class CodegenBuilder : public LightRefCountBase {
public:
    static CodegenBuilder& GetInstance();

    void GeneratorRegister(InterfaceType type, AutoPtr<CodeGenerator> generator);

private:
    GeneratorEntry generators;  // std::map<InterfaceType, AutoPtr<CodeGenerator>>
};
```

**稳定性**: **稳定** - 代码生成器入口

---

## HDI 后端接口

### HDICodeEmitter

**位置**: `idl_tool_2/codegen/HDI/hdi_code_emitter.h`

**HDICodeEmitter 接口**:
```cpp
class HDICodeEmitter : public CodeEmitter {
protected:
    virtual void EmitCode() override;
};
```

**子发射器接口**:
| 发射器 | 文件 | 接口定义 |
|--------|------|---------|
| `HDIInterfaceCodeEmitter` | HDI/cpp/cpp_interface_code_emitter.h | 生成接口头文件 |
| `HDIClientProxyCodeEmitter` | HDI/cpp/cpp_client_proxy_code_emitter.h | 生成客户端代理 |
| `HDIServiceStubCodeEmitter` | HDI/cpp/cpp_service_stub_code_emitter.h | 生成服务桩 |
| `HDICustomTypesCodeEmitter` | HDI/cpp/cpp_custom_types_code_emitter.h | 生成自定义类型 |
| `HDIServiceDriverCodeEmitter` | HDI/cpp/cpp_service_driver_code_emitter.h | 生成服务驱动 |

**稳定性**: **稳定** - HDI 后端已成熟

---

## SA 后端接口

### SACodeEmitter

**位置**: `idl_tool_2/codegen/SA/sa_code_emitter.h`

**SACodeEmitter 接口**:
```cpp
class SACodeEmitter : public CodeEmitter {
protected:
    virtual void EmitCode() override;
};
```

**子发射器接口**:
| 发射器 | 文件 | 接口定义 |
|--------|------|---------|
| `SACppInterfaceCodeEmitter` | SA/cpp/sa_cpp_interface_code_emitter.h | 生成 C++ 接口 |
| `SACppClientCodeEmitter` | SA/cpp/sa_cpp_client_code_emitter.h | 生成 C++ 客户端代理 |
| `SACppServiceStubCodeEmitter` | SA/cpp/sa_cpp_service_stub_code_emitter.h | 生成 C++ 服务桩 |
| `SACppCustomTypesCodeEmitter` | SA/cpp/sa_cpp_custom_types_code_emitter.h | 生成 C++ 自定义类型 |
| `SATsClientProxyCodeEmitter` | SA/ts/sa_ts_client_proxy_code_emitter.h | 生成 TS 客户端代理 |
| `SATsServiceStubCodeEmitter` | SA/ts/sa_ts_service_stub_code_emitter.h | 生成 TS 服务桩 |
| `SARustInterfaceCodeEmitter` | SA/rust/sa_rust_interface_code_emitter.h | 生成 Rust 接口 |

**稳定性**: **稳定** - SA 后端已成熟

---

## Metadata 模块接口

### MetadataBuilder

**位置**: `idl_tool_2/metadata/metadata_builder.h`

**MetadataBuilder 接口**:
```cpp
class MetadataBuilder {
public:
    std::shared_ptr<MetaComponent> Build();
};
```

**关键方法**:
- `Build()` - 从 AST 构建元数据（两遍算法）

**稳定性**: **稳定** - 元数据构建是 SA 模式的核心

---

### MetadataSerializer

**位置**: `idl_tool_2/metadata/metadata_serializer.h`

**MetadataSerializer 接口**:
```cpp
class MetadataSerializer {
public:
    void Serialize();
    void Deserialize();

    size_t GetDataSize() const;
    uintptr_t GetData() const;
};
```

**关键方法**:
- `Serialize()` - 指针转偏移量（位置无关存储）
- `Deserialize()` - 偏移量转指针
- `GetData()` - 获取二进制数据

**稳定性**: **稳定** - 元数据序列化是 SA 模式的核心

---

## Util 模块接口

### Options 类

**位置**: `idl_tool_2/util/options.h`

**Options 接口**:
```cpp
class Options {
public:
    static Options& GetInstance();
    bool Parse(int argc, char* argv[]);

    // 配置访问器
    bool DoShowUsage() const;
    bool DoCompile() const;
    bool DoGenerateCode() const;
    bool DoDumpAST() const;
    bool DoDumpMetadata() const;
    bool DoSaveMetadata() const;

    std::string GetGenerationDirectory() const;
    std::set<std::string> GetSourceFiles() const;
    InterfaceType GetInterfaceType() const;
    Language GetLanguage() const;
    SystemLevel GetSystemLevel() const;
};
```

**稳定性**: **稳定** - 命令行选项解析

---

### StringBuilder 类

**位置**: `idl_tool_2/util/string_builder.h`

**StringBuilder 接口**:
```cpp
class StringBuilder {
public:
    StringBuilder& Append(const char* format, ...);
    StringBuilder& Append(const std::string& str);
    StringBuilder& Append(char c);
    std::string ToString() const;
};
```

**关键方法**:
- `Append()` - 追加内容（支持格式化字符串）
- `ToString()` - 获取最终字符串

**稳定性**: **稳定** - 代码生成的核心工具

---

## 依赖方向

### 自底向上依赖

```
Util (工具库)
    ↑
    ├── AST
    ├── Lexer
    ├── Parser
    ├── MetadataBuilder
    ├── CodeEmitter
    └── CodeGenerator

CodeEmitter
    ↑
    StringBuilder
    ├── HDICodeEmitter
    ├── SACodeEmitter
    └── ...

MetadataBuilder
    ↑
    AST
```

**特点**:
- 无循环依赖
- 清晰的层次结构
- 下层模块不依赖上层模块

---

## 稳定性评估

### 稳定接口（不建议修改）

| 模块 | 接口 | 原因 |
|------|------|------|
| AST | `ASTNode`, `ASTType` | 核心数据结构，所有代码生成依赖 |
| Parser | `Parser`, `Lexer` | AST 构建的核心 |
| CodeEmitter | `CodeEmitter` | 所有后端的基础 |
| Metadata | `MetadataBuilder`, `MetadataSerializer` | SA 模式的核心 |

### 可扩展接口（谨慎修改）

| 模块 | 接口 | 修改影响 |
|------|------|------|
| Codegen | `CodeGenerator`, `CodegenBuilder` | 添加新后端 |
| HDI/SA Emitter | 各后端发射器 | 修改特定语言生成逻辑 |
| Util | `Options`, `StringBuilder` | 添加新命令行选项或优化字符串构建 |

---

## 可替换点

### 代码生成后端

**添加新的语言支持**：
1. 在 `idl_tool_2/codegen/` 创建新目录（如 `PYTHON/`）
2. 实现新的 `CodeEmitter` 子类
3. 在 `CodegenBuilder::GeneratorRegister()` 注册
4. 在 `util/common.h` 添加新的 `Language` 枚举值

**证据**: `idl_tool_2/codegen/code_generator.h:39-40`

### 类型系统扩展

**添加新类型支持**：
1. 在 `idl_tool_2/ast/` 创建新的类型节点类
2. 在 `ast_type.h` 的 `TypeKind` 枚举添加新类型
3. 在 `Parser` 中添加解析逻辑
4. 在各后端的 TypeEmitter 中添加类型映射

**证据**: `idl_tool_2/ast/ast_type.h:29-55`

---

## 关键结论

1. **清晰的接口层次**：基类定义明确，职责单一
2. **依赖方向正确**：自底向上，无循环依赖
3. **稳定与可扩展分离**：核心接口稳定，扩展点明确
4. **引用计数管理**：所有模块使用 `AutoPtr<T>` 和 `LightRefCountBase`

---

## 相关文档

- [01_Directory_Structure.md](01_Directory_Structure.md) - 模块职责与依赖关系
- [02_Architecture.md](02_Architecture.md) - 架构设计与数据流

---

**最后更新**: 2026-02-06
