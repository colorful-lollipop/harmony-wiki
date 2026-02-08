# 03 - 目录结构与代码地图 (CodeMap)

## 1. 顶层目录结构

```
foundation/ability/idl_tool/
├── idl_tool_2/              # 新版IDL工具主代码（本文档重点）
│   ├── ast/                 # 抽象语法树模块
│   ├── codegen/             # 代码生成模块
│   ├── hash/                # Hash生成模块
│   ├── lexer/               # 词法分析模块
│   ├── metadata/            # 元数据处理模块
│   ├── parser/              # 语法分析模块
│   ├── preprocessor/        # 预处理器模块
│   ├── util/                # 工具库
│   ├── main.cpp             # 程序入口
│   └── BUILD.gn             # 构建配置
├── ast/                     # 旧版AST模块（已废弃）
├── codegen/                 # 旧版代码生成（已废弃）
├── lexer/                   # 旧版词法分析（已废弃）
├── parser/                  # 旧版语法分析（已废弃）
├── metadata/                # 旧版元数据（已废弃）
├── preprocessor/            # 旧版预处理器（已废弃）
├── hash/                    # 旧版hash模块（已废弃）
├── util/                    # 旧版工具库（已废弃）
├── test/                    # 测试目录
├── wiki/                    # 本文档
├── README.md                # 项目说明
├── bundle.json              # 组件配置
└── BUILD.gn                 # GN构建配置
```

**注意**: 旧版代码（根目录下除 idl_tool_2/ 和 test/ 外）已废弃，新版本代码集中在 `idl_tool_2/` 目录。

---

## 2. 核心目录详解

### 2.1 idl_tool_2/ast/ - 抽象语法树

**职责**: 定义 IDL 语法结构的 AST 节点类型

```
idl_tool_2/ast/
├── ast.h/cpp                # AST 根容器类
├── ast_node.h/cpp           # AST 节点基类
├── ast_type.h/cpp           # 类型基类，TypeKind 枚举
├── ast_attribute.h/cpp      # 属性定义
├── ast_expr.h/cpp           # 表达式节点
├── ast_namespace.h/cpp      # 命名空间
├── ast_method.h/cpp         # 方法节点
├── ast_parameter.h/cpp      # 参数节点
├── ast_interface_type.h/cpp # 接口类型
├── ast_struct_type.h/cpp    # 结构体类型
├── ast_enum_type.h/cpp      # 枚举类型
├── ast_union_type.h/cpp     # 联合体类型
├── ast_array_type.h/cpp     # 数组类型
├── ast_map_type.h/cpp       # 映射类型
├── ast_set_type.h/cpp       # 集合类型
├── ast_ptr_type.h/cpp       # 智能指针类型
├── ast_sequenceable_type.h  # 序列化类型
├── ast_rawdata_type.h       # 原始数据类型
├── ast_fd_type.h            # 文件描述符类型
├── ast_fdsan_type.h         # FDsan类型
├── ast_smq_type.h           # SMQ类型
├── ast_native_buffer_type.h # 本地缓冲区类型
├── ast_pointer_type.h       # 指针类型
├── ast_void_type.h          # void类型
└── base/                    # 基础类型
    ├── ast_boolean_type.h/cpp
    ├── ast_byte_type.h/cpp
    ├── ast_char_type.h/cpp
    ├── ast_cstring_type.h/cpp
    ├── ast_double_type.h/cpp
    ├── ast_float_type.h/cpp
    ├── ast_integer_type.h/cpp
    ├── ast_long_type.h/cpp
    ├── ast_short_type.h/cpp
    ├── ast_string_type.h/cpp
    ├── ast_string16_type.h/cpp
    ├── ast_u16string_type.h/cpp
    ├── ast_uchar_type.h/cpp
    ├── ast_uint_type.h/cpp
    ├── ast_ulong_type.h/cpp
    └── ast_ushort_type.h/cpp
```

**关键类**:
| 类名 | 文件 | 说明 |
|------|------|------|
| `AST` | `ast.h` | 根容器，包含所有命名空间 |
| `ASTNode` | `ast_node.h` | 所有节点的基类 |
| `ASTType` | `ast_type.h` | 所有类型的基类 |
| `ASTInterfaceType` | `ast_interface_type.h` | 接口类型定义 |
| `ASTMethod` | `ast_method.h` | 接口方法 |
| `ASTParameter` | `ast_parameter.h` | 方法参数 |

### 2.2 idl_tool_2/lexer/ - 词法分析器

**职责**: 将 IDL 文件字符流转换为 Token 序列

```
idl_tool_2/lexer/
├── lexer.h/cpp              # Lexer 类实现
└── token.h/cpp              # Token 类型定义
```

**关键类**:
| 类名 | 文件 | 说明 |
|------|------|------|
| `Lexer` | `lexer.h:30` | 词法分析器主类 |
| `Token` | `token.h:26` | Token 结构体 |
| `TokenType` | `token.h:23` | Token 类型枚举 |

### 2.3 idl_tool_2/parser/ - 语法分析器

**职责**: 将 Token 序列解析为 AST

