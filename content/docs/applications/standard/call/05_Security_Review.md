# 安全评审

## 1. 威胁模型概述

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    应用进程 (com.ohos.callui)             │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐    │    │
│  │  │ UI 线程 │  │ Service │  │  数据存储            │    │    │
│  │  └─────────┘  └─────────┘  └─────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                            │                                     │
│                    系统 API 调用                                  │
│                            │                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   Telephony SA                          │    │
│  │  (可信区域，由系统框架保证)                                │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击面分析

| 攻击面 | 描述 | 风险等级 |
|-------|------|---------|
| 电话号码输入 | 用户输入拨打的电话号码 | 中 |
| 来电号码显示 | 展示来电号码信息 | 中 |
| 通话状态监听 | 监听通话状态变化 | 低 |
| 系统 API 调用 | 调用 telephony 系统能力 | 低 |
| IPC 通信 | ServiceAbility IPC 通信 | 低 |
| 权限请求 | 申请敏感系统权限 | 高 |
| 数据存储 | 本地偏好设置存储 | 低 |

## 2. 权限安全

### 2.1 权限声明清单

应用声明了以下敏感权限：

| 权限 | 用途 | 敏感度 | 必要性 |
|-----|------|-------|-------|
| `ohos.permission.PLACE_CALL` | 发起通话 | **高** | ✅ 必需 |
| `ohos.permission.ANSWER_CALL` | 接听电话 | **高** | ✅ 必需 |
| `ohos.permission.GET_TELEPHONY_STATE` | 获取通话状态 | 中 | ✅ 必需 |
| `ohos.permission.SET_TELEPHONY_STATE` | 设置通话状态 | **高** | ⚠️ 可选 |
| `ohos.permission.READ_CONTACTS` | 读取联系人 | **高** | ⚠️ 可选 |
| `ohos.permission.SEND_MESSAGES` | 发送短信 | **高** | ⚠️ 可选 |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | 中 | ⚠️ 可选 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 管理安全设置 | **高** | ⚠️ 可选 |

### 2.2 权限评估

#### ✅ 合理使用的权限

| 权限 | 评估 |
|-----|------|
| `ohos.permission.PLACE_CALL` | **合理** - 应用核心功能必需 |
| `ohos.permission.ANSWER_CALL` | **合理** - 来电接听必需 |
| `ohos.permission.GET_TELEPHONY_STATE` | **合理** - 通话状态监听必需 |

#### ⚠️ 需关注的权限

| 权限 | 风险 | 建议 |
|-----|------|-----|
| `ohos.permission.SET_TELEPHONY_STATE` | 可被滥用修改系统通话状态 | 仅在确实需要时使用 |
| `ohos.permission.READ_CONTACTS` | 可访问用户联系人 | 仅在联系人显示功能时请求 |
| `ohos.permission.SEND_MESSAGES` | 可发送短信 | 仅在拒接短信功能时请求 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 可修改安全设置 | 严格限制使用场景 |

### 2.3 权限请求位置

```typescript
// 文件: entry/src/main/module.json
{
  "requestPermissions": [
    {
      "name": "ohos.permission.PLACE_CALL",
      "reason": "$string:PLACE_CALL"           // 必须提供使用理由
    },
    {
      "name": "ohos.permission.ANSWER_CALL",
      "reason": "$string:ANSWER_CALL"
    }
  ]
}
```

## 3. 输入验证

### 3.1 电话号码验证

**证据位置**: `entry/src/main/ets/model/CallServiceProxy.ets:54-61`

```typescript
public dialCall(phoneNumber, accountId = 0, videoState = 0, dialScene = 0) {
  // 验证: 直接传递 phoneNumber 到系统 API
  return call.dial(phoneNumber, {
    accountId,
    videoState,
    dialScene
  });
}
```

**风险**: 未对 `phoneNumber` 进行格式验证

**建议**: 添加号码格式验证

