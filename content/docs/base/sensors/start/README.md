# OpenHarmony Sensors Start 组件 Wiki

**文档版本**: 1.1
**生成时间**: 2026-02-07
**仓库**: /base/sensors/start
**组件名称**: @ohos/start / start

---

## 文档覆盖范围

本文档集全面覆盖 `sensors_start` 组件的架构、构建、安全和使用说明。

### 已覆盖内容

- ✅ **项目概览**: 组件定位、边界、核心能力
- ✅ **目录结构**: 仓库文件组织和模块职责
- ✅ **架构说明**: 服务启动流程、依赖关系
- ⚠️ **N-API 文档**: 本组件不提供 N-API（配置型组件）
- ⚠️ **内部 API 文档**: 本组件无源代码，不提供内部 API
- ✅ **GN 构建系统**: 构建目标、编译产物、依赖关系
- ✅ **安全风险评审**: 权限配置、攻击面分析
- ✅ **常见问题 FAQ**: 构建和运行时问题定位
- ✅ **项目评估**: 完整的 ASSESSMENT.md 项目画像
- 🔴 **重要发现**: musl 版本权限配置差异分析

### 未覆盖内容

- ❌ `sensors_sensor` 组件的具体实现（位于外部仓库）
- ❌ `sensors_miscdevice` 组件的具体实现（位于外部仓库）
- ❌ `msdp` 组件的完整功能文档（需要查阅 msdp 相关仓库）
- ❌ `sensors.json` 和 `msdp.json` 配置文件（不在本仓库）
- ❌ `libsensor_service.z.so` 和 `libmiscdevice_service.z.so` 的实现细节

**说明**: 未覆盖内容需要查阅相关组件仓库的文档。

---

## 文档更新方式

本文档基于代码自动生成，更新步骤：

1. **代码变更后更新**:
   ```bash
   cd /base/sensors/start
   # 重新生成 Wiki（根据实际情况）
   ```

2. **手动更新指南**:
   - 如遇重大架构变更，需更新对应章节
   - 新增配置项需同步到 [06_Build_Artifacts.md](./06_Build_Artifacts.md) 和 [appendix/Config_Files.md](./appendix/Config_Files.md)
   - 权限变更需更新 [07_Security_Audit.md](./07_Security_Audit.md)

3. **版本记录**:
   - v1.1 (2026-02-07): 添加项目评估报告，完成 musl 版本权限差异分析
   - v1.0 (2026-02-06): 初始版本，基于 OpenHarmony 3.1

---

## 快速导航

**新人阅读顺序**（推荐）:

1. [SUMMARY.md](./SUMMARY.md) - 全站导航
2. [00_Overview.md](./00_Overview.md) - 项目概览（必读）
3. [01_Directory_Structure.md](./01_Directory_Structure.md) - 目录结构
4. [05_GN_Targets.md](./05_GN_Targets.md) - 构建系统
5. [07_Security_Audit.md](./07_Security_Audit.md) - 安全风险（重要）
6. [08_FAQ.md](./08_FAQ.md) - 常见问题

**按需查阅**:

- 架构设计: [02_Architecture.md](./02_Architecture.md)
- API 说明: [03_NAPI_API.md](./03_NAPI_API.md)（说明无 N-API）
- 内部接口: [04_Inner_API.md](./04_Inner_API.md)（说明无源代码）
- 编译产物: [06_Build_Artifacts.md](./06_Build_Artifacts.md)
- 配置详解: [appendix/Config_Files.md](./appendix/Config_Files.md)

---

## 重要提示

### 本组件特点

⚠️ **这是一个纯配置型组件**:

- **不包含源代码** (.cpp/.c/.h/.ts 等)
- **不提供 N-API/JS API**
- **不提供内部 API**
- **仅提供 INIT 系统启动配置文件**

### 与其他组件的关系

```
sensors_start (本仓库)
    ↓ 提供
    ├─→ sensors_sensor (外部仓库) - 传感器服务实现
    └─→ sensors_miscdevice (外部仓库) - 杂项设备服务实现（如震动器）
```

**关键点**:
- `sensors_sensor` 和 `sensors_miscdevice` 共享同一个 `sensors` 进程
- 本仓库提供统一的启动配置，避免重复启动进程
- 实际的服务实现代码在其他仓库

---

## 适用人群

- ✅ **系统架构师**: 了解传感器服务启动流程
- ✅ **构建工程师**: 了解构建配置和产物
- ✅ **安全审计人员**: 查看权限配置和风险分析
- ✅ **OpenHarmony 移植者**: 理解传感器服务启动机制

---

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| INIT | OpenHarmony Init System | OpenHarmony 系统初始化进程 |
| SA | System Ability | OpenHarmony 系统能力 |
| sa_main | System Ability Main | SA 进程管理器 |
| GN | Generate Ninja | OpenHarmony 构建系统 |
| msdp | Multi-Device System Platform | 多设备系统平台 |
| CFG | Configuration | INIT 系统配置文件 |
| RC | Run Control | INIT 系统配置文件（GN target 名称） |

---

## 质量保证

本文档遵循以下质量标准:

- ✅ 所有关键结论均可在仓库内找到直接证据（文件路径 + 行号）
- ✅ 不包含测试相关内容引用
- ✅ 术语统一，无歧义
- ✅ 所有链接均可跳转
- ✅ 所有证据均在 [wiki/_work/NOTES.md](./_work/NOTES.md) 中记录

---

## 工作笔记

详细的工作笔记和证据记录请查看:

- [工作笔记](./_work/NOTES.md) - 详细事实记录和证据
- [工作计划](./_work/PLAN.md) - 任务拆解和进度追踪

---

**最后更新**: 2026-02-07
**维护者**: OpenHarmony Sensors Team
