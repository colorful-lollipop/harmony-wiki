# OpenHarmony IDL Tool - 目录结构与模块职责

**目的**：详细说明项目目录结构、各模块职责以及模块间的依赖关系。

---

## 适用范围

本文档适用于：
- 理解 IDL Tool 代码组织结构的开发者
- 需要定位功能模块的工程师
- 希望修改或扩展 IDL Tool 的开发者

---

## 顶层目录结构

```
idl_tool/
├── idl_tool_2/              # 主代码库（新版本）
│   ├── ast/                # 抽象语法树模块
│   ├── codegen/            # 代码生成模块
│   ├── lexer/              # 词法分析器
│   ├── metadata/           # 元数据模块
│   ├── parser/             # 语法分析器
│   ├── preprocessor/        # 预处理器
│   ├── util/               # 工具库
│   └── main.cpp            # 主程序入口
├── figures/                # 图表资源
├── BUILD.gn               # GN 构建配置
├── bundle.json             # Bundle 配置
└── wiki/                  # 本文档系统
```

---

## 核心模块职责

### 1. AST 模块（`idl_tool_2/ast/`）

**职责**：定义和表示 IDL 语言的抽象语法树

**证据**: `idl_tool_2/ast/ast.h:68-369`

| 子模块 | 文件 | 职责 |
|--------|------|------|
| **基础类型** | base/ | 基本数据类型节点（boolean, byte, int, long, string 等） |
| **复合类型** | ast_array_type.h/cpp, ast_map_type.h/cpp 等 | 容器类型（list, map, array, set 等） |
| **接口类型** | ast_interface_type.h/cpp | 接口定义节点（方法、属性、继承） |
| **方法节点** | ast_method.h/cpp | 接口方法节点（参数、返回类型、属性） |
| **参数节点** | ast_parameter.h/cpp | 方法参数节点（名称、类型、方向） |
| **结构体** | ast_struct_type.h/cpp | 结构体定义节点（成员、父类） |
| **枚举** | ast_enum_type.h/cpp | 枚举定义节点（成员、基类型） |
| **联合体** | ast_union_type.h/cpp | 联合体定义节点（成员） |
| **命名空间** | ast_namespace.h/cpp | 命名空间层次结构 |
| **属性** | ast_attribute.h/cpp | AST 节点属性（接口/方法属性、参数方向） |
| **表达式** | ast_expr.h/cpp | 表达式节点（用于枚举值、常量） |

**关键类**:
- `ASTNode` - 所有 AST 节点的基类
- `ASTType` - 所有类型节点的基类（37 种 TypeKind）
- `AST` - 根节点，代表整个 IDL 文件

---

### 2. Parser 模块（`idl_tool_2/parser/`）

**职责**：将 IDL 源代码解析为 AST

**证据**: `idl_tool_2/parser/parser.h:63-346`

| 子模块 | 文件 | 职责 |
|--------|------|------|
| **Parser 类** | parser.h/cpp | 主解析器，递归下降算法 |
| **Lexer 类** | lexer/lexer.h/cpp | 词法分析器，tokenization |
| **类型检查** | intf_type_check.h/cpp | 接口类型完整性与一致性检查 |

**关键方法**:
- `Parse()` - 主解析入口
- `ParseInterface()` - 解析接口声明
- `ParseMethod()` - 解析方法声明
- `ParseType()` - 解析类型声明（enum, struct, union）
- `ParseExpression()` - 解析表达式

**依赖关系**:
```
Parser → Lexer (获取 tokens)
Parser → AST (构建 AST 节点)
Parser → IntfTypeChecker (语义验证)
```

---

### 3. Codegen 模块（`idl_tool_2/codegen/`）

**职责**：根据 AST 生成目标语言代码

**证据**: `idl_tool_2/codegen/code_emitter.h:79-156`

