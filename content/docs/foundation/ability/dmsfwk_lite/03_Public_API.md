# 03_Public_API - 对外接口文档

## 1. 接口概述

dmsfwk_lite 通过 **SAMGR（System Ability Manager）** 框架对外提供服务，采用 C/C++ 接口形式。

**注意**: 本组件**不直接提供 N-API/JS 接口**，JS 层调用通过 `ability_lite` 框架中转。

### 1.1 服务定位

| 属性 | 值 |
|------|-----|
| 服务名 | `dtbschedsrv` |
| Feature 名 | `dmslite` |
| 接口类型 | IUnknown（SAMGR 标准接口） |
| 头文件 | `interfaces/innerkits/dmsfwk_interface.h` |

### 1.2 获取接口方式

```c
#include "dmsfwk_interface.h"
#include "samgr_lite.h"

// 获取 DmsProxy 接口
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(
    DISTRIBUTED_SCHEDULE_SERVICE,  // "dtbschedsrv"
    DMSLITE_FEATURE                // "dmslite"
);

DmsProxy *dmsProxy = NULL;
int32_t errCode = iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&dmsProxy);
if (errCode == EC_SUCCESS) {
    // 使用 dmsProxy->StartRemoteAbility()
}
```

## 2. 数据结构

### 2.1 错误码定义 (DmsLiteCommonErrorCode)

**位置**: `interfaces/innerkits/dmsfwk_interface.h:31-55`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `DMS_EC_SUCCESS` | 0 | 成功 |
| `DMS_EC_START_ABILITY_SYNC_SUCCESS` | 1 | 同步启动成功 |
| `DMS_EC_START_ABILITY_ASYNC_SUCCESS` | 2 | 异步启动成功 |
| `DMS_EC_PARSE_TLV_FAILURE` | 3 | TLV 解析失败 |
| `DMS_EC_UNKNOWN_COMMAND_ID` | 4 | 未知命令 ID |
| `DMS_EC_GET_BMS_FAILURE` | 5 | 获取 BMS 失败 |
| `DMS_EC_GET_BUNDLEINFO_FAILURE` | 6 | 获取 BundleInfo 失败 |
| `DMS_EC_CHECK_PERMISSION_FAILURE` | 7 | 权限检查失败 |
| `DMS_EC_GET_ABILITYMS_FAILURE` | 8 | 获取 AbilityMS 失败 |
| `DMS_EC_REGISTE_IPC_CALLBACK_FAILURE` | 9 | 注册 IPC 回调失败 |
| `DMS_EC_FILL_WANT_FAILURE` | 10 | 填充 Want 失败 |
| `DMS_EC_START_ABILITY_SYNC_FAILURE` | 11 | 同步启动失败 |
| `DMS_EC_START_ABILITY_ASYNC_FAILURE` | 12 | 异步启动失败 |
| `DMS_EC_FAILURE` | 13 | 通用失败 |
| `DMS_EC_INVALID_PARAMETER` | 14 | 无效参数 |

**接收端错误码**:

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `DMS_REC_UNKNOWN_COMMAND_ID` | 29360300 | 接收端：未知命令 ID |
| `DMS_REC_PARSER_TLV_FAIL` | 29360301 | 接收端：TLV 解析失败 |
| `DMS_REC_PERMISSION_DENIED` | 29360302 | 接收端：权限拒绝 |
| `DMS_REC_OPEN_SESSION_FAIL` | 29360303 | 接收端：打开会话失败 |
| `DMS_REC_DEVICE_BUSY` | 29360304 | 接收端：设备忙 |
| `DMS_REC_PACKET_MARSHALL_FAIL` | 29360305 | 接收端：报文打包失败 |
| `DMS_REC_PACKET_UNMARSHALL_FAIL` | 29360306 | 接收端：报文解包失败 |
| `DMS_REC_FREEINSTALL_FAIL` | 29360307 | 接收端：自由安装失败 |

### 2.2 回调接口 (IDmsListener)

**位置**: `interfaces/innerkits/dmsfwk_interface.h:57-59`

```c
typedef struct {
    void (*OnResultCallback)(const void *data, int32_t ret);
} IDmsListener;
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `OnResultCallback` | 函数指针 | 结果回调函数 |

**回调参数**:
- `data`: 返回数据（当前未使用，为 NULL）
- `ret`: 结果码（见错误码定义）

### 2.3 调用方信息 (CallerInfo)

**位置**: `interfaces/innerkits/dmsfwk_interface.h:61-64`

```c
typedef struct {
    int32_t uid;
    char* bundleName;
} CallerInfo;
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `uid` | int32_t | 调用方应用 UID |
| `bundleName` | char* | 调用方包名（可选） |

### 2.4 DmsProxy 接口

**位置**: `interfaces/innerkits/dmsfwk_interface.h:66-70`

