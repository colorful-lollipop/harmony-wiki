# Connected NFC Tag - N-API 接口文档

## 1. 模块概述

### 1.1 模块信息

| 属性 | 值 |
|------|------|
| **模块名** | `connectedTag` |
| **N-API 版本** | 1 |
| **模块文件** | `libconnectedtag.z.so` |
| **注册位置** | `frameworks/js/napi/nfc_napi_entry.cpp:81-95` |
| **依赖组件** | `connected_nfc_tag` |

### 1.2 使用前提

```typescript
// 导入模块
import connectedTag from '@ohos.connectedTag';

// 权限声明 (module.json5)
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.NFC_TAG",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

---

## 2. API 清单

### 2.1 初始化类 API

| JS API | C++ 实现 | 同步/异步 | 用途 |
|--------|----------|----------|------|
| `init()` | `Init()` | Sync | 初始化 NFC |
| `uninit()` | `Uninit()` | Sync | 反初始化 NFC |
| `initialize()` | `Initialize()` | Sync | 初始化 NFC (异常抛出) |
| `uninitialize()` | `UnInitialize()` | Sync | 反初始化 NFC (异常抛出) |

### 2.2 读写类 API

| JS API | C++ 实现 | 同步/异步 | 用途 |
|--------|----------|----------|------|
| `readNdefTag(callback?)` | `ReadNdefTag()` | Async | 读取 NDEF (字符串) |
| `writeNdefTag(data, callback?)` | `WriteNdefTag()` | Async | 写入 NDEF (字符串) |
| `read(callback?)` | `ReadNdefData()` | Async | 读取 NDEF (二进制) |
| `write(data, callback?)` | `WriteNdefData()` | Async | 写入 NDEF (二进制) |

### 2.3 事件监听 API

| JS API | C++ 实现 | 同步/异步 | 用途 |
|--------|----------|----------|------|
| `on(event, callback)` | `On()` | Sync | 注册事件监听 |
| `off(event, callback?)` | `Off()` | Sync | 注销事件监听 |

### 2.4 枚举常量

| 常量 | 值 | 描述 |
|------|------|------|
| `NfcRfType.NFC_RF_LEAVE` | 0 | RF 场离开 |
| `NfcRfType.NFC_RF_ENTER` | 1 | RF 场进入 |

---

## 3. API 详细说明

### 3.1 init()

**功能**: 初始化 NFC 标签模块。

**签名**:
```typescript
function init(): boolean;
```

**返回值**:
| 类型 | 描述 |
|------|------|
| `boolean` | `true` 成功, `false` 失败 |

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:32-38`
- 调用: `NfcTagClient::GetInstance().Init()`

**错误处理**:
| 错误码 | 业务码 | 描述 |
|--------|--------|------|
| `NFC_SUCCESS` | 0 | 成功 |
| `NFC_GRANT_FAILED` | 201 | 权限不足 |
| 其他 | 3200101 | NFC 状态异常 |

---

### 3.2 uninit()

**功能**: 反初始化 NFC 标签模块。

**签名**:
```typescript
function uninit(): boolean;
```

**返回值**:
| 类型 | 描述 |
|------|------|
| `boolean` | `true` 成功, `false` 失败 |

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:40-46`
- 调用: `NfcTagClient::GetInstance().Uninit()`

---

### 3.3 initialize()

**功能**: 初始化 NFC 标签模块，失败时抛出异常。

**签名**:
```typescript
function initialize(): void;
```

**异常**:
| 错误码 | 业务码 | 异常类型 |
|--------|--------|----------|
| `NFC_GRANT_FAILED` | 201 | BusinessError |
| `NFC_INVALID_STATE` | 3200101 | BusinessError |

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:48-53`
- 调用: `CheckNfcStatusCodeAndThrow()` 抛出 JS 异常

---

### 3.4 uninitialize()

**功能**: 反初始化 NFC 标签模块，失败时抛出异常。

**签名**:
```typescript
function uninitialize(): void;
```

**异常**: 同 `initialize()`

---

### 3.5 readNdefTag()

**功能**: 读取 NDEF 数据（字符串格式）。

**签名**:
```typescript
function readNdefTag(callback?: AsyncCallback<string>): Promise<string>;
```

**参数**:
| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `callback` | `AsyncCallback<string>` | 否 | 回调函数 |

