# OAID 智能 Wiki

## 📖 简介

这是 **OpenHarmony OAID (Open Anonymous Device Identifier)** 项目的智能 Wiki 文档库。

OAID 是 OpenHarmony 提供的开放匿名设备标识符系统服务，为个性化广告投放提供非永久性设备标识，同时保护用户隐私数据。

---

## 🎯 文档定位

本文档服务于两类受众：

| 受众 | 需求 | 推荐阅读 |
|------|------|---------|
| **新人学习者** | 快速理解项目、上手使用 | [新人学习路线](#新人学习路线) |
| **安全研究员** | 攻击面分析、风险评估 | [安全研究路线](#安全研究路线) |

---

## 🚀 快速开始

### 新人学习路线

**30 分钟快速入门**:

1. 📘 **[项目概览](01_Overview.md)** - 了解 OAID 是什么、能做什么
2. 🗺️ **[代码地图](03_CodeMap.md)** - 找到关键代码位置

**1-2 小时深入理解**:

3. 🏗️ **[架构分析](02_Architecture.md)** - 组件关系和数据流
4. 🔌 **[接口文档](04_Interface.md)** - API 使用参考

### 安全研究路线

**30 分钟快速评估**:

1. 🎯 **[攻击面分析](05_AttackSurface.md)** - 外部输入和敏感操作
2. ⚠️ **[安全风险评估](06_SecurityReview.md)** - 11 项风险详细分析

---

## 📚 文档目录

### 核心内容

| 文档 | 描述 | 状态 |
|------|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览、快速开始 | ✅ 完成 |
| [02_Architecture.md](02_Architecture.md) | 架构与数据流分析 | ✅ 完成 |
| [03_CodeMap.md](03_CodeMap.md) | 目录结构与代码地图 | ✅ 完成 |
| [04_Interface.md](04_Interface.md) | 对外接口文档 | ✅ 完成 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 | ✅ 完成 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 | ✅ 完成 |
| [07_Build.md](07_Build.md) | 构建与产物 | ✅ 完成 |
| [08_Internals.md](08_Internals.md) | 内部实现细节 | ✅ 完成 |

### 导航与工具

| 文档 | 描述 |
|------|------|
| [SUMMARY.md](SUMMARY.md) | 完整目录导航，双路线推荐 |
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据汇总 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪 |

---

## 🏗️ 项目概览

### 基本信息

| 属性 | 值 |
|------|-----|
| **项目名称** | OAID (Open Anonymous Device Identifier) |
| **Bundle 名称** | @ohos/oaid |
| **系统能力 ID** | 6101 |
| **子系统** | advertising |
| **API 版本** | 10+ |
| **系统类型** | Standard System |

### 核心功能

- ✅ **生成匿名标识符** - 基于 UUID v4 算法
- ✅ **支持个性化广告** - 为广告 SDK 提供设备标识
- ✅ **用户可控重置** - 保护用户隐私
- ✅ **未成年人保护** - 集成 Ads Service 检测

### 权限要求

| 权限 | 类型 | 说明 |
|------|------|------|
| `ohos.permission.APP_TRACKING_CONSENT` | user_grant | 必须动态申请 |

---

## 🔒 安全摘要

### 发现的风险

| 严重程度 | 数量 | 说明 |
|---------|------|------|
| 🔴 **高危** | 2 | 需要立即修复 |
| 🟠 **中危** | 7 | 建议短期修复 |
| 🟡 **低危** | 2 | 可长期优化 |

### 关键风险

1. **R1: 手动 Mutex 管理** - 可能导致死锁
2. **R2: JSON 解析无限制** - 可能导致 DoS
3. **R3: 路径遍历风险** - 可能绕过信任列表
4. **R4: TOCTOU 竞争条件** - 文件操作安全问题

详细分析见 [安全风险评估](06_SecurityReview.md)

---

## 📊 代码统计

| 模块 | 文件数 | 行数 |
|------|--------|------|
| Services | 9 | ~2,000 |
| Interfaces (Innerkits) | 8 | ~800 |
| Interfaces (NAPI) | 4 | ~350 |
| Utils | 3 | ~150 |
| **总计** | **24** | **~3,300** |

---

## 🔗 快速链接

### 核心代码位置

| 功能 | 文件路径 | 行号 |
|------|---------|------|
| 服务主类 | `services/oaid_manager/src/oaid_service.cpp` | 1-439 |
| IPC 处理 | `services/oaid_manager/src/oaid_service_stub.cpp` | 1-356 |
| N-API 实现 | `interfaces/kits/js/napi/oaid/src/oaid.cpp` | 1-261 |
| 权限检查 | `services/oaid_manager/src/oaid_service_stub.cpp` | 45-80 |

### 外部资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [OAID API 参考](https://gitee.com/openharmony/docs/blob/master/en/application-dev/reference/apis/js-apis-advertising.md)
- [华为 HMS 广告示例](https://github.com/HMS-Core/hms-ads-demo-harmonyos)

---

## 🛠️ 构建说明

### 全量构建

```bash
gn gen out --args="target_os=\"ohos\""
ninja -C out domains/advertising/oaid:oaid_native_packages
```

### 产物列表

| 产物 | 路径 |
|------|------|
| 服务库 | `/system/lib/liboaid_service.z.so` |
| 客户端库 | `/system/lib/liboaid_client.z.so` |
| N-API 模块 | `/system/lib/module/identifier/liboaid_napi.z.so` |

详细构建说明见 [构建文档](07_Build.md)

---

## 📝 质量保证

### 文档标准

- ✅ 每个技术结论都有代码证据支撑
- ✅ 文件路径和行号格式：`path:line`
- ✅ 关键符号名完整标注
- ✅ Mermaid 图表辅助理解
- ✅ 双路线导航（新人/安全）

### 证据要求

| 类型 | 要求 |
|------|------|
| 架构结论 | 有目录结构或代码调用链支撑 |
| API 描述 | 有 N-API 注册点或函数定义定位 |
| 安全风险 | 有具体代码路径 + 符号 + 行号 |
| 构建配置 | 有 BUILD.gn 片段或 target 名称 |

---

## 🤝 维护说明

### 更新流程

1. **代码变更** → 更新 `_work/NOTES.md` 代码证据
2. **架构变更** → 更新 `02_Architecture.md`
3. **接口变更** → 更新 `04_Interface.md`
4. **安全修复** → 更新 `05_AttackSurface.md` 和 `06_SecurityReview.md`

### 验证清单

- [ ] 证据完整性检查
- [ ] 链接有效性检查
- [ ] 受众适配检查
- [ ] 术语一致性检查

---

## 📄 许可证

本项目 Wiki 基于 OAID 项目源代码生成，遵循 Apache License 2.0。

---

## 📞 反馈

如发现文档问题或需补充内容，请：

1. 检查 `_work/NOTES.md` 是否有相关代码证据
2. 查看 `SUMMARY.md` 是否有相关链接
3. 参考 `ASSESSMENT.md` 了解项目评估详情

---

**开始阅读**: [项目概览](01_Overview.md) | [目录导航](SUMMARY.md)

**安全审计**: [攻击面分析](05_AttackSurface.md) | [安全风险评估](06_SecurityReview.md)
