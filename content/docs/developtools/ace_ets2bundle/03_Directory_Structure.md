# 目录结构

## 顶层结构

```
ace_ets2bundle/
├── compiler/                    # ETS 编译器核心
│   ├── src/                    # TypeScript 源码
│   ├── components/             # 组件定义
│   ├── form_components/        # 卡片组件定义
│   ├── server/                 # 编译器服务
│   ├── codegen/                # 代码生成
│   ├── insight_intents/        # 意图识别
│   ├── config/                 # 配置
│   ├── sample/                 # 示例项目
│   ├── main.js                 # 主入口 (非 Interop)
│   ├── interop/                # Interop 版本
│   │   └── main.js             # Interop 主入口
│   └── package.json            # 依赖配置
├── arkui-plugins/              # ArkUI 插件系统
│   ├── common/                 # 公共基础设施
│   ├── ui-plugins/             # UI 转换插件
│   ├── ui-syntax-plugins/      # 语法检查插件
│   ├── interop-plugins/        # 互操作插件
│   ├── memo-plugins/           # 优化插件
│   ├── collectors/             # 数据收集器
│   └── BUILD.gn                # 构建配置
├── koala-wrapper/              # Koala 运行时包装器
│   ├── native/                 # Native C++ 实现
│   │   ├── include/            # 头文件
│   │   │   ├── common.h
│   │   │   └── memoryTracker.h
│   │   ├── src/                # C++ 源文件
│   │   │   ├── common.cc
│   │   │   ├── bridges.cc
│   │   │   ├── memoryTracker.cc
│   │   │   └── generated/      # 自动生成代码
│   │   │       └── bridges.cc
│   │   ├── BUILD.gn            # GN 构建配置
│   │   ├── meson.build         # Meson 构建配置
│   │   └── meson_options.txt   # Meson 选项
│   ├── koalaui/                # Koala UI 互操作
│   │   └── interop/            # 互操作层
│   │       └── src/cpp/        # C++ 互操作实现
│   │           ├── napi/       # N-API 绑定
│   │           ├── ani/        # ANI 互操作
│   │           ├── ets/        # ETS 互操作
│   │           └── wasm/       # WASM 互操作
│   └── tools/                  # 工具脚本
├── wiki/                       # 文档目录
│   ├── _work/                  # 工作区
│   ├── README.md               # 文档说明
│   ├── SUMMARY.md              # 导航索引
│   ├── 01_Overview.md          # 项目概览
│   ├── 02_Architecture.md      # 架构说明
│   ├── 03_Directory_Structure.md # 本文档
│   └── ...                     # 其他文档
├── BUILD.gn                    # 根构建配置
├── bundle.json                # 组件配置
├── LICENSE                    # 许可证
└── README.md                  # 项目说明
```

## compiler/src/ 详解

### 核心编译模块

```
src/
├── main.js                              # 主入口（非 Interop）
├── create.ts                            # 项目创建工具
├── ark_utils.ts                         # 通用工具函数
├── compile_info.ts                      # 编译状态管理
├── component_map.ts                     # 组件映射
├── constant_define.ts                   # 常量定义
├── create_ast_node_utils.ts             # AST 节点创建工具
├── do_arkTS_linter.ts                   # ArkTS Lint
├── ets_checker.ts                       # 类型检查服务
├── external_component_map.ts            # 外部组件映射
├── gen_abc.ts                           # ABC 生成
├── gen_abc_plugin.ts                    # ABC 插件
├── gen_aot.ts                           # AOT 编译
├── gen_merged_abc.ts                    # 合并 ABC 生成
├── gen_module_abc.ts                    # 模块 ABC 生成
├── import_path_expand.ts                # 导入路径扩展
├── log_message_collection.ts            # 日志收集
├── manage_workers.ts                    # Worker 管理
├── performance.ts                       # 性能监控
├── pre_define.ts                        # 预定义常量
├── pre_process.ts                       # 预处理入口
├── process_*.ts                         # 各类处理模块
│   ├── process_component_build.ts       # 组件 build 方法
│   ├── process_component_class.ts       # 组件类
│   ├── process_component_constructor.ts  # 组件构造
│   ├── process_component_member.ts      # 组件成员
│   ├── process_custom_component.ts      # 自定义组件
│   ├── process_import.ts                # 导入处理
│   ├── process_kit_import.ts            # Kit 导入
│   ├── process_lazy_import.ts           # 懒加载导入
│   ├── process_module_files.ts          # 模块文件
│   ├── process_module_package.ts        # 模块包
│   ├── process_sendable.ts              # Sendable 处理
│   ├── process_source_file.ts           # 源文件
│   ├── process_struct_componentV2.ts    # Struct V2
│   ├── process_system_module.ts        # 系统模块
│   ├── process_ui_syntax.ts             # UI 语法
│   ├── process_visual.ts               # 可视化 DSL
│   └── process_dts_file.ts             # 声明文件
├── resolve_ohm_url.ts                   # OhmURL 解析
├── validate_ui_syntax.ts                 # UI 语法验证
├── hvigor_error_code/                   # 错误码定义
│   ├── const/
│   │   ├── error_code_module.ts
│   │   └── error_code.ts
│   └── utils.ts
└── fast_build/                          # Fast Build 体系
    ├── ark_compiler/                    # Ark 编译器插件
    │   ├── common/
    │   │   ├── common_mode.ts
    │   │   ├── gen_abc.ts
    │   │   ├── ark_define.ts
    │   │   ├── ob_config_resolver.ts
    │   │   └── process_ark_config.ts
    │   ├── module/
    │   │   └── module_mode.ts
    │   ├── bundle/
    │   │   ├── bundle_mode.ts
    │   │   └── bundle_preview_mode.ts
    │   ├── rollup-plugin-gen-abc.ts     # ABC 生成插件
    │   ├── bytecode_obfuscator.ts      # 字节码混淆
    │   ├── cache/                       # 缓存
    │   └── error_code.ts               # 错误码
    ├── ets_ui/                          # ETS UI 插件
    │   ├── rollup-plugin-ets-checker.ts # 类型检查插件
    │   └── rollup-plugin-ets-typescript.ts # TS 转换插件
    ├── system_api/                      # 系统 API
    │   ├── api_check_define.ts         # API 检查定义
    │   ├── api_check_utils.ts          # API 检查工具
    │   └── rollup-plugin-system-api.ts # API 检查插件
    ├── memory_monitor/                  # 内存监控
    │   ├── memory_define.ts
    │   └── rollup-plugin-memory-monitor.ts
    └── meomry_monitor/                  # 内存监控（拼写变体）
```

