# 文档导航

## 阅读路线

### 新人学习路线

适合初次接触 HiTrace 的开发者，建议按以下顺序阅读：

1. **[项目概览](./index.md)** - 了解项目定位、核心能力（5分钟）
2. **[架构说明](./Architecture.md)** - 理解整体架构和数据流（15分钟）
3. **[N-API 参考](./NAPI_Reference.md)** - 使用 JS/ArkTS 接口（30分钟）
4. **[Native API 参考](./Native_API_Reference.md)** - 使用 C/C++ 接口（可选）
5. **[GN 构建系统](./Build_System.md)** - 了解编译配置（可选）

### 安全研究路线

适合进行安全审计和漏洞研究：

1. **[项目概览](./index.md)** - 快速了解项目范围
2. **[安全风险评审](./Security_Review.md)** - 攻击面分析与风险评估
3. **[架构说明](./Architecture.md)** - 理解数据流和信任边界
4. **[附录：关键调用链](./appendix/Callgraphs.md)** - 深入理解调用路径

### API 速查路线

已有经验，快速查找 API 用法：

- **[JS/ArkTS API](./NAPI_Reference.md)** - hiTraceChain / hiTraceMeter / bytrace
- **[Native C/C++ API](./Native_API_Reference.md)** - HiTraceChain / HiTraceMeter / HiTraceId
- **[配置参数](./appendix/Config_Flags.md)** - Feature Flags / 追踪标志位

## 完整文档列表

### 概览与架构
- [项目概览](./index.md)
- [架构说明](./Architecture.md)

### API 参考
- [N-API 参考 (JS)](./NAPI_Reference.md)
- [Native API 参考 (C/C++)](./Native_API_Reference.md)

### 构建与配置
- [GN 构建系统](./Build_System.md)

### 安全与附录
- [安全风险评审](./Security_Review.md)
- [附录：关键调用链](./appendix/Callgraphs.md)
- [附录：配置参数](./appendix/Config_Flags.md)

## 快速索引

### N-API 模块
| 模块名 | JS 命名空间 | 主要功能 |
|--------|-----------|---------|
| hiTraceChain | `@ohos.hiTraceChain` | 调用链追踪 |
| hiTraceMeter | `@ohos.hiTraceMeter` | 性能追踪 |
| bytrace | `@ohos.bytrace` | 遗留接口 |

### Native 模块
| 模块名 | 头文件 | 主要功能 |
|--------|-------|---------|
| libhitracechain | `hitrace/hitracechain.h` | 调用链核心 |
| hitrace_meter | `hitrace_meter/hitrace_meter.h` | 性能追踪 |
| hitrace_dump | `hitrace_dump.h` | 追踪数据转储 |
| libhitrace_option | `hitrace_option/hitrace_option.h` | 配置选项 |
