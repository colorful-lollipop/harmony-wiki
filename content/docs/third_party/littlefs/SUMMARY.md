# 阅读路线建议

> 本文档提供 littlefs OpenHarmony 集成文档的阅读路线建议，帮助不同角色的读者快速找到所需信息。

---

## 按角色分类的阅读路线

### 1. 集成开发者（将 littlefs 集成到 OH 项目）

**阅读顺序**:
1. ✅ [README](./README.md) - 了解 littlefs 在 OH 中的定位和快速开始
2. 📖 [03_Build_Integration](./03_Build_Integration.md) - 了解构建系统适配
3. 📖 [04_Usage_in_OH](./04_Usage_in_OH.md) - 查看依赖关系和使用场景

**关键信息**:
- littlefs.gni 的使用方式
- BUILD.gn 编译选项配置
- VFS 适配层实现

**预计耗时**: 30 分钟

---

### 2. 芯片厂商适配者（为特定芯片移植 littlefs）

**阅读顺序**:
1. ✅ [README](./README.md) - 了解 littlefs 的技术特点
2. 📖 [02_Patches](./02_Patches.md) - 了解 OH 适配方式（无 Patch）
3. 📖 [03_Build_Integration](./03_Build_Integration.md) - 了解构建系统适配
4. 📖 [04_Usage_in_OH](./04_Usage_in_OH.md) - 参考其他芯片的 HAL 层实现
5. 📖 [05_API_Differences](./05_API_Differences.md) - 了解 API 接口

**关键信息**:
- lfs_adapter.c 的 VFS 适配层实现
- littlefs_hal.c 的 HAL 层实现示例
- lfs_config 配置参数

**参考实现**:
- 海思 WS63V100: `device/soc/hisilicon/ws63v100/adapter/hals/utils/file`
- Rockchip RK2206: `device/soc/rockchip/rk2206/hdf_driver/fs`
- QEMU ARM MPS2: `device/qemu/arm_mps2_an386/liteos_m/board/fs`

**预计耗时**: 1 小时

---

### 3. 安全审计人员（评估 littlefs 的安全性）

**阅读顺序**:
1. ✅ [README](./README.md) - 了解 littlefs 的安全特性
2. 📖 [02_Patches](./02_Patches.md) - 检查 OH 特定修改（如有）
3. 📖 [06_Security](./06_Security.md) - 安全风险分析

**关键信息**:
- 已知 CVE 状态
- OH Patch 引入的新攻击面
- 安全升级策略

**预计耗时**: 20 分钟

---

### 4. 版本维护者（管理 littlefs 的升级和维护）

**阅读顺序**:
1. ✅ [README](./README.md) - 了解当前版本信息
2. 📖 [01_Overview](./01_Overview.md) - 详细了解库信息
3. 📖 [02_Patches](./02_Patches.md) - 检查 Patch 维护情况
4. 📖 [03_Build_Integration](./03_Build_Integration.md) - 了解构建系统适配
5. 📖 [06_Security](./06_Security.md) - 了解安全升级策略

**关键信息**:
- 当前版本与上游的对比
- Patch 升级建议
- 安全升级路线图

**预计耗时**: 45 分钟

---

### 5. 应用开发者（在 OH 应用中使用文件系统）

**阅读顺序**:
1. ✅ [README](./README.md) - 快速开始示例
2. 📖 [04_Usage_in_OH](./04_Usage_in_OH.md) - 了解使用场景

**关键信息**:
- 如何挂载 littlefs 文件系统
- 常用 API 使用示例
- 典型使用场景

**预计耗时**: 15 分钟

---

### 6. 技术调研者（了解 littlefs 的技术细节）

**阅读顺序**:
1. ✅ [README](./README.md) - 了解 littlefs 在 OH 中的定位
2. 📖 [01_Overview](./01_Overview.md) - 详细了解库信息
3. 📖 [02_Patches](./02_Patches.md) - 了解 OH 适配方式
4. 📖 [03_Build_Integration](./03_Build_Integration.md) - 了解构建系统适配
5. 📖 [04_Usage_in_OH](./04_Usage_in_OH.md) - 了解依赖关系
6. 📖 [05_API_Differences](./05_API_Differences.md) - 了解 API 接口
7. 📖 [06_Security](./06_Security.md) - 了解安全特性

**关键信息**:
- littlefs 的核心特性
- OH 适配的技术细节
- 依赖关系和架构图

**预计耗时**: 1.5 小时

---

## 按任务分类的阅读路线

