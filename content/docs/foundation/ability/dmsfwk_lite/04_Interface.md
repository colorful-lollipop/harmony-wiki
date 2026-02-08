# 对外接口文档

> dmsfwk_lite 提供的 C 接口定义、参数说明、调用方式及错误码参考。

## 一、接口概览

dmsfwk_lite 对外暴露的接口主要分为以下几类：

| 接口类型 | 接口名称 | 说明 |
|---------|---------|------|
| **主功能接口** | `StartRemoteAbility()` | 启动远端设备的 FA |
| **内部接口** | `StartRemoteAbilityInner()` | 内部启动接口（供 SAMgr 调用） |
| **回调定义** | `IDmsListener` | 操作结果回调 |
| **调用者信息** | `CallerInfo` | 调用方身份信息 |
| **权限校验** | `CheckRemotePermission()` | 远程权限检查 |

**证据来源**：`interfaces/innerkits/dmsfwk_interface.h:28-70` 接口定义。

---

## 二、主功能接口

### 2.1 StartRemoteAbility

启动远端设备的 Feature Ability。

#### 函数原型

```cpp
// interfaces/innerkits/dmsfwk_interface.h:68-69
int32_t StartRemoteAbility(const Want *want, const CallerInfo *callerInfo,
    const IDmsListener *callback);
```

#### 参数说明

| 参数名 | 类型 | 说明 | 必填 |
|-------|------|------|------|
| **want** | `const Want *` | 启动意图，包含目标设备 ID、包名、Ability 名 | 是 |
| **callerInfo** | `const CallerInfo *` | 调用者信息，包含 uid 和 bundleName | 是 |
| **callback** | `const IDmsListener *` | 结果回调，通知启动结果 | 否（可为空） |

#### 返回值

| 返回值 | 说明 |
|-------|------|
| `DMS_EC_SUCCESS` | 启动请求发送成功 |
| `DMS_EC_INVALID_PARAMETER` | 参数无效（空指针或字段缺失） |
| `DMS_EC_FAILURE` | 通用失败 |
| 其他错误码 | 见错误码表 |

#### 调用示例

```cpp
#include "dmsfwk_interface.h"

// 定义回调
void OnStartAbilityResult(const void *data, int32_t ret) {
    if (ret == DMS_EC_SUCCESS) {
        printf("远程启动成功\n");
    } else {
        printf("远程启动失败，错误码：%d\n", ret);
    }
}

// 调用接口
IDmsListener listener = {
    .OnResultCallback = OnStartAbilityResult
};

int32_t ret = StartRemoteAbility(&want, &callerInfo, &listener);
if (ret != DMS_EC_SUCCESS) {
    printf("请求发送失败，错误码：%d\n", ret);
}
```

---

### 2.2 StartRemoteAbilityInner

内部启动接口，供 SAMgr 消息处理调用。

#### 函数原型

```cpp
// include/dmslite_famgr.h:50-51
int32_t StartRemoteAbilityInner(const Want *want, const CallerInfo *callerInfo,
    const IDmsListener *callback);
```

#### 与 StartRemoteAbility 的区别

| 特性 | StartRemoteAbility | StartRemoteAbilityInner |
|------|-------------------|------------------------|
| 调用者 | 上层应用/框架 | SAMgr 消息处理 |
| 权限校验 | 调用方负责 | 已由调用方完成 |
| 消息封装 | 完整封装 | 直接发送 |

**证据来源**：`source/dmslite_famgr.c:45-83` 实现逻辑。

---

## 三、回调与数据结构

### 3.1 IDmsListener

操作结果回调接口。

#### 定义

```cpp
// interfaces/innerkits/dmsfwk_interface.h:57-59
typedef struct {
    void (*OnResultCallback)(const void *data, int32_t ret);
} IDmsListener;
```

#### 字段说明

| 字段 | 类型 | 说明 |
|-----|------|------|
| **OnResultCallback** | `void (*)(const void *data, int32_t ret)` | 结果回调函数指针 |

**证据来源**：`interfaces/innerkits/dmsfwk_interface.h:57-59`。

### 3.2 CallerInfo

调用者身份信息。

#### 定义

```cpp
// interfaces/innerkits/dmsfwk_interface.h:61-64
typedef struct {
    int32_t uid;
    char* bundleName;
} CallerInfo;
```

#### 字段说明

