# Wiki 目录导航

本文档提供了 OpenHarmony 分布式数据管理仓颉封装（distributeddatamgr_cangjie_wrapper）的完整导航。

## 新人阅读顺序

建议按照以下顺序阅读文档，以快速理解项目：

### 第一阶段：快速了解（30 分钟）
1. **[README.md](README.md)** - 了解文档范围和约定
2. **[00_Overview.md](00_Overview.md)** - 了解项目概况和核心概念

### 第二阶段：深入理解（1-2 小时）
3. **[01_Project_Boundaries.md](01_Project_Boundaries.md)** - 理解项目定位、边界和核心能力
4. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 了解代码组织和模块职责
5. **[03_Architecture.md](03_Architecture.md)** - 理解架构设计、组件关系和数据流

### 第三阶段：API 参考（根据需要查阅）
6. **[04_Public_API.md](04_Public_API.md)** - 查看对外仓颉 API 清单
7. **[05_Internal_API.md](05_Internal_API.md)** - 了解 FFI 层实现细节

### 第四阶段：构建与部署（1 小时）
8. **[06_GN_Targets.md](06_GN_Targets.md)** - 了解构建目标和依赖关系
9. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 了解编译产物和安装方式

### 第五阶段：安全与问题（30 分钟）
10. **[08_Security_Review.md](08_Security_Review.md)** - 了解安全风险和最佳实践
11. **[09_QA.md](09_QA.md)** - 查看常见问题和解决方案

### 附录（参考查阅）
12. **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 查看关键调用链
13. **[appendix/Config_Flags.md](appendix/Config_Flags.md)** - 查看配置标志

## 按角色查看文档

### 开发者（开发仓颉应用）
- 00_Overview.md - 快速入门
- 01_Project_Boundaries.md - 了解支持的功能
- 04_Public_API.md - API 参考
- 09_QA.md - 开发常见问题

### 维护者（参与代码贡献）
- 02_Directory_Structure.md - 代码组织
- 03_Architecture.md - 架构设计
- 05_Internal_API.md - FFI 层实现
- 06_GN_Targets.md - 构建系统
- 08_Security_Review.md - 安全风险

### 构建工程师
- 06_GN_Targets.md - 构建目标
- 07_Build_Artifacts.md - 编译产物

### 安全审计人员
- 08_Security_Review.md - 安全风险分析

## 文档关系图

```
README.md (元信息)
    │
    ├── SUMMARY.md (本文件 - 导航)
    │
    ├── 00_Overview.md ──┐
    ├── 01_Project_Boundaries.md ──┤
    ├── 02_Directory_Structure.md ──┤──→ 理解项目
    ├── 03_Architecture.md ──┘
    │
    ├── 04_Public_API.md ──┐
    ├── 05_Internal_API.md ──┤──→ 使用/维护 API
    │
    ├── 06_GN_Targets.md ──┐
    ├── 07_Build_Artifacts.md ──┘──→ 构建/部署
    │
    ├── 08_Security_Review.md ───→ 安全分析
    │
    ├── 09_QA.md ───────────────→ 问题排查
    │
    └── appendix/
            ├── Callgraphs.md ──→ 调用分析
            └── Config_Flags.md ──→ 配置参考
```

## 核心概念速查表

| 概念 | 说明 | 参考文档 |
|--------|------|----------|
| Cangjie FFI | 仓颉语言的 Foreign Function Interface，用于调用 C/C++ 代码 | 03_Architecture.md |
| KV Store | 分布式键值数据库 | 01_Project_Boundaries.md |
| RDB Store | 关系型数据库 | 01_Project_Boundaries.md |
| Preferences | 用户首选项（轻量级 KV 存储） | 01_Project_Boundaries.md |
| DataShare Predicates | 数据共享查询谓词 | 01_Project_Boundaries.md |
| Security Level | 数据安全级别（S1-S4） | 01_Project_Boundaries.md |
| RemoteDataLite | 所有数据管理类的基类 | 03_Architecture.md |

## 文档状态

| 文档 | 状态 | 最后更新 |
|--------|--------|----------|
| README.md | ✅ 已完成 | 2026-02-06 |
| SUMMARY.md | ✅ 已完成 | 2026-02-06 |
| 00_Overview.md | ⏳ 待完成 | - |
| 01_Project_Boundaries.md | ⏳ 待完成 | - |
| 02_Directory_Structure.md | ⏳ 待完成 | - |
| 03_Architecture.md | ⏳ 待完成 | - |
| 04_Public_API.md | ⏳ 待完成 | - |
| 05_Internal_API.md | ⏳ 待完成 | - |
| 06_GN_Targets.md | ⏳ 待完成 | - |
| 07_Build_Artifacts.md | ⏳ 待完成 | - |
| 08_Security_Review.md | ⏳ 待完成 | - |
| 09_QA.md | ⏳ 待完成 | - |
| appendix/Callgraphs.md | ⏳ 待完成 | - |
| appendix/Config_Flags.md | ⏳ 待完成 | - |

## 反馈与贡献

如发现文档错误或需要补充的内容，请通过以下方式反馈：

1. **代码贡献**: 参照代码结构更新相应文档
2. **Issue 反馈**: 在项目仓库提交 Issue 说明需要改进的内容

## 相关资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [仓颉语言官方仓库](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [分布式数据管理组件](https://gitcode.com/openharmony/distributeddatamgr_data_share)