| 子模块 | 目录 | 职责 |
|--------|------|------|
| **HDI 后端** | HDI/ | 硬件设备接口代码生成 |
|   - C 语言生成器** | HDI/c/ | 生成 C 语言接口、服务驱动、桩 |
|   - C++ 生成器** | HDI/cpp/ | 生成 C++ 接口、自定义类型、代理 |
|   - Java 生成器** | HDI/java/ | 生成 Java 客户端接口 |
|   - 类型发射器** | HDI/type/ | HDI 类型映射逻辑 |
| **SA 后端** | SA/ | 系统能力接口代码生成 |
|   - C++ 生成器** | SA/cpp/ | 生成 SA C++ 客户端代码、代理 |
|   - TypeScript 生成器** | SA/ts/ | 生成 SA TypeScript 代理 |
|   - Rust 生成器** | SA/rust/ | 生成 SA Rust 接口代码 |
|   - 类型发射器** | SA/type/ | SA 类型映射逻辑 |
| **基础框架** | code_emitter.h/cpp, code_generator.h/cpp | 代码生成基类 |

**关键类**:
- `CodeEmitter` - 所有后端的基类（虚方法 `OutPut()`）
- `CodeGenerator` - 代码生成器注册表
- `CodegenBuilder` - 管理多个 CodeGenerator 实例

**依赖关系**:
```
Codegen → AST (读取 AST 结构）
Codegen → Options (获取生成选项）
Codegen → Util (字符串构建、文件操作）
```

---

### 4. Metadata 模块（`idl_tool_2/metadata/`）

**职责**：生成和读取运行时类型信息（RTTI）

**证据**: `idl_tool_2/metadata/meta_component.h:1-20`

| 子模块 | 文件 | 职责 |
|--------|------|------|
| **元数据结构** | meta_*.h | 元数据类定义（接口、方法、参数等） |
| **构建器** | metadata_builder.h/cpp | AST → 元数据转换 |
| **序列化器** | metadata_serializer.h/cpp | 指针与偏移量转换 |
| **读取器** | metadata_reader.h/cpp | 文件 → 元数据转换 |
| **输出器** | metadata_dumper.h/cpp | 元数据格式化输出 |

**关键类**:
- `MetaComponent` - 根元数据容器
- `MetaInterface` - 接口元数据
- `MetaMethod` - 方法元数据
- `MetaParameter` - 参数元数据
- `MetaType` - 类型元数据（15 种 MetaTypeKind）

**依赖关系**:
```
MetadataBuilder → AST (遍历 AST 构建）
MetadataSerializer → MetadataBuilder (序列化）
MetadataReader → MetadataBuilder (反序列化）
```

---

### 5. Lexer 模块（`idl_tool_2/lexer/`）

**职责**：将 IDL 源代码转换为 token 流

**证据**: `idl_tool_2/lexer/lexer.h:23-95`

| 文件 | 职责 |
|------|------|
| **Lexer 类** | lexer.h/cpp | 词法分析器，tokenization 逻辑 |
| **Token 定义** | token.h/cpp | Token 类和 TokenType 枚举（85+ 种类型） |

**Token 类别**:
- 基础类型关键字（void, boolean, byte, int, long 等）
- 字符串类型关键字（String, String16, CString）
- 容器类型关键字（List, Set, Map, OrderedMap）
- 智能指针关键字（shared_ptr, unique_ptr, sptr）
- 特殊类型关键字（fd, fdsan, ashmem, NativeBuffer）
- 声明关键字（interface, enum, struct, union, extends）
- 包/导入关键字（package, import, sequenceable, rawdata）
- 属性关键字（oneway, callback, full, lite, mini, cacheable）
- 参数方向（in, out, inout）
- 操作符和分隔符（+, -, *, /, &, |, <<, >>, ., ,, ;, {} 等）

**依赖关系**:
```
Parser → Lexer (获取 tokens）
```

---

### 6. Preprocessor 模块（`idl_tool_2/preprocessor/`）

**职责**：预处理 IDL 源文件

**证据**: `idl_tool_2/preprocessor/preprocessor.h`

| 文件 | 职责 |
|------|------|
| **Preprocessor 类** | preprocessor.h/cpp | 文件预处理、宏展开、包含处理 |

