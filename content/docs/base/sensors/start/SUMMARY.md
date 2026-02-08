# Sensors Start Wiki 文档导航

**最后更新**: 2026-02-07
**文档版本**: 1.0

---

## 📚 文档目录

### 核心文档（必读）

| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [README.md](./README.md) | 文档说明、覆盖范围、更新方式 | 5 min |
| [00_Overview.md](./00_Overview.md) | 项目定位、边界、核心能力、运行环境 | 10 min |
| [SUMMARY.md](./SUMMARY.md) | 本导航页 | - |

### 架构与设计

| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构、文件组织、模块职责 | 5 min |
| [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、时序图 | 10 min |

### API 文档

| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [03_NAPI_API.md](./03_NAPI_API.md) | N-API 接口（本组件无 N-API） | 2 min |
| [04_Inner_API.md](./04_Inner_API.md) | 内部 API（本组件无源代码） | 2 min |

### 构建与产物

| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [05_GN_Targets.md](./05_GN_Targets.md) | GN 目标、构建配置、依赖关系 | 10 min |
| [06_Build_Artifacts.md](./06_Build_Artifacts.md) | 编译产物、安装路径、运行时加载 | 8 min |

### 安全与运维

| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [07_Security_Audit.md](./07_Security_Audit.md) | 安全风险评审、攻击面分析、修复建议 | 15 min |
| [08_FAQ.md](./08_FAQ.md) | 常见构建/运行/调试问题 | 10 min |

### 附录

| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [appendix/Config_Files.md](./appendix/Config_Files.md) | 配置文件详解、参数说明 | 15 min |

---

## 🚀 新人阅读路径

### 快速上手（30 分钟）

```
1. README.md (5 min)
   └─ 了解文档范围和更新方式

2. 00_Overview.md (10 min)
   └─ 理解项目定位和核心能力

3. 01_Directory_Structure.md (5 min)
   └─ 熟悉文件结构

4. 05_GN_Targets.md (10 min)
   └─ 了解构建流程
```

### 深入理解（1 小时）

在快速上手基础上，继续阅读:

```
5. 02_Architecture.md (10 min)
   └─ 理解服务启动流程

6. 06_Build_Artifacts.md (8 min)
   └─ 了解产物和运行时加载

7. 07_Security_Audit.md (15 min)
   └─ 掌握安全风险（重要！）

8. 08_FAQ.md (10 min)
   └─ 了解常见问题
```

### 完整掌握（2 小时）

完成深入理解后，查阅附录:

```
9. appendix/Config_Files.md (15 min)
   └─ 详细了解配置参数

10. 工作笔记 (参考)
    └─ wiki/_work/NOTES.md - 详细事实记录
    └─ wiki/_work/PLAN.md - 任务计划
```

---

## 📖 按角色查阅

### 系统架构师

必读:
- [00_Overview.md](./00_Overview.md) - 项目定位
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [07_Security_Audit.md](./07_Security_Audit.md) - 安全设计

参考:
- [01_Directory_Structure.md](./01_Directory_Structure.md)
- [appendix/Config_Files.md](./appendix/Config_Files.md)

### 构建工程师

必读:
- [05_GN_Targets.md](./05_GN_Targets.md) - 构建配置
- [06_Build_Artifacts.md](./06_Build_Artifacts.md) - 编译产物
- [08_FAQ.md](./08_FAQ.md) - 常见构建问题

参考:
- [01_Directory_Structure.md](./01_Directory_Structure.md)
- [appendix/Config_Files.md](./appendix/Config_Files.md)

### 安全审计人员

必读:
- [07_Security_Audit.md](./07_Security_Audit.md) - 安全风险评审
- [00_Overview.md](./00_Overview.md) - 项目背景
- [appendix/Config_Files.md](./appendix/Config_Files.md) - 权限配置

参考:
- [02_Architecture.md](./02_Architecture.md) - 攻击面分析

### OpenHarmony 移植者

必读:
- [00_Overview.md](./00_Overview.md) - 项目定位
- [05_GN_Targets.md](./05_GN_Targets.md) - 构建配置
- [06_Build_Artifacts.md](./06_Build_Artifacts.md) - 产物和安装
- [08_FAQ.md](./08_FAQ.md) - 常见问题

参考:
- [appendix/Config_Files.md](./appendix/Config_Files.md)
- [02_Architecture.md](./02_Architecture.md)

---

## 🔍 关键主题索引

### 权限与安全

- [msdp 服务权限列表](./07_Security_Audit.md#msdp-服务权限清单)
- [sensors 服务权限](./07_Security_Audit.md#sensors-服务权限)
- [安全风险分析](./07_Security_Audit.md#安全风险分析)

### 构建系统

- [GN Target: sensors.rc](./05_GN_Targets.md#target-sensorsrc)
- [GN Target: msdp.rc](./05_GN_Targets.md#target-msdprc)
- [编译产物](./06_Build_Artifacts.md#编译产物清单)

### 服务启动

- [sensors 服务配置](./appendix/Config_Files.md#sensors-服务配置)
- [msdp 服务配置](./appendix/Config_Files.md#msdp-服务配置)
- [启动流程](./02_Architecture.md#服务启动流程)

### 配置文件

- [sensors.cfg](./appendix/Config_Files.md#sensorscfg-详解)
- [msdp.cfg](./appendix/Config_Files.md#msdpcfg-详解)
- [musl vs 非 musl 差异](./appendix/Config_Files.md#musl-与-非-musl-配置差异)

---

## ⚠️ 重要提示

### 本组件特点

**sensors_start** 是一个**纯配置型组件**:

- ❌ 无源代码
- ❌ 无 N-API
- ❌ 无内部 API
- ✅ 仅提供 INIT 系统启动配置

### 外部依赖

本组件与以下组件协作（不在本仓库）:

| 组件 | 仓库 | 说明 |
|------|------|------|
| sensors_sensor | openharmony/sensors_sensor | 传感器服务实现（SA 3601） |
| sensors_miscdevice | openharmony/sensors_miscdevice | 杂项设备服务实现（SA 3602，如震动器） |
| msdp 相关组件 | - | 多设备系统平台服务实现 |

### 文档限制

以下内容**不在**本文档范围内:

- `sensors_sensor` 组件的实现细节
- `sensors_miscdevice` 组件的实现细节
- `msdp` 组件的完整功能文档
- `libsensor_service.z.so` 的实现
- `libmiscdevice_service.z.so` 的实现

**建议**: 查阅相关组件仓库获取详细信息。

---

## 🔗 相关资源

### OpenHarmony 官方文档

- [Init 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/subsystems/init-readme.md)
- [SA (System Ability) 框架](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/frameworkability/sa-framework.md)
- [权限系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/permission/permission-list.md)

### 相关仓库

- [sensors_sensor](https://gitee.com/openharmony/sensors_sensor)
- [sensors_miscdevice](https://gitee.com/openharmony/sensors_miscdevice)

### 工作笔记

- [NOTES.md](./_work/NOTES.md) - 详细事实记录和证据
- [PLAN.md](./_work/PLAN.md) - 任务计划和进度

---

## 📝 文档维护

### 更新记录

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.1 | 2026-02-07 | 添加项目评估报告 (ASSESSMENT.md)，完成 musl 版本权限差异分析 |
| 1.0 | 2026-02-06 | 初始版本，基于 OpenHarmony 3.1 |

### 贡献指南

如需更新文档:

1. 确保所有结论有代码证据
2. 更新 [NOTES.md](./_work/NOTES.md) 中的证据记录
3. 同步更新本文档中的相关章节
4. 保持术语统一

---

**最后更新**: 2026-02-07