**返回值**:
| 类型 | 描述 |
|------|------|
| `Promise<string>` | NDEF 数据字符串 |
| `callback(err, data)` | `err` 为空时 `data` 为 NDEF 数据 |

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:62-96`
- 异步上下文: `ReadAsyncContext`
- 工作队列: `DoAsyncWork()`

**调用链**:
```
readNdefTag()
  → ReadAsyncContext::executeFunc_
  → NfcTagClient::ReadNdefTag()
  → NfcTagProxy::ReadNdefTag() [IPC]
  → NfcTagService::ReadNdefTag()
  → hdiAdapter_.ReadNdefTag()
  → NfcTagHdiImpl::ReadNdefTag()
  → IConnectedNfcTag::ReadNdefTag() [HDI]
```

**示例**:
```typescript
// Promise 方式
try {
    await connectedTag.initialize();
    const data = await connectedTag.readNdefTag();
    console.log('NDEF data:', data);
} catch (err) {
    console.error('Read failed:', err);
}

// Callback 方式
connectedTag.readNdefTag((err, data) => {
    if (err) {
        console.error('Read failed:', err);
        return;
    }
    console.log('NDEF data:', data);
});
```

---

### 3.6 writeNdefTag()

**功能**: 写入 NDEF 数据（字符串格式）。

**签名**:
```typescript
function writeNdefTag(data: string, callback?: AsyncCallback<void>): Promise<void>;
```

**参数**:
| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `data` | `string` | 是 | 要写入的 NDEF 数据 |
| `callback` | `AsyncCallback<void>` | 否 | 回调函数 |

**参数校验**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:105-122`
- `ParseString()` 验证类型为 string
- 数据长度限制: 1-512 字节

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:98-148`
- 异步上下文: `WriteAsyncContext`

**示例**:
```typescript
const ndefData = 'Hello NFC Tag';
await connectedTag.writeNdefTag(ndefData);
console.log('Write success');
```

---

### 3.7 read()

**功能**: 读取 NDEF 数据（二进制格式）。

**签名**:
```typescript
function read(callback?: AsyncCallback<Array<number>>): Promise<Array<number>>;
```

**返回值**:
| 类型 | 描述 |
|------|------|
| `Promise<Array<number>>` | NDEF 数据字节数组 |
| `callback(err, data)` | `err` 为空时 `data` 为字节数组 |

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:150-183`
- 异步上下文: `ReadDataAsyncContext`
- 输出转换: `CreateNumberArray()`

**参数校验**:
| 检查 | 文件位置 |
|------|----------|
| 参数数量 | nfc_napi_adapter.cpp:157 |
| 类型验证 | nfc_napi_utils.cpp:187-210 |

**示例**:
```typescript
const data = await connectedTag.read();
console.log('NDEF bytes:', data);
```

---

### 3.8 write()

**功能**: 写入 NDEF 数据（二进制格式）。

**签名**:
```typescript
function write(data: Array<number>, callback?: AsyncCallback<void>): Promise<void>;
```

**参数**:
| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `data` | `Array<number>` | 是 | 要写入的字节数组 |

**参数校验**:
- 文件: `nfc_napi_utils.cpp:187-210`
- `ParseByteArray()` 验证:
  - 输入是否为数组
  - 每个元素是否在 0-255 范围内
  - 数组长度是否在有效范围内

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_adapter.cpp:185-236`
- 异步上下文: `WriteDataAsyncContext`

**示例**:
```typescript
const bytes = [0xD1, 0x01, 0x0E, 0x54, 0x02, 0x65, 0x6E, 0x48, 0x65, 0x6C, 0x6C, 0x6F];
await connectedTag.write(bytes);
console.log('Write success');
```

---

### 3.9 on()

**功能**: 注册 NFC 事件监听。

**签名**:
```typescript
function on(event: 'notify', callback: AsyncCallback<NfcRfType>): void;
```

**参数**:
| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `event` | `'notify'` | 是 | 事件类型（仅支持 notify） |
| `callback` | `AsyncCallback<NfcRfType>` | 是 | 回调函数 |

**事件类型**:
| 值 | 描述 |
|------|------|
| `NfcRfType.NFC_RF_ENTER` | NFC RF 场进入 |
| `NfcRfType.NFC_RF_LEAVE` | NFC RF 场离开 |

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_event.cpp:111-136`
- 事件注册: `EventRegister::Register()`
- 全局映射: `g_eventRegisterInfo`