| 字段 | 类型 | 说明 | 必填 |
|-----|------|------|------|
| **uid** | `int32_t` | 调用者用户 ID | 是 |
| **bundleName** | `char*` | 调用者包名 | 是 |

**证据来源**：`interfaces/innerkits/dmsfwk_interface.h:61-64`。

### 3.3 RequestData

内部请求数据结构。

#### 定义

```cpp
// include/dmslite_famgr.h:32-36
typedef struct {
    Want *want;
    CallerInfo *callerInfo;
    IDmsListener *callback;
} RequestData;
```

---

## 四、错误码定义

### 4.1 通用错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| 0 | `DMS_EC_SUCCESS` | 成功 |
| 1 | `DMS_EC_START_ABILITY_SYNC_SUCCESS` | 同步启动成功 |
| 2 | `DMS_EC_START_ABILITY_ASYNC_SUCCESS` | 异步启动成功 |
| 3 | `DMS_EC_PARSE_TLV_FAILURE` | TLV 解析失败 |
| 4 | `DMS_EC_UNKNOWN_COMMAND_ID` | 未知命令 ID |
| 5 | `DMS_EC_GET_BMS_FAILURE` | 获取 BMS 失败 |
| 6 | `DMS_EC_GET_BUNDLEINFO_FAILURE` | 获取 BundleInfo 失败 |
| 7 | `DMS_EC_CHECK_PERMISSION_FAILURE` | 权限校验失败 |
| 8 | `DMS_EC_GET_ABILITYMS_FAILURE` | 获取 AbilityMS 失败 |
| 9 | `DMS_EC_REGISTE_IPC_CALLBACK_FAILURE` | 注册 IPC 回调失败 |
| 10 | `DMS_EC_FILL_WANT_FAILURE` | 填充 Want 失败 |
| 11 | `DMS_EC_START_ABILITY_SYNC_FAILURE` | 同步启动失败 |
| 12 | `DMS_EC_START_ABILITY_ASYNC_FAILURE` | 异步启动失败 |
| 13 | `DMS_EC_FAILURE` | 通用失败 |
| 14 | `DMS_EC_INVALID_PARAMETER` | 无效参数 |

**证据来源**：`interfaces/innerkits/dmsfwk_interface.h:31-55`。

### 4.2 结果码（从设备返回）

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| 29360300 | `DMS_REC_UNKNOWN_COMMAND_ID` | 未知命令 |
| 29360301 | `DMS_REC_PARSER_TLV_FAIL` | TLV 解析失败 |
| 29360302 | `DMS_REC_PERMISSION_DENIED` | 权限拒绝 |
| 29360303 | `DMS_REC_OPEN_SESSION_FAIL` | 会话打开失败 |
| 29360304 | `DMS_REC_DEVICE_BUSY` | 设备忙 |
| 29360305 | `DMS_REC_PACKET_MARSHALL_FAIL` | 数据包封装失败 |
| 29360306 | `DMS_REC_PACKET_UNMARSHALL_FAIL` | 数据包解封失败 |
| 29360307 | `DMS_REC_FREEINSTALL_FAIL` | 自由安装失败 |

### 4.3 TLV 解析错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| 0 | `DMS_TLV_SUCCESS` | 成功 |
| 1 | `DMS_TLV_ERR_NO_MEM` | 内存不足 |
| 2 | `DMS_TLV_ERR_PARAM` | 参数错误 |
| 3 | `DMS_TLV_ERR_LEN` | 长度错误 |
| 4 | `DMS_TLV_ERR_OUT_OF_ORDER` | 节点顺序错误 |
| 5 | `DMS_TLV_ERR_BAD_NODE_NUM` | 节点数量错误 |
| 6 | `DMS_TLV_ERR_UNKNOWN_TYPE` | 未知类型 |
| 7 | `DMS_TLV_ERR_BAD_SOURCE` | 错误的数据源 |

**证据来源**：`include/dmslite_tlv_common.h:39-48`。

---

## 五、Want 结构说明

### 5.1 ElementName

目标设备与应用信息。

```cpp
// 来自 want_lite 接口
typedef struct ElementName {
    char* deviceId;      // 目标设备 ID
    char* bundleName;     // 目标包名
    char* abilityName;    // 目标 Ability 名
} ElementName;
```

### 5.2 Want

启动意图完整结构。