```
idl_tool_2/parser/
├── parser.h/cpp             # Parser 类实现
└── intf_type_check.h/cpp    # 接口类型检查
```

**关键类**:
| 类名 | 文件 | 说明 |
|------|------|------|
| `Parser` | `parser.h:27` | 语法分析器主类 |
| `IntfTypeCheck` | `intf_type_check.h` | 接口类型校验 |

### 2.4 idl_tool_2/codegen/ - 代码生成器

**职责**: 根据 AST 生成目标语言代码

```
idl_tool_2/codegen/
├── code_emitter.h/cpp           # 代码发射器基类
├── code_generator.h/cpp         # 代码生成器基类
├── HDI/                         # HDI代码生成
│   ├── hdi_code_emitter.h/cpp   # HDI发射器基类
│   ├── hdi_code_generator.h/cpp # HDI生成器
│   ├── hdi_type_emitter.h/cpp   # HDI类型发射器
│   ├── c/                       # C语言生成
│   │   ├── c_client_proxy_code_emitter.h/cpp
│   │   ├── c_service_stub_code_emitter.h/cpp
│   │   ├── c_interface_code_emitter.h/cpp
│   │   ├── c_custom_types_code_emitter.h/cpp
│   │   ├── c_service_driver_code_emitter.h/cpp
│   │   └── c_service_impl_code_emitter.h/cpp
│   ├── cpp/                     # C++语言生成
│   │   ├── cpp_client_proxy_code_emitter.h/cpp
│   │   ├── cpp_service_stub_code_emitter.h/cpp
│   │   ├── cpp_interface_code_emitter.h/cpp
│   │   ├── cpp_custom_types_code_emitter.h/cpp
│   │   ├── cpp_service_driver_code_emitter.h/cpp
│   │   └── cpp_service_impl_code_emitter.h/cpp
│   ├── java/                    # Java语言生成
│   │   ├── java_client_proxy_code_emitter.h/cpp
│   │   ├── java_client_interface_code_emitter.h/cpp
│   │   └── hdi_java_code_emitter.h/cpp
│   └── type/                    # HDI类型发射器
│       ├── hdi_array_type_emitter.h/cpp
│       ├── hdi_struct_type_emitter.h/cpp
│       ├── hdi_enum_type_emitter.h/cpp
│       ├── hdi_map_type_emitter.h/cpp
│       ├── hdi_string_type_emitter.h/cpp
│       └── ... (24+ 类型发射器)
└── SA/                          # SA代码生成
    ├── sa_code_emitter.h/cpp    # SA发射器基类
    ├── sa_code_generator.h.cpp  # SA生成器
    ├── sa_type_emitter.h.cpp    # SA类型发射器
    ├── cpp/                     # C++生成
    │   ├── sa_cpp_client_proxy_code_emitter.h/cpp
    │   ├── sa_cpp_service_stub_code_emitter.h/cpp
    │   ├── sa_cpp_interface_code_emitter.h.cpp
    │   ├── sa_cpp_custom_types_code_emitter.h.cpp
    │   └── sa_cpp_client_code_emitter.h.cpp
    ├── ts/                      # TypeScript生成
    │   ├── sa_ts_client_proxy_code_emitter.h/cpp
    │   ├── sa_ts_service_stub_code_emitter.h/cpp
    │   ├── sa_ts_interface_code_emitter.h/cpp
    │   └── sa_ts_code_emitter.h.cpp
    ├── rust/                    # Rust生成
    │   ├── sa_rust_interface_code_emitter.h/cpp
    │   └── sa_rust_code_emitter.h.cpp
    └── type/                    # SA类型发射器
        ├── sa_array_type_emitter.h.cpp
        ├── sa_map_type_emitter.h.cpp
        ├── sa_struct_type_emitter.h.cpp
        └── ... (29个类型发射器)
```

**关键类**:
| 类名 | 文件 | 说明 |
|------|------|------|
| `CodeGenerator` | `code_generator.h:32` | 生成器基类 |
| `CodegenBuilder` | `code_generator.h:44` | 生成器注册表 |
| `CodeEmitter` | `code_emitter.h:79` | 发射器基类 |
| `HDICodeGenerator` | `HDI/hdi_code_generator.h:24` | HDI生成器 |
| `SACodeGenerator` | `SA/sa_code_generator.h:27` | SA生成器 |

### 2.5 idl_tool_2/metadata/ - 元数据处理

**职责**: 处理运行时元数据（RTTI）

```
idl_tool_2/metadata/
├── meta_component.h         # 根元数据结构
├── meta_interface.h         # 接口元数据
├── meta_method.h            # 方法元数据
├── meta_patameter.h         # 参数元数据
├── meta_namespace.h         # 命名空间元数据
├── meta_type.h              # 类型元数据
├── meta_sequenceable.h      # 序列化类型元数据
├── meta_rawdata.h           # 原始数据元数据
├── metadata_builder.h/cpp   # AST → 元数据转换
├── metadata_serializer.h/cpp # 序列化/反序列化
├── metadata_reader.h.cpp    # 元数据读取
└── metadata_dumper.h.cpp    # 元数据输出
```

