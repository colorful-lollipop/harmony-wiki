# 阅读路线建议

本 Wiki 文档采用模块化组织，读者可根据自身角色和需求选择不同的阅读路线。

---

## 路线一：系统开发者（推荐 ⭐⭐⭐⭐⭐）

**目标**: 了解 libuv 在 OH 中的作用和使用方式

**阅读顺序**:

1. **[01_Overview.md](./01_Overview.md)** (15 min)
   - 了解 libuv 原始功能
   - 理解 OH 中的定位和作用

2. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** (20 min)
   - 查看谁在使用 libuv
   - 了解典型使用场景
   - 理解依赖关系

3. **[02_Patches.md](./02_Patches.md)** (30 min)
   - 深入了解 OH 定制化内容
   - 理解 FFRT 集成原理
   - 掌握 DFX 诊断框架集成

4. **[03_Build_Integration.md](./03_Build_Integration.md)** (15 min)
   - 了解 BUILD.gn 配置
   - 掌握编译选项

---

## 路线二：库维护者（推荐 ⭐⭐⭐⭐⭐）

**目标**: 维护和升级 libuv 库

**阅读顺序**:

1. **[02_Patches.md](./02_Patches.md)** (30 min)
   - 必须理解所有 OH 定制化点
   - 掌握条件编译宏的使用

2. **[03_Build_Integration.md](./03_Build_Integration.md)** (20 min)
   - 深入理解 BUILD.gn
   - 掌握多平台适配逻辑

3. **[05_API_Differences.md](./05_API_Differences.md)** (15 min)
   - 了解 API 差异
   - 掌握新增接口

4. **[06_Security.md](./06_Security.md)** (10 min)
   - 安全风险评估
   - 升级注意事项

---

## 路线三：安全工程师（推荐 ⭐⭐⭐⭐）

**目标**: 安全审计和风险评估

**阅读顺序**:

1. **[06_Security.md](./06_Security.md)** (15 min)
   - CVE 跟踪
   - 攻击面分析

2. **[02_Patches.md](./02_Patches.md)** (20 min)
   - 重点查看 OH 特有代码
   - 分析新增攻击面

3. **[03_Build_Integration.md](./03_Build_Integration.md)** (10 min)
   - 了解编译选项安全影响

---

## 路线四：应用开发者（推荐 ⭐⭐⭐）

**目标**: 使用 libuv 开发应用

**阅读顺序**:

1. **[01_Overview.md](./01_Overview.md)** (10 min)
   - 了解 libuv 基本功能

2. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** (10 min)
   - 了解如何在 OH 中使用
   - 查看依赖方式

---

## 路线五：快速参考（推荐 ⭐⭐⭐）

**目标**: 快速查找特定信息

**直接访问**:

- 查看 FFRT 集成: [02_Patches.md#FFRT 集成](./02_Patches.md#ffrt-集成)
- 查看 BUILD.gn: [03_Build_Integration.md](./03_Build_Integration.md)
- 查看依赖关系: [04_Usage_in_OH.md#依赖图](./04_Usage_in_OH.md#依赖图)
- 查看 QoS 定义: [05_API_Differences.md#QoS 分级](./05_API_Differences.md#qos-分级)

---

## 时间预算

| 路线 | 预计时间 | 深度 |
|-----|---------|-----|
| 系统开发者 | 80 min | ⭐⭐⭐⭐ |
| 库维护者 | 75 min | ⭐⭐⭐⭐⭐ |
| 安全工程师 | 45 min | ⭐⭐⭐⭐ |
| 应用开发者 | 20 min | ⭐⭐ |
| 快速参考 | 按需 | ⭐ |

---

## 相关资源

- [OpenHarmony 文档中心](https://docs.openharmony.cn)
- [libuv 官方文档](http://docs.libuv.org)
- [FFRT 文档](https://gitee.com/openharmony/ resources)

---

*建议: 首次阅读建议按照完整路线，后续按需查阅*