```typescript
// 建议的验证逻辑
function validatePhoneNumber(number: string): boolean {
  // 验证号码格式 (纯数字，允许特殊字符)
  const phoneRegex = /^[\d+\-*#]{1,20}$/;
  return phoneRegex.test(number);
}

public dialCall(phoneNumber: string, ...) {
  if (!validatePhoneNumber(phoneNumber)) {
    throw new Error('INVALID_PHONE_NUMBER');
  }
  return call.dial(phoneNumber, {...});
}
```

### 3.2 通话 ID 验证

**证据位置**: `entry/src/main/ets/ServiceAbility/ServiceAbility.ts:35-49`

```typescript
onConnect(want: Want): Stub {
  let callData: DefaultCallData = new DefaultCallData();
  callData.accountNumber = want.parameters?.accountNumber;
  callData.videoState = want.parameters?.videoState;
  callData.callType = want.parameters?.callType;
  // ...
  return new Stub('ServiceAbility');
}
```

**风险**: `want.parameters` 可被外部任意构造

**建议**: 添加参数验证

```typescript
function validateCallData(params: any): boolean {
  if (typeof params?.callId !== 'number') return false;
  if (typeof params?.callState !== 'number') return false;
  return true;
}
```

## 4. IPC 安全

### 4.1 IPC 服务端实现

**证据位置**: `entry/src/main/ets/ServiceAbility/ServiceAbility.ts:68-77`

```typescript
class Stub extends rpc.RemoteObject {
  onRemoteRequest(code, date, reply, option): boolean {
    LogUtils.i(TAG, 'Stub onRemoteRequest code:' + code);
    return true;  // ⚠️ 直接返回成功，无验证
  }
  
  constructor(descriptor) {
    super(descriptor);
  }
}
```

**风险**: 
1. `onRemoteRequest` 直接返回 `true`，无实际处理逻辑
2. 无调用方身份验证
3. 无参数校验

**建议**:

```typescript
class Stub extends rpc.RemoteObject {
  onRemoteRequest(code, data, reply, option): boolean {
    // 1. 验证调用方
    const callingUid = rpc.IPCSkeleton.getCallingUid();
    if (!this.isAllowedCaller(callingUid)) {
      LogUtils.e(TAG, 'Unauthorized IPC call from uid: ' + callingUid);
      return false;
    }

    // 2. 参数验证
    if (!data) {
      LogUtils.e(TAG, 'Invalid IPC data');
      return false;
    }

    // 3. 业务处理
    try {
      const result = this.handleRequest(code, data);
      reply.writeString(result);
      return true;
    } catch (e) {
      LogUtils.e(TAG, 'IPC request failed: ' + e);
      return false;
    }
  }

  private isAllowedCaller(uid: number): boolean {
    // 白名单机制，仅允许系统组件调用
    const ALLOWED_UIDS = [/* 系统 UID 列表 */];
    return ALLOWED_UIDS.includes(uid);
  }
}
```

### 4.2 CommonEvent 发布

**证据位置**: `entry/src/main/ets/model/CallServiceProxy.ets:296-307`

```typescript
public publish(data) {
  commonEvent.publish('callui.event.callEvent', {
    bundleName: 'com.ohos.callui',
    isOrdered: false,
    subscriberPermissions: ['ohos.permission.GET_TELEPHONY_STATE'],
    data: JSON.stringify(data)
  }, (res) => {
    LogUtils.i(TAG, 'callui.event.callEvent success')
  });
}
```

**评估**: ✅ 已设置订阅权限要求

## 5. 数据存储安全

### 5.1 偏好设置

**证据位置**: `mobiledatasettings/src/main/ets/common/utils/PreferenceUtil.ets`

```typescript
// 使用 Preferences 存储配置
import preferences from '@ohos.data.preferences';

// 读取
preferences.getValue('key', defaultValue);

// 写入
preferences.putValue('key', value);
```

**风险**: 
1. 偏好设置默认不加密
2. 敏感数据可能泄露

