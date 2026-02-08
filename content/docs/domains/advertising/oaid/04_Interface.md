# OAID 对外接口文档

## 接口总览

| 接口类型 | 接口数量 | 说明 |
|---------|---------|------|
| **N-API** | 2 | JavaScript 应用调用接口 |
| **IPC 接口** | 3 | C++ 层进程间通信接口 |
| **配置文件** | 2 | 服务运行时配置 |

---

## N-API 接口

### 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | `identifier.oaid` |
| **导入方式** | `import identifier from '@ohos.identifier.oaid'` |
| **API 版本** | API 10+ |
| **系统类型** | Standard System |

### getOAID

获取开放匿名设备标识符（OAID）。

#### 函数签名

```typescript
// Promise 方式
function getOAID(): Promise<string>;

// Callback 方式
function getOAID(callback: AsyncCallback<string>): void;
```

#### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| callback | `AsyncCallback<string>` | 否 | 回调函数（可选） |

#### 返回值

| 类型 | 说明 |
|------|------|
| `Promise<string>` | Promise 对象，返回 OAID 字符串 |

#### 返回值格式

```
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

示例：a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

#### 权限要求

| 权限 | 类型 | 说明 |
|------|------|------|
| `ohos.permission.APP_TRACKING_CONSENT` | user_grant | 必须动态申请并获得用户授权 |

#### 使用示例

**Promise 方式**:

```typescript
import { identifier } from '@kit.AdsKit';
import { BusinessError } from '@kit.BasicServicesKit';

async function getOAIDExample(): Promise<void> {
  try {
    const oaid = await identifier.getOAID();
    console.info('获取 OAID 成功:', oaid);
    
    // 检查是否为全 0（用户关闭开关）
    if (oaid === '00000000-0000-0000-0000-000000000000') {
      console.warn('用户已禁用广告追踪');
    }
  } catch (err) {
    const error = err as BusinessError;
    console.error('获取 OAID 失败:', error.code, error.message);
  }
}
```

**Callback 方式**:

```typescript
import { identifier } from '@kit.AdsKit';

identifier.getOAID((err, data) => {
  if (err.code) {
    console.error('获取失败:', err.code, err.message);
    return;
  }
  console.info('OAID:', data);
});
```

#### 错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| 17300001 | 系统内部错误 | 检查服务是否正常，稍后重试 |
| 401 | 参数错误 | 检查调用方式是否正确 |

#### 实现位置

```cpp
// interfaces/kits/js/napi/oaid/src/oaid.cpp:192-220
napi_value GetOAID(napi_env env, napi_callback_info info) {
    // 1. 解析参数
    // 2. 创建异步工作
    // 3. 加入异步队列
}
```

---

### resetOAID

重置开放匿名设备标识符。重置后将生成新的 OAID。

#### 函数签名

```typescript
function resetOAID(): void;
```

#### 参数说明

无参数。

#### 返回值

无返回值。

#### 权限要求

| 要求 | 说明 |
|------|------|
| **系统应用** | 调用应用必须是系统应用 |
| **白名单** | 应用 BundleName 必须在配置白名单中 |

#### 使用示例

```typescript
import { identifier } from '@kit.AdsKit';
import { BusinessError } from '@kit.BasicServicesKit';

function resetOAIDExample(): void {
  try {
    identifier.resetOAID();
    console.info('OAID 重置成功');
  } catch (err) {
    const error = err as BusinessError;
    // 错误码 202: 非系统应用
    // 错误码 17300002: 不在白名单
    console.error('重置失败:', error.code, error.message);
  }
}
```

#### 错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 202 | OAID_ERROR_CODE_NOT_SYSTEM_APP | 非系统应用调用系统 API |
| 17300002 | OAID_ERROR_NOT_IN_TRUST_LIST | 应用不在白名单中 |

#### 白名单配置

编辑 `/etc/advertising/oaid/oaid_service_config.json`：

```json
{
  "resetOAIDBundleName": [
    "com.example.system.app1",
    "com.example.system.app2"
  ]
}
```

#### 实现位置

```cpp
// interfaces/kits/js/napi/oaid/src/oaid.cpp:222-247
napi_value ResetOAID(napi_env env, napi_callback_info info) {
    // 1. 调用 Client 的 ResetOAID
    // 2. 处理错误码
    // 3. 抛出 JS 异常（如有错误）
}
```

---

## IPC 接口

### IOAIDService 接口定义

```cpp
// interfaces/innerkits/include/oaid_service_interface.h:26-46
class IOAIDService : public IRemoteBroker {
public:
    /**
     * Get open advertising id.
     * @return std::string, OAID.
     */
    virtual std::string GetOAID() = 0;

