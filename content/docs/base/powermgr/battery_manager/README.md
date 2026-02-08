# Battery Manager Wiki

> **目的**: battery_manager 项目工程文档主入口

**生成时间**: 2026-02-06 03:35:56
**最后更新**: 2026-02-07 10:15:00

---

## 🎯 文档导航

### 🟢 新人学习路线（推荐）

对于首次接触 battery_manager 项目的开发者，建议按以下顺序阅读：

1. **[00_Overview](00_Overview.md)** - 项目概览与定位
2. **[01_Project_Position](01_Project_Position.md)** - 项目边界与核心能力
3. **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构与模块职责
4. **[03_Architecture](03_Architecture.md)** - 系统架构说明
5. **[04_NAPI_API](04_NAPI_API.md)** - 对外 N-API 接口
6. **[05_Inner_API](05_Inner_API.md)** - 内部 API 接口
7. **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标
8. **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物说明
9. **[09_Troubleshooting](09_Troubleshooting.md)** - 常见问题与定位

**预计时间**: 2-3 小时
**详细路线**: 查看 [SUMMARY.md](SUMMARY.md#新人学习路线推荐)

---

### 🔴 安全研究路线（推荐）

对于安全研究员和安全审计工程师，建议按以下顺序阅读：

1. **[05_AttackSurface](05_AttackSurface.md)** - 攻击面分析 ⭐ 重点
2. **[08_Security_Review](08_Security_Review.md)** - 安全风险评审 ⭐ 重点
3. **[03_Architecture](03_Architecture.md)** - 系统架构（关注数据流和边界）
4. **[04_NAPI_API](04_NAPI_API.md)** - N-API 接口（关注参数校验）
5. **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构（快速定位代码）
6. **[05_Inner_API](05_Inner_API.md)** - 内部接口（调用链追踪）

**预计时间**: 3-4 小时
**详细路线**: 查看 [SUMMARY.md](SUMMARY.md#安全研究路线)

---

## 快速链接

### 按主题分类

| 主题 | 文档 | 适合人群 |
|------|------|---------|
| **概览** | [00_Overview](00_Overview.md) | 🟢 新人 |
| **定位** | [01_Project_Position](01_Project_Position.md) | 🟢 新人 |
| **结构** | [02_Directory_Structure](02_Directory_Structure.md) | 🟢 新人 |
| **架构** | [03_Architecture](03_Architecture.md) | 🟢 新人 / 🔴 安全 |
| **N-API** | [04_NAPI_API](04_NAPI_API.md) | 🟢 新人 / 🔴 安全 |
| **内部 API** | [05_Inner_API](05_Inner_API.md) | 🟢 新人 |
| **攻击面** | [05_AttackSurface](05_AttackSurface.md) | 🔴 安全 ⭐ |
| **构建** | [06_GN_Targets](06_GN_Targets.md), [07_Build_Artifacts](07_Build_Artifacts.md) | 🟢 新人 |
| **安全** | [08_Security_Review](08_Security_Review.md) | 🔴 安全 |
| **调试** | [09_Troubleshooting](09_Troubleshooting.md) | 🟢 新人 |

### 附录文档

| 附录 | 文档 |
|------|------|
| **调用链** | [appendix/Callgraphs.md](appendix/Callgraphs.md) |
| **配置开关** | [appendix/Config_Flags.md](appendix/Config_Flags.md) |

---

## 覆盖范围

### 已覆盖

- ✅ 项目概览与定位
- ✅ 目录结构与模块职责
- ✅ 系统架构与数据流
- ✅ 对外 N-API 接口（含 API 清单表）
- ✅ 内部 API 接口
- ✅ GN 构建目标与依赖关系
- ✅ 编译产物说明
- ✅ 攻击面分析（新增）
- ✅ 安全风险评审（7 条可被利用点）

### 未覆盖

- ⚠️ 测试相关代码（test/ 目录）
- ⚠️ 第三方依赖实现细节
- ⚠️ OpenHarmony 基础库内部实现

---

## 文档约定

- 所有代码引用使用 **代码证据** 标注（文件路径 + 行号）
- 不引用测试代码作为业务证据
- 使用 `TODO(需确认)` 标注需要进一步验证的内容
- 术语统一使用 OpenHarmony 官方术语

---

## 如何使用本 Wiki

### 阅读建议

1. **新人开发者**: 按照推荐顺序从 [00_Overview](00_Overview.md) 开始阅读
2. **安全研究员**: 先阅读 [05_AttackSurface](05_AttackSurface.md) 和 [08_Security_Review](08_Security_Review.md)
3. **开发新功能**: 先阅读 [01_Project_Position](01_Project_Position.md) 确认功能边界
4. **调试问题**: 参考 [09_Troubleshooting](09_Troubleshooting.md) 的故障排查方法
5. **安全开发**: 阅读 [08_Security_Review](08_Security_Review.md) 了解安全最佳实践

### 维护指南

#### 代码变更后

1. 更新对应模块的 Wiki 文档
2. 更新 `wiki/_work/NOTES.md` 中的发现
3. 如有新增接口，更新 [04_NAPI_API](04_NAPI_API.md)
4. 如有架构变更，更新 [03_Architecture](03_Architecture.md)
5. 如有安全相关变更，更新 [05_AttackSurface](05_AttackSurface.md) 和 [08_Security_Review](08_Security_Review.md)

#### 建议更新时机

- 每次 API 变更后
- 每次架构调整后
- 每次安全评审后
- 每次 GN 配置调整后

---

## 代码证据标准

### 证据要求

所有关键结论必须能在仓库内找到直接证据：
- 文件路径（必要时含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

### 示例

**好**:
```
电池电量查询接口:
- 文件: frameworks/napi/src/battery_info.cpp
- 行号: 37-46
- 符号: BatterySOC()
- 代码: int32_t capacity = g_battClient.GetCapacity();
```

**不好**:
```
电池信息查询功能:
- 功能: 查询电池信息
```

---

## 版本信息

| 项目 | 版本 | 说明 |
|------|------|------|
| battery_manager | 3.1 | bundle.json 中定义的版本 |

---

## 参考资源

### 官方文档

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [电源管理子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/电源管理子系统.md)
- [N-API 开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/napi)

### 相关仓库

- [powermgr_power_manager](https://gitee.com/openharmony/powermgr_power_manager)
- [powermgr_battery_statistics](https://gitee.com/openharmony/powermgr_battery_statistics)
- [powermgr_battery_lite](https://gitee.com/openharmony/powermgr_battery_lite)

---

## 反馈与贡献

如有文档问题或建议，请通过以下方式反馈：

- 项目 Issue: [Gitee](https://gitee.com/openharmony/powermgr_battery_manager/issues)
- 提交 PR: 参考 [Contributing Guide](https://gitee.com/openharmony/docs/blob/master/zh-cn/contribute/贡献方式.md)

---

**返回**: [00_Overview](00_Overview.md)
