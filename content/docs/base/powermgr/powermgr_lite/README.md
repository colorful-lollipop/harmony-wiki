# powermgr_lite Wiki

> 本 Wiki 为 OpenHarmony powermgr_lite 项目提供全面的工程文档,帮助开发者快速理解项目架构、API、构建和安全机制。

---

## 文档覆盖范围

### 已覆盖内容
- [x] 项目概览与核心能力
- [x] 目录结构与模块职责
- [x] 系统架构 (组件图、数据流、线程模型)
- [x] 对外 API (C API + JavaScript JSI 接口)
- [x] 内部 API 与模块接口
- [x] GN 构建系统
- [x] 编译产物与安装路径
- [x] 安全风险评审
- [x] 常见问题与定位路径

### 未覆盖内容 (TODO)
- [ ] 性能基准测试数据
- [ ] 功耗统计接口详细说明
- [ ] 与其他 powermgr 组件的集成 (power_manager, battery_manager, etc.)

---

## 快速导航

| 文档 | 描述 | 适用人群 |
|------|------|---------|
| [00_Overview](00_Overview.md) | 项目概览、核心能力、关键概念 | 所有人 |
| [01_Project_Boundaries](01_Project_Boundaries.md) | 项目定位、边界、运行环境 | 新人入门 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块职责 | 架构理解 |
| [03_Architecture](03_Architecture.md) | 架构说明: 组件图、数据流、线程模型 | 架构设计 |
| [04_NAPI_JS_API](04_NAPI_JS_API.md) | 对外 API: C API + JSI 绑定 | 应用开发者 |
| [05_Internal_API](05_Internal_API.md) | 内部 API: 模块接口、依赖方向 | 模块开发者 |
| [06_GN_Targets](06_GN_Targets.md) | GN 目标梳理 | 构建工程师 |
| [07_Build_Artifacts](07_Build_Artifacts.md) | 编译产物与加载关系 | 部署工程师 |
| [08_Security_Assessment](08_Security_Assessment.md) | 安全风险评审与可利用点 | 安全审计 |
| [09_Troubleshooting](09_Troubleshooting.md) | 常见构建/运行/调试问题 | 问题定位 |

---

## 文档更新方式

### 随代码更新
本 Wiki 应随代码变更同步更新,维护流程如下:

1. **API 变更**: 更新 [04_NAPI_JS_API.md](04_NAPI_JS_API.md) 和 [05_Internal_API.md](05_Internal_API.md)
2. **架构变更**: 更新 [02_Directory_Structure.md](02_Directory_Structure.md) 和 [03_Architecture.md](03_Architecture.md)
3. **构建变更**: 更新 [06_GN_Targets.md](06_GN_Targets.md) 和 [07_Build_Artifacts.md](07_Build_Artifacts.md)
4. **安全修复**: 更新 [08_Security_Assessment.md](08_Security_Assessment.md)

### 更新检查清单
- [ ] 所有代码引用包含文件路径
- [ ] 关键结论包含行号
- [ ] 术语在所有文档中保持一致
- [ ] 相关文档间链接正确
- [ ] 无测试目录内容引用

---

## 文档元数据

| 项目 | 值 |
|------|-----|
| 仓库名 | powermgr_lite |
| 子系统 | powermgr |
| 版本 | 3.1 |
| 系统类型 | mini (LiteOS-M), small (LiteOS-A) |
| 语言 | C (主要), C++ (屏保组件) |
| 文档生成时间 | 2026-02-06 |

---

## 相关仓库

- [powermgr_power_manager](https://gitee.com/openharmony/powermgr_power_manager) - 完整电源管理
- [powermgr_battery_manager](https://gitee.com/openharmony/powermgr_battery_manager) - 电池管理
- [powermgr_battery_lite](https://gitee.com/openharmony/powermgr_battery_lite) - 轻量电池管理
- [powermgr_thermal_manager](https://gitee.com/openharmony/powermgr_thermal_manager) - 热管理

---

## 贡献指南

如发现文档错误或有改进建议,请:
1. 更新对应的 Markdown 文件
2. 确保包含代码证据 (文件路径 + 行号)
3. 更新 `wiki/_work/NOTES.md` 中的发现记录
4. 检查 `wiki/SUMMARY.md` 链接完整性
