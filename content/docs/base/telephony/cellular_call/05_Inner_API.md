# 05 - 内部模块接口

## 目的

本文档描述 Cellular Call 模块**内部各模块之间的接口和依赖关系**，帮助开发者理解模块边界、接口稳定性等级和内部 API 使用方式。

## 适用范围

- **读者对象**：需要修改或扩展模块功能的开发者
- **前置知识**：了解模块结构和架构设计
- **使用场景**：
  - 新增通话功能
  - 修改控制逻辑
  - 排查模块间通信问题

---

## 5.1 模块依赖关系

### 5.1.1 模块依赖图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           模块依赖关系图                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                          ┌─────────────────┐                               │
│                          │  CellularCallService │                            │
│                          │    (Manager)      │                             │
│                          └────────┬────────┘                               │
│                    ┌──────────────┼───────────────┐                        │
│                    ↓              ↓               ↓                         │
│         ┌─────────────────┐ ┌───────────┐ ┌─────────────────┐             │
│         │   CSControl    │ │ IMSControl│ │ SatelliteControl│             │
│         │   (Control)    │ │ (Control) │ │   (Control)    │             │
│         └────────┬────────┘ └─────┬─────┘ └────────┬────────┘             │
│                  │                │                 │                       │
│                  ↓                ↓                 ↓                       │
│         ┌─────────────────┐ ┌───────────┐ ┌─────────────────┐             │
│         │CellularCall    │ │ImsCall   │ │SatelliteCall    │             │
│         │ConnectionCS    │ │Connection │ │Connection       │             │
│         │   (Connection) │ │(Connection)│ │   (Connection) │             │
│         └─────────────────┘ └───────────┘ └─────────────────┘             │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                          公共模块                                      │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌────────────────────────────┐ │  │
│  │  │  BaseRequest │ │ Supplement   │ │ CellularCallSupplement     │ │  │
│  │  │   (请求基类) │ │ Request      │ │   (补充业务)               │ │  │
│  │  └──────────────┘ └──────────────┘ └────────────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.1.2 依赖方向说明

| 依赖方向 | 说明 |
|----------|------|
| Manager → Control | Manager 层调用 Control 层实现通话控制 |
| Control → Connection | Control 层调用 Connection 层与 RIL 通信 |
| Control ← Common | Control 层使用 Common 层的公共工具 |
| 所有层 → Utils | 所有层均可使用 Utils 层的工具函数 |

---

## 5.2 管理层 (Manager) 接口

### 5.2.1 CellularCallService 核心接口

**头文件**: `services/manager/include/cellular_call_service.h`

#### 生命周期接口

| 方法 | 用途 | 证据位置 |
|------|------|----------|
| `OnStart()` | SA 启动 | `cellular_call_service.h:48` |
| `OnStop()` | SA 停止 | `cellular_call_service.h:53` |
| `Dump()` | 服务状态转储 | `cellular_call_service.h:62` |

#### Control 管理接口

| 方法 | 用途 | 证据位置 |
|------|------|----------|
| `GetCsControl(slotId)` | 获取 CS Control | `cellular_call_service.h:596` |
| `GetImsControl(slotId)` | 获取 IMS Control | `cellular_call_service.h:604` |
| `GetSatelliteControl(slotId)` | 获取 Satellite Control | `cellular_call_service.h:613` |
| `SetCsControl(slotId, csControl)` | 设置 CS Control | `cellular_call_service.h:622` |
| `SetImsControl(slotId, imsControl)` | 设置 IMS Control | `cellular_call_service.h:630` |

### 5.2.2 CellularCallStub IPC 接口

**头文件**: `services/manager/include/cellular_call_stub.h`

#### 远程请求处理

| 方法 | 用途 | 证据位置 |
|------|------|----------|
| `OnRemoteRequest()` | IPC 请求分发 | `cellular_call_stub.h` |
| `InitFuncMap()` | 初始化函数映射表 | `cellular_call_stub.h` |

#### 权限检查

**证据位置**: `services/manager/src/cellular_call_stub.cpp:45-50`

