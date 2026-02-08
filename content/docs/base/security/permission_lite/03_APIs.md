# 对外 API

## API 概览

Permission Lite 提供三类 API：

| 类型 | 调用方 | 位置 |
|------|--------|------|
| **C 接口** | 系统服务、系统应用 | `interfaces/kits/pms_interface.h` |
| **IPC 认证接口** | SAMGR | `interfaces/innerkits/ipc_auth_interface.h` |
| **JS 接口** | ACE Lite 应用 | `services/js_api/include/perm_module.h` |

---

## C 接口（NDK API）

### 头文件

```
interfaces/kits/pms_interface.h
```

### API 清单

#### 1. CheckPermission

```c
int CheckPermission(int uid, const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| uid | int | 调用方 UID，范围 [0, INT_MAX] |
| permissionName | const char * | 权限名称 |

**返回值**：

| 值 | 含义 |
|----|------|
| 1 | 有权限 |
| 0 | 无权限 |

**代码证据**：`interfaces/kits/pms_interface.h:63`

#### 2. CheckSelfPermission

```c
int CheckSelfPermission(const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| permissionName | const char * | 权限名称 |

**返回值**：

| 值 | 含义 |
|----|------|
| 1 | 调用者有权限 |
| 0 | 调用者无权限 |

**代码证据**：`interfaces/kits/pms_interface.h:75`

#### 3. QueryPermission

```c
int QueryPermission(const char *identifier, PermissionSaved **permissions, int *permNum);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用包名 |
| permissions | PermissionSaved ** | 输出：权限数组指针 |
| permNum | int * | 输出：权限数量 |

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 错误码 | 失败（见 PmsErrorCode） |

**注意**：调用方需释放 `permissions` 指向的内存

**代码证据**：`interfaces/kits/pms_interface.h:91`

#### 4. GrantPermission

```c
int GrantPermission(const char *identifier, const char *permName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用包名 |
| permName | const char * | 权限名称 |

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 错误码 | 失败 |

**代码证据**：`interfaces/kits/pms_interface.h:105`

#### 5. RevokePermission

```c
int RevokePermission(const char *identifier, const char *permName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用包名 |
| permName | const char * | 权限名称 |

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 错误码 | 失败 |

**代码证据**：`interfaces/kits/pms_interface.h:119`

#### 6. GrantRuntimePermission

```c
int GrantRuntimePermission(int uid, const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| uid | int | 用户 ID，范围 [0, INT_MAX] |
| permissionName | const char * | 权限名称 |

**说明**：运行时授予敏感权限

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 错误码 | 失败 |

**代码证据**：`interfaces/kits/pms_interface.h:152`

#### 7. RevokeRuntimePermission

```c
int RevokeRuntimePermission(int uid, const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| uid | int | 用户 ID，范围 [0, INT_MAX] |
| permissionName | const char * | 权限名称 |

**说明**：运行时回收敏感权限

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 错误码 | 失败 |

**代码证据**：`interfaces/kits/pms_interface.h:169`

#### 8. UpdatePermissionFlags

```c
int UpdatePermissionFlags(const char *identifier, const char *permissionName, const int flags);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用包名 |
| permissionName | const char * | 权限名称 |
| flags | int | 权限标志 |

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 错误码 | 失败 |

**代码证据**：`interfaces/kits/pms_interface.h:135`

---

## IPC 认证接口

### 头文件

```
interfaces/innerkits/ipc_auth_interface.h
```

### API 清单

#### 1. GetCommunicationStrategy

```c
int GetCommunicationStrategy(RegParams params, PolicyTrans **policies, unsigned int *policyNum);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| params | RegParams | 注册参数（服务信息） |
| policies | PolicyTrans ** | 输出：策略数组 |
| policyNum | unsigned int * | 输出：策略数量 |

**调用方**：仅限 SAMGR

**返回值**：

| 值 | 含义 |
|----|------|
| 0 | 成功 |
| 非 0 | 失败 |

**代码证据**：`services/ipc_auth/include/ipc_auth.h:27`

#### 2. IsCommunicationAllowed

```c
int IsCommunicationAllowed(AuthParams params);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| params | AuthParams | 认证参数（调用方信息） |

**调用方**：仅限 SAMGR

**返回值**：

| 值 | 含义 |
|----|------|
| 1 | 允许通信 |
| 0 | 拒绝通信 |

**代码证据**：`services/ipc_auth/include/ipc_auth.h:29`

---

## JS 接口（ACE Lite JSI）

### 头文件

```
services/js_api/include/perm_module.h
```

### API 清单

#### check

```cpp
static JSIValue CheckSelfPerm(const JSIValue thisVal, const JSIValue* args, uint8_t argsNum);
```

**JS 调用**：`check(permissionName)`

| 参数 | JS 类型 | 说明 |
|------|---------|------|
| args[0] | string | 权限名称 |

**返回值**：JS boolean

| 值 | 含义 |
|----|------|
| true | 有权限 |
| false | 无权限 |

**代码证据**：`services/js_api/include/perm_module.h:27`

**模块注册**：

```cpp
JSI::SetModuleAPI(exports, "check", PermModule::CheckSelfPerm);
```

---

## 类型定义

### PermissionSaved

```c
typedef struct {
    char name[PERM_NAME_LEN];      // 权限名
    char desc[PERM_DESC_LEN];       // 描述
    enum GrantTime when;            // 授予时机
    enum GrantType type;            // 授予类型
    int flags;                      // 标志位
} PermissionSaved;
```

### GrantTime

| 值 | 含义 |
|----|------|
| INUSE | 使用时授予 |
| ALWAYS | 始终授予 |

### GrantType

| 值 | 含义 |
|----|------|
| USER_GRANT | 用户授予 |
| SYSTEM_GRANT | 系统授予 |

---

## 错误码

错误码定义位置：`interfaces/kits/pms_types.h:117-62`

| 错误码 | 宏定义 | 含义 |
|--------|--------|------|
| 0 | `PERM_ERRORCODE_SUCCESS` | 成功 |
| 10 | `PERM_ERRORCODE_INVALID_PARAMS` | 无效参数 |
| 11 | `PERM_ERRORCODE_INVALID_PERMNAME` | 无效权限名 |
| 12 | `PERM_ERRORCODE_MALLOC_FAIL` | 内存分配失败 |
| 13 | `PERM_ERRORCODE_OPENFD_FAIL` | 打开文件描述符失败 |
| 14 | `PERM_ERRORCODE_READFD_FAIL` | 读取文件描述符失败 |
| 15 | `PERM_ERRORCODE_WRITEFD_FAIL` | 写入文件描述符失败 |
| 16 | `PERM_ERRORCODE_JSONPARSE_FAIL` | JSON 解析失败 |
| 17 | `PERM_ERRORCODE_COPY_ERROR` | 字符串拷贝失败 |
| 18 | `PERM_ERRORCODE_FIELD_TOO_LONG` | 字段过长 |
| 19 | `PERM_ERRORCODE_PERM_NOT_EXIST` | 权限不存在 |
| 20 | `PERM_ERRORCODE_UNLINK_ERROR` | 删除权限文件失败 |
| 21 | `PERM_ERRORCODE_FILE_NOT_EXIST` | 文件不存在 |
| 22 | `PERM_ERRORCODE_MEMSET_FAIL` | memset 失败 |
| 23 | `PERM_ERRORCODE_STAT_FAIL` | stat 失败 |
| 24 | `PERM_ERRORCODE_PATH_INVALID` | 无效路径 |
| 25 | `PERM_ERRORCODE_TOO_MUCH_PERM` | 权限过多 |
| 26 | `PERM_ERRORCODE_TASKID_NOT_EXIST` | 进程 ID 不存在 |
| 27 | `PERM_ERRORCODE_PERM_NUM_ERROR` | 权限数量异常 |
| 28 | `PERM_ERRORCODE_GENERATE_UDID_FAILED` | 生成 UDID 失败 |
| 29 | `PERM_ERRORCODE_SPRINTFS_FAIL` | sprintf_s 失败 |

---

## 使用示例

### C 接口调用示例

```c
#include "pms_interface.h"

int CheckAndInstall(const char *hapPath)
{
    // 检查安装权限
    int ret = CheckPermission(0, "ohos.permission.INSTALL_BUNDLE");
    if (ret != 1) {
        return -1;  // 无权限
    }
    return 0;  // 允许安装
}
```

**代码来源**：参考 `README.md` 中的示例代码

### IPC 认证策略配置

```c
// policy_preset.h
FeaturePolicy bmsFeature[] = {
    {
        "BmsFeature",
        {
            {
                .type = FIXED,
                .fixedUid = {2, 3, 8}
            },
            {
                .type = RANGE,
                .uidMin = 100,
                .uidMax = __INT_MAX__,
            },
        }
    },
};
```

**代码来源**：`services/ipc_auth/include/policy_preset.h`

---

## 调用链

### 权限校验调用链

```
App/Service
    │
    ▼
┌───────────────────┐
│  C 接口 (pms_)    │  CheckPermission()
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│   pms_client      │  IPC 调用
└─────────┬─────────┘
          │ IPC
          ▼
┌───────────────────┐
│   pms_server      │  权限查找
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│   权限存储        │  读取权限文件
└───────────────────┘
```

### IPC 认证调用链

```
Caller Process
    │
    ▼ IPC 请求
┌───────────────────┐
│      SAMGR       │  拦截 IPC
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│    ipc_auth       │  IsCommunicationAllowed()
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  策略配置 (preset)│  查询访问策略
└───────────────────┘
```

---

## 相关文档

- 项目概览 → `01_Overview.md`
- 架构说明 → `02_Architecture.md`
- 构建配置 → `04_Build.md`
- 安全评审 → `05_Security.md`
