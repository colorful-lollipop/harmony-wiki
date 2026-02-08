# Wiki 导航与阅读指南

## 新人阅读顺序

建议按照以下顺序阅读本文档：

### 第一阶段：快速了解（15分钟）
1. [项目概览](00_Overview.md) - 了解项目定位、核心能力和运行环境
2. [目录结构](01_Directory_Structure.md) - 了解代码组织方式

### 第二阶段：深入理解（1小时）
3. [架构说明](02_Architecture.md) - 理解组件关系和数据流
4. [对外 API](03_Public_API.md) - 了解如何使用本模块
5. [编译产物](06_Build_Artifacts.md) - 了解构建输出

### 第三阶段：开发参考（按需查阅）
6. [内部 API](04_Internal_API.md) - 内部模块接口说明
7. [GN 构建目标](05_GN_Targets.md) - 构建系统详解
8. [安全风险分析](07_Security_Analysis.md) - 安全注意事项
9. [问题排查](08_Troubleshooting.md) - 常见问题解决

### 附录（参考查阅）
- [调用链](appendix/Callgraphs.md) - 关键调用流程
- [配置开关](appendix/Config_Flags.md) - Feature 开关说明

## 文档索引

### 按主题分类

**架构设计**
- [00_Overview.md](00_Overview.md) - 项目定位与边界
- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构
- [02_Architecture.md](02_Architecture.md) - 架构说明

**接口文档**
- [03_Public_API.md](03_Public_API.md) - 对外 C API 和 JS API
- [04_Internal_API.md](04_Internal_API.md) - 内部模块接口

**构建与产物**
- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建目标
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物说明

**安全与调试**
- [07_Security_Analysis.md](07_Security_Analysis.md) - 安全风险评审
- [08_Troubleshooting.md](08_Troubleshooting.md) - 问题排查

**参考**
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置开关

## 术语表

| 术语 | 说明 |
|------|------|
| BMS | Bundle Manager Service，包管理服务 |
| HAP | HarmonyOS Ability Package，应用安装包 |
| BundleKit | 对外 API 接口层 |
| SA | System Ability，系统服务 |
| IPC | Inter-Process Communication，进程间通信 |
| UID/GID | User ID / Group ID，用户/组标识 |

## 贡献指南

如发现文档错误或有改进建议，请：
1. 确认问题并收集代码证据
2. 修改对应 markdown 文件
3. 更新相关链接和索引
4. 提交变更说明