```cpp
// 来自 want_lite 接口
typedef struct Want {
    ElementName* element;      // 目标信息
    char* action;              // 动作
    char* entity;              // 实体
    void* data;                // 携带数据
    int32_t dataLength;        // 数据长度
    int32_t flags;             // 标志位
} Want;
```

### 5.3 关键标志位

| 标志位 | 说明 |
|-------|------|
| `Want.FLAG_ABILITYSLICE_MULTI_DEVICE` | 使能分布式启动（必须设置） |

**证据来源**：`README_zh.md:104`「want.setFlags(Want.FLAG_ABILITYSLICE_MULTI_DEVICE)」。

---

## 六、调用流程与时序

### 6.1 完整调用链

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  调用方   │───>│ dmslite  │───>│ famgr    │───>│ session  │───>│DSoftBus  │
│          │    │ feature  │    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     │                │                │                │                │
     │ 1. StartRemoteAbility          │                │                │
     │                │                │                │                │
     │         2. 参数校验             │                │                │
     │                │                │                │                │
     │         3. 消息封装             │                │                │
     │                │                │                │                │
     │                │        4. SendDmsMessage        │                │
     │                │                │                │                │
     │                │                │      5. 会话建立                │
     │                │                │                │                │
     │                │                │      6. 消息发送                │
     │                │                │                │                │
     │                │                │                │   ┌──────────┐│
     │                │                │                │──>│ 从设备    ││
     │                │                │                │   │ dmslite  ││
     │                │                │                │   └──────────┘│
     │                │                │                │               │
     │                │                │                │   7. 消息解析 │
     │                │                │                │               │
     │                │                │                │   8. FA 启动 │
     │                │                │                │               │
     │                │                │       9. 回调通知               │
     │<───────────────│<───────────────│<───────────────│<──────────────┘
                     │                │                │
                10. InvokeCallback   │                │
```

### 6.2 消息封装格式

dmsfwk_lite 使用 TLV 格式封装跨设备消息：

| 字段 | 类型 | 说明 |
|-----|------|------|
| **COMMAND_ID** | `uint16_t` | 命令 ID（DMS_MSG_CMD_START_FA = 0x01） |
| **DMS_VERSION** | `uint16_t` | 协议版本（当前为 200） |
| **CALLEE_BUNDLE_NAME** | `string` | 被调用方包名 |
| **CALLEE_ABILITY_NAME** | `string` | 被调用方 Ability 名 |
| **CALLER_SIGNATURE** | `string` | 调用方签名（用于权限校验） |
| **CALLER_PAYLOAD** | `raw data` | 可选的携带数据 |

**证据来源**：`source/dmslite_famgr.c:114-135` 消息封装逻辑。

---

## 七、权限与安全

### 7.1 调用权限要求

| 权限 | 说明 | 来源 |
|------|------|------|
| 无特殊权限 | 不需要用户授权 | - |
| 签名校验 | 自动校验调用方与目标方签名是否一致 | `source/dmslite_permission.c:111-114` |

### 7.2 签名校验流程

```
1. 获取调用方信息（uid、bundleName）
2. 根据 uid 查询调用方签名
3. 获取目标包签名
4. 比对两个签名
5. 签名一致 → 允许操作
   签名不一致 → 返回 DMS_EC_CHECK_PERMISSION_FAILURE
```

**证据来源**：`source/dmslite_permission.c:103-115` 签名比对逻辑。

---

## 八、常见问题

### Q1：返回 DMS_EC_INVALID_PARAMETER

**原因**：参数校验失败。

**排查步骤**：
1. 检查 Want 是否为 NULL
2. 检查 Want->element 是否为 NULL
3. 检查 element->deviceId、bundleName、abilityName 是否有效

**代码位置**：`source/dmslite_famgr.c:88-90` 参数校验。

### Q2：返回 DMS_EC_CHECK_PERMISSION_FAILURE

**原因**：签名校验失败。

**排查步骤**：
1. 确认调用方应用已安装
2. 确认目标应用已安装在从设备
3. 确认两个应用使用相同签名证书签名

### Q3：返回 DMS_EC_PARSE_TLV_FAILURE

**原因**：TLV 消息解析失败。

**排查步骤**：
1. 检查消息格式是否正确
2. 检查 TLV 节点顺序是否正确（必须递增）

---

## 九、相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计 |
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 |
| [06_SecurityReview.md](./06_SecurityReview.md) | 安全风险评估 |

---

*文档版本：v1.0*  
*最后更新：2026-02-07*