**依赖关系**:
```
Parser → Preprocessor (预处理源文件）
```

---

### 7. Util 模块（`idl_tool_2/util/`）

**职责**：提供通用工具和辅助功能

**证据**: `idl_tool_2/util/common.h:16-72`

| 文件 | 职责 |
|------|------|
| **common.h** | 公共定义（SystemLevel, GenMode, Language, InterfaceType 枚举） |
| **autoptr.h/cpp** | 智能指针类（引用计数） |
| **file.h/cpp** | 文件读写操作 |
| **light_refcount_base.h/cpp** | 引用计数基类 |
| **logger.h/cpp** | 日志输出（Logger::E, Logger::W, Logger::I） |
| **options.h/cpp** | 命令行选项解析（Options::GetInstance(), Parse()） |
| **string.h/cpp** | 字符串工具类 |
| **string_builder.h/cpp** | 高效字符串构建器 |
| **string_pool.h/cpp** | 字符串池（去重、减少内存占用） |

**依赖关系**:
```
所有模块 → Util（工具函数）
Parser → Options (命令行参数）
Codegen → StringBuilder (代码生成）
```

---

## 模块依赖关系图

```mermaid
graph TD
    main_cpp[main.cpp 主程序] --> Options[Options]
    main_cpp --> Parser[Parser]
    main_cpp --> CodeGenerator[CodeGenerator]

    Parser --> Lexer[Lexer]
    Parser --> AST[AST]
    Parser --> Preprocessor[Preprocessor]
    Parser --> IntfTypeChecker[IntfTypeChecker]

    Lexer --> Token[Token]

    CodeGenerator --> CodeEmitter[CodeEmitter]
    CodeGenerator --> MetadataBuilder[MetadataBuilder]

    MetadataBuilder --> AST
    MetadataBuilder --> MetaComponent[MetaComponent]
    MetadataBuilder --> MetaInterface[MetaInterface]
    MetadataBuilder --> MetaMethod[MetaMethod]
    MetadataBuilder --> MetaParameter[MetaParameter]
    MetadataBuilder --> MetaType[MetaType]

    MetadataSerializer --> MetadataBuilder
    MetadataReader --> MetaComponent

    class ASTNode[ASTNode 基类] -.->|基础| ASTType[ASTType]
    class ASTType -.->|基础| ASTInterfaceType[ASTInterfaceType]
    class ASTType -.->|基础| ASTStructType[ASTStructType]
    class ASTType -.->|基础| ASTEnumType[ASTEnumType]
    class ASTType -.->|基础| ASTUnionType[ASTUnionType]

    ASTInterfaceType --> ASTMethod[ASTMethod]
    ASTMethod --> ASTParameter[ASTParameter]

    AST --> ASTNamespace[ASTNamespace]
```

---

## 关键文件映射

| 功能 | 文件路径 | 说明 |
|------|---------|------|
| **主程序入口** | idl_tool_2/main.cpp | 命令行工具入口 |
| **GN 构建配置** | BUILD.gn | 定义 `idl` 可执行文件 |
| **Bundle 配置** | bundle.json | 组件元数据（@ohos/idl_tool v3.1） |
| **IDL 语法** | IDL-CPP.md | C++ IDL 语法文档 |
| **IDL 构建** | IDL-CPP-GN.md | IDL GN 构建指南 |

---

## 关键结论

1. **模块化设计**：清晰的单职责原则，各模块职责明确
2. **依赖单向**：上层模块依赖下层模块，无循环依赖
3. **基类抽象**：ASTNode、ASTType、CodeEmitter 等提供良好的抽象
4. **工具复用**：Util 模块提供通用功能，各模块复用
5. **后端独立**：HDI 和 SA 后端相互独立，互不影响

---

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览与核心能力
- [02_Architecture.md](02_Architecture.md) - 完整架构设计与数据流
- [04_Internal_API.md](04_Internal_API.md) - 内部模块接口详解
- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建系统
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物说明

---

**最后更新**: 2026-02-06
