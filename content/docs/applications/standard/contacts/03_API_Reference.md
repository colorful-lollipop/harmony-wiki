# API 参考

## 概述

本项目为 **纯 ArkTS 应用**，不提供 N-API 绑定，而是**消费**系统 Native APIs。本章档记录应用调用的 Native API 模式。

> **重要说明**: 本应用不包含 C/C++ 代码，无 N-API 实现层。所有 API 调用均通过 `@ohos.*` 模块进行。

## Native API 调用清单

### 能力管理 (Ability)

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.app.ability.UIAbility` | `import Ability from '@ohos.app.ability.UIAbility'` | Ability 基类 | `MainAbility.ts:16` |
| `@ohos.app.ability.Want` | `import Want from '@ohos.app.ability.Want'` | Intent 包装器 | `MainAbility.ts:20` |
| `@kit.AbilityKit` | `import { AbilityConstant } from '@kit.AbilityKit'` | Ability 常量 | `MainAbility.ts:24` |
| `@ohos.app.ability.AbilityStage` | `import AbilityStage from '@ohos.app.ability.AbilityStage'` | Stage 入口 | `MyAbilityStage.ts` |

#### UIAbility 生命周期方法

```typescript
// 证据来源: entry/src/main/ets/MainAbility/MainAbility.ts

class MainAbility extends Ability {
    onCreate(want: Want, launchParam: AbilityConstant.LaunchParam) {
        // 应用创建时调用
        globalThis.context = this.context;
    }
    
    onNewWant(want: Want, launchParam: AbilityConstant.LaunchParam) {
        // 新 Intent 到达时调用
    }
    
    onDestroy() {
        // 应用销毁时调用
    }
    
    onWindowStageCreate(windowStage: Window.WindowStage) {
        // 窗口创建时调用
        windowStage.loadContent('pages/index', this.storage);
    }
    
    onWindowStageDestroy() {
        // 窗口销毁时调用
    }
    
    onForeground() {
        // 来到前台时调用
    }
    
    onBackground() {
        // 退到后台时调用
    }
}
```

### 窗口管理 (Window)

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.window` | `import Window from '@ohos.window'` | 窗口管理 | `MainAbility.ts:17` |

#### 窗口 API 使用

```typescript
// 证据来源: entry/src/main/ets/MainAbility/MainAbility.ts:94-101

// 获取顶层窗口
Window.getTopWindow(this.context).then((windowObj) => {
    // 获取窗口属性
    windowObj.getProperties().then((windowProperties) => {
        this.updateBreakpoint(windowProperties.windowRect.width);
    });
    
    // 监听窗口大小变化
    windowObj.on('windowSizeChange', (data) => {
        this.updateBreakpoint(data.width);
    });
});
```

### 电话能力 (Telephony)

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.telephony.call` | `import call from '@ohos.telephony.call'` | 拨打电话 | `StaticSubscriber.ts` |
| `@ohos.telephony.sim` | `import sim from '@ohos.telephony.sim'` | SIM 卡管理 | `StaticSubscriber.ts` |

#### 拨打电话 API

```typescript
// 典型用法
import call from '@ohos.telephony.call';

// 发起呼叫
call.makeCall(phoneNumber: string): void

// 挂断呼叫
call.endCall(callId: number): void
```

### 通知管理 (Notification)

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.notificationManager` | `import notificationManager from '@ohos.notificationManager'` | 通知管理 | `MyAbilityStage.ts` |

### 静态订阅 (StaticSubscriber)

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.application.StaticSubscriberExtensionAbility` | - | 静态事件订阅 | `StaticSubscriber.ts` |

```typescript
// 证据来源: entry/src/main/ets/StaticSubscriber/StaticSubscriber.ts
import StaticSubscriberExtensionAbility from '@ohos.application.StaticSubscriberExtensionAbility';

export default class StaticSubscriber extends StaticSubscriberExtensionAbility {
    onReceiveEvent(event: Want) {
        // 处理系统事件
    }
}
```

### 多线程 Worker

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.worker` | `import worker from '@ohos.worker'` | Worker 多线程 | `Worker.ts` |
| `@ohos.buffer` | `import buffer from '@ohos.buffer'` | 缓冲区操作 | `WorkerWrapper.ts` |

#### Worker API 使用