**参数校验**:
| 检查 | 值 | 文件位置 |
|------|------|----------|
| 参数数量 | 2 | nfc_napi_event.cpp:122 |
| 事件名类型 | string | nfc_napi_event.cpp:120 |
| 回调类型 | function | nfc_napi_event.cpp:120 |
| 事件类型支持 | "notify" | nfc_napi_event.cpp:221 |
| 字符串长度限制 | 64 字节 | nfc_napi_event.cpp:31 |

**示例**:
```typescript
connectedTag.on('notify', (err, type) => {
    if (err) {
        console.error('Event error:', err);
        return;
    }
    if (type === connectedTag.NfcRfType.NFC_RF_ENTER) {
        console.log('NFC field detected');
    } else if (type === connectedTag.NfcRfType.NFC_RF_LEAVE) {
        console.log('NFC field removed');
    }
});
```

---

### 3.10 off()

**功能**: 注销 NFC 事件监听。

**签名**:
```typescript
function off(event: 'notify', callback?: AsyncCallback<void>): void;
```

**参数**:
| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `event` | `'notify'` | 是 | 事件类型 |
| `callback` | `AsyncCallback<void>` | 否 | 要注销的回调（可选） |

**行为说明**:
- `callback` 为空: 注销该事件的所有回调
- `callback` 指定: 只注销指定回调

**C++ 实现**:
- 文件: `frameworks/js/napi/nfc_napi_event.cpp:138-175`
- 注销: `EventRegister::Unregister()`

**示例**:
```typescript
// 注销指定回调
const handler = (err, type) => { /* ... */ };
connectedTag.on('notify', handler);
// ...
connectedTag.off('notify', handler);

// 注销所有回调
connectedTag.off('notify');
```

---

## 4. 错误码说明

### 4.1 业务错误码映射

| 内部错误码 | 业务错误码 | 描述 |
|-----------|-----------|------|
| `NFC_SUCCESS` | 0 | 成功 |
| `NFC_GRANT_FAILED` | 201 | 权限校验失败 |
| `NFC_SYS_PERM_FAILED` | 801 | 系统权限不足 |
| `NFC_INVALID_PARAMETER` | 401 | 参数无效 |
| 其他 | 3200101 | NFC 状态异常 |

### 4.2 错误对象结构

```typescript
interface BusinessError extends Error {
    code: number;      // 业务错误码
    message: string;   // 错误描述
}
```

### 4.3 常见错误处理

```typescript
import connectedTag from '@ohos.connectedTag';

try {
    await connectedTag.initialize();
} catch (err) {
    if (err.code === 201) {
        console.error('Permission denied. Check module.json5 permissions.');
    } else if (err.code === 401) {
        console.error('Invalid parameter.');
    } else if (err.code === 801) {
        console.error('System capability not supported.');
    } else if (err.code === 3200101) {
        console.error('NFC service abnormal. Try reinitialize.');
    }
}
```

---

## 5. 完整使用示例

```typescript
import connectedTag from '@ohos.connectedTag';

class NfcTagHandler {
    // 初始化
    async init(): Promise<void> {
        await connectedTag.initialize();
    }

    // 读取 NDEF
    async readNdef(): Promise<string> {
        return await connectedTag.readNdefTag();
    }

    // 写入 NDEF
    async writeNdef(data: string): Promise<void> {
        await connectedTag.writeNdefTag(data);
    }

    // 监听 RF 状态
    onRfStateChange(callback: (type: number) => void): void {
        connectedTag.on('notify', (err, type) => {
            if (err) {
                console.error('Event error:', err);
                return;
            }
            callback(type);
        });
    }

    // 释放资源
    async uninit(): Promise<void> {
        await connectedTag.uninit();
    }
}

// 使用
const handler = new NfcTagHandler();
await handler.init();

handler.onRfStateChange((type) => {
    const state = type === connectedTag.NfcRfType.NFC_RF_ENTER ? '进入' : '离开';
    console.log(`NFC RF 场${state}`);
});

const data = await handler.readNdef();
console.log('Read data:', data);

await handler.writeNdef('Hello from OpenHarmony!');
await handler.uninit();
```

---

## 6. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 内部 API | [03_InnerAPI.md](./03_InnerAPI.md) |
| 构建说明 | [04_Build.md](./04_Build.md) |
| 安全评估 | [05_Security.md](./05_Security.md) |
