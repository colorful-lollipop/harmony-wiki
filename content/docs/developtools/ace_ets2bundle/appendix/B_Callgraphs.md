# 附录 B：关键调用链

## 编译调用链

### 完整编译流程

```
npm run build
    │
    └──► main.js (项目初始化)
            │
            ├──► 读取 module.json / manifest.json
            ├──► 初始化 projectConfig
            ├──► 加载系统模块
            └──► 调用编译流程
                    │
                    ├──► pre_process.ts (预处理)
                    │       │
                    │       └──► process_ui_syntax.ts (UI 语法转换)
                    │               │
                    │               └──► arkui-plugins/ (插件转换)
                    │                       │
                    │                       ├──► uiSyntaxLinterTransform()
                    │                       ├──► uiTransform()
                    │                       └──► interopTransform()
                    │
                    ├──► ets_checker.ts (类型检查)
                    │       │
                    │       ├──► createLanguageService()
                    │       ├──► runArkTSLinter()
                    │       └──► serviceChecker()
                    │
                    └──► gen_abc_plugin.ts (字节码生成)
                            │
                            ├──► rollup-plugin-gen-abc.ts
                            │       │
                            │       └──► es2abc (子进程)
                            │               │
                            │               └──► ABC 字节码 (.abc)
                            │
                            └──► gen_aot.ts (AOT 编译，可选)
                                    │
                                    └──► ark_aot_compiler
```

## UI 语法转换调用链

```
process_ui_syntax.ts
    │
    ├──► processDecorator (装饰器处理)
    │       │
    │       ├──► @Component → 组件注册
    │       ├──► @Entry → 页面入口标记
    │       ├──► @State/@Prop/@Link → 状态管理
    │       └──► @Preview → 预览支持
    │
    ├──► processBuild (build 方法转换)
    │       │
    │       ├──► Text/Button/Column 等组件
    │       ├──► ForEach/LazyForEach 循环
    │       ├──► If/Branch 条件
    │       └──► 自定义组件
    │
    └──► processComponent (组件实例化)
            │
            ├──► 构造函数生成
            ├──► 属性传递
            └──► 事件绑定
```

## Koala Native 调用链

```
arkui-plugins/ (TypeScript)
    │
    ├──► require('./es2panda.node')
    │       │
    │       └──► N-API 胶水层 (convertors-napi.cc)
    │               │
    │               ├──► KOALA_INTEROP_宏展开
    │               ├──► 类型转换 (InteropTypeConverter)
    │               └──► 函数调用转发
    │
    └──► koala-wrapper/native/src/*.cc
            │
            ├──► common.cc (上下文管理)
            ├──► bridges.cc (手动桥接)
            ├──► generated/bridges.cc (自动生成)
            │
            └──► libes2panda_public.so (es2panda 核心)
                    │
                    ├──► AST 解析
                    ├──► 语义分析
                    ├──► 字节码生成
                    └──► 优化器
```

## Fast Build 调用链

```
Rollup 编译
    │
    ├──► rollup-plugin-ets-typescript.ts
    │       │
    │       └──► TypeScript 编译
    │               │
    │               └──► AST 生成
    │
    ├──► rollup-plugin-ets-checker.ts
    │       │
    │       ├──► ArkTS 类型检查
    │       └──► 错误报告
    │
    ├──► rollup-plugin-system-api.ts
    │       │
    │       └──► API 可用性检查
    │               │
    │               └──► checkPermissionValue()
    │
    └──► rollup-plugin-gen-abc.ts
            │
            ├──► 模块转换
            │       │
            │       └──► process_module_files.ts
            │
            └──► ABC 生成
                    │
                    ├──► es2abc 调用
                    │       │
                    │       └──► .abc 输出
                    │
                    └──► 产物打包
```

## 模块间依赖关系

```
                    ┌─────────────────────────────────────┐
                    │           compiler/main.js           │
                    └─────────────────┬───────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌───────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│  pre_process.ts   │   │    ets_checker.ts     │   │   gen_abc_plugin.ts   │
│  (预处理)         │   │    (类型检查)          │   │   (字节码生成)        │
└─────────┬─────────┘   └───────────┬───────────┘   └───────────┬───────────┘
          │                         │                           │
          │         ┌───────────────┼───────────────┐           │
          │         │               │               │           │
          ▼         ▼               ▼               ▼           ▼
┌───────────────────────────┐ ┌───────────────────────────────┐
│ arkui-plugins/            │ │ fast_build/                   │
│ - ui-syntax-plugins/      │ │ - ark_compiler/              │
│ - ui-plugins/             │ │ - ets_ui/                    │
│ - interop-plugins/        │ │ - system_api/                │
└───────────────────────────┘ └───────────────────────────────┘
          │                         │                           │
          └─────────────────────────┼───────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────────┐
                    │        koala-wrapper/               │
                    │        - native/src/*.cc            │
                    │        - koalaui/interop/           │
                    └─────────────────┬───────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────┐
                    │    libes2panda_public.so/dll        │
                    │    (es2panda 编译器核心)              │
                    └─────────────────────────────────────┘
```

## 相关文档

- [架构说明](02_Architecture.md)
- [编译器核心](04_Compiler_Core.md)
- [ArkUI 插件系统](05_ArkUI_Plugins.md)
- [Koala 包装器](06_Koala_Wrapper.md)