```typescript
// 证据来源: entry/src/main/ets/workers/base/Worker.ts

import worker from '@ohos.worker';

// 创建 Worker
const workerInstance = new worker.Worker('path/to/worker.ts');

// 发送消息
workerInstance.postMessage({
    type: 'task',
    data: payload
});

// 接收消息
workerInstance.onmessage = (e) => {
    const result = e.data;
};
```

### 权限控制 (AbilityAccessCtrl)

| API 模块 | 导入语句 | 用途 | 证据路径 |
|----------|----------|------|----------|
| `@ohos.abilityAccessCtrl` | `import abilityAccessCtrl from '@ohos.abilityAccessCtrl'` | 权限管理 | `PermissionManager.ets:16` |

#### 权限请求 API

```typescript
// 证据来源: common/src/main/ets/permission/PermissionManager.ets:37-61

import abilityAccessCtrl, { Permissions } from '@ohos.abilityAccessCtrl';
import { BusinessError } from '@ohos.base';

class PermissionManager {
    async initPermissions() {
        const requestPermissions: Permissions[] = [
            "ohos.permission.READ_CONTACTS",
            "ohos.permission.WRITE_CONTACTS",
            "ohos.permission.READ_CALL_LOG",
            "ohos.permission.WRITE_CALL_LOG",
            "ohos.permission.MANAGE_VOICEMAIL"
        ];
        
        const AtManager = abilityAccessCtrl.createAtManager();
        
        AtManager.requestPermissionsFromUser(
            globalThis.context,
            requestPermissions
        ).then((data) => {
            // 处理授权结果
            const authResults = data.authResults;
        }).catch((err: BusinessError) => {
            // 处理错误
        });
    }
}
```

## API 调用模式

### 1. 异步 API 调用模式

```typescript
// Promise 风格
async function fetchData(): Promise<Contact[]> {
    const result = await repository.getContacts();
    return result;
}

// Callback 风格
function fetchDataWithCallback(callback: (data: Contact[]) => void): void {
    repository.getContacts((data) => {
        callback(data);
    });
}
```

### 2. 错误处理模式

```typescript
import { BusinessError } from '@ohos.base';

try {
    // 调用可能失败的 API
    await someNativeApi();
} catch (err) {
    const businessError = err as BusinessError;
    console.error(`Error: ${businessError.code}, ${businessError.message}`);
}
```

### 3. 上下文获取模式

```typescript
// 从 globalThis 获取上下文
const context = globalThis.context as Context;

// 常见用法
const rdbStore = await context.getRdbStore(config);
```

## 数据契约 URI

### Contacts DataAbility URI

| 数据类型 | URI | 证据路径 |
|----------|-----|----------|
| 联系人 | `datashare:///com.ohos.contactsdataability` | `Contacts.ets:22` |
| 联系人详情 | `datashare:///com.ohos.contactsdataability/contacts/contact` | `Contacts.ets:23` |

```typescript
// 证据来源: feature/contact/src/main/ets/contract/Contacts.ets

export class Contacts extends ContactsColumns {
    static readonly CONTENT_URI: string = 'datashare:///com.ohos.contactsdataability';
    static readonly CONTACT_URI: string = Contacts.CONTENT_URI + '/contacts/contact';
}
```

## API 使用注意事项

### 1. 权限检查

| 权限 | API | 检查位置 |
|------|-----|----------|
| READ_CONTACTS | 联系人查询 | Repository 层 |
| WRITE_CONTACTS | 联系人写入 | Repository 层 |
| READ_CALL_LOG | 通话记录查询 | Call 模块 |
| PLACE_CALL | 拨打电话 | Dialer 模块 |

### 2. 线程限制

| API | 主线程 | Worker |
|-----|--------|--------|
| UI 操作 | ✅ | ❌ |
| RDB 操作 | ✅ | ✅ |
| Worker.postMessage | ✅ | ✅ |
| Window API | ✅ | ❌ |

### 3. 异步要求

以下 API 必须异步调用：

- `Window.getTopWindow()`
- `windowStage.loadContent()`
- `rdbStore.executeQuery()`
- `abilityAccessCtrl.requestPermissionsFromUser()`
- `worker.postMessage()` (同步发送，消息处理异步)

## 外部相关 API

以下 API 由相关仓库提供，不在本项目实现范围内：

| API 来源 | 用途 | 相关仓库 |
|----------|------|----------|
| ContactsDataAbility | 联系人数据存储 | `applications_contactsdata` |
| CallAbility | 通话能力 | `applications_call` |
| MMS Ability | 短信能力 | `applications_mms` |