## arkui-plugins/ 详解

```
arkui-plugins/
├── common/                      # 公共基础设施
│   ├── abstract-visitor.ts     # AST 访问者基类
│   ├── program-visitor.ts      # 程序遍历器
│   ├── plugin-context.ts       # 插件上下文
│   ├── predefines.ts           # 常量定义
│   ├── utils.ts                # 工具函数
│   ├── cache/                  # 缓存
│   └── ...                     # 其他工具
├── ui-plugins/                 # UI 插件核心
│   ├── index.ts                # uiTransform() 入口
│   ├── component-transformer.ts # 组件转换器
│   ├── checked-transformer.ts  # 检查阶段转换器
│   ├── entry-translators/      # 入口翻译器
│   ├── property-translators/   # 属性翻译器
│   │   ├── index.ts
│   │   ├── base.ts
│   │   ├── builderParam.ts
│   │   ├── computed.ts
│   │   ├── consume.ts
│   │   ├── consumer.ts
│   │   ├── event.ts
│   │   ├── factory.ts
│   │   ├── index.ts
│   │   ├── link.ts
│   │   ├── local.ts
│   │   ├── localStoragePropRef.ts
│   │   ├── ...
│   │   └── ...
│   ├── struct-translators/     # Struct 翻译器
│   ├── builder-lambda-translators/ # Builder Lambda
│   ├── type-translators/       # 类型翻译器
│   └── interop/               # 互操作集成
├── ui-syntax-plugins/          # 语法检查插件
│   ├── index.ts               # uiSyntaxLinterTransform()
│   ├── ui-factory.ts          # UI 工厂
│   ├── processor/             # 规则处理器
│   ├── rules/                 # 60+ 语法规则
│   ├── transformers/           # 语法转换器
│   ├── utils.ts               # 工具函数
│   └── checked-transformer.ts
├── interop-plugins/            # 互操作插件
│   ├── index.ts               # interopTransform()
│   ├── decl_transformer.ts    # 声明转换
│   ├── emit_transformer.ts    # 发射转换
│   ├── function-transformer.ts # 函数转换
│   ├── signature-transformer.ts # 签名转换
│   └── types.ts               # 类型定义
├── memo-plugins/              # Memo 优化插件
│   └── index.ts               # unmemoizeTransform()
├── collectors/                # 数据收集器
└── BUILD.gn
```

## koala-wrapper/native/src/ 详解

```
src/
├── common.cc                 # 核心功能（上下文管理、库加载）
├── bridges.cc                # 手动桥接函数
├── memoryTracker.cc          # 内存追踪器
└── generated/
    └── bridges.cc           # 自动生成的 AST 桥接函数
```

## 排除的目录

以下目录不计入架构分析：

| 目录模式 | 说明 |
|---------|------|
| `test/` | 测试代码 |
| `tests/` | 测试套件 |
| `node_modules/` | npm 依赖 |
| `out/` | 构建产物 |
| `build/` | 构建产物 |
| `.git/` | Git 元数据 |

## 相关文档

- [架构说明](02_Architecture.md)
- [编译器核心](04_Compiler_Core.md)
- [ArkUI 插件系统](05_ArkUI_Plugins.md)
