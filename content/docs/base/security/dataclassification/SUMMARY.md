# 文档导航

## 快速入门

- [项目概览](index.md) - 快速了解 dataclassification 模块

## 架构与设计

- [架构说明](Architecture.md) - 组件图、数据流、线程模型
- [关键调用链](appendix/Callgraphs.md) - 入口到核心逻辑

## API 参考

- [内部 API](Inner_API.md) - C 接口定义、使用方式、错误码

## 构建与配置

- [构建指南](Build.md) - GN Targets、编译产物、配置开关
- [配置开关](appendix/Config_Flags.md) - feature flags 详解

## 安全

- [安全风险评审](Security.md) - 攻击面、风险分析、修复建议

## 文档说明

- [README](README.md) - 文档覆盖范围、更新方式

---

## 阅读路径推荐

### 新人入门
```
index.md → Architecture.md → Inner_API.md
```

### 开发者查阅
```
Inner_API.md → Build.md → Security.md
```

### 安全审计
```
Security.md → Architecture.md → Inner_API.md
```
