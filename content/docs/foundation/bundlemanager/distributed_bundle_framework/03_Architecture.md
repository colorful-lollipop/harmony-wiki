# 系统架构

---

## 目的

本文档说明 DBMS 服务的架构设计，包括组件关系、数据流、线程模型和关键时序。

---

## 适用范围

- ✅ 整体架构设计
- ✅ 组件关系图
- ✅ 数据流说明
- ✅ 线程模型
- ✅ 关键时序图

---

## 架构概览

### 分层架构

```
┌────────────────────────────────────────────────────────────────┐
│                        应用层 (JavaScript/ArkTS)          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ distributedBundle                                 │    │
│  │ bundle.distributedBundleManager                    │    │
│  │ - getRemoteAbilityInfo()                         │    │
│  │ - getRemoteAbilityInfos()                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────┐
│                   N-API 绑定层                        │
│  ┌───────────────────────────────────────────────────┐      │
│  │ native_module.cpp (JS 接口绑定）            │      │
│  │ distributed_bundle_mgr.cpp                        │      │
│  │ distributed_bundle.cpp                         │      │
│  │ - 参数解析（ParseElementName）                  │      │
│  │ - 类型转换（napi_create_object）             │      │
│  │ - 异步调度（napi_create_async_work）          │      │
│  └───────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────┐
│               IPC 代理层 (DistributedBmsProxy)         │
│  ┌───────────────────────────────────────────────────┐      │
│  │ SendRequest() - 发送 IPC 请求               │      │
│  │ MessageParcel/MessageParcel 序列化               │      │
│  └───────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
                          │ IPC (HIDL/Binder)
                          ▼
┌────────────────────────────────────────────────────────────────┐
│          系统服务层 (DistributedBms SA = 402)        │
│  ┌───────────────────────────────────────────────────┐      │
│  │ DistributedBmsHost (IPC 处理）            │      │
│  │ OnRemoteRequest() - 分发请求               │      │
│  └───────────────────────────────────────────────────┘      │
│                        │                               │
│                        ▼                               │
│  ┌───────────────────────────────────────────┐      │
│  │ DistributedBms (业务逻辑）        │      │
│  │ GetRemoteAbilityInfo()                   │      │
│  │ - VerifyCallingPermission()             │      │
│  │ - VerifyCallingPermissionOrAclCheck() │      │
│  │ - CheckAclData()                     │      │
│  │ - Query Bundle Manager SA             │      │
│  └───────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────┐
│                 外部系统服务（跨系统调用）                 │
│  ┌──────────────────────────┐  ┌───────────────────┐  │
│  │ Bundle Manager      │  │ Device Manager     │  │
│  │ SA = 401              │  │ Service            │  │
│  └──────────────────────────┘  └───────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 核心组件

### 1. JS API 层

**组件**: N-API 绑定层

**职责**:
- JS 参数解析和类型转换
- 异步工作调度
- 错误处理和结果返回

**文件**:
- `interfaces/kits/js/distributedBundle/native_module.cpp`
- `interfaces/kits/js/distributebundlemgr/native_module.cpp`
- `interfaces/kits/js/distributedBundle/distributed_bundle.cpp`
- `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp`

**证据**: `interfaces/kits/js/distributedBundle/native_module.cpp:31-43`

### 2. IPC 层

**组件**: Proxy 和 Stub

#### Proxy（客户端）
**类**: DistributedBmsProxy
**职责**: 将本地调用转换为 IPC 消息，发送到远程服务

**文件**: `interfaces/inner_api/src/distributed_bms_proxy.cpp`

**证据**: `interfaces/inner_api/include/distributed_bms_proxy.h:27-32`

#### Stub（服务端）
**类**: DistributedBmsHost
**职责**: 接收 IPC 消息，分发到对应的处理函数

**文件**: `services/dbms/src/distributed_bms_host.cpp`

**证据**: `services/dbms/include/distributed_bms_host.h:26-39`

### 3. 服务层

**组件**: DistributedBms

**职责**: 实现业务逻辑，包括：
- 权限验证
- ACL 检查
- 跨设备查询
- 数据缓存

**文件**: `services/dbms/src/distributed_bms.cpp`

**证据**: `services/dbms/include/distributed_bms.h:33-168`

### 4. 辅助服务

| 服务 | 用途 | SA ID |
|------|------|--------|
| Bundle Manager | 查询 Bundle 信息 | 401 |
| Device Manager | 设备发现和管理 | - |
| Account Manager | 账号信息 | - |

---

## 数据流

### 典型查询流程

```
1. 应用调用 JS API
   distributedBundle.getRemoteAbilityInfo(elementName, locale)

