# 目录

## 快速入门

- [首页](README.md) - 项目概述与导航
- [概览](00_Overview.md) - 项目定位、核心能力、运行环境

## 架构设计

- [架构说明](01_Architecture.md) - 组件图、数据流、线程模型、关键时序
- [附录: 关键调用链](appendix/Callgraphs.md) - 入口→核心逻辑调用链

## API 参考

- [N-API 接口文档](02_NAPI.md) - JS/ArkTS API 清单、参数、错误码
- [Inner API 接口文档](03_InnerAPI.md) - Native 模块接口、依赖方向

## 工程实践

- [构建配置](04_Build.md) - GN Targets、编译产物、安装路径
- [附录: 配置开关](appendix/Config_Flags.md) - 关键宏/Feature Flags

## 安全与运维

- [安全风险评审](05_Security.md) - 攻击面、信任边界、风险点与修复建议
- [常见问题](06_FAQ.md) - 构建/运行/调试问题定位

---

## 新人阅读顺序推荐

### 开发者 (使用 API)

```
README.md → 00_Overview.md → 02_NAPI.md → 06_FAQ.md
```

### 系统开发者 (扩展框架)

```
README.md → 00_Overview.md → 01_Architecture.md → 03_InnerAPI.md → 05_Security.md
```

### 平台开发者 (构建/适配)

```
README.md → 00_Overview.md → 04_Build.md → 05_Security.md → 06_FAQ.md
```

---

## 快速导航

| 主题 | 文档 | 关键章节 |
|------|------|----------|
| API 使用 | 02_NAPI.md | API 清单表 |
| 架构理解 | 01_Architecture.md | 组件图、数据流 |
| 构建产物 | 04_Build.md | 产物清单表 |
| 安全检查 | 05_Security.md | 攻击面清单 |
| 问题定位 | 06_FAQ.md | 问题分类索引 |
