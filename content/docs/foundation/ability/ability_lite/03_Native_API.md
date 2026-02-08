# 对外 Native API

## 目的

本文档描述 ability_lite 对外提供的 Native C/C++ API，供 Native 应用开发使用。

## 适用范围

- Native 应用开发者
- 需要调用 Ability 管理功能的模块

## API 清单表

### Ability 生命周期 API

**头文件**: `interfaces/kits/ability_lite/ability.h`

| API | 类型 | 参数 | 返回值 | 说明 |
|-----|------|------|--------|------|
| `OnStart(const Want &want)` | 虚函数 | want: 启动信息 | void | Ability 启动时回调 |
| `OnActive(const Want &want)` | 虚函数 | want: 激活信息 | void | Ability 进入前台 |
| `OnInactive()` | 虚函数 | - | void | Ability 失去焦点 |
| `OnBackground()` | 虚函数 | - | void | Ability 进入后台 |
| `OnStop()` | 虚函数 | - | void | Ability 停止 |
| `OnConnect(const Want &want)` | 虚函数 | want: 连接信息 | SvcIdentity* | Service Ability 连接 |
| `OnDisconnect(const Want &want)` | 虚函数 | want: 断开信息 | void | Service Ability 断开 |

**代码位置**: `interfaces/kits/ability_lite/ability.h:87-142`

```cpp
class Ability : public AbilityContext {
public:
    virtual void OnStart(const Want &want);
    virtual void OnInactive();
    virtual void OnActive(const Want &want);
    virtual void OnBackground();
    virtual void OnStop();
    virtual const SvcIdentity *OnConnect(const Want &want);
    virtual void OnDisconnect(const Want &want);
    virtual void MsgHandle(uint32_t funcId, IpcIo *request, IpcIo *reply);
};
```

### Ability 管理 API

**头文件**: `interfaces/kits/ability_lite/ability_manager.h`

| API | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `StartAbility(const Want *want)` | want: 目标 Ability 信息 | int (0=成功) | 启动 Ability |
| `StartAbilityWithCallback(const Want *want, IAbilityStartCallback cb)` | want, cb: 回调函数 | int | 带回调的启动 |
| `StopAbility(const Want *want)` | want: 目标 Ability 信息 | int | 停止 Service Ability |
| `ConnectAbility(const Want *want, const IAbilityConnection *conn, void *data)` | want, conn: 连接回调, data: 用户数据 | int | 连接 Service Ability |
| `DisconnectAbility(const IAbilityConnection *conn)` | conn: 连接对象 | int | 断开 Service Ability |

**代码位置**: `interfaces/kits/ability_lite/ability_manager.h:70-109`

```cpp
// 启动回调类型
typedef void (*IAbilityStartCallback)(const uint8_t resultCode, const void *resultMessage);

// API 函数
int StartAbility(const Want *want);
int StartAbilityWithCallback(const Want *want, IAbilityStartCallback iAbilityStartCallback);
int StopAbility(const Want *want);
int ConnectAbility(const Want *want, const IAbilityConnection *conn, void *data);
int DisconnectAbility(const IAbilityConnection *conn);
```

### Want 操作 API

**头文件**: `interfaces/kits/want_lite/want.h`

| API | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `ClearWant(Want *want)` | want | void | 清理 Want 内存 |
| `SetWantElement(Want *want, ElementName element)` | want, element | bool | 设置 Element |
| `SetWantData(Want *want, const void *data, uint16_t dataLength)` | want, data, length | bool | 设置数据 |
| `SetIntParam(Want *want, const char *key, uint8_t keyLen, int32_t value)` | want, key, keyLen, value | bool | 设置整型参数 |
| `SetStrParam(Want *want, const char *key, uint8_t keyLen, const char *value, uint8_t valueLen)` | want, key, keyLen, value, valueLen | bool | 设置字符串参数 |

**代码位置**: `interfaces/kits/want_lite/want.h:101-127`

```cpp
typedef struct {
    ElementName *element;
    SvcIdentity *sid;
    void *data;
    uint16_t dataLength;
    const char *appPath;
    uint32_t mission;
} Want;

void ClearWant(Want *want);
bool SetWantElement(Want *want, ElementName element);
bool SetWantData(Want *want, const void *data, uint16_t dataLength);
bool SetIntParam(Want *want, const char *key, uint8_t keyLen, int32_t value);
bool SetStrParam(Want *want, const char *key, uint8_t keyLen, const char *value, uint8_t valueLen);
```

### Ability 注册 API

**头文件**: `interfaces/kits/ability_lite/ability_loader.h`

| API/宏 | 参数 | 说明 |
|--------|------|------|
| `REGISTER_AA(className)` | className: Ability 类名 | 注册 Ability 类 |