2. N-API 参数解析
   ParseElementName() → {deviceId, bundleName, moduleName, abilityName}

3. 异步工作创建
   napi_create_async_work() → 异步执行线程

4. IPC 代理调用
   DistributedBmsProxy::GetRemoteAbilityInfo()
   SendRequest(GET_REMOTE_ABILITY_INFO, data, reply)

5. IPC 消息传输
   MessageParcel → IPC (Binder/HIDL) → DBMS SA

6. SA 接收和处理
   DistributedBmsHost::OnRemoteRequest()
   → HandleGetRemoteAbilityInfo()

7. 服务业务逻辑
   DistributedBms::GetRemoteAbilityInfo()
   - VerifyCallingPermission()
   - VerifyCallingPermissionOrAclCheck()
   - Query Bundle Manager SA

8. 跨设备查询（如需要）
   DeviceManager::CheckSystemAbility()
   → 远程设备 DBMS SA → 获取 RemoteAbilityInfo

9. 结果返回
   RemoteAbilityInfo → MessageParcel → IPC → DistributedBmsProxy

10. 结果转换
   ConvertRemoteAbilityInfo() → napi_create_object()

11. 回调/Promise 完成
   napi_resolve_deferred() 或 napi_call_function()
```

---

## 线程模型

### N-API 线程

**主线程**:
- JS 执行
- N-API 参数解析
- 对象构造
- 回调执行

**工作线程**:
- `napi_create_async_work()` 创建的异步工作
- 执行实际的 IPC 调用
- 避免阻塞主线程

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:366-401`

### IPC 线程

**DBMS 服务线程**:
- SA 运行在独立的进程中（d-bms 进程）
- 通过 Binder/HIDL IPC 与客户端通信
- 线程池处理并发请求

**跨进程 IPC**:
- 客户端进程 → DBMS 进程（同步/异步）
- DBMS 进程 → Bundle Manager SA 进程
- DBMS 进程 → Device Manager SA 进程

**证据**: `services/dbms/sa_profile/distributedbms.cfg:11-16`

---

## IPC 接口

### 接口描述符

```cpp
DECLARE_INTERFACE_DESCRIPTOR(u"ohos.appexecfwk.IDistributedbms");
```

**证据**: `interfaces/inner_api/include/distributed_bms_interface.h:33`

### 命令码枚举

| 命令码 | 值 | 方法 |
|--------|-----|------|
| GET_REMOTE_ABILITY_INFO | 0 | 获取远程 Ability 信息 |
| GET_REMOTE_ABILITY_INFOS | 1 | 批量获取远程 Ability 信息 |
| GET_ABILITY_INFO | 2 | 获取 Ability 信息 |
| GET_ABILITY_INFOS | 3 | 批量获取 Ability 信息 |
| GET_ABILITY_INFO_WITH_LOCALE | 6 | 获取 Ability 信息（带本地化）|
| GET_ABILITY_INFOS_WITH_LOCALE | 7 | 批量获取 Ability 信息（带本地化）|
| GET_DISTRIBUTED_BUNDLE_INFO | 8 | 获取分布式 Bundle 信息 |
| GET_DISTRIBUTED_BUNDLE_NAME | 9 | 获取分布式 Bundle 名称 |

**证据**: `interfaces/inner_api/include/distributed_bundle_ipc_interface_code.h`

---

## 关键时序

