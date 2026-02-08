# 文档导航

## 首页与概览
- [首页](README.md) - 覆盖范围、更新方式
- [项目概述](01_Overview.md) - 定位、核心能力、运行环境

## 架构与 API
- [架构说明](02_Architecture.md) - 组件图、数据流、线程模型
- [内部 API](03_InnerAPIs.md) - C++ 接口、模块接口

## 构建与安全
- [构建指南](04_Build.md) - GN Targets、编译产物
- [安全评审](05_Security.md) - 攻击面、风险分析
- [问题排查](06_Troubleshooting.md) - 构建/调试问题

---

## 新人阅读路线

```
1. 了解项目
   └─ 01_Overview.md
   
2. 理解架构
   └─ 02_Architecture.md
   
3. 学习 API
   └─ 03_InnerAPIs.md
   
4. 根据需要查阅
   ├─ 04_Build.md (构建相关)
   ├─ 05_Security.md (安全相关)
   └─ 06_Troubleshooting.md (问题排查)
```

---

## 快速跳转

| 主题 | 文档 |
|-----|------|
| 核心类 | [GERender](03_InnerAPIs.md#gerender) |
| 效果容器 | [GEVisualEffectContainer](03_InnerAPIs.md#gevisualeffectcontainer) |
| 效果类型 | [效果列表](01_Overview.md#效果类型) |
| 构建命令 | [构建命令](04_Build.md#构建命令) |
| 安全机制 | [安全评审](05_Security.md) |
