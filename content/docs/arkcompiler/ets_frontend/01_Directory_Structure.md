# 目录结构与模块职责

## 顶层目录

```
arkcompiler/ets_frontend/
├── es2panda/              # [核心] JS/TS 编译器
├── ets2panda/             # [核心] ETS 编译器 (Enhanced TypeScript)
├── merge_abc/             # ABC 文件合并工具
├── arkguard/              # 代码保护/混淆
├── legacy_bin/            # API8 旧版编译器 (已废弃)
├── test/                  # SDK/XTS 测试用例 [测试目录]
├── testTs/                # 系统测试用例 [测试目录]
├── test262/               # Test262 测试配置 [测试目录]
├── test_ecma_bcopt/       # 字节码优化测试 [测试目录]
├── BUILD.gn               # 主构建入口
├── bundle.json            # 组件配置
├── ark_config.gni         # 编译配置
└── README.md              # 项目说明
```

## es2panda 模块 (JS/TS 编译器)

**职责**: 将 JavaScript/TypeScript 代码编译为 ARK 字节码

```
es2panda/
├── aot/                   # AOT 编译器入口
│   └── *.cpp             # AOT 模式编译逻辑
├── binder/                # 符号绑定与解析
│   ├── binder.cpp/h      # 绑定器主实现
│   └── *.cpp             # 各类声明/表达式绑定
├── compiler/             # 编译优化与代码生成
│   ├── base/             # 基础编译结构
│   ├── instruction/      # 指令生成
│   └── *.cpp             # 编译通道实现
├── ir/                   # 中间表示与字节码生成
│   ├── ast_to_ir.cpp     # AST → IR 转换
│   ├── bytecode_builder  # 字节码构建器
│   └── *.cpp             # IR 节点定义
├── lexer/                 # 词法分析
│   ├── lexer.cpp/h       # 词法分析器
│   ├── token.cpp/h      # Token 定义
│   └── *.cpp             # 各类 Token 识别
├── parser/                # 语法解析
│   ├── parser.cpp/h     # 解析器主逻辑
│   ├── program.cpp      # Program 节点
│   └── *.cpp             # 各类语法结构解析
├── typescript/           # TypeScript 支持
│   ├── tsprogram.cpp    # TypeScript Program
│   └── *.cpp            # TS 特定语法处理
├── util/                  # 工具类
│   ├── arena.cpp/h      # 内存池管理
│   ├── logger.cpp/h     # 日志工具
│   └── *.cpp            # 其他工具函数
└── test/                 # 单元测试 [测试目录]
```

## ets2panda 模块 (ETS 编译器)

**职责**: 编译 OpenHarmony 应用专用的 ETS (Enhanced TypeScript) 语言

```
ets2panda/
├── aot/                   # AOT 编译器入口
├── bindings/native/       # N-API 绑定 (Native Interop)
│   ├── src/common-interop.cpp    # Promise/Async 处理
│   ├── src/convertors-napi.cpp  # 类型转换
│   └── src/win-dynamic-node.cpp  # 动态模块加载
├── checker/               # 类型检查器
├── varbinder/             # 变量绑定
│   ├── JSBinder.cpp/h    # JS 绑定
│   ├── TSBinder.cpp/h    # TS 绑定
│   └── *.cpp             # 变量/作用域管理
├── parser/                # 语法解析 (ETS 专用)
├── compiler/              # 编译逻辑
├── ir/                   # 中间表示
├── linter/                # 代码检查
├── driver/                # 编译驱动
│   ├── build_system/     # 构建系统集成
│   └── dependency_analyzer/  # 依赖分析
├── evaluate/              # 求值器
├── lexer/                 # 词法分析
└── public/                # 公共 API 头文件
```

## merge_abc 模块

**职责**: 合并多个 ARK 字节码文件为一个

```
merge_abc/
├── protos/               # Protocol Buffer 模板
│   └── *.proto           # ABC 文件结构定义
├── scripts/              # 构建脚本
└── src/                 # 序列化/反序列化实现
    ├── main.cpp         # 入口点
    ├── mergeProgram.cpp/h  # 程序合并逻辑
    ├── protobufSnapshotGenerator.cpp  # Protobuf 生成
    └── assembly*.cpp    # 各类型组装器
```

## arkguard 模块

**职责**: 代码保护与混淆

```
arkguard/
├── *.cpp                # 混淆/保护实现
└── BUILD.gn             # 构建配置
```

## 模块依赖关系

```
                    ┌─────────────────┐
                    │   merge_abc     │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  es2panda     │   │  ets2panda    │   │   arkguard    │
│  (JS/TS)      │   │  (ETS)        │   │  (混淆)       │
└───────┬───────┘   └───────┬───────┘   └───────────────┘
        │                   │
        └─────────┬─────────┘
                  ▼
        ┌─────────────────────┐
        │   runtime_core      │
        │   (共享依赖)        │
        └─────────────────────┘
```

## 排除目录 (测试相关)

以下目录不计入模块职责分析：

- `test/` - SDK/XTS 测试
- `testTs/` - 系统测试
- `test262/` - ECMAScript 标准测试
- `test_ecma_bcopt/` - 字节码优化测试
- `**/test/` - 各模块内测试用例