    /**
     * Reset open advertising id.
     * @return int32_t, error code.
     */
    virtual int32_t ResetOAID() = 0;

    /**
     * RegisterObserver
     * @param observer - Remote config observer.
     * @return int32_t, error code.
     */
    virtual int32_t RegisterObserver(const sptr<IRemoteConfigObserver>& observer) = 0;

    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.cloud.oaid.IOAIDService");
};
```

### IPC 命令码

```cpp
// interfaces/innerkits/include/oaid_service_ipc_interface_code.h
enum class OAIDInterfaceCode {
    GET_OAID = 1,                          // 获取 OAID
    RESET_OAID = 2,                        // 重置 OAID
    REGISTER_CONTROL_CONFIG_OBSERVER = 3,  // 注册配置观察者
};
```

### GetOAID (IPC)

#### 调用方式

```cpp
// 客户端调用
std::string oaid = oaidServiceProxy->GetOAID();
```

#### 权限检查

| 检查点 | 位置 | 说明 |
|--------|------|------|
| Client 端检查 | `oaid_service_client.cpp:142` | `CheckPermission(APP_TRACKING_CONSENT)` |
| Server 端检查 | `oaid_service_stub.cpp:172` | `CheckPermission(APP_TRACKING_CONSENT)` |

#### 返回值

| 值 | 说明 |
|----|------|
| OAID 字符串 | 成功获取 |
| 空字符串 | 获取失败（如权限不足）|
| `00000000-0000-0000-0000-000000000000` | 用户关闭开关 |

#### 实现位置

**客户端**:
```cpp
// interfaces/innerkits/src/oaid_service_client.cpp:140-165
std::string OAIDServiceClient::GetOAID() {
    // 1. 检查权限
    if (!CheckPermission(OAID_TRACKING_CONSENT_PERMISSION)) {
        return OAID_ALLZERO_STR;
    }
    // 2. 加载服务
    // 3. 调用 Proxy
    return oaidServiceProxy_->GetOAID();
}
```

**服务端**:
```cpp
// services/oaid_manager/src/oaid_service.cpp:287-293
std::string OAIDService::GetOAID() {
    std::string oaid = GainOAID();
    // 日志脱敏处理
    std::string target = oaid.substr(0, 9).append(OAID_VIRTUAL_STR);
    return oaid;
}
```

---

### ResetOAID (IPC)

#### 调用方式

```cpp
// 客户端调用
int32_t result = oaidServiceProxy->ResetOAID();
```

#### 权限检查

| 检查点 | 位置 | 验证内容 |
|--------|------|----------|
| 白名单检查 | `oaid_service_stub.cpp:197` | `LoadAndCheckOaidTrustList(bundleName)` |
| 系统应用检查 | `oaid_service_stub.cpp:207` | `CheckSystemApp()` |

#### 返回值

| 返回值 | 常量 | 说明 |
|--------|------|------|
| 0 | ERR_OK | 重置成功 |
| 202 | OAID_ERROR_CODE_NOT_SYSTEM_APP | 非系统应用 |
| 17300002 | OAID_ERROR_NOT_IN_TRUST_LIST | 不在白名单 |
| 17300001 | ERR_SYSYTEM_ERROR | 系统错误 |

#### 实现位置

**服务端**:
```cpp
// services/oaid_manager/src/oaid_service.cpp:295-312
int32_t OAIDService::ResetOAID() {
    // 1. 生成新 UUID
    std::string resetOaid = GetUUID();
    if (resetOaid.empty()) {
        return ERR_SYSYTEM_ERROR;
    }
    // 2. 更新内存和数据库
    oaid_ = resetOaid;
    WriteValueToKvStore(OAID_KVSTORE_KEY, resetOaid);
    // 3. 通知观察者
    notifyKit(NOTIFY_RESET_OAID_CODE);
    DelayedSingleton<OaidObserverManager>::GetInstance()->OnUpdateOaid(resetOaid);
    return ERR_OK;
}
```

---

### RegisterObserver (IPC)

#### 功能说明

注册远程配置观察者，用于接收配置变更通知。

#### 调用方式

```cpp
sptr<IRemoteConfigObserver> observer = new MyObserver();
int32_t result = oaidServiceProxy->RegisterObserver(observer);
```

#### 权限检查

| 检查点 | 位置 | 验证内容 |
|--------|------|----------|
| UID 检查 | `oaid_service_stub.cpp:333` | `uid == HA_UID (7508)` |

#### 返回值

| 返回值 | 常量 | 说明 |
|--------|------|------|
| 0 | ERR_OK | 注册成功 |
| 401 | ERR_INVALID_PARAM | UID 错误 |
| 17100002 | ERR_NULL_POINTER | 观察者为空 |

#### 实现位置

```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:330-354
int32_t OAIDServiceStub::HandleRegisterControlConfigObserver(MessageParcel& data, MessageParcel& reply) {
    // 1. 检查 UID
    if (uid != HA_UID) {
        return ERR_INVALID_PARAM;
    }
    // 2. 读取 RemoteObject
    // 3. 注册观察者
    return RegisterObserver(observer);
}
```

---

## 配置文件接口

### 主配置文件

**路径**: `/etc/advertising/oaid/oaid_service_config.json`

**用途**: 服务运行时配置

#### 配置项说明

| 配置项 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `resetOAIDBundleName` | string[] | 否 | 允许重置 OAID 的 BundleName 列表 |
| `providerBundleName` | string | 否 | Ads Provider 的 BundleName |
| `providerAbilityName` | string | 否 | Ads Provider 的 Ability 名称 |
| `providerTokenName` | string | 否 | Ads Provider 的 Token |

#### 配置示例

```json
{
  "resetOAIDBundleName": [
    "com.example.system.app1",
    "com.example.system.app2"
  ],
  "providerBundleName": "com.example.adprovider",
  "providerAbilityName": "AdsServiceAbility",
  "providerTokenName": "example_token_123"
}
```

#### 代码读取位置

```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:93-141
bool LoadAndCheckOaidTrustList(const std::string &bundleName) {
    // 1. 读取配置文件
    GetOneCfgFile(OAID_TRUSTLIST_EXTENSION_CONFIG_PATH.c_str(), pathBuff, PATH_MAX);
    // 2. 解析 JSON
    cJSON *root = cJSON_Parse(fileContent.c_str());
    // 3. 检查白名单
    cJSON *oaidTrustConfig = cJSON_GetObjectItem(root, "resetOAIDBundleName");
}
```

### 扩展配置文件

**路径**: `/etc/advertising/oaid/oaid_service_config_ext.json`

**用途**: 覆盖主配置文件，用于自定义配置

**优先级**: 扩展配置 > 主配置

---

## 权限声明接口

### 权限清单

| 权限 | 值 | 类型 | 说明 |
|------|-----|------|------|
| `ohos.permission.APP_TRACKING_CONSENT` | "ohos.permission.APP_TRACKING_CONSENT" | user_grant | 广告追踪权限 |

### 权限申请方式

**静态声明** (module.json5):

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.APP_TRACKING_CONSENT",
        "reason": "$string:tracking_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

**动态申请**:

```typescript
import { abilityAccessCtrl, common } from '@kit.AbilityKit';

