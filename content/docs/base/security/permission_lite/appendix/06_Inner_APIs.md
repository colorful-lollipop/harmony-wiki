# 内部 API（Inner APIs）

## 概述

本文档描述 Permission Lite 的内部 API，供系统服务调用。这些 API 不对第三方应用暴露，通过 SAMGR 或 IPC 间接使用。

---

## PMS 内部 API

### 头文件

```
services/pms/include/pms.h
services/pms/include/pms_inner.h
services/pms/include/perm_operate.h
```

### API 清单

#### 1. SaveOrUpdatePermissions

```c
int SaveOrUpdatePermissions(const char *identifier, PermissionTrans permissions[], int permNum, enum IsUpdate isUpdate);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用标识（包名） |
| permissions | PermissionTrans[] | 权限数组 |
| permNum | int | 权限数量 |
| isUpdate | enum IsUpdate | FIRST_INSTALL / UPDATE |

**返回值**：0=成功，非0=失败

**调用场景**：应用安装时保存权限

**代码证据**：`services/pms/include/pms.h:27`

---

#### 2. DeletePermissions

```c
int DeletePermissions(const char *identifier);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用标识（包名） |

**返回值**：0=成功，非0=失败

**调用场景**：应用卸载时删除权限

**代码证据**：`services/pms/include/pms.h:29`

---

#### 3. IsPermissionValid

```c
int IsPermissionValid(const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| permissionName | const char * | 权限名称 |

**返回值**：1=有效，0=无效

**代码证据**：`services/pms/include/pms.h:31`

---

#### 4. IsPermissionRestricted

```c
int IsPermissionRestricted(const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| permissionName | const char * | 权限名称 |

**返回值**：1=受限，0=不受限

**代码证据**：`services/pms/include/pms.h:33`

---

#### 5. QueryAppCapabilities

```c
int QueryAppCapabilities(const char *identifier, unsigned int **caps, unsigned int *capNum);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用标识 |
| caps | unsigned int ** | 输出：能力数组 |
| capNum | unsigned int * | 输出：能力数量 |

**返回值**：0=成功，非0=失败

**代码证据**：`services/pms/include/pms.h:35`

---

#### 6. LoadPermissions

```c
int LoadPermissions(const char *identifier, int uid);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用标识 |
| uid | int | 用户 ID |

**返回值**：0=成功，非0=失败

**说明**：加载应用权限到内存

**代码证据**：`services/pms/include/pms.h:37`

---

#### 7. UnLoadPermissions

```c
int UnLoadPermissions(int uid);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| uid | int | 用户 ID |

**返回值**：0=成功，非0=失败

**说明**：从内存卸载权限

**代码证据**：`services/pms/include/pms.h:39`

---

#### 8. CheckPermissionStat

```c
int CheckPermissionStat(int uid, const char *permissionName);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| uid | int | 用户 ID |
| permissionName | const char * | 权限名称 |

**返回值**：1=已授予，0=未授予

**代码证据**：`services/pms/include/pms.h:41`

---

#### 9. QueryPermissionString

```c
char *QueryPermissionString(const char *identifier, int *errCode);
```

| 参数 | 类型 | 说明 |
|------|------|------|
| identifier | const char * | 应用标识 |
| errCode | int * | 输出：错误码 |

**返回值**：权限 JSON 字符串（调用方释放）

**代码证据**：`services/pms/include/pms.h:43`

---

## IPC 认证内部 API

### 头文件

```
services/ipc_auth/include/ipc_auth_lite.h
services/ipc_auth/include/policy_define.h
```

### 数据结构

#### RegParams（注册参数）

```c
typedef struct {
    const char *serviceName;    // 服务名
    const char *featureName;    // 特性名
    int uid;                    // UID
} RegParams;
```

#### AuthParams（认证参数）

```c
typedef struct {
    int callerUid;              // 调用方 UID
    const char *targetService;   // 目标服务名
} AuthParams;
```

#### PolicyTrans（策略传输）

```c
typedef struct {
    const char *featureName;     // 特性名
    FixedPolicy fixed;           // 固定策略
    RangePolicy range;           // 范围策略
} PolicyTrans;
```

#### FixedPolicy（固定策略）

```c
typedef struct {
    PolicyType type;            // FIXED/RANGE/BUNDLENAME
    int fixedUid[FIXED_UID_MAX]; // 固定 UID 列表
} FixedPolicy;
```

#### RangePolicy（范围策略）

```c
typedef struct {
    int uidMin;                  // UID 最小值
    int uidMax;                  // UID 最大值
} RangePolicy;
```

---

## 服务注册 API

### 头文件

```
services/pms_base/include/permission_service.h
```

### Feature 注册

```c
// SAMGR Feature 注册
const char PERMISSION_SERVICE[] = "permissionms";
const char PERM_INNER[] = "PermInnerFeature";

// 注册函数
static void Init()
{
    SamgrLite *sm = SAMGR_GetInstance();
    sm->RegisterFeature(PERMISSION_SERVICE, reinterpret_cast<Feature *>(PermissionService::GetInstance()));
    sm->RegisterFeatureApi(PERMISSION_SERVICE, PERM_INNER, GetPermissionFeatureApi());
}
APP_FEATURE_INIT(Init);
```

**代码证据**：`services/pms_base/src/permission_service.c`

---

## HAL 层 API

### 头文件

```
services/pms/include/hals/hal_pms.h
```

### API 清单

| 函数 | 描述 |
|------|------|
| `HalPmsInit()` | 初始化 HAL 层 |
| `HalPmsReadPermission()` | 读取权限 |
| `HalPmsWritePermission()` | 写入权限 |
| `HalPmsRemovePermission()` | 删除权限 |

**说明**：HAL 层封装底层存储操作，屏蔽硬件差异

---

## 接口稳定性标注

### 稳定接口（Stable）

以下接口为系统服务间调用，稳定性高：

| API | 稳定性 | 调用方 |
|-----|--------|--------|
| `CheckPermission()` | Stable | 系统服务 |
| `GrantPermission()` | Stable | 系统服务 |
| `GetCommunicationStrategy()` | Stable | SAMGR |

### 实验接口（Experimental）

以下接口为内部使用，可能变更：

| API | 稳定性 | 说明 |
|-----|--------|------|
| `QueryPermissionString()` | Experimental | 内部调试用 |
| `LoadPermissions()` | Experimental | 内部缓存管理 |

---

## 相关文档

- 架构说明 → `02_Architecture.md`
- API 接口 → `03_APIs.md`
- 构建配置 → `04_Build.md`
- 调用链图谱 → `appendix/07_Callgraphs.md`