**关键类**:
| 类名 | 文件 | 说明 |
|------|------|------|
| `MetaComponent` | `meta_component.h` | 根容器 |
| `MetadataBuilder` | `metadata_builder.h` | 构建器 |
| `MetadataSerializer` | `metadata_serializer.h` | 序列化器 |
| `MetadataReader` | `metadata_reader.h` | 读取器 |

### 2.6 idl_tool_2/util/ - 工具库

**职责**: 通用工具类和辅助函数

```
idl_tool_2/util/
├── autoptr.h                # 智能指针
├── common.h                 # 公共定义（枚举等）
├── file.h/cpp               # 文件操作
├── light_refcount_base.h/cpp # 引用计数基类
├── logger.h/cpp             # 日志
├── options.h/cpp            # 命令行选项
├── string_builder.h.cpp     # 字符串构建器
├── string_helper.h.cpp      # 字符串辅助
└── string_pool.h.cpp        # 字符串池
```

**关键类**:
| 类名 | 文件 | 说明 |
|------|------|------|
| `Options` | `options.h:28` | 命令行选项管理 |
| `File` | `file.h:27` | 文件操作封装 |
| `StringBuilder` | `string_builder.h:25` | 字符串构建 |
| `Logger` | `logger.h:26` | 日志输出 |
| `AutoPtr` | `autoptr.h:25` | 智能指针 |

---

## 3. 关键文件导航

### 3.1 程序入口

```
idl_tool_2/main.cpp:127
```
**main() 函数** - 程序入口，协调解析和代码生成流程。

### 3.2 核心处理流程

```
1. 选项解析
   idl_tool_2/util/options.cpp:64
   
2. 文件预处理
   idl_tool_2/preprocessor/preprocessor.cpp
   
3. 词法分析
   idl_tool_2/lexer/lexer.cpp:118
   
4. 语法分析
   idl_tool_2/parser/parser.cpp:56
   
5. 元数据构建
   idl_tool_2/metadata/metadata_builder.cpp
   
6. 代码生成
   idl_tool_2/codegen/code_generator.cpp:22
```

### 3.3 类型系统

```
1. 类型基类
   idl_tool_2/ast/ast_type.h:29
   
2. TypeKind 枚举
   idl_tool_2/ast/ast_type.h:37
   
3. 接口类型
   idl_tool_2/ast/ast_interface_type.h:28
   
4. HDI 类型发射
   idl_tool_2/codegen/HDI/hdi_type_emitter.h:30
   
5. SA 类型发射
   idl_tool_2/codegen/SA/sa_type_emitter.h:32
```

---

## 4. 代码阅读路线

### 4.1 快速了解（30分钟）

```
1. 入口和流程
   idl_tool_2/main.cpp:127-178
   
2. 选项定义
   idl_tool_2/util/options.h:28-100
   
3. Token 类型
   idl_tool_2/lexer/token.h:23-123
   
4. AST 节点基类
   idl_tool_2/ast/ast_node.h:25-60
   
5. 类型枚举
   idl_tool_2/ast/ast_type.h:37-77
   
6. 接口类型
   idl_tool_2/ast/ast_interface_type.h:28-80
```

### 4.2 深入理解（2小时）

```
1. 词法分析器
   idl_tool_2/lexer/lexer.cpp:118-289
   
2. 语法分析器
   idl_tool_2/parser/parser.cpp:56-140
   
3. 代码生成策略
   idl_tool_2/codegen/HDI/hdi_code_generator.cpp:36-115
   idl_tool_2/codegen/SA/sa_code_generator.cpp:32-54
   
4. 元数据处理
   idl_tool_2/metadata/metadata_builder.cpp
   
5. 文件操作
   idl_tool_2/util/file.cpp:48-156
```

### 4.3 扩展开发（按需）

```
1. 添加新类型
   - 在 ast/ 添加新 AST 类型
   - 在 lexer/token.h 添加 Token 类型
   - 在 parser/ 添加解析逻辑
   - 在 codegen/ 添加发射逻辑
   
2. 添加新语言
   - 在 codegen/HDI/ 或 codegen/SA/ 创建新目录
   - 继承 CodeEmitter 实现发射器
   - 在 CodeGenerator 注册
```

---

## 5. 模块依赖图

```mermaid
graph TB
    subgraph "Frontend"
        LEX[lexer]
        PAR[parser]
    end
    
    subgraph "AST"
        AST[ast]
    end
    
    subgraph "Backend"
        CG[codegen]
        META[metadata]
    end
    
    subgraph "Util"
        UTIL[util]
    end
    
    subgraph "Main"
        MAIN[main.cpp]
        PRE[preprocessor]
    end
    
    UTIL --> LEX
    UTIL --> AST
    UTIL --> META
    UTIL --> CG
    
    LEX --> PAR
    AST --> PAR
    
    PAR --> MAIN
    AST --> MAIN
    META --> MAIN
    CG --> MAIN
    PRE --> MAIN
```

---

**上一步**: [02_Architecture.md](02_Architecture.md)
**下一步**: [04_Interface.md](04_Interface.md)
