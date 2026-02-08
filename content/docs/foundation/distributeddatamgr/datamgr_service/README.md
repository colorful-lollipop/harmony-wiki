# OpenHarmony Distributed Data Manager Wiki

**分布式数据管理服务 (DDS/Distributed Data Service)**

---

## 项目概述

本 Wiki 描述 OpenHarmony 分布式数据管理服务的服务端实现，位于仓库 `distributeddatamgr_datamgr_service`。

**关键说明**：本仓库为**纯服务端实现**，不包含 N-API/JavaScript 绑定层。JS API 在独立仓库（`kv_store`、`relational_store` 等）。

---

## 覆盖范围

| 主题 | 状态 | 说明 |
|-----|------|------|
| 项目定位与核心能力 | ✅ 完成 | README, 功能边界 |
| 四层架构 | ✅ 完成 | App/Service/Framework/Adapter |
| 系统能力 (SA) | ✅ 完成 | SA ID: 1301, 进程: distributeddata |
| 核心模块职责 | ✅ 完成 | KVDB/RDB/Cloud/Object/DataShare/UDMF/UTD |
| Feature 系统 | ✅ 完成 | 插件化架构 |
| GN 构建配置 | ✅ 完成 | 8个 feature 开关 |
| IPC 接口清单 | ✅ 完成 | 125 个 RPC 方法详细列表 |
| 攻击面分析 | ✅ 完成 | 外部输入、敏感操作、信任边界 |
| 内部实现细节 | ✅ 完成 | 核心类、生命周期、安全机制 |
| 安全风险评估 | ⚠️ 持续更新 | 威胁模型与风险点 |
| N-API | ❌ 不适用 | JS API 在独立仓库 |

---

## 快速导航

```
├── 📖 概览
│   └── [README](README.md)
│   └── [概览](00_Overview.md)
│
├── 🏗️ 架构
│   └── [架构说明](01_Architecture.md)
│   └── [内部实现细节](06_Internals.md)
│
├── 🔨 构建
│   └── [构建配置](02_Build.md)
│
├── 🔒 安全
│   └── [攻击面分析](05_AttackSurface.md)
│   └── [IPC 接口清单](04_Interface.md)
│   └── [安全评审](03_Security.md)
│
├── 📁 附录
│   └── [术语表](appendix/Glossary.md)
│   └── [错误码](appendix/ErrorCodes.md)
│
└── 📋 导航
    └── [SUMMARY](SUMMARY.md) - 双路线阅读指南
```

---

## 关键概念

### 数据隔离三元组
```
(user, app, database) → 唯一确定一个数据库
```

### 服务进程
- **进程名**: `distributeddata`
- **SA ID**: `1301`
- **主产物**: `libdistributeddataservice.z.so`

### 五种核心能力
1. **KV/RDB 跨设备同步** - 自动同步和一致性保证
2. **分布式对象持久化** - 完整生命周期管理
3. **跨应用数据静默共享** - 无需启动提供方进程
4. **设备-云同步** - 数据库数据同步到云端
5. **UDMF 统一数据管理** - 系统级数据标准

---

## 源码位置

| 层次 | 路径 | 说明 |
|-----|------|-----|
| App 层 | `services/distributeddataservice/app/` | SystemAbility 入口 |
| Service 层 | `services/distributeddataservice/service/` | 业务逻辑实现 |
| Framework 层 | `services/distributeddataservice/framework/` | 框架接口与基础设施 |
| Adapter 层 | `services/distributeddataservice/adapter/` | 系统服务适配器 |

---

## 文档维护

### 更新时机
- 新增/删除 Feature 模块
- 修改 GN feature 开关
- 架构重大变更
- 新增安全风险点

### 验证方法
```bash
# 1. 检查链接存在性
grep -r "](" wiki/ | grep -v ".md)"

# 2. 验证 GN feature 开关与 bundle.json 一致
grep "datamgr_service_" wiki/*.md
grep "datamgr_service_" datamgr_service.gni bundle.json
```

---

## 相关资源

- **官方文档**: https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/分布式数据管理子系统.md
- **应用开发指南**: https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/database
- **JS API**: https://gitee.com/openharmony/distributeddatamgr_kv_store (独立仓库)

---

## 更新日志

| 日期 | 版本 | 变更 |
|-----|------|------|
| 2026-02-07 | 1.1 | 新增 IPC 接口清单 (125 个 RPC)、攻击面分析、内部实现细节文档 |
| 2026-02-06 | 1.0 | 初始版本，完成架构与构建文档 |
