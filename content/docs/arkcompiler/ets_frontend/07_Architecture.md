# 架构详解

## 编译流程总览

### 数据流图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         编译流程                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  源代码 (.js/.ts/.ets)                                                   │
│         │                                                               │
│         ▼                                                               │
│  ┌─────────────────┐                                                    │
│  │    Lexer        │  词法分析                                          │
│  │  (tokenize)     │  Source → Tokens                                  │
│  └────────┬────────┘                                                    │
│           │ Tokens                                                      │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │    Parser       │  语法解析                                          │
│  │  (parse)        │  Tokens → AST                                      │
│  └────────┬────────┘                                                    │
│           │ AST                                                         │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │    Binder       │  符号绑定                                          │
│  │  (bind)         │  AST → Annotated AST                              │
│  └────────┬────────┘                                                    │
│           │ Annotated AST                                               │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │    Checker      │  类型检查 (ETS 专用)                               │
│  │  (check)        │  类型验证                                          │
│  └────────┬────────┘                                                    │
│           │ Checked AST                                                 │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │    IR Builder   │  中间表示                                          │
│  │  (generate)     │  AST → IR                                          │
│  └────────┬────────┘                                                    │
│           │ IR                                                          │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │ Bytecode Gen    │  字节码生成                                        │
│  │  (emit)         │  IR → Bytecode                                    │
│  └────────┬────────┘                                                    │
│           │ Bytecode                                                    │
│           ▼                                                             │
│  ┌─────────────────┐                                                    │
│  │  Bytecode Opt   │  字节码优化 (可选)                                 │
│  │  (optimize)     │  优化 Passes                                       │
│  └────────┬────────┘                                                    │
│           │ Optimized Bytecode                                         │
│           ▼                                                             │
│  .abc (ARK Bytecode)                                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 关键编译 Passes

| Pass | 模块 | 描述 |
|------|------|------|
| Lexer | lexer | 词法分析，生成 Token 序列 |
| Parser | parser | 语法分析，生成 AST |
| Binder | binder/varbinder | 符号解析，作用域分析 |
| Checker | checker | 类型检查 (ETS) |
| IR Generation | ir | 构建中间表示 |
| Bytecode Emission | ir/compiler | 字节码生成 |
| Optimization | compiler | 字节码优化 |

## es2panda 架构 (JS/TS)

### 模块职责

#### Lexer (词法分析器)

**入口**: `es2panda/lexer/lexer.cpp`

**职责**:
- 源代码 → Token 序列
- 识别关键字、标识符、字面量、运算符
- 处理 Unicode 和转义字符

**关键类**:
- `Lexer` - 主词法分析器
- `Token` - Token 定义
- `TokenType` - Token 类型枚举

#### Parser (语法解析器)

**入口**: `es2panda/parser/parser.cpp`

**职责**:
- Token 序列 → AST
- 构建程序结构 (函数、类、表达式)
- 错误恢复

**关键类**:
- `Parser` - 主解析器
- `Program` - 顶层 AST 节点
- `Expression` - 表达式基类
- `Statement` - 语句基类

#### Binder (符号绑定器)

**入口**: `es2panda/binder/binder.cpp`

**职责**:
- 符号解析与作用域管理
- 声明收集
- 引用消解

**关键类**:
- `Binder` - 主绑定器
- `Scope` - 作用域
- `Variable` - 变量

#### IR & Bytecode (中间表示与字节码)

**入口**: `es2panda/ir/ir.cpp`

**职责**:
- AST → IR 转换
- 字节码指令生成
- 常量池管理

**关键类**:
- `IRBuilder` - IR 构建器
- `BytecodeBuilder` - 字节码构建器
- `Ins` - 指令定义

## ets2panda 架构 (ETS)

### 模块职责

#### VarBinder (变量绑定器)

**入口**: `ets2panda/varbinder/`

**职责**:
- ETS 专用符号绑定
- 支持 TypeScript 扩展

**关键类**:
- `TSBinder` - TypeScript 绑定器
- `JSBinder` - JavaScript 绑定器
- `Variable` - 变量

#### Checker (类型检查器)

**入口**: `ets2panda/checker/`

**职责**:
- 类型推断
- 类型兼容性检查
- 错误报告

#### N-API Bindings (Native 互操作)

**入口**: `ets2panda/bindings/native/`

**职责**:
- Native ↔ JS 类型转换
- Promise/Async 支持
- 动态模块加载

## 线程模型

### 编译线程

```
主线程
  │
  ├── 词法分析 (单线程)
  ├── 语法解析 (单线程)
  ├── 符号绑定 (单线程)
  ├── IR 生成 (可配置多线程)
  └── 字节码输出 (单线程)
```

### 并行编译

```bash
# 指定线程数
es2abc --thread 4 input.js
```

**线程使用**:
- `--thread 0` - 自动 (CPU 核心数)
- `--thread N` - 使用 N 个线程

### 内存管理

- **Arena 分配器**: 编译过程使用内存池
- **引用计数**: 循环引用检测
- **智能指针**: C++ RAII 管理

## 错误处理

### 错误传播

```
编译错误
    │
    ▼
┌──────────────┐
│ ErrorReporter │  收集并报告错误
└──────┬───────┘
       │
       ▼
    退出码 1
```

### 错误类型

| 错误类型 | 退出码 | 描述 |
|----------|--------|------|
| 词法错误 | 1 | 无效 Token |
| 语法错误 | 1 | 语法结构错误 |
| 语义错误 | 1 | 类型/引用错误 |
| 系统错误 | 2 | IO/内存错误 |

## 扩展点

### 编译插件

可扩展 Pass:
- 自定义优化 Pass
- 代码生成器
- 分析工具

### 配置选项

| 选项 | 描述 |
|------|------|
| `--opt-level` | 优化级别 (0/1/2) |
| `--debug-info` | 调试信息 |
| `--dump-ast` | 输出 AST |

## 相关文档

- [命令行接口](02_CLI_Reference.md)
- [目录结构](01_Directory_Structure.md)
- [N-API 绑定](03_NAPI_Bindings.md)
