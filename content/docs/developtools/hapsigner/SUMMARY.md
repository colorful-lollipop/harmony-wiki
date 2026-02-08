# SUMMARY

## 目录

### 入门指南

- [简介](README.md)
- [项目概览](00_Overview.md)
- [目录结构](01_Directory_Structure.md)

### 架构与设计

- [架构说明](02_Architecture.md)
- [API 参考](03_API_Reference.md)
- [构建系统](04_Build_System.md)

### 安全与运维

- [安全风险分析](05_Security_Analysis.md)
- [常见问题](06_Troubleshooting.md)

### 附录

- [调用链](appendix/Callgraphs.md)

---

## 新人阅读顺序

建议按以下顺序阅读本文档：

1. **[README.md](README.md)** - 了解 Wiki 覆盖范围和更新方式
2. **[00_Overview.md](00_Overview.md)** - 理解项目定位、核心能力和运行环境
3. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 熟悉代码组织结构
4. **[02_Architecture.md](02_Architecture.md)** - 理解系统架构和组件关系
5. **[04_Build_System.md](04_Build_System.md)** - 了解如何构建项目
6. **[03_API_Reference.md](03_API_Reference.md)** - 掌握 API 使用方法
7. **[05_Security_Analysis.md](05_Security_Analysis.md)** - 了解安全风险点
8. **[06_Troubleshooting.md](06_Troubleshooting.md)** - 掌握问题定位方法

---

## 快速链接

### 核心概念

- [项目定位](00_Overview.md#项目定位)
- [核心能力](00_Overview.md#核心能力)
- [运行环境](00_Overview.md#运行环境)

### 关键模块

- [Java 签名工具](01_Directory_Structure.md#hapsigntool)
- [C++ 签名工具](01_Directory_Structure.md#hapsigntool_cpp)
- [二进制签名工具](01_Directory_Structure.md#binary_sign_tool)

### 重要接口

- [命令行接口](03_API_Reference.md#命令行接口)
- [C++ ServiceApi](03_API_Reference.md#c-service-api)
- [Java ServiceApi](03_API_Reference.md#java-service-api)

### 构建相关

- [GN 构建目标](04_Build_System.md#gn-构建目标)
- [Maven 构建](04_Build_System.md#maven-构建)
- [编译产物](04_Build_System.md#编译产物)
