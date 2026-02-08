# ets_frontend 概览

## 项目定位

`ets_frontend` 是 OpenHarmony ARK Runtime 子系统的前端编译工具，负责将应用代码（ETS/JS/TS）转换为 ARK 字节码文件。

### 核心能力

1. **多语言支持**: 支持 ECMAScript 2015+ 标准，包括 JavaScript、TypeScript 和 ETS (Enhanced TypeScript)
2. **字节码生成**: 产出可在 ARK Runtime 上运行的 .abc 字节码文件
3. **AOT 编译**: 支持提前编译 (Ahead-of-Time) 生成优化字节码
4. **代码保护**: 提供代码混淆和保护能力 (arkguard 模块)

### 架构位置

```
应用代码 (ETS/JS/TS)
        │
        ▼
┌───────────────────────┐
│   ets_frontend        │  ← 本项目
│   (前端编译器)         │
└───────────────────────┘
        │
        ▼
   ARK 字节码 (.abc)
        │
        ▼
┌───────────────────────┐
│  arkcompiler_ets_runtime │  ← 运行时执行
└───────────────────────┘
```

## 关键特性

| 特性 | 描述 | 相关模块 |
|------|------|----------|
| 词法分析 | 源代码 → Token 序列 | es2panda/lexer |
| 语法解析 | Token → AST | es2panda/parser |
| 语义分析 | 符号绑定、类型检查 | es2panda/binder, ets2panda/varbinder |
| 字节码生成 | AST → ARK 字节码 | es2panda/ir |
| 模块合并 | 多文件合并为单一 ABC | merge_abc |
| Native 互操作 | JS ↔ Native 桥接 | ets2panda/bindings/native |

## 运行环境

- **操作系统**: Linux, Windows, macOS
- **构建工具**: GN + Ninja
- **依赖**: runtime_core, typescript, protobuf, icu, abseil-cpp, hilog

## 版本信息

- **当前版本**: 3.1 (from bundle.json)
- **许可证**: Apache License 2.0

## 相关文档

- [目录结构](01_Directory_Structure.md) - 详细模块划分
- [命令行接口](02_CLI_Reference.md) - 编译工具使用
- [架构详解](07_Architecture.md) - 编译流程详解
