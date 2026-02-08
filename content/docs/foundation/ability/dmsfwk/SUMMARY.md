# DMSFWK Wiki 导航

## 新人阅读路线

建议阅读顺序：

1. **[00_Overview.md](00_Overview.md)** - 项目概述、核心能力、运行环境
2. **[01_Architecture.md](01_Architecture.md)** - 系统架构、模块职责、数据流
3. **[02_NAPI.md](02_NAPI.md)** - JS API 接口清单与使用
4. **[03_InnerAPI.md](03_InnerAPI.md)** - C++ 内部模块接口
5. **[04_Build.md](04_Build.md)** - GN 构建配置与编译产物
6. **[05_Security.md](05_Security.md)** - 安全风险与修复建议

## 文档索引

### 概览
- [00_Overview.md](00_Overview.md) - 项目定位、核心功能、关键概念

### 架构
- [01_Architecture.md](01_Architecture.md) - 组件图、数据流、线程模型

### 接口
- [02_NAPI.md](02_NAPI.md) - N-API 模块、JS 方法、错误码
- [03_InnerAPI.md](03_InnerAPI.md) - Inner API 模块、依赖关系

### 构建
- [04_Build.md](04_Build.md) - GN targets、编译产物、配置开关

### 安全
- [05_Security.md](05_Security.md) - 威胁模型、攻击面、修复建议

### 附录
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链

## 快速索引

### 按模块索引

| 模块 | 说明 | 文档位置 |
|------|------|----------|
| dtbschedmgr | 分布式调度管理器 | 01_Architecture.md |
| dtbabilitymgr | 分布式能力管理器 | 01_Architecture.md |
| dtbcollabmgr | 分布式协作管理器 | 01_Architecture.md |
| continuation | 接续管理 | 02_NAPI.md, 03_InnerAPI.md |
| ability_connection | 能力连接 | 02_NAPI.md, 03_InnerAPI.md |

### 按功能索引

| 功能 | 说明 | 文档位置 |
|------|------|----------|
| 远程启动 | Remote Ability Startup | 00_Overview.md, 01_Architecture.md |
| 远程迁移 | Remote Ability Continuation | 02_NAPI.md |
| 远程绑定 | Remote Ability Binding | 02_NAPI.md |
| 远程调用 | Remote Call | 01_Architecture.md |
| 音视频传输 | AV Stream Transport | 01_Architecture.md, 02_NAPI.md |