async function requestPermission(context: common.Context): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  const result = await atManager.requestPermissionsFromUser(
    context, 
    ['ohos.permission.APP_TRACKING_CONSENT']
  );
  return result.authResults[0] === 0;
}
```

---

## 错误码汇总

### 公共错误码

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 0 | ERR_OK | 成功 |
| 401 | ERR_INVALID_PARAM | 无效参数 |
| 403 | ERR_PERMISSION_ERROR | 权限错误 |

### OAID 特有错误码

| 错误码 | 常量 | 说明 | 触发场景 |
|--------|------|------|----------|
| 202 | OAID_ERROR_CODE_NOT_SYSTEM_APP | 非系统应用 | resetOAID 被非系统应用调用 |
| 17300001 | ERR_SYSYTEM_ERROR | 系统内部错误 | 服务异常、UUID 生成失败 |
| 17300002 | OAID_ERROR_NOT_IN_TRUST_LIST | 不在信任列表 | resetOAID 调用者不在白名单 |
| 17300003 | ERR_WRITE_PARCEL_FAILED | 写入 Parcel 失败 | IPC 通信错误 |

---

## 接口对比表

| 特性 | getOAID | resetOAID |
|------|---------|-----------|
| **调用者** | 任意应用 | 仅限系统应用 |
| **权限要求** | APP_TRACKING_CONSENT | 系统应用 + 白名单 |
| **用户授权** | 需要 | 不需要（系统级） |
| **异步方式** | Promise / Callback | 同步 |
| **返回值** | string | void |
| **错误处理** | Promise reject / err 回调 | 抛出异常 |

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构分析](02_Architecture.md)
- [攻击面分析](05_AttackSurface.md)
- [安全风险评估](06_SecurityReview.md)
