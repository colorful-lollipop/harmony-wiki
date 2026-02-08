# 关键调用链

## 概述

本文档梳理 ability_runtime 中的关键调用链路，帮助理解代码执行流程。

## Ability 启动流程

### 1. JS 层启动

```
用户调用
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  JS/TS 应用层                                                    │
│  abilityManager.startAbility(want)                               │
│  └── 实现：frameworks/js/napi/ability_manager/ability_manager_module.cpp  │
└─────────────────────────────────────────────────────────────────┘
    │
    │ N-API 调用
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  Native 绑定层                                                    │
│  NAPI_StartAbility(env, callback)                                │
│  └── 实现：frameworks/js/napi/ability_manager/ability_manager_impl.cpp │
└─────────────────────────────────────────────────────────────────┘
    │
    │ IPC 调用
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  客户端代理层                                                     │
│  AbilityManagerClient::StartAbility()                           │
│  └── 实现：services/abilitymgr/src/ability_manager_client.cpp     │
└─────────────────────────────────────────────────────────────────┘
    │
    │ Binder IPC
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  服务端存根层                                                     │
│  AbilityManagerStub::StartAbility()                             │
│  └── 实现：services/abilitymgr/src/ability_manager_stub.cpp      │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 分发
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  业务处理层                                                       │
│  AbilityManagerService::StartAbility()                          │
│  └── 实现：services/abilitymgr/src/ability_manager_service.cpp   │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 生命周期调度
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  生命周期处理                                                     │
│  LifecycleDeal::PerformLifecycleTransition()                     │
│  └── 实现：services/abilitymgr/src/lifecycle_deal/               │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 应用调度
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  应用调度层                                                       │
│  AppScheduler::ScheduleAbilityTransaction()                      │
│  └── 实现：services/appmgr/src/app_scheduler.cpp                 │
└─────────────────────────────────────────────────────────────────┘
```

### 完整调用链（源码位置）

| 步骤 | 组件 | 源文件 |
|------|------|--------|
| 1 | JS API | `frameworks/js/napi/ability_manager/ability_manager_module.cpp` |
| 2 | N-API | `frameworks/js/napi/ability_manager/ability_manager_impl.cpp` |
| 3 | 客户端 | `services/abilitymgr/src/ability_manager_client.cpp` |
| 4 | IPC Stub | `services/abilitymgr/src/ability_manager_stub.cpp` |
| 5 | 主服务 | `services/abilitymgr/src/ability_manager_service.cpp` |
| 6 | 生命周期 | `services/abilitymgr/src/lifecycle_deal/lifecycle_deal.cpp` |
| 7 | 应用调度 | `services/appmgr/src/app_scheduler.cpp` |

## ConnectAbility 流程

```
客户端调用 connectAbility()
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  N-API 层                                                        │
│  NAPIAbilityConnection::Connect()                                │
│  └── 存储回调到 maps_                                             │
└─────────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  连接管理                                                         │
│  AbilityConnectManager::Connect()                               │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 回调处理
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  连接回调                                                         │
│  IAbilityConnection::OnAbilityConnectDone()                      │
│  └── 通知客户端连接成功                                            │
└─────────────────────────────────────────────────────────────────┘
```

## 数据流图

### Want 参数传递

```
JS 层
  │
  │ want 对象
  ▼
N-API 层（Want Unmarshalling）
  │
  │ Parcel 序列化
  ▼
IPC 层（MessageParcel）
  │
  │ Binder 驱动
  ▼
AMS 服务端
  │
  │ Want 反序列化
  ▼
业务处理
```

### 生命周期状态流转

```
onCreate()
  │
  ▼
AbilityThread::OnCreate()
  │
  ▼
LifecycleDeal::NotifyLifecycleTransition()
  │
  ▼
onStart()
  │
  ▼
onForeground()
  │
  ▼
onBackground()
  │
  ▼
onStop()
```

## 权限校验流程

```
IPC 请求
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  权限校验入口                                                     │
│  AbilityManagerStub::CheckPermission()                           │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 获取调用者信息
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  调用者身份验证                                                   │
│  IPCSkeleton::GetCallingTokenID()                               │
│  IPCSkeleton::GetCallingUid()                                    │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 权限检查
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  权限验证核心                                                     │
│  PermissionVerification::VerifyCallingPermission()              │
│  └── 验证：权限名、TokenID、用户ID                                │
└─────────────────────────────────────────────────────────────────┘
    │
    │ 结果
    ▼
允许/拒绝操作
```

## 相关文档

- [架构说明](03_Architecture.md)
- [安全风险评审](08_Security_Review.md)
