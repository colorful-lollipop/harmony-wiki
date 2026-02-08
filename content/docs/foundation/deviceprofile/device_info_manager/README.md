# DeviceProfile Wiki 文档中心

**项目**: OpenHarmony DeviceProfile (deviceprofile_device_info_manager)  
**定位**: 分布式设备信息管理部件  
**版本**: v2.0  
**更新**: 2026-02-07  

---

## 文档简介

本文档是 DeviceProfile 项目的工程 Wiki，面向两类受众：

| 受众 | 目标 | 推荐路线 |
|------|------|----------|
| **新人学习者** | 快速理解项目架构、掌握 API 使用 | [新人学习路线](#新人学习路线) |
| **安全研究员** | 识别攻击面、分析安全风险 | [安全研究路线](#安全研究路线) |

---

## 新人学习路线

建议按以下顺序阅读：

1. **[01_Overview.md](01_Overview.md)** - 理解项目定位、核心能力和运行环境
2. **[02_Architecture.md](02_Architecture.md)** - 了解架构设计和数据流
3. **[03_CodeMap.md](03_CodeMap.md)** - 快速定位代码位置
4. **[04_Interface.md](04_Interface.md)** - 学习 IPC 接口使用
5. **[07_Build.md](07_Build.md)** - 了解构建配置和产物
6. **[08_Internals.md](08_Internals.md)** - 深入内部实现（可选）

**预计时间**: 30-45 分钟  
**学习目标**: 能够独立阅读代码、理解接口调用流程

---

## 安全研究路线

建议按以下顺序阅读：

1. **[05_AttackSurface.md](05_AttackSurface.md)** - 快速识别攻击面和外部输入
2. **[06_SecurityReview.md](06_SecurityReview.md)** - 详细安全风险分析
3. **[02_Architecture.md](02_Architecture.md)** - 了解信任边界和数据流
4. **[04_Interface.md](04_Interface.md)** - 分析 IPC 接口参数
5. **[08_Internals.md](08_Internals.md)** - 检查内部实现细节

**预计时间**: 20-30 分钟  
**学习目标**: 能够定位漏洞点、评估可利用性

---

## 文档索引

| 文档 | 说明 | 主要受众 |
|------|------|----------|
| [01_Overview.md](01_Overview.md) | 项目概览、定位、能力、环境 | 全部 |
| [02_Architecture.md](02_Architecture.md) | 架构设计、组件图、数据流、时序 | 全部 |
| [03_CodeMap.md](03_CodeMap.md) | 目录结构、关键文件导航 | 新人 |
| [04_Interface.md](04_Interface.md) | IPC 接口清单、参数、错误码 | 全部 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面、外部输入、敏感操作 | 安全 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估（代码级） | 安全 |
| [07_Build.md](07_Build.md) | GN 构建、产物、Feature 开关 | 工程 |
| [08_Internals.md](08_Internals.md) | 核心类、生命周期、Owner 关系 | 深入 |

---

## 快速参考

### 核心事实

| 属性 | 值 |
|------|-----|
| **项目类型** | 系统服务 (SA) |
| **SA ID** | 6001 |
| **进程名** | deviceprofile |
| **对外接口** | IPC (C++ Innerkits) |
| **N-API** | ❌ 无 |
| **存储** | KV Store + RDB |
| **权限** | `ohos.permission.ACCESS_SERVICE_DP` |

### 关键文件位置

```
服务入口: services/core/include/distributed_device_profile_service_new.h:42
客户端:   interfaces/innerkits/core/include/distributed_device_profile_client.h:46
权限管理: services/core/src/permissionmanager/permission_manager.cpp:218
SA配置:   sa_profile/6001.json
权限配置: permission/permission.json
```

---

## 更新与维护

- 本文档随代码变更同步更新
- 代码证据标注格式：`文件路径:行号`
- 发现错误或遗漏请提交 Issue

---

## 相关链接

- [OpenHarmony 官方文档](https://www.openharmony.cn/)
- [DeviceProfile 源码仓库](https://gitee.com/openharmony/deviceprofile_device_info_manager)
