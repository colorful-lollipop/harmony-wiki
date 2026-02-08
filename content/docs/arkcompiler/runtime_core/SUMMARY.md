# 全站导航

> 本文档提供 ArkCompiler Runtime Core Wiki 的完整导航与新人阅读指南。

## 快速导航

| 文档 | 目的 | 阅读时长 |
|------|------|----------|
| [README](README.md) | 了解 Wiki 结构与使用方式 | 5 min |
| [项目概览](00_Overview.md) | 了解 Runtime Core 是什么 | 10 min |
| [项目定位与边界](01_Project_Boundary.md) | 明确项目范围与核心能力 | 15 min |
| [目录结构](02_Directory_Structure.md) | 熟悉代码组织 | 20 min |
| [架构说明](03_Architecture.md) | 理解组件关系与数据流 | 30 min |
| [对外 API](04_Public_API.md) | 开发 Native 扩展必读 | 40 min |
| [内部 API](05_Inner_API.md) | 运行时开发必读 | 40 min |
| [GN Targets](06_GN_Targets.md) | 构建系统详解 | 30 min |
| [编译产物](07_Build_Artifacts.md) | 产物清单与路径 | 20 min |
| [安全风险](08_Security.md) | 安全审计必读 | 30 min |
| [问题排查](09_Troubleshooting.md) | 调试与故障定位 | 20 min |

## 按角色阅读

### 我是 Native 开发者（使用 ANI/N-API）

阅读顺序：
1. [项目概览](00_Overview.md) - 了解整体
2. [对外 API](04_Public_API.md) - 掌握 ANI 接口
3. [编译产物](07_Build_Artifacts.md) - 了解库文件
4. [安全风险](08_Security.md) - 了解安全约束

重点关注：
- ANI 函数列表与使用方式
- 参数校验与错误码处理
- 内存管理规则

### 我是运行时开发者（修改 Runtime Core）

阅读顺序：
1. [项目概览](00_Overview.md)
2. [项目定位与边界](01_Project_Boundary.md) - 明确修改边界
3. [目录结构](02_Directory_Structure.md) - 找到代码位置
4. [架构说明](03_Architecture.md) - 理解设计
5. [内部 API](05_Inner_API.md) - 掌握模块接口
6. [GN Targets](06_GN_Targets.md) - 了解构建

重点关注：
- 模块职责与依赖方向
- 线程模型与同步机制
- GC 与内存管理接口

### 我是构建/集成工程师

阅读顺序：
1. [项目概览](00_Overview.md)
2. [GN Targets](06_GN_Targets.md) - 构建系统详解
3. [编译产物](07_Build_Artifacts.md) - 产物清单
4. [目录结构](02_Directory_Structure.md) - 代码组织

重点关注：
- GN target 类型与依赖
- 产物输出路径
- 安装配置

### 我是安全审计人员

阅读顺序：
1. [项目概览](00_Overview.md)
2. [架构说明](03_Architecture.md) - 理解攻击面
3. [安全风险](08_Security.md) - 详细风险分析
4. [对外 API](04_Public_API.md) - 外部接口风险

重点关注：
- 攻击面清单
- 可被利用点详情
- 修复建议

## 附录导航

- [关键调用链](appendix/Callgraphs.md) - 入口到核心逻辑的调用路径
- [配置与宏](appendix/Config_Flags.md) - 关键编译配置与特性开关

## 术语速查

| 术语 | 说明 | 相关文档 |
|------|------|----------|
| ANI | Ark Native Interface，Ark 原生接口 | [对外 API](04_Public_API.md) |
| N-API | Node-API，JS 与原生代码互操作接口 | [对外 API](04_Public_API.md) |
| ABC | Ark Bytecode，Ark 字节码格式 | [目录结构](02_Directory_Structure.md) |
| ETS | Extended TypeScript，ArkTS 语言 | [项目概览](00_Overview.md) |
| GC | Garbage Collection，垃圾回收 | [架构说明](03_Architecture.md) |
| AOT | Ahead-of-Time，提前编译 | [GN Targets](06_GN_Targets.md) |
| JIT | Just-in-Time，即时编译 | [架构说明](03_Architecture.md) |
| IR | Intermediate Representation，中间表示 | [内部 API](05_Inner_API.md) |
| ISA | Instruction Set Architecture，指令集架构 | [项目概览](00_Overview.md) |

## 外部链接

- [OpenHarmony 文档中心](https://gitee.com/openharmony/docs)
- [ArkCompiler 设计文档](../docs/)
- [Issue 反馈](https://gitee.com/openharmony/arkcompiler_runtime_core/issues)