**建议**: 
- 对敏感配置使用加密存储
- 使用 `@ohos.security.crypto` 加密敏感数据

### 5.2 AppStorage 安全

**证据位置**: `entry/src/main/ets/model/CallManager.ets:141-144`

```typescript
AppStorage.SetOrCreate('AccountNumber', formattedNumber);
```

**风险**: `AppStorage` 是进程内存储，相对安全，但内存中仍可能泄露

**建议**: 通话结束后及时清理敏感数据

## 6. 竞态条件

### 6.1 通话状态竞态

**风险场景**: 
```
时间 T1: 用户 A 接听电话 (callId=1)
时间 T2: 用户 B 挂断电话 (callId=1)
时间 T3: 回调返回，更新界面显示"已接通"
```

**证据位置**: `entry/src/main/ets/model/CallManager.ets:121-149`

```typescript
async update(callData) {
  // ⚠️ 竞态窗口：callData 可能在回调间变化
  if (this.callData != undefined && this.callData.callId === callData.callId) {
    // ...
  }
}
```

**建议**: 
- 使用原子操作更新状态
- 维护最近一次有效状态

## 7. 可利用点清单

### 7.1 高风险项

| 编号 | 问题 | 风险等级 | 位置 |
|-----|------|---------|------|
| H-01 | 未验证的 IPC 调用 | **高** | `ServiceAbility.ts:69-71` |
| H-02 | 电话号码未验证格式 | **高** | `CallServiceProxy.ts:54-61` |
| H-03 | ServiceAbility 参数未校验 | **高** | `ServiceAbility.ts:38-46` |
| H-04 | 通话状态回调无权限控制 | **中** | `TelephonyApi.ts:33-46` |

### 7.2 中风险项

| 编号 | 问题 | 风险等级 | 位置 |
|-----|------|---------|------|
| M-01 | 偏好设置未加密存储 | **中** | `PreferenceUtil.ets` |
| M-02 | 通话结束后敏感数据未清理 | **中** | `CallManager.ts` |
| M-03 | 通话状态竞态窗口 | **中** | `CallManager.ts:121-149` |
| M-04 | LogUtils 可能泄露敏感信息 | **低** | 全局日志 |

### 7.3 低风险项

| 编号 | 问题 | 风险等级 | 位置 |
|-----|------|---------|------|
| L-01 | CommonEvent 事件名称硬编码 | **低** | `CallServiceProxy.ts:298` |

## 8. 安全建议

### 8.1 紧急修复项 (立即处理)

1. **IPC 调用方验证**
   - 在 `Stub.onRemoteRequest` 添加 UID 验证
   - 仅允许系统组件调用

2. **电话号码格式验证**
   - 添加正则验证
   - 限制号码长度和字符集

### 8.2 短期改进项

1. **加密敏感存储**
   - 使用 `@ohos.security.crypto` 加密
   - 通话结束后清理 AppStorage

2. **通话状态原子操作**
   - 使用锁或原子变量
   - 消除竞态窗口

### 8.3 长期优化项

1. **权限最小化**
   - 审查并移除不必要的权限
   - 使用前动态申请

2. **安全编码规范**
   - 禁止硬编码敏感信息
   - 完善异常处理

## 9. 检查范围

### 9.1 已检查代码

| 模块 | 文件数 | 关键发现 |
|-----|-------|---------|
| entry | 30+ | IPC 未验证、号码未校验 |
| mobiledatasettings | 15+ | 偏好设置未加密 |
| common | 5+ | 无安全问题 |

### 9.2 未检查范围

| 范围 | 原因 |
|-----|------|
| 系统 API 内部实现 | 黑盒 |
| 测试代码 | 不在范围内 |
| 构建脚本 | 不在范围内 |

## 10. 相关文档

- [项目概述](01_Overview.md)
- [架构设计](02_Architecture.md)
- [API 参考](03_API_Reference.md)
- [构建指南](04_Build_Guide.md)
