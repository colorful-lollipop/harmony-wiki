# 全文导航

本文档为 OpenHarmony Mechbody Controller 的工程 Wiki，提供从新手到专家的完整学习路径。

---

## 快速导航

| 章节 | 描述 | 优先级 |
|------|------|--------|
| [README](README.md) | 项目概述、文档结构、更新方式 | ⭐⭐⭐ |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境 | ⭐⭐⭐ |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型、时序 | ⭐⭐⭐ |
| [03_CodeMap](03_CodeMap.md) | 目录结构、代码地图、功能映射 | ⭐⭐⭐ |
| [03_NAPI_Reference](03_NAPI_Reference.md) | JS API 清单、参数、返回值、调用链 | ⭐⭐⭐ |
| [04_Inner_API](04_Inner_API.md) | 内部模块接口、依赖关系、生命周期 | ⭐⭐ |
| [05_AttackSurface](05_AttackSurface.md) | 攻击面分析、信任边界、攻击路径 | ⭐⭐⭐ |
| [05_Build_Targets](05_Build_Targets.md) | GN targets、编译产物、运行时加载 | ⭐⭐ |
| [06_Security_Review](06_Security_Review.md) | 安全风险、风险点、修复建议 | ⭐⭐⭐ |
| [07_Troubleshooting](07_Troubleshooting.md) | 构建/运行/调试问题与定位 | ⭐ |

---

## 新人阅读路线

### 路线 A：应用开发者（使用 N-API）

```
1. README.md           → 了解文档结构
2. 01_Overview.md     → 项目定位与能力
3. 03_NAPI_Reference.md → 学习 API 使用
4. 07_Troubleshooting.md → 问题排查
```

### 路线 B：系统开发者（扩展服务）

```
1. README.md
2. 01_Overview.md
3. 02_Architecture.md → 系统架构
4. 04_Inner_API.md   → 内部模块
5. 05_Build_Targets.md → 构建配置
```

### 路线 C：安全审计

```
1. README.md
2. 06_Security_Review.md → 安全风险
3. 02_Architecture.md → 信任边界
4. 03_NAPI_Reference.md → API 安全
```

---

## 模块快速索引

### N-API 模块

| API 分类 | 方法 | 文件位置 |
|----------|------|----------|
| 事件监听 | `on`, `off` | js_mech_manager.cpp |
| 设备管理 | `getAttachedMechDevices`, `setUserOperation` | js_mech_manager.cpp |
| 相机追踪 | `setCameraTrackingEnabled`, `getCameraTrackingLayout` | js_mech_manager.cpp |
| 运动控制 | `rotate`, `rotateToEulerAngles`, `rotateBySpeed` | js_mech_manager.cpp |
| 状态查询 | `getCurrentAngles`, `getRotationLimits` | js_mech_manager.cpp |

### 内部模块

| 模块 | 职责 | 关键文件 |
|------|------|----------|
| **Controller** | 高层控制逻辑 | mc_controller_manager.cpp |
| **Connect** | 蓝牙连接管理 | mc_connect_manager.cpp |
| **Motion** | 运动规划 | mc_motion_manager.cpp |
| **Transport** | 协议传输 | mc_send_adapter.cpp |

### 关键入口

| 入口 | 类型 | 路径 |
|------|------|------|
| SystemAbility | 服务入口 | mechbody_controller_service.cpp |
| N-API | JS 绑定 | js_mech_manager.cpp |
| ANI | ETS 绑定 | ani_constructor.cpp |
| 适配器加载 | 动态库 | load_mechbody_adapter.cpp |

---

## 附录

- [附录 A：关键调用链](appendix/Callgraphs.md)
- [附录 B：配置开关](appendix/Config_Flags.md)

---

## 反馈与贡献

如发现文档错误或不清晰，请：

1. 在 `wiki/_work/NOTES.md` 记录问题
2. 修改对应 `.md` 文件
3. 提交 PR 或 Issue
