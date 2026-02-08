# 架构说明

## 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      分布式服务 (Distributed Service)           │
│                 调用 DATASL_* API 进行跨设备安全校验               │
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    数据传输管控模块 (dataclassification)          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    dev_slinfo_mgr.c                       │   │
│  │   DATASL_OnStart() / DATASL_OnStop()                     │   │
│  │   DATASL_GetHighestSecLevel()  - 同步接口                 │   │
│  │   DATASL_GetHighestSecLevelAsync() - 异步接口             │   │
│  └──────────────────────────┬──────────────────────────────┘   │
│                             │                                  │
│              ┌──────────────┼──────────────┐                   │
│              ▼              ▼              ▼                   │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐│
│  │  dev_slinfo_adpt │ │ dev_slinfo_list  │ │   dev_slinfo_log ││
│  │       .c        │ │       .c         │ │       .h         ││
│  │  SDK 适配层      │ │  异步回调链表    │ │   日志宏定义     ││
│  │  dlopen SDK     │ │  pthread_mutex   │ │                  ││
│  └──────────────────┘ └──────────────────┘ └──────────────────┘│
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   libdslm_sdk.z.so                               │
│               (设备安全等级 SDK - 动态加载)                        │
│   RequestDeviceSecurityInfo / GetDeviceSecurityLevelValue        │
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   设备安全等级管理模块                             │
│              (device_security_level subsystem)                   │
└─────────────────────────────────────────────────────────────────┘
```

## 模块职责

### dev_slinfo_mgr.c
**主入口模块**，负责：
- 导出 `DATASL_*` 系列公共 API
- 参数校验与错误处理
- 调用适配层获取设备安全等级

> 证据：`frameworks/datatransmitmgr/dev_slinfo_mgr.c`

### dev_slinfo_adpt.c
**SDK 适配层**，负责：
- 通过 `dlopen` 动态加载 `libdslm_sdk.z.so`
- 解析 SDK 函数指针（`dlsym`）
- 调用设备安全等级查询接口
- 处理同步/异步查询逻辑

> 证据：`frameworks/datatransmitmgr/dev_slinfo_adpt.c:43-101`

### dev_slinfo_list.c
**回调链表管理**，负责：
- 维护异步回调参数链表
- 使用 `pthread_mutex` 保证线程安全
- 回调查找与执行

> 证据：`frameworks/datatransmitmgr/dev_slinfo_list.c`

### dev_slinfo_log.h
**日志基础设施**，定义日志宏：
- `DATA_SEC_LOG_DEBUG`
- `DATA_SEC_LOG_INFO`
- `DATA_SEC_LOG_WARN`
- `DATA_SEC_LOG_ERROR`

> 证据：`interfaces/inner_api/datatransmitmgr/include/dev_slinfo_log.h`

## 数据流

### 同步查询流程

```
1. 调用方传入 UDID + levelInfo 输出参数
2. 参数校验 (udidLen: 1-64)
3. 调用 GetDeviceSecLevelByUdid() 查询设备安全等级
4. 映射设备安全等级 → 数据安全等级
5. 返回结果给调用方
```

> 证据：`dev_slinfo_mgr.c:69-81`

### 异步查询流程

```
1. 调用方传入 UDID + 回调函数
2. 参数校验
3. 将回调注册到链表 (UpdateCallbackListParams)
4. 异步调用 RequestDeviceSecurityInfoAsync
5. SDK 通过 OnApiDeviceSecInfoCallback 回调
6. 链表查找匹配的回调参数
7. 执行原始回调函数
8. 从链表移除已执行的回调
```

> 证据：`dev_slinfo_adpt.c:184-233`

## 线程模型

### 线程安全机制

- **同步 API**：`DATASL_GetHighestSecLevel` 无需加锁
- **异步 API**：`DATASL_GetHighestSecLevelAsync` 使用 `pthread_mutex` 保护链表
- **全局状态**：`g_callbackList`、`g_deviceSecEnv` 在首次初始化时创建

### 关键全局变量

| 变量 | 类型 | 保护方式 |
|-----|------|---------|
| `g_callbackList` | `struct DATASLListParams*` | pthread_mutex |
| `g_deviceSecEnv` | `DeviceSecEnv` | 单次初始化后只读 |

> 证据：`dev_slinfo_adpt.c:25-27`、`dev_slinfo_list.c:22`

## 状态机

```
┌─────────────┐
│   UNINIT    │ ←─────────────────────────┐
└──────┬──────┘                          │
       │ DATASL_OnStart()                │
       ▼                                 │
┌─────────────┐     DATASL_OnStop()      │
│   READY     │ ←────────────────────────┘
└──────┬──────┘
       │
       │ DATASL_GetHighestSecLevel[Async]
       ▼
┌─────────────┐
│   BUSY     │ (可选：表示查询进行中)
└─────────────┘
```

> 证据：`dev_slinfo_mgr.c:46-67`