### 任务：快速集成 littlefs

1. [README](./README.md) - 快速开始
2. [03_Build_Integration](./03_Build_Integration.md) - 构建系统配置

**预计耗时**: 20 分钟

---

### 任务：深度了解 OH 适配

1. [README](./README.md) - 适配概述
2. [02_Patches](./02_Patches.md) - Patch 分析
3. [03_Build_Integration](./03_Build_Integration.md) - 构建适配
4. [04_Usage_in_OH](./04_Usage_in_OH.md) - 使用分析

**预计耗时**: 1 小时

---

### 任务：安全评估

1. [README](./README.md) - 安全特性概述
2. [06_Security](./06_Security.md) - 安全风险分析

**预计耗时**: 20 分钟

---

### 任务：版本升级

1. [02_Patches](./02_Patches.md) - Patch 升级建议
2. [06_Security](./06_Security.md) - 安全升级策略

**预计耗时**: 30 分钟

---

## 文档详细说明

### [README](./README.md)

**内容**:
- littlefs 在 OH 中的定位和作用
- OH 适配概述
- 技术特点（断电恢复、磨损均衡、内存边界）
- 版本信息
- 快速开始示例
- 常见问题

**适用人群**: 所有读者

**预计耗时**: 15 分钟

---

### [01_Overview](./01_Overview.md)

**内容**:
- 库名称、版本、许可证
- 原始功能一句话描述
- 上游地址
- 该库在 OH 中的作用和定位

**适用人群**: 技术调研者、版本维护者

**预计耗时**: 10 分钟

---

### [02_Patches](./02_Patches.md)

**内容**:
- Patch 清单表（如有）
- 每个 Patch 的详细分析（如有）
- 修改目的、修改内容、OH 价值、回归风险
- **特殊情况**: 标注无 Patch，说明适配层集成方式

**适用人群**: 集成开发者、芯片厂商适配者、安全审计人员、版本维护者

**预计耗时**: 15 分钟

---

### [03_Build_Integration](./03_Build_Integration.md)

**内容**:
- BUILD.gn 结构说明（littlefs.gni）
- 关键编译选项（defines、configs、flags）
- 与上游构建系统的差异（Makefile vs GN）
- lfs_conf.h 中的 OH 配置参数

**适用人群**: 集成开发者、芯片厂商适配者、版本维护者

**预计耗时**: 20 分钟

---

### [04_Usage_in_OH](./04_Usage_in_OH.md)

**内容**:
- 谁在使用（直接依赖者列表）
- 使用方式（静态链接、动态链接、头文件引用）
- 关键使用场景
- 依赖关系图（Mermaid）
- VFS 适配层实现分析（lfs_adapter.c）

**适用人群**: 集成开发者、芯片厂商适配者、应用开发者

**预计耗时**: 30 分钟

---

### [05_API_Differences](./05_API_Differences.md)

**内容**:
- OH 新增的 API（如有）
- 行为变更的 API（如有）
- 废弃或禁用的功能（如有）
- **特殊情况**: 如无差异，标注"无 API 差异"

**适用人群**: 芯片厂商适配者、技术调研者

**预计耗时**: 10 分钟

---

### [06_Security](./06_Security.md)

**内容**:
- 该库已知 CVE 和在 OH 版本中的修复状态
- OH Patch 引入的新攻击面（如有）
- 建议的安全升级策略
- littlefs 的安全特性（CRC 校验、原子提交等）

**适用人群**: 安全审计人员、版本维护者

**预计耗时**: 20 分钟

---

## 关键发现摘要

### 无代码 Patch

**重要**: littlefs 在 OpenHarmony 中**没有代码级 Patch**，核心代码与上游完全一致。

**原因**: 通过适配层（lfs_adapter.c）和构建系统适配（littlefs.gni）实现集成，保持代码纯净性。

### 版本同步

**重要**: OH 版本（v2.11.2）与上游最新版本保持一致，无需升级。

### 适配层集成

**重要**: OH 通过 VFS 适配层（lfs_adapter.c）和 HAL 层（littlefs_hal.c）实现 littlefs 集成，职责分离清晰。

---

## 附录：工作目录

文档创建过程中的工作文件存放在 `wiki/_work/` 目录：

| 文件 | 说明 |
|------|------|
| **ASSESSMENT.md** | Phase 0 项目评估结果 |
| **NOTES.md** | 分析过程记录 |
| **PLAN.md** | 任务进度跟踪 |

---

**最后更新时间**: 2026-02-08
