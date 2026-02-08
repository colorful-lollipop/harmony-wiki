# 文档导航

## 新人阅读路线（推荐顺序）

```
1. 概览 → 2. 目录结构 → 3. N-API 接口 → 4. 内部架构 → 5. GN 构建 → 6. 安全评审
```

## 完整文档列表

### 1. 快速入门

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [README.md](./README.md) | Wiki 首页 | 项目概述、更新方式、相关链接 |
| [index.md](./index.md) | 模块概览 | 项目定位、目录结构、API 示例代码 |
| [01_Overview.md](./01_Overview.md) | 项目概览 | 一句话定义、能力边界、运行环境、快速开始 |

### 2. 架构与设计

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [02_Architecture.md](./02_Architecture.md) | 系统架构 | 组件图、数据流、线程模型、时序图 |
| [03_CodeMap.md](./03_CodeMap.md) | 目录结构与代码地图 | 目录职责说明、核心文件定位、代码导航图 |

### 3. 接口文档

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [04_NAPI_Reference.md](./04_NAPI_Reference.md) | JS/TS API | 完整 API 清单表、注册入口、错误码 |
| [05_InnerAPI.md](./05_InnerAPI.md) | 内部模块接口 | C++ 服务接口、IPC 代码、数据类型 |

### 4. 构建与编译

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [06_Build.md](./06_Build.md) | GN 构建配置 | Targets 清单、依赖关系、产物路径 |

### 5. 安全与权限

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 | 外部输入清单、敏感操作清单、信任边界图 |
| [06_SecurityReview.md](./06_SecurityReview.md) | 安全风险评估 | 输入验证、内存安全、权限与鉴权、并发安全、逻辑漏洞分析 |

### 6. 内部实现

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [08_Internals.md](./08_Internals.md) | 内部实现细节 | 核心类职责、内部 API 契约、资源生命周期 |

---

## 模块索引

| 功能 | 对应文档 | 关键文件 |
|------|---------|---------|
| **JS API** | N-API Reference | `interfaces/kits/js/@ohos.telephony.data.d.ts` |
| **服务实现** | Architecture | `services/src/cellular_data_service.cpp` |
| **状态机** | Architecture / CodeMap | `services/src/state_machine/` |
| **IPC 客户端** | InnerAPI | `frameworks/native/cellular_data_client.cpp` |
| **N-API 绑定** | N-API Reference | `frameworks/js/napi/src/napi_cellular_data.cpp` |
| **项目概览** | Overview | `01_Overview.md` |
| **目录结构** | CodeMap | `03_CodeMap.md` |
| **攻击面分析** | AttackSurface | `05_AttackSurface.md` |
| **安全风险评估** | SecurityReview | `06_SecurityReview.md` |
| **内部实现细节** | Internals | `08_Internals.md` |
| **构建配置** | Build | `BUILD.gn`, `frameworks/native/BUILD.gn` |

---

## 代码证据索引

| 类别 | 文件路径 | 行号 | 说明 |
|------|----------|------|------|
| **SA ID** | `sa_profile/4007.json:5` | SA ID: 4007 |
| **SA 依赖** | `sa_profile/4007.json:9` | 依赖 core_service (SA 4010) |
| **SA 主类** | `services/include/cellular_data_service.h:32` | `CellularDataService` 类定义 |
| **N-API 注册** | `frameworks/js/napi/src/napi_cellular_data.cpp:1515-1528` | `napi_module_register` 调用 |
| **N-API 函数** | `frameworks/js/napi/src/napi_cellular_data.cpp:1481-1512` | 20 个 JS API 注册 |
| **JS 声明** | `interfaces/kits/js/@ohos.telephony.data.d.ts:30-494` | TypeScript 接口定义 |
| **IPC IDL** | `frameworks/native/ICellularDataManager.idl:20-61` | 40 个 IPC 方法定义 |
| **IPC 接口码** | `interfaces/innerkits/cellular_data_ipc_interface_code.h:22-62` | `CellularDataInterfaceCode` 枚举 |
| **状态机类** | `services/include/state_machine/cellular_data_state_machine.h:37-38` | `CellularDataStateMachine` 类定义 |
| **APN 管理器** | `services/include/apn_manager/apn_manager.h` | `ApnManager` 类定义 |
| **主构建文件** | `BUILD.gn:36-159` | `tel_cellular_data` target 定义 |
| **组件配置** | `bundle.json:1-102` | 组件元信息 |
| **权限检查** | `services/src/cellular_data_service.cpp:100-109` | `CheckPermission` 调用 |