**代码位置**: `interfaces/kits/ability_lite/ability_loader.h`

```cpp
#define REGISTER_AA(className) \
    AbilityLoader::GetInstance().RegisterAbility(#className, new className())
```

### Ability 连接回调

**头文件**: `interfaces/kits/ability_lite/ability_connection.h`

| 回调 | 参数 | 说明 |
|------|------|------|
| `OnAbilityConnectDone(want, sid, resultCode)` | want, sid: 服务标识, resultCode | 连接完成回调 |
| `OnAbilityDisconnectDone(want, resultCode)` | want, resultCode | 断开完成回调 |

**代码位置**: `interfaces/kits/ability_lite/ability_connection.h`

```cpp
typedef struct {
    void (*OnAbilityConnectDone)(const Want *want, const SvcIdentity *sid, uint32_t resultCode);
    void (*OnAbilityDisconnectDone)(const Want *want, uint32_t resultCode);
} IAbilityConnection;
```

## 错误码定义

**头文件**: `interfaces/kits/ability_lite/ability_errors.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERR_OK` | 0 | 成功 |
| `PARAM_NULL_ERROR` | 1 | 参数为空 |
| `PARAM_CHECK_ERROR` | 9 | 参数校验失败 |
| `MEMORY_MALLOC_ERROR` | 2 | 内存分配失败 |
| `COMMAND_ERROR` | 0x7fff | 通用命令错误 |

**代码位置**: `interfaces/kits/ability_lite/ability_errors.h`

```cpp
enum AppexecfwkErrors {
    ERR_OK = 0,
    PARAM_NULL_ERROR = 1,
    MEMORY_MALLOC_ERROR = 2,
    PARAM_CHECK_ERROR = 9,
    COMMAND_ERROR = 0x7fff,
};
```

## 生命周期状态

**头文件**: `interfaces/kits/ability_lite/ability_state.h`

| 状态 | 值 | 说明 |
|------|-----|------|
| `STATE_UNINITIALIZED` | 0 | 未初始化 |
| `STATE_INITIAL` | 1 | 初始/停止状态 |
| `STATE_INACTIVE` | 2 | 可见但无焦点 |
| `STATE_ACTIVE` | 3 | 前台有焦点 |
| `STATE_BACKGROUND` | 4 | 后台 |

**代码位置**: `interfaces/kits/ability_lite/ability_state.h`

```cpp
enum AbilityLifecycleState {
    STATE_UNINITIALIZED = 0,
    STATE_INITIAL = 1,
    STATE_INACTIVE = 2,
    STATE_ACTIVE = 3,
    STATE_BACKGROUND = 4,
};
```

## 使用示例

### 创建 Page Ability

```cpp
#include "ability.h"
#include "ability_loader.h"

using namespace OHOS;

class MyAbility : public Ability {
public:
    void OnStart(const Want &want) override {
        // 初始化操作
    }
    
    void OnActive(const Want &want) override {
        // 进入前台
    }
    
    void OnBackground() override {
        // 进入后台
    }
    
    void OnStop() override {
        // 清理操作
    }
};

// 注册 Ability
REGISTER_AA(MyAbility);
```

### 启动 Ability

```cpp
#include "ability_manager.h"
#include "want.h"

Want want;
memset_s(&want, sizeof(Want), 0, sizeof(Want));

ElementName element;
SetElementDeviceID(&element, "");
SetElementBundleName(&element, "com.example.app");
SetElementAbilityName(&element, "MainAbility");

SetWantElement(&want, element);
int result = StartAbility(&want);
ClearWant(&want);
```

### 连接 Service Ability

```cpp
#include "ability_manager.h"
#include "ability_connection.h"

void OnConnectDone(const Want *want, const SvcIdentity *sid, uint32_t resultCode) {
    // 连接成功
}

void OnDisconnectDone(const Want *want, uint32_t resultCode) {
    // 断开连接
}

IAbilityConnection conn = {
    .OnAbilityConnectDone = OnConnectDone,
    .OnAbilityDisconnectDone = OnDisconnectDone,
};

Want want;
// ... 设置 want ...
ConnectAbility(&want, &conn, nullptr);
```

## 权限/前置条件

| API | 权限要求 | 前置条件 |
|-----|----------|----------|
| `StartAbility` | 无 | 目标 Ability 已安装 |
| `StopAbility` | 无 | 目标 Service Ability 正在运行 |
| `ConnectAbility` | 无 | 目标 Service Ability 已注册 |
| `DisconnectAbility` | 无 | 已建立连接 |

## 相关链接

- [N-API / JS API](04_NAPI_JS_API.md)
- [内部 API](05_Inner_API.md)
- [架构说明](02_Architecture.md)
