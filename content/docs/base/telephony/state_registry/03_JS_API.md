# JS API 参考

## 模块概述

`@ohos.telephony.observer` 模块提供电信状态变化的观察者注册与注销能力。通过注册观察者，应用可以在状态发生变化时收到异步回调通知。

**入口文件**：`interfaces/kits/js/@ohos.telephony.observer.d.ts`

## API 清单

### 注册类 API

| JS API 名称 | 命名空间 | 同步/异步 | 对应 C++ 入口 | 绑定文件 |
|-------------|----------|-----------|---------------|----------|
| `on` | observer | 异步 | `RegisterObserver` | `frameworks/js/napi/observer/napi_observer.cpp` |

### 注销类 API

| JS API 名称 | 命名空间 | 同步/异步 | 对应 C++ 入口 | 绑定文件 |
|-------------|----------|-----------|---------------|----------|
| `off` | observer | 异步 | `UnregisterObserver` | `frameworks/js/napi/observer/napi_observer.cpp` |

## on() API 详解

### 函数签名

```typescript
function on<T>(type: string, options: { slotId?: number }, callback: AsyncCallback<T>): void
```

### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| type | string | 是 | 事件类型，见下方事件类型表 |
| options.slotId | number | 否 | SIM 卡槽 ID，`0` 为主卡，`1` 为副卡，未指定则使用默认 |
| callback | AsyncCallback\<T\> | 是 | 事件回调函数 |

### 事件类型与回调数据类型

| 事件类型 | 回调数据类型 | 权限要求 | N-API 文件 |
|----------|--------------|----------|------------|
| networkStateChange | [NetworkState](../telephony-core/NetworkState.md) | ohos.permission.GET_NETWORK_INFO | `napi_observer_network.cpp` |
| signalInfoChange | [SignalInformation](https://gitee.com/openharmony/telephony_core_services/blob/master/interfaces/kits/js/@ohos.telephony.md) | 无 | `napi_observer_signal.cpp` |
| cellInfoChange | [CellInformation](https://gitee.com/openharmony/telephony_core_services/blob/master/interfaces/kits/js/@ohos.telephony.md) | ohos.permission.LOCATION + ohos.permission.APPROXIMATELY_LOCATION | `napi_observer_cell.cpp` |
| cellularDataConnectionStateChange | CellularDataConnectionState | 无 | `napi_observer_data.cpp` |
| cellularDataFlowChange | CellularDataFlowType | 无 | `napi_observer_data.cpp` |
| callStateChange | [CallStateInfo](https://gitee.com/openharmony/telephony_core_services/blob/master/interfaces/kits/js/@ohos.telephony.md) | ohos.permission.READ_CALL_LOG | `napi_observer_call.cpp` |
| simStateChange | [SimStateData](#simstatedata) | 无 | `napi_observer_sim.cpp` |

### 错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| 401 | 参数错误 | 检查 type、options、callback 参数格式 |
| 405 | 权限不足 | 检查是否申请了所需权限 |
| 406 | 服务不可用 | 电信服务未就绪，延迟重试 |
| 410 | 状态异常 | slotId 对应的卡槽不存在或无效 |

### 使用示例

```typescript
import observer from '@ohos.telephony.observer';

// 注册通话状态变化监听
observer.on('callStateChange', { slotId: 0 }, (err, data: Record<string, string | number>) => {
    if (err) {
        console.error(`注册失败: ${err.message}`);
        return;
    }
    console.log(`通话状态: ${data.state}, 号码: ${data.number}`);
});

// 注册网络状态变化监听
observer.on('networkStateChange', (err, data) => {
    if (err) {
        console.error(`注册失败: ${err.message}`);
        return;
    }
    console.log(`网络类型: ${data.networkType}`);
});
```

## off() API 详解

### 函数签名

```typescript
function off<T>(type: string, callback?: AsyncCallback<T>): void
```

### 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| type | string | 是 | 事件类型，与 on() 的 type 对应 |
| callback | AsyncCallback\<T\> | 否 | 指定的回调函数，为空则注销该类型所有回调 |

### 错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| 401 | 参数错误 | 检查 type 参数格式 |
| 406 | 服务不可用 | 电信服务未就绪，延迟重试 |

### 使用示例

```typescript
import observer from '@ohos.telephony.observer';

// 注销指定的回调
observer.off('callStateChange', (err) => {
    if (err) {
        console.error(`注销失败: ${err.message}`);
        return;
    }
    console.log('通话状态监听已注销');
});

// 注销该类型的所有回调
observer.off('networkStateChange');
```

## 数据类型定义

### SimStateData

```typescript
/**
 * SIM 卡状态数据
 * @typedef {Object} SimStateData
 * @property {string} type - SIM 卡类型
 * @property {string} state - SIM 卡状态
 * @property {string} reason - 状态变化原因
 */
interface SimStateData {
    type: string;
    state: string;
    reason: string;
}
```

### NetworkState

```typescript
/**
 * 网络状态数据
 * @typedef {Object} NetworkState
 * @property {string} networkType - 网络类型
 * @property {boolean} isConnected - 是否已连接
 * @property {string} operatorName - 运营商名称
 */
interface NetworkState {
    networkType: string;
    isConnected: boolean;
    operatorName: string;
}
```

## 权限声明

### 必选权限

| 权限名 | 用途 | 申请方式 | 风险等级 |
|--------|------|----------|----------|
| ohos.permission.GET_NETWORK_INFO | 获取网络状态信息 | user_grant | normal |
| ohos.permission.READ_CALL_LOG | 读取通话记录 | user_grant | dangerous |
| ohos.permission.LOCATION | 精确定位 | user_grant | dangerous |
| ohos.permission.APPROXIMATELY_LOCATION | 模糊定位 | user_grant | normal |

### 权限申请示例

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

async function requestPermissions() {
    const atManager = abilityAccessCtrl.createAtManager();
    const bundleInfo = await bundleManager.getBundleInfoForSelf(
        bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );
    const tokenId = bundleInfo.appInfo.accessTokenId;
    
    const permissions = [
        'ohos.permission.GET_NETWORK_INFO',
        'ohos.permission.READ_CALL_LOG'
    ];
    
    // 跳转到权限申请页面让用户授权
    // 实际开发中需要在 module.json5 中声明权限
}
```

**权限声明位置**：`module.json5` 中 `requestPermissions` 字段。

## 完整调用链

### 事件注册调用链

```
App JS (observer.on)
    ↓
napi_observer_xxx.cpp (N-API 入口)
    ↓
napi_observer_utils.cpp (参数解析与校验)
    ↓
telephony_state_registry_service (RegisterObserver)
    ↓
telephony_observer_proxy (代理转发)
    ↓
core_service (事件订阅)
```

**证据来源**：`frameworks/js/napi/observer/napi_observer.cpp` N-API 实现。

### 事件回调调用链

```
Modem Event (硬件事件)
    ↓
core_service (底层事件通知)
    ↓
telephony_state_registry_service (NotifyChange)
    ↓
telephony_observer_proxy (代理回调)
    ↓
napi_observer_xxx.cpp (JS 回调触发)
    ↓
App JS (用户回调函数)
```

## 最佳实践

1. **及时注销**：应用进入后台或不再需要监听时，调用 `off()` 注销观察者
2. **错误处理**：始终检查 `err` 参数，处理可能的错误情况
3. **权限检查**：在调用 API 前确认权限已授予
4. **槽位管理**：多卡设备注意正确处理不同 slotId

## 相关文档

- [架构设计](02_Architecture.md)
- [Native API](04_Native_API.md)
- [安全评审](07_Security_Review.md)