```cpp
auto callingUid = IPCSkeleton::GetCallingUid();
if (callingUid != FOUNDATION_UID &&
    !TelephonyPermission::CheckPermission(Permission::CONNECT_CELLULAR_CALL_SERVICE)) {
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

### 5.2.3 CellularCallHandler 事件处理

**头文件**: `services/manager/include/cellular_call_handler.h`

---

## 5.3 控制层 (Control) 接口

### 5.3.1 ControlBase 基类

**头文件**: `services/control/include/control_base.h`

**基类方法**:

| 方法 | 用途 |
|------|------|
| `Dial()` | 拨号（虚函数） |
| `HangUp()` | 挂断（虚函数） |
| `Answer()` | 接听（虚函数） |
| `Reject()` | 拒绝（虚函数） |
| `HoldCall()` | 保持（虚函数） |
| `UnHoldCall()` | 取消保持（虚函数） |

### 5.3.2 CSControl 接口

**头文件**: `services/control/include/cs_control.h`

**继承**: `ControlBase`

**CS 特有方法**:

| 方法 | 用途 | 证据位置 |
|------|------|----------|
| `Dial()` | CS 拨号 | `cs_control.h:49` |
| `HangUp()` | CS 挂断 | `cs_control.h:63` |
| `Answer()` | CS 接听 | `cs_control.h:76` |
| `Reject()` | CS 拒绝 | `cs_control.h:89` |

### 5.3.3 IMSControl 接口

**头文件**: `services/control/include/ims_control.h`

**继承**: `ControlBase`

**IMS 特有方法**:

| 方法 | 用途 |
|------|------|
| `Dial()` | IMS 拨号 |
| `HangUp()` | IMS 挂断 |
| `Reject()` | IMS 拒绝 |
| `Answer()` | IMS 接听 |
| `HoldCall()` | IMS 保持 |
| `UnHoldCall()` | IMS 取消保持 |
| `SwitchCall()` | IMS 切换 |
| `CombineConference()` | 合并会议 |
| `SeparateConference()` | 分离会议 |

---

## 5.4 连接层 (Connection) 接口

### 5.4.1 BaseConnection 基类

**头文件**: `services/connection/include/base_connection.h`

**基类方法**:

| 方法 | 用途 |
|------|------|
| `SendRequest()` | 发送请求（虚函数） |
| `ProcessResponse()` | 处理响应（虚函数） |

### 5.4.2 Connection 模块分类

| 模块 | 说明 | 头文件 |
|------|------|--------|
| **CellularCallConnectionCS** | CS 通话连接 | `cellular_call_connection_cs.h` |
| **CellularCallConnectionIMS** | IMS 通话连接 | `cellular_call_connection_ims.h` |
| **CellularCallConnectionSatellite** | 卫星通话连接 | `cellular_call_connection_satellite.h` |

---

## 5.5 补充业务接口

### 5.5.1 CellularCallSupplement

**头文件**: `services/utils/include/cellular_call_supplement.h`

**方法**:

| 方法 | 用途 |
|------|------|
| `SetCallTransfer()` | 设置呼叫转移 |
| `GetCallTransfer()` | 查询呼叫转移 |
| `SetCallWaiting()` | 设置呼叫等待 |
| `GetCallWaiting()` | 查询呼叫等待 |
| `SetCallRestriction()` | 设置呼叫限制 |
| `GetCallRestriction()` | 查询呼叫限制 |

---

## 5.6 稳定性标注

### 5.6.1 接口稳定性等级

| 等级 | 说明 | 示例 |
|------|------|------|
| **稳定** | 公开 API，外部模块使用 | IPC 接口、Inner API |
| **不稳定** | 内部使用，可能变更 | 模块间内部调用 |
| **实验性** | 新功能，可能变更 | 新增接口 |

### 5.6.2 稳定性判断依据

| 接口类型 | 稳定性 | 判断依据 |
|----------|--------|----------|
| **interfaces/** | ✅ 稳定 | Inner API，官方接口 |
| **services/manager/** | ✅ 稳定 | IPC 实现，官方接口 |
| **services/control/** | ⚠️ 不稳定 | 内部实现，可能变更 |
| **services/connection/** | ⚠️ 不稳定 | 内部实现，可能变更 |
| **services/utils/** | ⚠️ 不稳定 | 工具类，可能变更 |
| **vendor/** | ❌ 禁止使用 | 样例代码，仅供参考 |

---

## 5.7 跨模块调用示例

### 5.7.1 Manager 调用 Control 示例

```cpp
// 位置: services/manager/src/cellular_call_service.cpp

// 获取对应域的 Control
std::shared_ptr<CSControl> csControl = GetCsControl(slotId);
if (csControl != nullptr) {
    int32_t result = csControl->Dial(callInfo, isEcc);
    return result;
}
```

### 5.7.2 Control 调用 Connection 示例

```cpp
// 位置: services/control/src/cs_control.cpp

// 发起拨号请求
auto connection = GetConnection(slotId);
if (connection != nullptr) {
    int32_t result = connection->SendRequest(request);
    return result;
}
```

---

## 相关跳转

| 目标 | 链接 |
|------|------|
| 接口规范 | [04_Interfaces.md](./04_Interfaces.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| 构建配置 | [06_GN_Build.md](./06_GN_Build.md) |
| 安全评审 | [07_Security_Review.md](./07_Security_Review.md) |

---

*最后更新：2026-02-06*
