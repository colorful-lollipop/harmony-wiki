# Wiki 导航

本文档提供后台任务管理模块Wiki的全站导航和阅读路线指引。

---

## 快速导航

### 基础入门
| 文档 | 描述 | 推荐度 |
|------|------|--------|
| [首页概览](index.md) | 模块定位、核心能力、运行环境 | ⭐⭐⭐⭐⭐ |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型 | ⭐⭐⭐⭐⭐ |
| [目录结构](01_Directory_Structure.md) | 源码目录组织与模块职责 | ⭐⭐⭐⭐ |

### 接口文档
| 文档 | 描述 | 适用场景 |
|------|------|----------|
| [N-API接口](03_NAPI_Reference.md) | JS/ArkTS对外接口完整参考 | 应用开发 |
| [内部API](04_Inner_API.md) | C++内部接口说明 | 系统开发 |

### 工程实践
| 文档 | 描述 | 适用场景 |
|------|------|----------|
| [GN构建](05_GN_Build.md) | 构建配置、产物说明 | 编译集成 |
| [安全风险](06_Security.md) | 攻击面分析、安全建议 | 安全审计 |
| [常见问题](07_FAQ.md) | 构建/运行/调试问题 | 故障排查 |

### 附录
| 文档 | 描述 |
|------|------|
| [附录-调用链](appendix/Callgraphs.md) | 关键调用链追踪 |
| [附录-配置项](appendix/Config_Flags.md) | 编译配置与Feature Flags |

---

## 按角色阅读路线

### 👤 应用开发者（使用JS/ArkTS接口）

```
首页概览 → N-API接口 → 常见问题
```

重点关注：
- [短时任务接口](03_NAPI_Reference.md#短时任务接口) - requestSuspendDelay/cancelSuspendDelay
- [长时任务接口](03_NAPI_Reference.md#长时任务接口) - startBackgroundRunning/stopBackgroundRunning
- [能效资源接口](03_NAPI_Reference.md#能效资源接口) - applyEfficiencyResources
- [错误码说明](03_NAPI_Reference.md#error-code) - 异常处理

### 👤 系统开发者（对接内部服务）

```
首页概览 → 架构说明 → 内部API → GN构建
```

重点关注：
- [Service架构](02_Architecture.md#service架构) - 三大子服务
- [IPC接口](04_Inner_API.md#ipc接口定义) - 跨进程调用
- [Helper API](04_Inner_API.md#backgroundtaskmgrhelper) - C++接口

### 👤 安全审计人员

```
首页概览 → 架构说明 → 安全风险
```

重点关注：
- [攻击面清单](06_Security.md#攻击面清单)
- [可被利用点](06_Security.md#可被利用点分析)
- [信任边界](06_Security.md#信任边界)

### 👤 构建/集成工程师

```
首页概览 → GN构建 → 目录结构
```

重点关注：
- [Targets清单](05_GN_Build.md#targets清单)
- [编译产物](05_GN_Build.md#编译产物清单)
- [Feature Flags](05_GN_Build.md#feature-flags)

---

## 文档规范

### 术语统一

| 术语 | 英文 | 说明 |
|------|------|------|
| 短时任务 | Transient Task | 延迟挂起机制，限制后台运行时间 |
| 长时任务 | Continuous Task | 持续后台运行，需通知栏提示 |
| 能效资源 | Efficiency Resources | 特权资源申请，如CPU、GPS等 |
| 后台模式 | Background Mode | 长时任务的类型标识 |

### 代码引用格式

- 文件路径：`path/to/file.cpp`
- 行号引用：`path/to/file.cpp:123`
- 函数/类名：`ClassName::MethodName`
- 宏定义：`MACRO_NAME`

### 证据标注

- ✅ 已确认 - 有代码证据支持
- ⚠️ TODO(需确认) - 证据不足需补充
- ❓ 待验证 - 推测性结论

---

## 链接检查清单

- [x] [README.md](README.md) - 本文档存在
- [x] [index.md](index.md) - 首页概览
- [x] [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构
- [x] [02_Architecture.md](02_Architecture.md) - 架构说明
- [x] [03_NAPI_Reference.md](03_NAPI_Reference.md) - N-API接口
- [x] [04_Inner_API.md](04_Inner_API.md) - 内部API
- [x] [05_GN_Build.md](05_GN_Build.md) - GN构建
- [x] [06_Security.md](06_Security.md) - 安全风险
- [x] [07_FAQ.md](07_FAQ.md) - 常见问题
- [x] [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链附录
- [x] [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置项附录