### 时序 1：查询本地 Ability 信息

```
应用                  N-API层              Proxy层                DBMS SA               Bundle SA
  │                      │                      │                      │                    │
  ├─ getRemoteAbilityInfo ─┼── ParseElementName ──┼── SendRequest ───────┼────── OnStart        │
  │                      │                      │                      │                    │
  │                      │                      │                      │                    │
  │                      │                      │          VerifyCallingPermission                     │
  │                      │                      │                      │                    │
  │                      │                      │                      │                    │
  │                      │                      │                      │                    │
  │                      │                      ├─ QueryRemoteAbility ──────────────────────────┤
  │                      │                      │                      │                    │
  │                      │                      │                      │
  │                      │                      │                      │
  │                      │                      │                      ├─ GetBundleInfo ──────┤
  │                      │                      │                      │                    │
  │                      │                      │                      │                    │
  │                      │                      │                      │
  │                      ◄─────────────────────────── ReplyMessage ────┼── napi_resolve    │
  │                      │                      │                      │                    │
  │                      │                      │              ConvertRemoteAbilityInfo              │
  │                      │                      │                      │                    │
  │                      │                      │                      │                    │
  │                      ◄── callback/Promise resolve ────────────┼─────────────────────┘
  │                      │                      │                      │
```

### 时序 2：查询跨设备 Ability 信息

```
应用                  N-API层              Proxy层                DBMS SA               Device Manager      远程DBMS
  │                      │                      │                      │                     │              │
  ├─ getRemoteAbilityInfo ─┼── ParseElementName ──┼── SendRequest ───────┼────── CheckAclData ───────────────────┤
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │   VerifyCallingPermission                │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      ├─ CheckSystemAbility ──────────────────┼───────────────┤
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │  Remote GetRemoteAbilityInfo ◄──┼───────┤
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │                     ◄─────────── ReplyMessage ────┼── napi_resolve  │
  │                      │                      │                      │                     │              │
  │                      │                      │                      │                     │              │
  │                      │                      │              ConvertRemoteAbilityInfo             │
  │                      │                      │                      │                     │              │
  │                      │                      │                     │              │
  │                      │                      │                     │              │
  │                      ◄── callback/Promise resolve ────────────┼─────────────────────────────────────┘
  │                      │                      │                      │
```

---

## SA 生命周期

### 启动流程

```
系统启动
    │
    ▼
┌─────────────────────────────┐
│ init 进程启动 d-bms      │
│ sa_main distributedbms.cfg  │
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│ DBMS SA 构造              │
│ DistributedBms()           │
│ - 注册 SA ID = 402       │
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│ OnStart() 调用            │
│ - InitDeviceManager()        │
│ - Publish(this)             │
│ - 订阅设备状态事件         │
└─────────────────────────────┘
    │
    ▼
SA 注册完成，等待按需启动
```

**证据**: `services/dbms/src/distributed_bms.cpp:125-165`

### 按需启动

**触发条件**: deviceonline 事件

**配置**: `services/dbms/sa_profile/402.json:11-26`

```json
"start-on-demand": {
    "deviceonline": [
        {
            "name": "deviceonline",
            "value": "on"
        }
    ]
}
```

### 停止流程

```
设备离线
    │
    ▼
触发 stop-on-demand
    │
    ▼
┌─────────────────────────────┐
│ OnStop() 调用            │
│ - 取消事件订阅           │
└─────────────────────────────┘
```

**证据**: `services/dbms/src/distributed_bms.cpp:136-142`

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| SA ID = 402 | services/dbms/src/distributed_bms.cpp:115 |
| SA 进程名 = d-bms | services/dbms/sa_profile/distributedbms.cfg:12 |
| IPC 接口描述符 | interfaces/inner_api/include/distributed_bms_interface.h:33 |
| Proxy 类定义 | interfaces/inner_api/include/distributed_bms_proxy.h:27 |
| Stub 类定义 | services/dbms/include/distributed_bms_host.h:26 |
| N-API 异步工作 | interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:366 |
