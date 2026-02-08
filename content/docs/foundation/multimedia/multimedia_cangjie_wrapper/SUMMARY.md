# 全站导航

> multimedia_cangjie_wrapper Wiki 导航中心

---

## 核心文档

### 1. 入门必读
- [README](README.md) - Wiki 首页与使用说明
- [项目概览](00_Overview.md) - 项目定位、核心能力、运行环境

### 2. 架构理解
- [架构说明](01_Architecture.md) - 组件图、数据流、线程模型
- [目录结构](02_Directory_Structure.md) - 代码组织与模块职责

### 3. API 参考
- [N-API 接口](03_NAPI_Interfaces.md) - 对外 JS/仓颉 API 详述
  - CameraKit API
  - ImageKit API
  - MediaKit API
  - MediaLibraryKit API
- [内部 API](04_Inner_API.md) - 模块内部接口与依赖

### 4. 构建与产物
- [GN Targets](05_GN_Targets.md) - 构建目标与依赖关系
- [编译产物](06_Build_Artifacts.md) - 输出文件与安装路径

### 5. 运维与调试
- [安全风险分析](07_Security_Analysis.md) - 攻击面与风险点
- [FAQ 与调试](08_FAQ_Debugging.md) - 常见问题与排查

### 6. 附录
- [调用链附录](appendix/Callgraphs.md) - 关键调用链
- [配置标志附录](appendix/Config_Flags.md) - 宏与 Feature Flags

---

## 新人阅读路线

### 路线一：快速了解（15分钟）
1. [README](README.md) - 了解项目定位
2. [项目概览](00_Overview.md) - 核心能力与限制
3. [架构说明](01_Architecture.md) - 组件关系

### 路线二：接口开发（30分钟）
1. [目录结构](02_Directory_Structure.md) - 代码组织
2. [N-API 接口](03_NAPI_Interfaces.md) - 重点阅读对应 Kit 部分
3. [FAQ 与调试](08_FAQ_Debugging.md) - 常见问题

### 路线三：深度贡献（1小时）
1. [项目概览](00_Overview.md)
2. [架构说明](01_Architecture.md)
3. [内部 API](04_Inner_API.md)
4. [GN Targets](05_GN_Targets.md)
5. [安全风险分析](07_Security_Analysis.md)

---

## 按角色导航

### 应用开发者（使用 Kit 接口）
→ 重点阅读：
- [项目概览](00_Overview.md) - 约束与限制
- [N-API 接口](03_NAPI_Interfaces.md) - API 使用指南
- [FAQ 与调试](08_FAQ_Debugging.md) - 问题排查

### 框架开发者（维护 Wrapper）
→ 重点阅读：
- [架构说明](01_Architecture.md)
- [内部 API](04_Inner_API.md)
- [GN Targets](05_GN_Targets.md)
- [安全风险分析](07_Security_Analysis.md)

### 安全审计
→ 重点阅读：
- [安全风险分析](07_Security_Analysis.md)
- [N-API 接口](03_NAPI_Interfaces.md) - 权限部分
- [架构说明](01_Architecture.md) - 信任边界
