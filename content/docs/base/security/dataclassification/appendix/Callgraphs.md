# 关键调用链

## 同步查询调用链

```
DATASL_GetHighestSecLevel()
    │
    ├── 验证参数 (queryParams, levelInfo 非 NULL)
    │   └── dev_slinfo_mgr.c:73-75
    │
    └── GetHighestSecLevelByUdid()
        │
        ├── 验证 udidLen 范围 (1-64)
        │   └── dev_slinfo_mgr.c:31
        │
        └── GetDeviceSecLevelByUdid()
            │
            ├── 验证 SDK 句柄有效性
            │   └── dev_slinfo_adpt.c:137-150
            │
            ├── 调用 requestDeviceSecurityInfo()
            │   └── dev_slinfo_adpt.c:165
            │
            ├── 调用 getDeviceSecurityLevelValue()
            │   └── dev_slinfo_adpt.c:172
            │
            └── 释放 DeviceSecurityInfo
                └── dev_slinfo_adpt.c:168-169, 175-176
```

**Mermaid 图示**：

```mermaid
flowchart TD
    A[DATASL_GetHighestSecLevel] --> B[参数校验]
    B --> C[GetHighestSecLevelByUdid]
    C --> D[udidLen 范围校验]
    D --> E[GetDeviceSecLevelByUdid]
    E --> F{SDK 句柄有效?}
    F -->|否| G[返回错误]
    F -->|是| H[requestDeviceSecurityInfo]
    H --> I[getDeviceSecurityLevelValue]
    I --> J[释放资源]
    J --> K[映射设备等级到数据等级]
    K --> L[返回结果]
```

> 证据来源：`dev_slinfo_mgr.c:69-81`、`dev_slinfo_adpt.c:134-182`

---

## 异步查询调用链

```
DATASL_GetHighestSecLevelAsync()
    │
    ├── 验证参数 (queryParams, callback 非 NULL, udidLen 范围)
    │   └── dev_slinfo_mgr.c:103
    │
    └── UpdateCallbackListParams()
        │
        ├── 分配链表节点
        │   └── dev_slinfo_adpt.c:314-318
        │
        ├── 拷贝 UDID 到链表节点
        │   └── dev_slinfo_adpt.c:320-326
        │
        ├── 检查链表长度 (MAX: 128)
        │   └── dev_slinfo_adpt.c:329-332
        │
        └── 插入链表节点
            └── dev_slinfo_adpt.c:335
```

**Mermaid 图示**：

```mermaid
flowchart TD
    A[DATASL_GetHighestSecLevelAsync] --> B[参数校验]
    B --> C[UpdateCallbackListParams]
    C --> D[分配链表节点]
    D --> E[memcpy_s 拷贝 UDID]
    E --> F{链表已满?}
    F -->|是| G[触发最旧回调并移除]
    F -->|否| H[插入链表尾部]
    H --> I[GetDeviceSecLevelByUdidAsync]
    I --> J[返回成功]
```

> 证据来源：`dev_slinfo_mgr.c:98-116`、`dev_slinfo_adpt.c:307-337`

---

## SDK 异步回调调用链

```
OnApiDeviceSecInfoCallback()
    │
    ├── 验证参数 (identify, info 非 NULL)
    │   └── dev_slinfo_adpt.c:187-205
    │
    ├── 调用 getDeviceSecurityLevelValue()
    │   └── dev_slinfo_adpt.c:211
    │
    ├── 映射设备等级到数据等级
    │   └── dev_slinfo_adpt.c:215
    │
    ├── 释放 DeviceSecurityInfo
    │   └── dev_slinfo_adpt.c:217
    │
    ├── 查找匹配的回调
    │   └── LookupCallback()
    │       │
    │       ├── 按 UDID 遍历链表
    │       │   └── dev_slinfo_list.c:125-140
    │       │
    │       └── 找到后移除链表节点
    │           └── dev_slinfo_list.c:132-137
    │
    └── 执行原始回调函数
        └── dev_slinfo_list.c:144
```

**Mermaid 图示**：

```mermaid
flowchart TD
    A[OnApiDeviceSecInfoCallback] --> B[参数校验]
    B --> C[getDeviceSecurityLevelValue]
    C --> D[GetDataSecLevelByDevSecLevel]
    D --> E[freeDeviceSecurityInfo]
    E --> F[LookupCallback]
    F --> G[遍历链表匹配 UDID]
    G --> H{找到匹配?}
    H -->|否| I[直接返回]
    H -->|是| J[移除链表节点]
    J --> K[执行回调函数]
```

> 证据来源：`dev_slinfo_adpt.c:184-233`

---

## 初始化调用链

```
DATASL_OnStart()
    │
    └── StartDevslEnv()
        │
        └── InitDeviceSecEnv()
            │
            ├── dlopen libdslm_sdk.z.so
            │   └── dev_slinfo_adpt.c:45
            │
            ├── dlsym RequestDeviceSecurityInfo
            │   └── dev_slinfo_adpt.c:64-65
            │
            ├── dlsym FreeDeviceSecurityInfo
            │   └── dev_slinfo_adpt.c:72-73
            │
            ├── dlsym GetDeviceSecurityLevelValue
            │   └── dev_slinfo_adpt.c:80-81
            │
            └── dlsym RequestDeviceSecurityInfoAsync
                └── dev_slinfo_adpt.c:88-89
```

**Mermaid 图示**：

```mermaid
flowchart TD
    A[DATASL_OnStart] --> B[StartDevslEnv]
    B --> C[InitDeviceSecEnv]
    C --> D[dlopen libdslm_sdk.z.so]
    D --> E[dlsym 解析 4 个函数指针]
    E --> F{全部解析成功?}
    F -->|否| G[关闭句柄返回错误]
    F -->|是| H[初始化链表和互斥锁]
    H --> I[返回成功]
```

> 证据来源：`dev_slinfo_mgr.c:46-59`、`dev_slinfo_adpt.c:43-102`

---

## 去初始化调用链

```
DATASL_OnStop()
    │
    └── FinishDevslEnv()
        │
        ├── DestroyDeviceSecEnv()
        │   │
        │   ├── memset_s 清零 g_deviceSecEnv
        │   │   └── dev_slinfo_adpt.c:96
        │   │
        │   └── dlclose SDK 句柄
        │       └── dev_slinfo_adpt.c:33-34
        │
        └── DestroyPthreadMutex()
            └── dev_slinfo_list.c:155-157
```

> 证据来源：`dev_slinfo_mgr.c:61-67`、`dev_slinfo_adpt.c:29-41`

---

## 调用关系图

```mermaid
flowchart TB
    subgraph 用户态
        A[分布式服务] --> B[DATASL_GetHighestSecLevel]
        A --> C[DATASL_GetHighestSecLevelAsync]
        A --> D[DATASL_OnStart]
        A --> E[DATASL_OnStop]
    end

    subgraph dataclassification 模块
        B --> F[dev_slinfo_mgr.c]
        C --> F
        D --> F
        E --> F
        F --> G[dev_slinfo_adpt.c]
        F --> H[dev_slinfo_list.c]
    end

    subgraph 设备安全 SDK
        G --> I[libdslm_sdk.z.so]
        I --> J[RequestDeviceSecurityInfo]
        I --> K[GetDeviceSecurityLevelValue]
        I --> L[RequestDeviceSecurityInfoAsync]
    end
```