```c
typedef struct {
    INHERIT_IUNKNOWN;
    int32_t (*StartRemoteAbility)(const Want *want, const CallerInfo *callerInfo,
        const IDmsListener *callback);
} DmsProxy;
```

| 方法 | 说明 |
|------|------|
| `StartRemoteAbility` | 启动远程 FA |

## 3. API 详情

### 3.1 StartRemoteAbility

**功能**: 向远程设备发送启动 FA 请求。

**原型**:
```c
int32_t (*StartRemoteAbility)(const Want *want, const CallerInfo *callerInfo,
    const IDmsListener *callback);
```

**参数**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `want` | `const Want*` | 启动参数，包含设备 ID、包名、Ability 名等 |
| `callerInfo` | `const CallerInfo*` | 调用方信息（UID、包名） |
| `callback` | `const IDmsListener*` | 结果回调（可为 NULL） |

**返回值**:

| 返回值 | 说明 |
|--------|------|
| `DMS_EC_SUCCESS` (0) | 请求发送成功（异步） |
| `DMS_EC_FAILURE` (13) | 通用失败 |
| `DMS_EC_INVALID_PARAMETER` (14) | 参数无效 |
| 其他 | 见错误码定义 |

**Want 结构要求**:

```c
// want->element 必须设置以下字段：
want->element->deviceId    // 远程设备 ID（网络 ID）
want->element->bundleName  // 被调用方包名
want->element->abilityName // 被调用方 Ability 名

// want->data / want->dataLength 可选，透传数据
```

**调用权限**:
- 仅允许 `foundation` (UID=7) 或 `shell` (UID=0/2) 进程调用

**代码位置**: `source/dmslite_famgr.c:45-83`

**调用示例**:

```c
#include "dmsfwk_interface.h"
#include "samgr_lite.h"
#include "want.h"

// 构造 Want
Want want = {0};
ElementName element = {0};
SetElementDeviceID(&element, "remote_device_network_id");
SetElementBundleName(&element, "com.example.remoteapp");
SetElementAbilityName(&element, "MainAbility");
SetWantElement(&want, element);

// 构造 CallerInfo
CallerInfo callerInfo = {
    .uid = getuid(),
    .bundleName = "com.example.localapp"
};

// 定义回调
void OnResult(const void *data, int32_t ret) {
    printf("Remote start result: %d\n", ret);
}

IDmsListener listener = {
    .OnResultCallback = OnResult
};

// 获取接口并调用
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(
    DISTRIBUTED_SCHEDULE_SERVICE, DMSLITE_FEATURE);
DmsProxy *proxy = NULL;
iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&proxy);

int32_t result = proxy->StartRemoteAbility(&want, &callerInfo, &listener);
```

## 4. 使用约束

### 4.1 前置条件

| 条件 | 说明 |
|------|------|
| 组网成功 | 本地和远程设备必须在同一 LAN，分布式组网成功 |
| 应用已安装 | 远程设备必须已安装目标 FA |
| 签名一致 | 调用方和被调用方必须有匹配的签名 |
| UID 权限 | 调用方必须是 foundation(7) 或 shell(0/2) |

### 4.2 并发限制

- 当前仅支持**单会话**，同时只能处理一个远程启动请求
- `IsDmsBusy()` 会检查当前是否有未完成的请求
- 请求超时时间为 **60 秒**

### 4.3 数据限制

| 项目 | 限制 |
|------|------|
| 消息大小 | 最大 1024 字节 |
| 包名字符串 | 无明确限制，但受消息大小限制 |
| Ability 名字符串 | 同上 |
| 透传数据 | 受消息大小限制 |

## 5. 典型使用场景

### 5.1 应用层调用流程

```
应用 (JS/Java)
    │
    ▼
ability_lite (AAFwk)
    │ 识别 FLAG_ABILITYSLICE_MULTI_DEVICE
    ▼
dmsfwk_lite
    │ 构建 TLV 消息
    ▼
DSoftBus
    │ 网络传输
    ▼
远程设备 dmsfwk_lite
    │ 解析消息、权限检查
    ▼
远程设备 ability_lite
    │ 启动 FA
    ▼
远程应用
```

### 5.2 应用层代码示例

```java
// Java 代码示例（来自 README.md）
import ohos.aafwk.ability.Ability;
import ohos.aafwk.content.Want;
import ohos.bundle.ElementName;

// 创建 Want 实例
Want want = new Want();
ElementName name = new ElementName(
    remote_device_id,           // 远程设备 ID
    "ohos.dms.remote_bundle_name",  // 目标包名
    "remote_ability_name"       // 目标 Ability 名
);
want.setElement(name);

// 设置分布式标志（必须）
want.setFlags(Want.FLAG_ABILITYSLICE_MULTI_DEVICE);

// 启动远程 FA
startAbility(want);
```

## 6. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构设计（含时序图）
- [04_Internal_API](04_Internal_API.md) - 内部接口
- [06_Security](06_Security.md) - 安全机制
